## Task

The Nautilus DevOps team want to install and set up a simple httpd web server on all app servers in Stratos DC. They also want to deploy a sample web page using Ansible. Therefore, write the required playbook to complete this task as per details mentioned below.

We already have an inventory file under /home/thor/ansible directory on jump host. Write a playbook playbook.yml under /home/thor/ansible directory on jump host itself. Using the playbook perform below given tasks:

Install httpd web server on all app servers, and make sure its service is up and running.

Create a file /var/www/html/index.html with content:

This is a Nautilus sample file, created using Ansible!

Using lineinfile Ansible module add some more content in /var/www/html/index.html file. Below is the content:

Welcome to xFusionCorp Industries!

Also make sure this new line is added at the top of the file.

The /var/www/html/index.html file's user and group owner should be apache on all app servers.

The /var/www/html/index.html file's permissions should be 0744 on all app servers.


## Solution

```yaml
---
- name: Service httpd
  hosts: all
  become: yes
  tasks:
    - name: Install the latest version of httpd
      ansible.builtin.yum:
        name: httpd
        state: present
    - name: Start httpd
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
    - name: Create the index.html file
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: |
            This is a Nautilus sample file, created using Ansible!
    - name: Add a line to a file
      ansible.builtin.lineinfile:
        path: /var/www/html/index.html
        line: Welcome to xFusionCorp Industries!
        insertbefore: BOF
        mode: '0644'        
        owner: 'apache'
        group: 'apache'
```
