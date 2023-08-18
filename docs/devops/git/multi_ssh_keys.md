---
title: "Managing Multiple Github Accounts"
description: "Cloud Computing Documentation"
---

#### Managing Multiple SSH Keys 

SSH Keys must be unique across Github.com Accounts/Organizations. The below method can also be used for other git providers such as bitbucket, gitlab, etc.

1. Create One SSH Key for each Organization.

    | :exclamation:  SSH with an SSH key and passphrase are highly recommended.  |
    |-----------------------------------------|
    
    
    account| command
    -------|--------
    netgtz | `ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_netgtz -C "netgtz@gmail.com"`
    gtz4all |` ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_gtz4all -C "gtz4all@gmail.com"`


2. Create/Modify SSH Config

    - Create file if it doesn't already exist
      ```
      ~/.ssh/config
      touch ~/.ssh/config
      chmod 600 ~/.ssh/config
      ```

    - SSH Config file content -~/.ssh/config
      ```
      # netgtz
      Host github.com-netgtz
          HostName github.com
          AddKeysToAgent yes
          User git
          IdentityFile ~/.ssh/id_rsa_netgtz
       
      # gtz4all
      Host github.com-gtz4all
          HostName github.com
          AddKeysToAgent yes
          User git
          IdentityFile ~/.ssh/id_rsa_gtz4all
       
      # default uses ~/.ssh/id_rsa
      Host *
        AddKeysToAgent yes
        UpdateHostKeys yes
      ```

3. Add Keys to SSH Agent

    - ssh-add
      ```
      ssh-add ~/.ssh/id_rsa_netgtz
      ssh-add ~/.ssh/id_rsa_gtz4all
      ```
    
    - ssh-add validation
      ```
      ## List all keys
      ssh-add -l
       
      ## Delete all keys
      ssh-add -D
      ```
    
    - SSH Agent Troubleshoot
      ```
      ## ssh-agent error
      $ ssh-add ~/.ssh/id_rsa_netgtz
      Could not open a connection to your authentication agent.
       
      ## restart ssh-agent
      $ killall ssh-agent; eval `ssh-agent`
      ssh-agent: no process found
      Agent pid 5144
       
      ## run ssh-add
      $ ssh-add ~/.ssh/id_rsa_ukgepic
      Enter passphrase for /home/kg48037/.ssh/id_rsa_netgtz:
      Identity added: /home/kg48037/.ssh/id_rsa_netgtz (netgtz@gmail.com)
      ```

4. Add SSH Public keys to GitHub Accounts/Organizations
 
    ![image](https://github.com/gtz4all/gtz4all.github.io/assets/40032360/cacd8073-d785-422d-9495-36f897adab27)

5. Validate SSH Access
   The below commands are used to validate the configuration in ~/.ssh/config. the "host" value is used to reference the key that must be used.

    - github.com-netgtz
      ```
      $ ssh -T git@github.com-netgtz
      Hi netgtz! You've successfully authenticated, but GitHub does not provide shell access.
      ```
      
    - github.com-gtz4all
      ```
      $ ssh -T git@github.com-gtz4all
      Hi gtz4all! You've successfully authenticated, but GitHub does not provide shell access.
      ```

#### Cloning Repos from multiple Github Accounts
When cloning a repo, you can copy the SSH command provided by the repository.

  - The below command will use the default SSH key.
    Default | Command	| SSH Key
    -------|----------|---------
    default | `git clone git@github.com:netgtz/terraform-gcp-vm.git` | `~/.ssh/id_rsa`
    
  
  - The below command will use the default SSH key.
    SSH Config host | Command	| SSH Key
    -------|----------|---------
    github.com-netgtz | `git clone git@github.com-netgtz:netgtz/terraform-gcp-vm.git` | `~/.ssh/id_rsa_netgtz`
    github.com-gtz4all | `git clone git@github.com-gtz4all:gtz4all/terraform-gcp-vpc.git` | `~/.ssh/id_rsa_gtz4all`
    

  - Validate cloned repository is using ssh config hosts. See `[remote "origin"]` url.
    ```
    ~/terraform-gcp-vm$ cat .git/config
    ```

- To update the `[remote "origin"]` url of already existing repositories, use  `git remote set-url origin`.
    SSH Config host | Command	
    -------|----------
    github.com-netgtz | `git remote set-url origin git@github.com-netgtz:netgtz/terraform-gcp-vm.git`
    github.com-gtz4all | `git remote set-url origin git@github.com-gtz4all:gtz4all/terraform-gcp-vpc.git`

