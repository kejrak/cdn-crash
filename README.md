# CDN - CRASH COURSE WITH ANSIBLE

## Ansible role: cdn_connection

### Description

Setup connection between **control node (~localhost)** and **master node**, which will run docker containers. Role generates ssh key, creates config file and set authorized key on **master node**.

### Variables

All variables which can be overridden are stored in defaults/main.yaml file as well as in table below.

| Name                 | Default Value | Description                                             |
| -------------------- | ------------- | ------------------------------------------------------- |
| `cdn_connection_ssh` | {}            | Location and name for generating SSH key for connection |

### Overrides

You can **override** the default value of **cdn_connection_ssh** in inventory file stored in groups_vars/all.yal

```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

cdn_connection_ssh: "{{ ssh }}"
```


## Ansible role: cdn_connection

### Description

Setup connection between **control node (~localhost)** and **master node**, which will run docker containers. Role generates ssh key, creates config file and set authorized key on **master node**.

### Variables

All variables which can be overridden are stored in defaults/main.yaml file as well as in table below.

| Name                 | Default Value | Description                                             |
| -------------------- | ------------- | ------------------------------------------------------- |
| `cdn_connection_ssh` | {}            | Location and name for generating SSH key for connection |

### Overrides

You can **override** the default value of **cdn_connection_ssh** in inventory file stored in groups_vars/all.yal

```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

cdn_connection_ssh: "{{ ssh }}"
```

## Ansible role: cdn_docker

### Description

Install and configure **docker engine** on master node based on provided unix distribution, which will run docker containers as VMs with systemd installed. The default settings creates multiple docker containes with **ipvlan** network driver included in *cdn_containers* group. It also notify the host about created dokcker containers, so it can manage ssh connection from **control node**.

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

You can **override** the default values in inventory file stored in groups_vars/all.yal
Its **required** to **overrride** the **cdn_docekr_network_settings**.

Like this **example**:

```
cdn_docker_network_settings:
  subnet: "192.168.1.0/24"
  gateway: "192.168.1.1"
  interface: "enp0s25"
```

## Local Testing 

With Anacoda Virtual Environment
```
conda create -p venv/
conda install --file requirements.txt -y
conda activate venv/
```

With local pip package manager
```
pip3 install -r requirements.txt
```