# 4640-w7-lab-start-w25

Create a new SSH key pair:

 `ssh-keygen -t ed25519 -f ~/.ssh/4640lab7`

Import the public key:
`./import_lab_key ~/.ssh/4640lab7.pub`

check the terraform configuration file:

vim main.tf

Run the terraform to build two ec2:
`terraform init` 

`terraform plan -out plan`

`terraform apply “plan”`

Edit ./ansible/inventory/hosts.yml:

```yaml
all:
  children:
    web:
      hosts:
        server-one:
          ansible_host: 35.86.115.18
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/4640lab7
        server-two:
          ansible_host: 54.185.246.25
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/4640lab7
```

Edit ./ansible/playbook.yml:

```yaml
- name: Configure web servers
  hosts: all
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
    - name: create directory structure for web documents
      ansible.builtin.file:
        path: /web/html
        state: directory
        mode: '0755'
    - name: copy nginx conf file to server
      ansible.builtin.copy:
        src: ./files/nginx.conf
        dest: /etc/nginx/sites-available/default
        mode: '0644'
    - name: create link to nginx config file to enable it
      ansible.builtin.file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link
    - name: Generate index.html file from template
      ansible.builtin.template:
        src: ./templates/index.html.j2
        dest: /web/html/index.html
        mode: '0644'
    - name: reload nginx service
      ansible.builtin.service:
        name: nginx
        state: reloaded

```

Run playbook:

```bash
ansible-playbook playbook.yml
```
![screenshot1](https://github.com/user-attachments/assets/3c0bbb15-a0aa-4f05-af46-2a258d18d5df)
