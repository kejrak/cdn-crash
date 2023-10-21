# CDN - CRASH COURSE WITH ANSIBLE

## Prerequisities

- **Master node** with **ssh-server** installed (Debian Bullseye)
- An user account (**cdn** user is a default user for this playbook, it can be overridden)
- This node will hosts all of your docker containers

## Inventory

To create or edit the **inventory** file, navigate to the path *inventory/inventory.yaml*.

The Docker containers will function as additional **virtual machines** (VMs). They will be created on your master node and port-forwarded with selected **SSH port**.

Inventory in **.ini** format.
```
[cdn_master]
master_node ansible_port=22

[cdn_webservers]
web_server_node ansible_port=1022 nginx_port=81
web_proxy_node ansible_port=1123 nginx_port=82

[cdn_prometheus]
prometheus_node ansible_port=1222 prometheus_port=9090

[cdn_grafana]
grafana_node ansible_port=1322 grafana_port=3000

[cdn_containers:children]
cdn_webservers
cdn_monitoring
```

The *group_vars/all.yaml* file contains hosts-specific variables used for connecting to all hosts listed in the inventory file. These variables **shall be overriden** base on your specific needs. It is **required** to change the **ansible_host** which will be main host for the docker containers.
```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

ansibe_host: 192.168.1.46
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

- Multiple Docker containers with the default network driver will be created.
- These containers are included in the **cdn_containers** group.
- Notifications will be sent to the **control node (~localhost)** about the Docker containers.
- The containers will be added to the known hosts.
  
The role ensures that Docker Engine is set up correctly on the master node and manages the creation and configuration of Docker containers using the default network driver. Furthermore, it facilitates communication between the master node and the **control node (~localhost)** by notifying the control node about the created Docker containers and adding them to the known hosts list.

### Variables
All variables which can be overridden are stored in defaults/main.yaml file as well as in table below.

| Name                        | Default Value    | Description                                                                                              |
| --------------------------- | ---------------- | -------------------------------------------------------------------------------------------------------- |
| `cdn_docker_ssh`            | {}               | Location and name for generating SSH key for connection (**private_key_location**, **private_key_name**) |
| `cdn_docker_hosts`          | "cdn_containers" | Inventory group name of hosts as docker containers                                                       |
| `cdn_docker_project`        | " ~/cdn-project  | Location on master node to create project files                                                          |
| `cdn_docker_os`             | {}               | System distribution and version (**distribution**, **version**)                                          |
| `cdn_docker_settings`       | {}               | Docker settings (**user**, **password**, **image**)                                                      |
| `cdn_docker_network_name`   | "cdn-network"    | Docker network name                                                                                      |
| `cdn_docker_network_driver` | "ipvlan"         | Docker network driver                                                                                    |

### Overrides

You can **override** the default values in inventory file stored in *group_vars/all.yaml*.  

```
cdn_docker_ssh: "{{ ssh }}"

cdn_docker_settings:
  user: "{{ ansible_user }}"
  password: "admin"
  image: "cdn-debian-bullseye"

cdn_docker_network_name: "cdn-network"
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
cdn_nginx_proxy_pass_address: "{{ ansible_host }}" 
cdn_nginx_proxy_pass_port: "81"
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

You can use `--ask-pass` (for the first time) argument for *ssh password* with `-K` for *privilege escalation*.
```
ansible-playbook playbook.yaml --ask-pass -K 
```

