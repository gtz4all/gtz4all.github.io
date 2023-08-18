---
title: "Google Cloud Platform Ansible Collection"
---

The Google Cloud Platform Ansible Collection provides a series of Ansible modules and plugins for interacting with the Google Cloud Platform. 
It's similar to GCloud with some limitations. 

## Use Case: 1
Create a list of all Compute Resources that contain IPv4 IP address and related resources such as Firewall Rules and Routes.


#### GCP Authentication
- `auth_kind` string / required

  Choices:
  - "application"
      `user credentials - gcloud login`
  - "machineaccount"
      `A GCP VM with Access Role`
  - "serviceaccount"
      `A GCP Service Account`
      
 ###### Basics
 
- Get a list of all firewall rules and save them to a file
  ```yml
  ---
  - name: Data Collection
    hosts: localhost
    connection: local
    gather_facts: no
    vars:
      project: playground-project
    tasks:
      - name: get gcp firewall rules
        gcp_compute_firewall_info:
          project: '(( project ))'
          auth_kind: application
        register: gcp_fw

      - name: save gcp fw rules to file
        copy:
          content: '(( gcp_fw.resources | to_yaml ))'
          dest: '(( project ))-gcp_fw_rules.yml'
  ```
  
  - the above register output can be used to remove or modify firewall rules

  ```yaml
      - name: delete firewall
        google.cloud.gcp_compute_firewall:
          name: "(( item.name ))"
          project: repd-e-adm-01
          auth_kind: application
          state: absent
        with_items: '(( gcp_fw.resources ))'
  ```

#### Challenges
Getting a list of all created subnets becomes a challenge when using this ansible module collections. the module requires a region, but there is no ansible module to get a region list to iterate through. 

- gcloud command is used to get a region list.

  ```yaml
      - name: Network Subnet List
        command: |
          gcloud compute regions list --format="json"
        register: gcp_regions

       - name: get info on a subnetwork
        gcp_compute_subnetwork_info:
          region: "(( item.name ))"
          project: '(( project ))'
          auth_kind: application
        register: gcp_subnetworks
        with_items: "(( gcp_regions.stdout | from_json  ))"
  ```
- instead of using a gcloud command to assist the ansible module, it can be used to just get all subnets without the need to for a region.
  ```yaml
      - name: Network Subnet List
        command: |
          gcloud compute networks subnets list --project "(( project ))" --format="json"
        register: gcp_subnets

      - name: debug
        debug:
          msg: "(((gcp_subnets.stdout | from_json)))"
  ```
