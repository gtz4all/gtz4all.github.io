---
title: "Tips and Tricks"
---

## Reference

The purpose of this section is to provide code snippets, tips and solution to simple issues.

??? note "Git Commands"
    ```yaml
    git add -A .; git commit -m "new content"; git push -u origin master
    ```

??? note "Side by Side Code Block"

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

??? note "Mermaid Sample Code"

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

??? note "Manage Docker containers/images"

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

??? note "Connect to Pod Shell"
     
    ## Testing
    
    ```bash
    cloud_user# kubectl exec --stdin --tty nginx-deployment-5cfcccdb74-ck2nj -- /bin/bash
    root@nginx-deployment-5cfcccdb74-ck2nj:/#
    ```

    > hugo server -D --bind=192.168.1.55 --baseURL=http://192.168.1.55
    > grep -RiIl '25000' | xargs sed -i 's/25000/1000/g'
    > grep -rnw 'CUST01/' -e '25000'
    > echo theme = \"ananke\" >> config.toml
