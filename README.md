# CDN - CRASH COURSE WITH ANSIBLE

## Prerequisities

- **Master node** with **ssh-server** installed (Debian Bullseye)
- An user account (**cdn** user is a default user for this playbook, it can be overrided)
- This node will hosts all of your docker containers

## Inventory

To create or edit the **inventory** file, navigate to the path *inventory/inventory.yaml*. This file will be based on your network specifications. It is **required** to change addresses in inventory file according to your network.

The Docker containers will function as additional **virtual machines** (VMs) on your local network. They will be derived from the **interface** of your master node.

Inventory in **.ini** format.
```
[cdn_master]
master_node ansible_host=192.168.1.46

[cdn_webservers]
web_server_node ansible_host=192.168.1.10
web_proxy_node ansible_host=192.168.1.20

[cdn_monitoring]
prometheus_node ansible_host=192.168.1.30
grafana_node ansible_host=192.168.1.40

[cdn_containers:children]
cdn_webservers
cdn_monitoring
```

Inventory in **.yaml** format.
```
all:
  children:
    cdn_master:
      hosts:
        master_node:
          ansible_host: 192.168.1.46
    cdn_webservers:
      hosts:
        web_server_node:
          ansible_host: 192.168.1.10
        web_proxy_node:
          ansible_host: 192.168.1.20
    cdn_monitoring:
      hosts:
        prometheus_node:
          ansible_host: 192.168.1.30
        grafana_node:
          ansible_host: 192.168.1.40
    cdn_containers:
      children:
        cdn_webservers:
        cdn_monitoring
```

The *group_vars/all.yaml* file contains hosts-specific variables used for connecting to all hosts listed in the inventory file. These variables **shall be overriden** base on your specific needs.
```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

ansible_user: "cdn"
ansible_connection: ssh
ansible_ssh_private_key_file: "{{ ssh.private_key_location }}/{{ ssh.private_key_name }}"
```

## Ansible role: cdn_connection

### Tags
-----
`cdn_connection`

### Description

Setups connection between **control node (~localhost)** and **master node**, which will run docker containers. Role generates ssh key, creates config file and set authorized key on **master node**.

### Variables

All variables which can be overridden are stored in *defaults/main.yaml* file as well as in table below.

| Name                 | Default Value | Description                                                                                              |
| -------------------- | ------------- | -------------------------------------------------------------------------------------------------------- |
| `cdn_connection_ssh` | {}            | Location and name for generating SSH key for connection (**private_key_location**, **private_key_name**) |

### Overrides

You can **override** the default value of **cdn_connection_ssh** in inventory file stored in *groups_vars/all.yaml*.

```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

cdn_connection_ssh: "{{ ssh }}"
```

## Ansible role: cdn_docker

### Tags
-----
`cdn_docker`

### Description

The role installs and configures the Docker Engine on the master node based on the provided Unix distribution. The master node will run Docker containers as VMs with systemd installed.

Default Settings:

- Multiple Docker containers with the **ipvlan** network driver will be created.
- These containers are included in the **cdn_containers** group.
- Notifications will be sent to the **control node (~localhost)** about the Docker containers.
- The containers will be added to the known hosts.
  
The role ensures that Docker Engine is set up correctly on the master node and manages the creation and configuration of Docker containers using the ipvlan network driver. Furthermore, it facilitates communication between the master node and the **control node (~localhost)** by notifying the control node about the created Docker containers and adding them to the known hosts list.

### Variables
All variables which can be overridden are stored in defaults/main.yaml file as well as in table below.

| Name                          | Default Value    | Description                                                                                              |
| ----------------------------- | ---------------- | -------------------------------------------------------------------------------------------------------- |
| `cdn_docker_ssh`              | {}               | Location and name for generating SSH key for connection (**private_key_location**, **private_key_name**) |
| `cdn_docker_hosts`            | "cdn_containers" | Inventory group name of hosts as docker containers                                                       |
| `cdn_docker_project`          | " ~/cdn-project  | Location on master node to create project files                                                          |
| `cdn_docker_os`               | {}               | System distribution and version (**distribution**, **version**)                                          |
| `cdn_docker_settings`         | {}               | Docker settings (**user**, **password**, **image**)                                                      |
| `cdn_docker_network_name`     | "cdn-network"    | Docker network name                                                                                      |
| `cdn_docker_network_driver`   | "ipvlan"         | Docker network driver                                                                                    |
| `cdn_docker_network_settings` | {}               | Docker netwokrk settings (**subnet**, **gateway**, **interface**)                                        |

### Overrides

You can **override** the default values in inventory file stored in *group_vars/all.yaml*.  
It is **required** to **override** the **cdn_docker_network_settings** variables.

```
cdn_docker_network_settings:
  subnet: "192.168.1.0/24"
  gateway: "192.168.1.1"
  interface: "enp0s25"
```

## Ansible role: cdn_nginx_browser

### Tags
-----
`cdn_nginx_browser`

### Description

Installs and configure **nginx server** as a browser for static files.

### Variables
All variables which can be overridden are stored in *defaults/main.yaml* file as well as in table below.

| Name                              | Default Value | Description                           |
| --------------------------------- | ------------- | ------------------------------------- |
| `cdn_nginx_browser_listen`        | "80"          | Port, that nginx server listents to   |
| `cdn_nginx_browser_absolute_path` | "/opt/files"  | An absolute path for the static files |

### Overrides

You can **override** the default values in inventory file stored in *group_vars/all.yaml*.  

## Ansible role: cdn_nginx_proxy

### Tags
-----
`cdn_nginx_proxy`

### Description

Installs and configure **nginx server** as a reverse proxy with cache.

### Variables
All variables which can be overridden are stored in *defaults/main.yaml* file as well as in table below.

| Name                           | Default Value | Description                                                              |
| ------------------------------ | ------------- | ------------------------------------------------------------------------ |
| `cdn_nginx_browser_listen`     | "80"          | Port, that nginx server listens to                                       |
| `cdn_nginx_proxy_cache`        | {}            | Nginx reverse proxy cache settings (**path**, **inactive**, **max_size** |
| `cdn_nginx_proxy_pass_address` | ""            | Server address to redirect                                               |
| `cdn_nginx_proxy_pass_port`    | ""            | Application port to redirect                                             |


### Overrides

You can **override** the default values in inventory file stored in *group_vars/all.yaml*.  
It is **required** to **override** the **cdn_nginx_proxy_pass_address** and **cdn_nginx_proxy_pass_port** variables.

```
cdn_nginx_proxy_pass_address: "192.168.1.10"
cdn_nginx_proxy_pass_port: "80"
```

## Local Testing 

With [Anacoda](https://anaconda.org/) Virtual Environment
```
conda create -p venv/
conda install --file requirements.txt -y
conda activate venv/
```

With **Python 3 pip** package manager
```
pip3 install -r requirements.txt
```

If you have Ansible installed locally, **lucky you**.

## Run the Playbook!

If you haven't configured the SSH connection between your control node and master node for the first time, make a manual connection to your **master node**.

```
ssh-keyscan <your master node address> >> ~/.ssh/known_hosts
```

After that, you can use `--ask-pass` argument for *ssh password* with `-K` for *privilege escalation*.
```
ansible-playbook playbook.yaml --ask-pass -K 
```

If you have configured the ssh conection and overrired the **roles** vars in **inventory file**, use `-K` for *privilege escalation*.
```
ansible-playbook playbook.yaml -K
```
