---
title: "Gcloud Command Overview"
description: "gcloud commands"
---

#### Review current configuration

- Current configuration

  ```shell
  $ gcloud config configurations list
  NAME     IS_ACTIVE  ACCOUNT               PROJECT              COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
  default  True       kg48037@mykronos.com  shd-cus-host01-7f07
  ```

- update account

  ```shell
  $ gcloud config set account kg48037@mykronos.com
  Updated property [core/account].
  ```

- update project

  ```shell
  $ gcloud config set project rdy-dev-cld-usa-dev01
  Updated property [core/project].
  ```

- New configuration

  ```shell
  $ gcloud config configurations list
  NAME     IS_ACTIVE  ACCOUNT               PROJECT                COMPUTE_DEFAULT_ZONE  COMPUTE_DEFAULT_REGION
  default  True       kg48037@mykronos.com  rdy-dev-cld-usa-dev01
  ```
