## Task
Several new developers and DevOps engineers just joined the xFusionCorp industries. They have been assigned the Nautilus project, and as per the onboarding process we need to create user accounts for new joinees on at least one of the app servers in Stratos DC. We also need to create groups and make new users members of those groups. We need to accomplish this task using Ansible. Below you can find more information about the task.

There is already an inventory file ~/playbooks/inventory on jump host.

On jump host itself there is a list of users in ~/playbooks/data/users.yml file and there are two groups — admins and developers —that have list of different users. Create a playbook ~/playbooks/add_users.yml on jump host to perform the following tasks on app server 3 in Stratos DC.

a. Add all users given in the users.yml file on app server 3.

b. Also add developers and admins groups on the same server.

c. As per the list given in the users.yml file, make each user member of the respective group they are listed under.

d. Make sure home directory for all of the users under developers group is /var/www (not the default i.e /var/www/{USER}). Users under admins group should use the default home directory (i.e /home/devid for user devid).

e. Set password BruCStnMT5 for all of the users under developers group and LQfKeWWxWD for of the users under admins group. Make sure to use the password given in the ~/playbooks/secrets/vault.txt file as Ansible vault password to encrypt the original password strings. You can use ~/playbooks/secrets/vault.txt file as a vault secret file while running the playbook (make necessary changes in ~/playbooks/ansible.cfg file).

f. All users under admins group must be added as sudo users. To do so, simply make them member of the wheel group as well.



## Solution

- Create ansible.cfg file:
```
[defaults]
host_key_checking = False
vault_password_file = ~/playbooks/secrets/vault.txt
```

- Inventory file:
```
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=banner

[app_server3]
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=banner
```

- Create users.yml file:
```yaml
admins:
  - rob
  - david
  - joy

developers:
  - tim
  - ray
  - jim
```

- Content of vault.txt file: `P@ss3or432`

- Create user-pass.yml password file for the groups:
```yaml
---
admin_password: "LQfKeWWxWD"
dev_password: "BruCStnMT5"
```
  - Encrypt the user password (enter the password in the `vault.txt` file - the default password):
```bash
ansible-vault encrypt ~/playbooks/secrets/encrypted_passwords.yml
```

- Create playbook:
```yaml
---
- name: Manage users and groups on app server 1
  hosts: app_servers
  become: yes
  vars_files:
    - ~/playbooks/secrets/encrypted_passwords.yml
    - ~/playbooks/data/users.yml

  tasks:
    - name: Ensure groups exist
      ansible.builtin.group:
        name: "{{ item }}"
        state: present
      loop:
        - developers
        - admins
        - wheel  # For sudo access

    - name: Create users with correct groups and home directories
      ansible.builtin.user:
        name: "{{ item }}"
        group: "{{ 'admins' if item in admins else 'developers' }}"
        groups: "{{ 'wheel' if item in admins else '' }}"
        home: "{{ '/home/' ~ item if item in admins else '/var/www' }}"
        shell: /bin/bash
        password: "{{ (admin_password if item in admins else dev_password) | password_hash('sha512') }}"
      loop: "{{ admins + developers }}"

    - name: Set correct permissions for /var/www
      ansible.builtin.file:
        path: /var/www
        state: directory
        owner: root
        group: developers
        mode: '0775'
```

- Run the playbook:
```bash
ansible-playbook ~/playbooks/add_users.yml -i ~/playbooks/inventory
```
