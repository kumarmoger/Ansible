==========================
Configuration Management 
==========================

=> Below things comes into configuration management

	a) Installing required softwares in the machines

	b) Copy required files from one machine to another machine

	c) OS Patching/Updates

=> We can perform this configuration management in 2 ways

		1) Manual Configuration Management

		2) Automated Configuration Management

============================================
Problems with Manual Configuration Mgmt
============================================

1) Time Taking process

2) Repeated Work

3) Human Errors

4) Complex Rollback process

Note: To overcome these problem we are going to automate configuration management in the project.

=> To automate configuration management we have several tools

		1) puppet

		2) chef

		3) Ansible (trending)

================
What is Ansible
================		

-> It is an open source software developed by Michael DeHaan and its ownership is under RedHat.

=> Ansible was written in Python language.

-> Ansible is an automation tool that provides a way to define configuration as code.


=======================
Ansible Architecture
=======================

1) Control Node

2) Managed Nodes

3) Host Inventory File

4) Playbooks


=> The machine which contains ansible software is called as Controlling Node.

=> The machines which are managing by Controlling Node are called as Managed Nodes.

=> Host inventory file contains managed nodes information.

=> Playbook is a YML/YAML which contains set of taks (configuration as code).

===========================
Ansible Ad-Hoc Commands
===========================

=> To run ad-hoc commands we will follow below syntax


Syntax :  

$ ansible [all/group-name/private-ip] -m <module> -a <args>

Ex:

		$ ansible all -m ping

		$ ansible webservers -m ping

		$ ansible dbservers -m ping


=> We have several modules in ansible to perform configuration management

		1) ping

		2) shell

		3) yum / apt

		4) service

		5) copy
         
        6) service
 


		$ ansible all -m ping

		$ ansible all -m shell -a date

		$ ansible all -m shell -a uptime

		$ ansible all -m yum -a "name=git"

==================
Ansible Playbooks
==================

=> Playbook is a YML file

=> Playbook contains one or more tasks

=> Using playbooks we can define what task to be formed and where that task to be formed

=> We will give playbook as a input for ansible control node to perform tasks in managed nodes.

=> To write ansible playbooks we should learn YML/YAML first.

============
YML or YAML
============

=> YML stands for yet another markup language

=> It is used to store the data in both human & machine readable format.

=> YML files will have extension as .yml or .yaml

===========================
01 :: Sample YML File data
===========================

Note: Indentation (spacing) is very important is yml.

---

id: 101
name: Ashok
gender: Male
hobbies:
	- music
	- chess
	- cricket
	- swimming
...

===============
02 : YML FILE
===============

---
person:
 id: 101
 name: Ashok
 gender: Male
 address:
  city: Hyd
  state: TG
  country: India
 hobbies:
  - music
  - chess
...

===============
03 : YML FILE
===============

---
emp:
 id: 101
 name: Raj
 company: Microsoft
 job-info:
  skills: 
  	- DevOps
  	- Cloud
  	- Linux
  exp: 5 years
 address:
  city: Hyd
  state: TG
...

#### Website To validate YML syntax : https://www.yamllint.com/


==================
Writing Playbooks
==================

=> Playbook contains 3 sections

		1) Host section

		2) Variable Section

		3) Task Section


=> Host section represents the targeted machines to execute the task.

=> Variables section is used to declare variables required for playbook execution.

=> Task section is used to declare what operation we want to perform using Ansible.	


=================================
01-Playbook to ping managed nodes
=================================

---
- hosts: all
  gather_facts: yes
  tasks:
  - name: ping all managed nodes
    ping:
...

# It will check the syntax of a playbook
$ ansible-playbook <playbook-yml-file> --syntax-check

# It will display which hosts would be effected by a playbook before run
$ ansible-playbook <playbook-yml-file> --list-hosts

# Run the playbook Using below command
$ ansible-playbook <playbook-yml-file>

# Run the playbook in verbose mode
$ ansible-playbook <playbook-yml-file> -vvv

=================================
What is gather facts in ansible 
=================================

=> In Ansible, gathering facts refers to the process of collecting information about the target machines before executing tasks. 

    Ex: OS, memory, cpu architecture etc....

=> This information will be collected automatically using "setup" module.


=================================
What is debug keyword in ansible
=================================

=> debug keyword is used to print a msg when playbook is getting executed.

