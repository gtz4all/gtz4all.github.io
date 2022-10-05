---
title: "Tips and Tricks"
draft: false
---



{{< tips "Hugo Static Pages" >}}
[Ansible](https://www.ansible.com/) is an automation tool written in python

##### Adding a new chapter to a page
```python
hugo new --kind chapter <name>/_index.en.md
```
Adding another section to chapter
```python
hugo new --kind chapter <name>/<subname>/_index.en.md
```
Adding content to chapter
```python
hugo new --kind chapter <name>/<new-content>.en.md
```
{{</ tips >}}

{{< tips "Git" >}}
#### Git Update
```yaml
git add -A .; git commit -m "new content"; git push -u origin master
```
{{</ tips >}}

{{< tips "Markdown" >}}
###### Adding Comments
```bash
[comment]: <> (a reference style link.)
```
###### Side by Side Code Block
The usual [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
does not cover some of the more advanced Markdown tricks, but here
is one. You can combine verbatim HTML with your Markdown. 
This is particularly useful for tables.
Notice that with **empty separating lines** we can use Markdown inside HTML:

<table>
<tr>
<th>Json 1</th>
<th>Json 2</th>
</tr>
<tr>
<td>

```bash
{
  "id": 5,
  "username": "mary",
  "email": "mary@example.com",
  "order_id": "f7177da"
}
```

</td>
<td>

```bash
{
  "id": 5,
  "username": "mary",
  "email": "mary@example.com",
  "order_id": "f7177da"
}
```

</td>
</tr>
</table>
{{</ tips >}}


{{< tips "Mermaid Diagram" >}}
###### Mermaid Sample Code
  This is just sample code. **due to shortcode "tips", detailed information just as names and description are not showing.
  on a normal .md file, this will be displayed.**

<table>
<tr>
<th>Vertical</th>
<th>Horizonal</th>
</tr>
<tr>
<td>

<div class="mermaid">
graph TD;
    subgraph "Device Connectivity"
    1(DISTRT1)---|Gi1/12|3(ACCSW1);
    2(DISTRT2)---|Gi1/12|3(ACCSW1);
    1(DISTRT1)---|Gi1/13|4(ACCSW2);
    2(DISTRT2)---|Gi1/13|4(ACCSW2);
    1(DISTRT1)---|Gi1/14|5(ACCSW3);
    2(DISTRT2)---|Gi1/14|5(ACCSW3);
    1(DISTRT1)---|Gi1/15|6(ACCSW4);
    2(DISTRT2)---|Gi1/15|6(ACCSW4);
    1(DISTRT1)---|Gi1/16|7(ACCSW5);
    2(DISTRT2)---|Gi1/16|7(ACCSW5);
    1(DISTRT1)---|Gi1/17|8(ACCSW6);
    2(DISTRT2)---|Gi1/17|8(ACCSW6);
end
</div>
<script async src="https://unpkg.com/mermaid@8.2.3/dist/mermaid.min.js"></script>

</td>
<td>

<div class="mermaid">
graph LR
  subgraph "Requester Pre Provision Trigger URL"
  Requester-URL -->|Provision|B{CI/CD Complete Provisioning}
  B -->|Switch Variables|E[Pull Switch Vars from Archive]
  B -->|Validate Switch|E.A[Pull switch variables]
  B -->|Validate Switch Code|E.B[Build Code Standard Variables]
  B -->|Email Notification|E.C[Generate Requester Token/Email]
  E.C -->|Requester Email|1[Email w/Token, Switch OSPF/Initial Cfg]
end
</div>
<script async src="https://unpkg.com/mermaid@8.2.3/dist/mermaid.min.js"></script>

</td>
</tr>
</table>

{{</ tips >}}

{{< tips "Notice Boxes" >}}

{{% notice note %}}
A notice disclaimer
{{% /notice %}}

{{% notice info %}}
A notice disclaimer
{{% /notice %}}

{{% notice tip %}}
A tip disclaimer
{{% /notice %}}

{{% notice warning %}}
A warning disclaimer
{{% /notice %}}

{{</ tips >}}

{{< tips "Docker Cleanup" >}}
Remove all Docker containers/images.

###### List All Container IDs
```yaml
## list all containes
docker ps -a
## list all containers IDs ( -q = IDs)
docker ps -aq
```
###### Stop All Running Containers
```yaml
## list all running containers
docker ps
### stop all containers
docker stop $(docker ps -aq)
```
###### Remove All Containers
```yaml
## Remove all stopped containers
docker rm $(docker ps -aq)
```
###### Remove All Images
```yaml
docker rmi $(docker images -q)
```
{{</ tips >}}

{{< tips "Native HTML Dropdown" >}}
# Questions

<details><summary>Which command do you use to create a new swarm?</summary>
<p>

```
docker swarm init --advertise-addr <MANAGER-IP>
```
</p>
</details>

<details><summary>What is this flag --advertise-addr for?</summary>
<p>

```
This flag configures the IP address for the manager node and The other nodes in the swarm must be able to access the manager at the IP address.
```
</p>
</details>

<details><summary>How do you know the current status of the swarm?</summary>
<p>

```
docker info // you can find the info under the swarm section
```
</p>
</details>

{{</ tips >}}

{{< tips "Kubernetes" >}}
#### Connect to Pod Shell
```bash
cloud_user# kubectl exec --stdin --tty nginx-deployment-5cfcccdb74-ck2nj -- /bin/bash
root@nginx-deployment-5cfcccdb74-ck2nj:/#
```
{{</ tips >}}

{{< tips "Hugo Binding" >}}
hugo server -D --bind=192.168.1.55 --baseURL=http://192.168.1.55
{{</ tips >}}

{{< tips "Search and Replace" >}}
grep -RiIl '25000' | xargs sed -i 's/25000/1000/g'
grep -rnw 'CUST01/' -e '25000'
{{</ tips >}}

{{< tips "More Comming Soon" >}}
echo theme = \"ananke\" >> config.toml
{{</ tips >}}