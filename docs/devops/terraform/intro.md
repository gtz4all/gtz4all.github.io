---
title: "Introduction"
---

![terraform](../../assets/images/terraform.png "terraform"){: style="width:100px"} [Terraform Documentation](/devops/terraform/intro)

Infrastructure as code ( IaC) solves several problems with manual and static infrastructure management. IaC provides the ability to build and destroy an environment multiple times if required. IaC can be used to create environment lifecycle where changes can transition from development, testing to production. Terraform is an IaC (Infrastructure as Code) tool used to build and modify infrastructure safely and efficiently. It keeps track of its environment state so rerunning the tool against a code with no changes will not generate any changes.

#### Terraform Flow

![terraform-intro](../../assets/terraform-intro.drawio){: style="height:150px;width:150px"}


#### Known Issues

- Empty Lists - https://github.com/hashicorp/terraform/issues/23562
