---
title: "Gcloud Command Overview"
description: "gcloud commands"
---

#### Review current configuration

- Current configuration

  ```shell
  $ gcloud config configurations list
  NAME     IS_ACTIVE  ACCOUNT               PROJECT              COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
  default  True       gtz007@cloudgtz.com  gtz-cld-dev-shd01
  ```

- update account

  ```shell
  $ gcloud config set account gtz007@cloudgtz.com
  Updated property [core/account].
  ```

- update project

  ```shell
  $ gcloud config set project gtz-cld-dev01
  Updated property [core/project].
  ```

- New configuration

  ```shell
  $ gcloud config configurations list
  NAME     IS_ACTIVE  ACCOUNT               PROJECT                COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
  default  True       gtz007@cloudgtz.com  gtz-cld-dev01
  ```
## GCP Authentication Types
There are two ways to authenticate in GCP depending on the use case. The following CLI Commands will store credentials in the current configuration. If no configuration exists, it will create a configuration named default.

#### GCLOUD CLI Authentication

- `gcloud auth login --no-launch-browser --update-adc`

  - This obtains access credentials for the user account and store them in ~/.config/gcloud/ via a web-based authorization flow.
  - `--no-launch-browser` flag will print a URL that can be copied to any browser.
  - `--update-adc` writes the obtained credentials to the well-known location for Application Default Credentials (ADC)
  - This command can be used to authenticate both GCloud CLI commands and SDK 

#### GCP SDK Authentication (optional)

- `gcloud auth application-default login --no-launch-browser`

  - This obtains access credentials via a web-based authorization flow for the user account and store them in the 'well-known location for Application Default Credentials' file named `application_default_credentials.json` in `~/.config/gcloud/` 
  - `--no-launch-browser` flag will print a URL that can be copied to any browser.


## Switching Between GCP Configurations

- Review current configuration
  - ```
    $ gcloud config list
    [core]
    account = gtz007@cloudgtz.com
    disable_usage_reporting = True
    project = gtz-cld-dev01
    Your active configuration is: [default]
    ```
    
  - ```
    $ gcloud config configurations list
    NAME     IS_ACTIVE  ACCOUNT               PROJECT                COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
    default  True       gtz007@cloudgtz.com  gtz-cld-dev01
    sbox     False
    $ gcloud config configurations activate sbox
    Activated [sbox].
    ```
    
  - ```
    $ gcloud config list
    [core]
    disable_usage_reporting = True
     
    Your active configuration is: [sbox]
    ```
- Provide user credentials to new gcloud configuration - sbox
    
  - ```
    $ gcloud auth login --no-launch-browser --update-adc
    Go to the following link in your browser:
     
        https://accounts.google.com/o/oauth2/auth?response_type=code
     
    Enter authorization code: 4/0AbUR2VNmI2-EonA-P57ik6-Y6B842zi9l0piWRm7KNlqFgQtM1G6f-HToDYBHbJwuEeTTw
     
    Application default credentials (ADC) were updated.
     
    You are now logged in as [cloud_user_p_4591b375@linuxacademygclabs.com].
    Your current project is [None].  You can change this setting by running:
      $ gcloud config set project PROJECT_ID
     
    $ gcloud config set project playground-s-11-899e0a9b
    Updated property [core/project].
    ```

- validate gcloud configuration credentials
    
  - ```
    $ gcloud compute networks list
    NAME     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    default  AUTO         REGIONAL
    switch back to the default gcloud configuration and validate credentials
    ```
  - ```
    $ gcloud config configurations activate default
    Activated [default].
     ```
  - ```
    $ gcloud compute networks list
    NAME                    SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    gtz-cld-shdvpc01       CUSTOM       GLOBAL
     ```
