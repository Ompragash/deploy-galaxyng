# Ansible Galaxy NG: Up and Running

[Galaxy NG](https://github.com/ansible/galaxy_ng) is the next generation Galaxy server to host Ansible contents. It is based on [Pulp](https://github.com/pulp/pulp_ansible).

###  Environment
----------------
Tested on:
  - AlmaLinux 8.6, ~=9
  - Fedora 36
  - RHEL & CentOS ~=8, ~=9

### Requirements
----------------
- python ~=3.9
- ansible-core ~=2.13
- For better _performance_, it's recommended to have a minimum of **2 CPUs** and **4 GiB RAM**

### Deployment Instruction
--------------------------
Install required OS packages
```bash
dnf install -y git python39
```

Install `ansible-core`
```bash
pip3 install ansible-core
```

Clone [deploy-galaxyng](https://github.com/Ompragash/deploy-galaxyng) git repo
```bash
git clone https://github.com/Ompragash/deploy-galaxyng.git
cd deploy-galaxyng
```
> _**Note:** Run `ls -l` and check the files in `deploy-galaxyng` directory_

Install geerlingguy.postgresql Role and pulp_installer Collection
```bash
ansible-galaxy role install geerlingguy.postgresql,3.3.1
ansible-galaxy collection install pulp.pulp_installer:3.18.5
```

> _**Note:** If `ansible*` utilities aren't found then add `/usr/local/bin` to `PATH` by running this command:_ echo 'export PATH=$PATH:/usr/local/bin' > ~/.bashrc && source ~/.bashrc

Install pulp_installer dependencies
```bash
ansible-galaxy install -r ~/.ansible/collections/ansible_collections/pulp/pulp_installer/requirements.yml
```

Explore the files in `deploy-galaxyng` directory:
- **inventory**           - Contains HOST IP/FQDN where Galaxy NG will be deployed
- **install-vars.yml**    - Contains variables needed by the pulp_installer roles to setup and configure pulp services & Galaxy NG
- **deploy-galaxyng.yml** - Main playbook that invokes pulp* roles
- **ansible.cfg**         - Project based Ansible configuration file that displays overall time taken for the playbook execution and time time taken per task basis.

With the current _inventory_ file, ansible manages _localhost_ via _local_ connection. Modify inventory file as per need to manage remote servers.

Open _install-vars.yml_ |-
  - Set _admin_ password to *pulp_default_admin_password* variable
  - Refer [version_matrix](docs/vertion_matrix.md) and set version to _galaxy-ng_, _pulpcore_, _pulp-ansible_ & _pulp-container_ variables. Current setting installs Galaxy NG 4.5.0.

Run `deploy-galaxyng.yml` 
```bash
ansible-playbook deploy-galaxyng.yml -e "@install-vars.yml" -i inventory
```
> _**Note:** Playbook execution takes around ~13 minutes to complete._

On successful execution, open `https://localhost/` in any browser and login with username (**admin**) & password (value of **pulp_default_admin_password** variable)

Yayy!🎉 All set! 
Finally, Publish/download Ansible content to Self-hosted Galaxy NG server!



