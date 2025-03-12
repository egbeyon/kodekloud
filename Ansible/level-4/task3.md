## Task

Nautilus Application development team wants to test the Apache and PHP setup on one of the app servers in Stratos Datacenter. They want the DevOps team to prepare an Ansible playbook to accomplish this task. Below you can find more details about the task.

There is an inventory file ~/playbooks/inventory on jump host.

Create a playbook ~/playbooks/httpd.yml on jump host and perform the following tasks on App Server 3.

a. Install httpd and php packages (whatever default version is available in yum repo).

b. Change default document root of Apache to /var/www/html/myroot in default Apache config /etc/httpd/conf/httpd.conf. Make sure /var/www/html/myroot path exists (if not please create the same).

c. There is a template ~/playbooks/templates/phpinfo.php.j2 on jump host. Copy this template to the Apache document root you created as phpinfo.php file and make sure user owner and the group owner for this file is apache user.

d. Start and enable httpd service.


## Solution

```yaml
---
- hosts: app3 # create a different block for stapp03 and call it app3 on the inventory file
  become: true
  tasks:
    - name: Install httpd and php packages
      yum:
        name:
          - httpd
          - php
        state: present

    - name: Create custom document root directory
      file:
        path: /var/www/html/myroot
        state: directory
        owner: apache
        group: apache
        mode: '0755'

    - name: Change Apache document root in httpd.conf
      lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: '^DocumentRoot "/var/www/html"'
        line: 'DocumentRoot "/var/www/html/myroot"'
        state: present
      notify: restart_httpd

    - name: Change directory directive in httpd.conf
      lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: '^<Directory "/var/www/html">'
        line: '<Directory "/var/www/html/myroot">'
        state: present
      notify: restart_httpd

    - name: Copy phpinfo.php template to document root
      template:
        src: ~/playbooks/templates/phpinfo.php.j2
        dest: /var/www/html/myroot/phpinfo.php
        owner: apache
        group: apache
        mode: '0644'
      notify: restart_httpd

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: true

  handlers:
    - name: restart_httpd
      service:
        name: httpd
        state: restarted
```
