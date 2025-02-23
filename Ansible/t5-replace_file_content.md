## Task

There is some data on all app servers in Stratos DC. The Nautilus development team shared some requirement with the DevOps team to alter some of the data as per recent changes they made. The DevOps team is working to prepare an Ansible playbook to accomplish the same. Below you can find more details about the task.

Write a playbook.yml under /home/thor/ansible on jump host, an inventory is already present under /home/thor/ansible directory on Jump host itself. Perform below given tasks using this playbook:

We have a file /opt/sysops/blog.txt on app server 1. Using Ansible replace module replace string xFusionCorp to Nautilus in that file.

We have a file /opt/sysops/story.txt on app server 2. Using Ansiblereplace module replace the string Nautilus to KodeKloud in that file.

We have a file /opt/sysops/media.txt on app server 3. Using Ansible replace module replace string KodeKloud to xFusionCorp Industries in that file.


## Solution

```yaml
- name: Replace string in files
  hosts: all
  become: yes
  tasks:
    - name: Replace xFusionCorp with Nautilus in blog.txt
      replace:
        path: /opt/dba/blog.txt
        regexp: 'xFusionCorp'
        replace: 'Nautilus'
      delegate_to: app_server_1

    - name: Replace Nautilus with KodeKloud in story.txt
      replace:
        path: /opt/dba/story.txt
        regexp: 'Nautilus'
        replace: 'KodeKloud'
      delegate_to: app_server_2

    - name: Replace KodeKloud with xFusionCorp Industries in media.txt
      replace:  
        path: /opt/dba/media.txt
        regexp: 'KodeKloud'
        replace: 'xFusionCorp Industries'
      delegate_to: app_server_3
```

-ref: https://prathapreddy-mudium.medium.com/ansible-replace-module-c8ec77e0b400#:~:text=The%20replace%20module%20in%20Ansible%20is%20used%20to%20replace%20all,or%20scripts%20with%20new%20values.
