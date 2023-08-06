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

You can **override** the default value of **cdn_connection_ssh** in inventory file stored in group_vars/all.yal

```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

cdn_connection_ssh: "{{ ssh }}"
```


## Ansible role: cdn_connection

### Description

Setups connection between **control node (~localhost)** and **master node**, which will run docker containers. Role generates ssh key, creates config file and set authorized key on **master node**.

### Variables

All variables which can be overridden are stored in defaults/main.yaml file as well as in table below.

| Name                 | Default Value | Description                                                                                              |
| -------------------- | ------------- | -------------------------------------------------------------------------------------------------------- |
| `cdn_connection_ssh` | {}            | Location and name for generating SSH key for connection (**private_key_location**, **private_key_name**) |

### Overrides

You can **override** the default value of **cdn_connection_ssh** in inventory file stored in groups_vars/all.yaml

```
ssh:
  private_key_location: "~/.ssh"
  private_key_name: "cdn"

cdn_connection_ssh: "{{ ssh }}"
```

## Ansible role: cdn_docker

### Description

Installs and configures **docker engine** on master node based on provided unix distribution, which will run docker containers as VMs with **systemd** installed. The default setting creates multiple docker containers with **ipvlan** network driver which are included in *cdn_containers* group. It also notifies the **control node (~localhost)** about docker containers and adds them to **known hosts**.

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

You can **override** the default values in inventory file stored in group_vars/all.yaml  
It is **required** to **overrride** the **cdn_docekr_network_settings**.

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

## Configure the inventory.yaml

In inventory/inventory.yaml, add yours free addresses for hosts.
It's mandatory to use the same addresses in your **cnd_docker_network_settings** CIDR range (**subnet** and **gateway**).

## Run the playbook.yaml

For the first time, if you haven't configure the ssh connection between you control and master node,
use `--ask-pass` argument for **SSH password**.
```
ansible-playbook playbook.yaml -K --ask-pass
```

If you configured the ssh conection and overired the **roles** vars in groups_vars, just use **-K** for *Privilege Escalation*.
```
ansible-playbook playbook.yaml -K
```
