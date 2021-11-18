# Apache Role

This role deploys and configures Apache Web server.
To run the deployment you have to configure inventory first.

## Static inventory
INI-like or YAML file defining which hosts to manage. Put it to the same directory as `playbook.yml`.

<details>
<summary>hosts.ini</summary>

```
[www]
35.232.181.231

[mysql]
10.128.0.35
```
</details>

Run the following command to review your inventory:

```
$ ansible-inventory --graph -i hosts.ini 
@all:
  |--@mysql:
  |  |--10.128.0.35
  |--@ungrouped:
  |--@www:
  |  |--35.232.181.231
```

Please keep in mind that in your case the IP addresses will be different.

## Dynamic inventory
Inventory configuration for GCP.

### Pre-requesites
* `google-auth` Python module has to be installed. Run `sudo pip install requests google-auth` if it's not.
* Create a [Google IAM service account](https://cloud.google.com/iam/docs/creating-managing-service-accounts) and grant it enough permissions to get the GCE resources details
* Download the Google IAM service account's key (json file) to the Ansible Controle Node. For example - to the `/opt/ansible/inventory/` folder.

Create `gcp.yaml` file like below in the same directory as `playbook.yml`.

<details>
<summary>gcp.yaml</summary>

```
plugin: gcp_compute
projects:
  - 
auth_kind: serviceaccount
service_account_file: /opt/ansible/inventory/service-account.json
keyed_groups:
  # Create groups from GCE labels
  - prefix: gcp
    key: labels
hostnames:
  # List host by name (or private_ip) instead of the default public ip
  #- name
  #- public_ip
  - private_ip
compose:
  # Set an inventory parameter to use the Public IP or Private IP address to connect to the host
  ansible_host: networkInterfaces[0].networkIP
  #ansible_host: networkInterfaces[0].accessConfigs[0].natIP
```

</details>

Also, you may find [this article](https://devopscube.com/ansible-dymanic-inventry-google-cloud/) heplful.

Run the following command to review your inventory:

```
$ ansible-inventory --graph -i gcp.yaml
@all:
  |--@gcp_role_mysql:
  |  |--10.128.0.40
  |--@gcp_role_www:
  |  |--10.128.0.39
  |--@ungrouped:
```

### Ansible Config

Create `ansible.cfg` file like below in the same directory as `playbook.yml`.

<details>
<summary>ansible.cfg</summary>

```
[defaults]
inventory = gcp.yaml
host_key_checking = False
record_host_keys = False
remote_user = abondar
private_key_file = /home/user/.creds/ssh-keys/user.rsa

[privilege_escalation]
become = True
become_user = root
become_method = sudo
become_ask_pass = False
```

</details>

### Deployment
Change folder to `../../` and run this:
```
ansible-playbook playbook.yml
```