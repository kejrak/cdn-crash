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