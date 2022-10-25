---
title: " Google Secret Manager"
---

Secret Manager can be used to store API keys, passwords, and any other sensitive data.

#### Requirements

- GCP Project Access
- Terraform 

#### Create Secret

Create secrets to store self-manage certificate public and private keys.

- self-managed-cert.crt 

    ```
    -----BEGIN CERTIFICATE-----
    MYCERTVALUES
    -----END CERTIFICATE-----
    ```

- self-managed-cert.key

    ```
    -----BEGIN RSA PRIVATE KEY-----
    MYCERTKEY
    -----END RSA PRIVATE KEY-----
    ```
  
 - create secret

    ```hlc
    ## Creates secrets
    resource "google_secret_manager_secret" "cert" {
      project = "rdy-dev-cld-usa-dev01"
      secret_id = "self-managed-cert"
      replication {
        automatic = true
      }
    }

    resource "google_secret_manager_secret" "key" {
      project = "rdy-dev-cld-usa-dev01"
      secret_id = "self-managed-cert-key"
      replication {
        automatic = true
      }
    }

    ## adds content to secrets
    resource "google_secret_manager_secret_version" "cert" {
      secret = google_secret_manager_secret.cert.id
      secret_data = file("self-managed-cert.cert")
    }

    resource "google_secret_manager_secret_version" "key" {
      secret = google_secret_manager_secret.key.id
      secret_data = file("self-managed-cert.key")
    }
    ```

- read secret data

    ```hlc
    ## gets secret_data
    data "google_secret_manager_secret_version" "cert" {
      project = "rdy-dev-cld-usa-dev01"
      secret = google_secret_manager_secret.cert.id
    }

    data "google_secret_manager_secret_version" "key" {
      project = "rdy-dev-cld-usa-dev01"
      secret = google_secret_manager_secret.key.id
    }
    ```

- consuming secret data using terraform outputs

    ```hlc
    output "cert_secret_value" {
      value = data.google_secret_manager_secret_version.cert.secret_data
      sensitive = true
    }
    output "key_secret_value" {
      value = data.google_secret_manager_secret_version.key.secret_data
      sensitive = true
    }
    ```

- ```terraform apply``` does not output secret_data by default. use ```terraform ouput -json``` to print secret values

    ```hlt
    $ terraform apply

    < omitted data>

    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    cert_secret_value = <sensitive>
    key_secret_value = <sensitive>
    ```

    ```hlc    
    $ terraform output -json
    {
      "cert_secret_value": {
        "sensitive": true,
        "type": "string",
        "value": "  -----BEGIN CERTIFICATE-----\n  MYCERTVALUES\n  -----END CERTIFICATE-----"
      },
      "key_secret_value": {
        "sensitive": true,
        "type": "string",
        "value": "  -----BEGIN RSA PRIVATE KEY-----\n  MYCERTKEY\n  -----END RSA PRIVATE KEY-----"
      }
    }
    ```
