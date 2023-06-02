---
title: "GCP Multi Account Authentication"
---

#### GCLOUD CLI Configurations

[Gcloud CLI Configurations](https://cloud.google.com/sdk/docs/configurations) can be used to store and separate multiple account properties including Project ID, active user or service account. 

```
gcloud config configurations --help
```
```
Available commands for gcloud config configurations:

      activate                Activates an existing named configuration.
      create                  Creates a new named configuration.
      delete                  Deletes a named configuration.
      describe                Describes a named configuration by listing its
                              properties.
      list                    Lists existing named configurations.
      rename                  Renames a named configuration.

For detailed information on this command and its flags, run:
  gcloud config configurations --help
```

#### GCP Configurations

- Initially, gcloud CLI starts with a single configuration named `default`. 

    ```
    $ gcloud config list
    [core]
    account = kg48037@mykronos.com
    disable_usage_reporting = True
    project = rdy-dev-cld-usa-dev01

    Your active configuration is: [default]
    ```

- Additional configurations can be created to switch between accounts.

    ```
    $ gcloud config configurations create acloudsbox
    Created [acloudsbox].
    Activated [acloudsbox].
    ```
    ```
    $ gcloud config configurations list
    NAME        IS_ACTIVE  ACCOUNT                 PROJECT                   COMPUTE_DEFAULT_ZONE
    default     False      kg48037@mykronos.com    rdy-dev-cld-usa-dev01
    acloudsbox  True
    ```
    ```
    $ gcloud config list
    [core]
    disable_usage_reporting = True

    Your active configuration is: [acloudsbox]
    ```
#### GCP Authentication Types
There are two ways to authenticate in GCP depending on the use case. The following CLI Commands will store credentials in the current configuration. 
If no configuration exists, it will create a configuration named default.

###### [GCLOUD CLI Authentication](https://cloud.google.com/sdk/gcloud/reference/auth/login)
`gcloud auth login --no-launch-browser --update-adc` This command can be used to authenticate both GCloud CLI commands and SDK 
- access credentials for the user account and store them in ~/.config/gcloud/ via a web-based authorization flow.
- `--no-launch-browser` flag will print a URL that can be copied to any browser.
- `--update-adc` writes the obtained credentials to the well-known location for Application Default Credentials (ADC)

###### [GCP SDK Authentication](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)
`gcloud auth application-default login --no-launch-browser` This command is not required when using flag `--update-adc` with `gcloud auth login`.
- access credentials via a web-based authorization flow for the user account and store them in the `well-known location for Application Default Credentials` file named application_default_credentials.json in `~/.config/gcloud/` 
- `--no-launch-browser` flag will print a URL that can be copied to any browser.

#### Switching Between GCP Configurations

- Review current configuration
    ```
    $ gcloud config list
    [core]
    account = admin@gtz4all.com
    disable_usage_reporting = True
    project = rdy-dev-cld-usa-dev01

    Your active configuration is: [default]
    ```
    ```
    $ gcloud config configurations list
    NAME     IS_ACTIVE  ACCOUNT               PROJECT                COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
    default  True       admin@gtz4all.com  gtz4all-dev-cld01
    sbox     False
    $ gcloud config configurations activate sbox
    Activated [sbox].
    ```
    ```
    $ gcloud config list
    [core]
    disable_usage_reporting = True

    Your active configuration is: [sbox]
    ```

- Provide user credentials to new gcloud configuration - sbox
    ```
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
    ```
    $ gcloud compute networks list
    NAME     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    default  AUTO         REGIONAL
    ```
- switch back to the default gcloud configuration and validate credentials
    ```
    $ gcloud config configurations activate default
    Activated [default].
    ```
    ```
    $ gcloud compute networks list
    NAME                    SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    dev01-vpc01       CUSTOM       GLOBAL
    ```