---
- hosts: all
  gather_facts: yes
  tasks:
  - name: ping nodes
    ping: 
  - name:  print os family
    debug:
     msg: "The os is {{ansible_os_family}}"
...

======================================
What is register keyword in ansible
======================================

=> register keyword in ansible allow you to capture the output of a task and store it into a variable for later use.

Note: one task output we can register and we can use it in another task like below


---
- hosts: localhost
  tasks:
  - name: Get Today's Date
    command: date
    register: i
  - name: Print date 
    debug:
     msg: "Current Date {{i.stdout}}"
...


==============================
Error Handling in Playbooks
==============================

=> If we get any error in task execution then playbook execution will be terminated abnormally
 (in the middle).

=> If we get any error in first task execution then remaining tasks will not be executed.

=> We can handle errors in playbook and we can continue remaining tasks execution using 'ignore_errors' concept.

---
- hosts: localhost
  tasks:
  - name: get date
    command: dates
    ignore_errors: yes 
  - name: get whoami
    command: whoami
...



===========================================
02-Playbook to create file & copy content
===========================================

---
- hosts: all
  tasks:
  - name: create a file
    file:
     path: /home/ansible/f1.txt
     state: touch
  - name: copy content to a file
    copy: content="welcome\n" dest="/home/ansible/f1.txt"
...

===========================================
03-Playbook to host static website
===========================================

---
- hosts: webservers
  become: true
  tasks:
  - name: install httpd server
    yum:
     name: httpd
     state: latest
  - name: copy index.html
    copy:
     src: index.html
     dest: /var/www/html/index.html
  - name: start httpd service 
    service:
     name: httpd
     state: started
...

=================
Handlers & Tags
=================

-> In playbook all tasks will be executed by default in sequential order.

=> Using Handlers we can execute tasks based on other tasks status.

Note: If 2nd task status is changed then only i want to execute 3rd task.

-> Handlers are used to notify the tasks to execute.

=> 'notify' keyword we will use to inform handler to execute.

---
- hosts: webservers
  become: true
  tasks:
  - name: install httpd package
    yum:
     name: httpd
     state: latest
  - name: copy index.html file
    copy:
     src: index.html
     dest: /var/www/html/index.html
    notify:
      start httpd service
  handlers: 
  - name: start httpd service
    service:
     name: httpd
     state: started
...

-> Using Tag we can map task to a tag-name

-> Using tag name we can execute particular task and we can skip particular task available in our playbook.


---
- hosts: webservers
  become: true
  tasks:
  - name: install httpd package
    yum:
     name: httpd
     state: latest
    tags:
    - install
  - name: copy index.html file
    copy:
     src: index.html
     dest: /var/www/html/index.html
    tags: 
    - copy
    notify:
      start httpd service
  handlers: 
  - name: start httpd service
    service:
     name: httpd
     state: started
...

# to display all tags available in playbook
$ ansible-playbook handlers_tags.yml --list-tags

# Execute a task whose tag name is install
$ ansible-playbook handlers_tags.yml --tags "install"

# Execute the tasks whose tags names are install and copy
$ ansible-playbook handlers_tags.yml --tags "install,copy"

# Execute all the tasks in playbook by skipping install task
$ ansible-playbook handlers_tags.yml --skip-tags "install"


===============
variables
==============

=> Variables are used to store the data in key-value format

Ex:

id=101
name=ashok
age=20
gender=male

=> In Ansible also we can use variables

1) Runtime Variables

2) Playbook variables


==================
Runtime Variables 
==================

=> We can pass variable value in runtime like below

---
- hosts: webservers
  become: true
  tasks:
  - name: install package
    yum:
     name: "{{pkg_name}}"
     state: latest
...

$ ansible-playbook <yml> --extra-vars pkg_name=httpd

===================
Playbook Variables 
===================

=> We can declare variable value with in the playbook only like below

---
- hosts: webservers
  become: true
  vars:
   pkg_name: httpd
  tasks:
  - name: install package
    yum:
     name: "{{pkg_name}}"
     state: latest
  - name: print a msg
    debug:
     msg: "{{pkg_name}}" installed
...

===============
Ansible Vault
===============

=> It is used to secure our playbooks

=> Using Ansible vault concept, we can encrypt & decrypt our playbooks.

Encryption : Converting data from readable format to un-readable format.

Decryption : Convert data from un-readable format to readable format.


