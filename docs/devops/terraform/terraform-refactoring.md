---
title: Terraform State Refactoring
---

!!! note

    Always create a local backup of your state file before making any changes to it. If using a Cloud Bucket as your backend, enable versioning.

#### Terraform State/Workspace Renaming

1. list current terraform workspaces

    ```hlc
    terraform init
    terraform workspace list
    ```

2. select terraform workspace

    ```hlc
     terraform workspace select <workspace-name>
    ```

3. create a local terraform state backup

    ```hlc
     terraform state pull > <new-workspace-name>.tfstate
     ```

4. create new workspace and copy local <new-workspace-name>.tfstate

    ```hlc
    terraform workspace new -state=<new-workspace-name>.tfstate <new-workspace-name> 
    ```

5. validate terraform state in new workspace

    ```hlc
    terraform plan
    ```

6. delete old terraform workspace

    !!! note

        default workspace cannot be deleted.
  
    ```hlc
    terraform workspace delete -force <old-workspace-name>
    ```

#### Terraform State/Workspace Refactoring
  
  This can be used to move state between buckets and workspaces. Note: This moves all resources within a state file.

1. list current terraform workspaces

    ```hlc
    terraform init
    terraform workspace list
    ```

2. switch to current terraform workspace 

    ```hlc
    terraform workspace select <current-workspace-name>
    ```

3. validate current terraform state

    ```hlc
    terraform plan
    ```

4. create a local terraform state backup
  
    ```hlc
    terraform state pull > <new-workspace-name>.tfstate 
    ```
    
5. comment out/edit the backend block in backend.tf if exists. 
  This will force a terraform init which will switch to a new remote or local backend 
    
    ```
    # backend "gcs" {
    #   bucket = "***"
    #   prefix = "terraform/state/***/***"
    # }  
    ```
    
6. switch to new backend - this will copy any current workspaces from all 

    ```hlc
    terraform init -migrate-state
    ```

###### Optional Steps
  This used to copy local state file into a newly created workspace.
  
7. create new workspace and copy <new-workspace-name>.tfstate

    ```hlc
    terraform workspace new -state=<new-workspace-name>.tfstate <new-workspace-name> 
    ```

8. validate terraform state

    ```hlc
    terraform plan
    ```

9. validate terraform workspace

    ```hlc
    terraform workspace list
    ```

#### Terraform Resource State Migration 
  
  This can be used to move specific resources from one state file to another.

1. create a local terraform state backup

```hlc
 terraform state pull > <workspace-name>.tfstate 
```

2. list all terraform resources in state file

    ```hlc
    terraform state list
    ```

    - output

      ```
      gtz4all@admin:~/terraform$ terraform state list
      module.new-vpcs["gtz4all-core-vnet01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-prd-snet01"]
      module.new-vpcs["gtz4all-core-net01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-npr-snet01"]
      module.new-vpcs["gtz4all-core-net01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-dev-snet01"]
      module.new-vpcs["gtz4all-core-net01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-shd-snet01"]
      ```

3. move a resource from current workspace state file into a local state file

    ```hlc
    terraform state mv -state-out=<new-state-name>.tfstate \
    'module.xxx["xxx"]' \     #<--- resource name from state list - wrap resource inside single quotes
     'module.xxx["xxx"]'       #<--- new resource name - use same name as state list - wrap resource inside single quotes
    ```
    !!! note

        since resource value is using double quotes, single quotes are being used outside resource.

    ```hlc
    gtz4all@admin:~/terraform$  terraform state mv -state-out=prd.tfstate \
    > 'module.new-vpcs["gtz4all-core-vnet01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-prd-snet01"]' \
    > 'module.new-vpcs["gtz4all-core-vnet01"].module.subnet.google_compute_subnetwork.subnetwork["us-east1/gtz4all-use1-prd-snet01"]'
    ```

3. edit variables file and remove any resources that have been move out of state file.
  
4. validate changes by running ```terraform plan/apply``` - expected msg: ```No changes. Your infrastructure matches the configuration.```


5. migrate moved resources to another workspace
  
    - create variables files with the resources that were moved

    - create new workspace and copy local <new-state-name>.tfstate

      ```hlc
      terraform workspace new -state=<new-state-name>.tfstate <new-workspace-name> 
      ```

6. validate by running ```terraform plan/apply``` - expected msg: ```No changes. Your infrastructure matches the configuration.```

