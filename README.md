# Ansible Beginner

- [Resource PDF](https://kodekloud.com/wp-content/uploads/2021/11/ansible-for-beginners-for-pdf.pdf)

## Why Ansinle?

- Provisioning
- Configuration Management
- Continuous Delivery (CD)
- Application Deployment
- Security Compliance

## Scripts VS Ansible

- Scripts
  - Time
  - Coding skill
  - Maintenance
- Ansible
  - Simple
  - Powerful
  - Agentless

## What is YAML?

- YAML Ain't Markup Language
- YAML is a human friendly data serialization standard for all programming languages
- YAML is a superset of JSON
- YAML is a superset of XML
- YAML is a superset of INI

```yaml
---
- name: Install Apache
  hosts: web
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
    - name: Start Apache
      service:
        name: httpd
        state: started
```

### Key value pair

```yaml
Fruit: Apple
Vegetable: Carrot
Meat: Beef
```

### Array/List

```yaml
Fruit:
  - Apple
  - Orange
  - Banana
Vegetable:
  - Carrot
  - Potato
  - Tomato
Meat:
  - Beef
  - Chicken
  - Pork
```

### Dictionary

```yaml
Fruit:
  Apple:
    Color: Red
    Price: 1.99
  Orange:
    Color: Orange
    Price: 2.99
  Banana:
    Color: Yellow
    Price: 0.99
Vegetable:
  Carrot:
    Color: Orange
    Price: 0.99
  Potato:
    Color: Brown
    Price: 0.99
  Tomato:
    Color: Red
    Price: 0.99
```

### Notes

- Dictionary - Unordered
- Array/List - Ordered

## Playbook

- Playbook is a file containing one or more plays
- Playbook is written in YAML format
- Playbook is the starting point of any automation
- Playbook contains plays
- Playbook is executed using the ansible-playbook command

```yaml
# playbook.yml
---
- name: Install Apache
  hosts: web
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
    - name: Start Apache
      service:
        name: httpd
        state: started
```

```bash
ansible-playbook playbook.yml
```

## Ansible Inventory

- Ansible inventory is a file that contains the list of managed nodes

```bash
# /etc/ansible/hosts
[web]
web1
web2

[db]
db1

[servers:children]
web
db
```

## Ansible Modules

- System modules
- Commands modules
- Files modules
- Database modules
- Cloud modules
- Windows modules
- More...

```yaml
# playbook.yml
---
- name: Install Apache
  hosts: web
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
    - name: Start Apache
      service:
        name: httpd
        state: started # started, restarted, reloaded or stopped
    - name: Create index.html
      copy:
        content: '<h1>Hello World</h1>'
        dest: /var/www/html/index.html
    - name: Create ssl directory
      file:
        path: /etc/ssl/certs
        state: directory
```

## Ansible Variables

Stores information that varies with each host

```yaml
# Inventory
Web1 ansible_host=server1.example.com ansible_user=ansible ansible_password=ansible
db ansible_host=server2.example.com ansible_user=ansible ansible_password=ansible
```

```yaml
# playbook.yml
---
- name: Install Apache
  hosts: web
  become: yes
  vars:
    http_port: '{{ http_port }}'
    max_clients: '{{ max_clients }}'
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
    - lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: '^Listen'
        line: 'Listen {{ http_port }}' # {{ http_port }} is a variable
---
- name: Install SQL
  hosts: db
  become: yes
  vars:
    http_port: '{{ http_port }}'
    max_clients: '{{ max_clients }}'
  tasks:
    - name: Install SQL
      yum:
        name: mariadb-server
        state: present
```

```yaml
# Variables
http_port: 80
max_clients: 200
```

## Ansible Conditionals

```yaml
# playbook.yml
---
- name: Install NGINX
  hosts: all
  tasks:
    - name: Install NGINX on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == 'RedHat'
    - name: Install NGINX on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == 'Debian'
```

### Conditionals in Loops

```yaml
# playbook.yml
---
- name: Install NGINX
  hosts: all
  vars:
    packages:
      - name: nginx
        required: yes
      - name: httpd
        required: no
      - name: mysql
        required: yes
      - name: apache
        required: yes
  tasks:
    - name: Install "{{ item.name }}" package on Debian
      apt:
        name: '{{ item.name }}'
        state: present
      when: ansible_os_family == 'Debian' and item.required == 'yes'
      loop: '{{ packages }}'
```

### Conditionals and Register

```yaml
# playbook.yml
---
- name: Check status of a service and email if its down
  hosts: all
  tasks:
    - name: Check status of a service
      shell: systemctl httpd status
      register: result
    - name: Email if service is down
      mail:
        to: admin@example.server
        subject: Service is Alert
        body: Httpd Service is down
      when: result.stdout.find('down') != -1
```

## Ansible Loop

```yaml
# playbook.yml
---
- name: Create users
  hosts: all
  tasks:
    - user: name= '{{ item.name }}' uid= '{{ item.uid }}' state= present
      loop:
        - name: user1
          uid: 1001
        - name: user2
          uid: 1002
        - name: user3
          uid: 1003
```

```yaml
# playbook.yml
---
- name: Create users
  hosts: all
  tasks:
    - user: name= '{{ item }} state= present
      with_items: # with_file, with_url, with_mongodb
        - user1
        - user2
        - user3
```

## Ansible Roles

- Roles are the recommended way for structuring your Ansible content
- Roles are a set of Ansible tasks, handlers, templates, files, variables, and other Ansible components
- Roles are used to organize and structure your Ansible content

```bash
# Create a role
ansible-galaxy init role_name # (geerlingguy.apache)
# Install a role
ansible-galaxy install geerlingguy.apache
```

```yaml
# playbook.yml
---
- name: Install Apache
  hosts: web
  become: yes
  roles:
    - geerlingguy.apache
```