# Encrypt playbook
$ ansible-vault encrypt <yml>

Note: To encrypt a playbook we need to set one password.

# see encrypted playbook
cat <yml-file-name>

# see orignal content of playbook
ansible-vault view <yml-file-name>

# to edit encrypted playbook
ansible-vault edit <yml-file-name>

# how to run encrypted playbook
ansible-playbook <yml-file-name> --ask-vault-pass

# decrypt playbook
ansible-playbook decrypt <yml>

### Assignment : If we forgot vault-password how to recover it ?


===============
Ansible Roles
===============

=> If we write more functionalities in single playbook then it will become difficult to manage that playbook.

=> We can divide large playbooks into small chunks using Ansible Roles concept.

=> In below playbook we have 3 tasks including handler...

---
- hosts: webservers
  become: true
  tasks:
  - name: install httpd package
    yum:
     name: httpd
     state: latest
    tags:
    - install
  - name: copy index.html file
    copy:
     src: index.html
     dest: /var/www/html/index.html
    tags: 
    - copy
    notify:
      start httpd service
  handlers: 
  - name: start httpd service
    service:
     name: httpd
     state: started
...

### To work with Roles we will use Ansible-Galaxy Concept ###

Syntax: $ ansible-galaxy init <role-name>

===========================
Working with Ansible Roles
===========================

### Step-1: Connect with control node and switch to ansible user

$ sudo su ansible
$ cd ~

### Step-2 : Create a role using 'ansible-galaxy'

$ mkdir roles

$ cd roles

$ ansible-galaxy init apache

$ sudo yum install tree

$ tree apache

Note: Here we can seperate yml files for everything

    tasks/main.yml

    handlers/main.yml

    vars/main.yml

    files/


### Step-3 : Create all tasks inside "tasks/main.yml" like below

---
# tasks file for apache
- name: install httpd
  yum:
    name: httpd
    state: latest
- name: copy index.html
  copy:
    src=index.html
    dest=/var/www/html/
  notify:
    - restart apache
...

### Step-4 : Copy/keep required files into "files" directory

Note: keep index.html file in files directory

### Step-5 : configure handlers in "handlers/main.yml"

---
# handlers file for apache
- name: restart apache
  service:
    name: httpd
    state: restarted
...

Note: With above 5 steps our "apache"  role is ready now we can execute that role like below

### Step-6 : Create main playbook to invoke role using role name

$ cd ~
$ vi invoke-roles.yml

---
- hosts: all
  become: true
  roles:
    - apache
...

===========================
Ansible Playbook with Loop
===========================

=> Loop is used to execute same code multiple times..

ex: install multiple softwares using single task

    create multiple users using single task

    create multiple files using single task

## Requirement : write a playbook to install git, maven and java softwares.

---
- hosts: all
  become: yes
  tasks:
  - name: install multiple packages using loop
    yum:
     name: "{{pkg_name}}"
     state: present
    loop:
    - git
    - maven
    - java
    - tree
  - name: create multiple users
    user:
     name: "{{uname}}"
     state: present
    loop:
     - devuser1
     - devuser2
     - devuser3
     - devuser4
...

===========================================
Create users along with pwd using playbook
==========================================

---
- name: Ansible Playbook with loop and password
  hosts: all
  become: yes
  vars:
    users_list:
      - { name: "devuser1", password: "Password@123" }
      - { name: "devuser2", password: "Welcome@123" }
      - { name: "devuser3", password: "AshokIT@123" }

  tasks:
    - name: Create users with password
      user:
        name: "{{ item.name }}"
        password: "{{ item.password | password_hash('sha512') }}"
        shell: /bin/bash
        state: present
      loop: "{{ users_list }}"
...


==========================
Scenarion based question
==========================

We have 2 managed nodes one is belongs to Red hat family and another one is belongs to debian family..

Write an ansible playbook to install java software in both managed nodes.

---
- hosts: all
  gather_facts: yes
  become: yes
  tasks:
  - name: install java in RED Hat family
    yum: 
     name: java
     state: present
    when: ansible_os_family == "RedHat"
  - name: install java in Debian family
    apt: 
     name: java
     state: present
    when: ansible_os_family == "Debian"
...

================
Ansible Tower
================

=> It is a web application provided by RED Hat company

=> Ansible Tower is license based software.

=> Ansible tower provides User interface to manage Ansible playbooks scheduling and execution.