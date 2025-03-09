

## Task

The Nautilus DevOps team is trying to setup a simple Apache web server on all app servers in Stratos DC using Ansible. They also want to create a sample html page for now with some app specific data on it. Below you can find more details about the task.


You will find a valid inventory file /home/thor/playbooks/inventory on jump host (which we are using as an Ansible controller).


Create a playbook index.yml under /home/thor/playbooks directory on jump host. Using blockinfile Ansible module create a file facts.txt under /root directory on all app servers and add the following given block in it. You will need to enable facts gathering for this task.

Ansible managed node architecture is <architecture>



(You can obtain the system architecture from Ansible's gathered facts by using the correct Ansible variable while taking into account Jinja2 syntax)


Install httpd server on all apps. After that make a copy of facts.txt file as index.html under /var/www/html directory. Make sure to start httpd service after that.

Note: Do not create a separate role for this task, just add all of the changes in index.yml playbook.

## Solution
```yaml
---
- name: Manage facts and install httpd
  hosts: all  # app_servers
  become: yes
  gather_facts: yes

  tasks:
    - name: Create facts.txt with architecture details
      blockinfile:
        path: /root/facts.txt
        create: yes
        block: |
          Ansible managed node architecture is {{ ansible_architecture }}

    - name: Install httpd server
      package:
        name: httpd
        state: present

    - name: Copy facts.txt to index.html
      copy:
        src: /root/facts.txt
        dest: /var/www/html/index.html
        remote_src: yes

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes
```
