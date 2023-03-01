# my-Ansible
# ad-hoc commands
```
ansible-doc shell
ansible localhost -m setup #facts about localhost
ansible-playbook myscript.yaml --syntax-check
```
# playbook.yml
```
---
- name: Install packeges
  hosts: webservers
  gather_facts: false
  become: yes
  tasks:

  - name: Current ip
    debug:
      msg: "https://{{ ansible_facts.default_ipv4.address }}:80"
  
  - name: Modyfy file
    shell:
      cmd: "echo hello > mynewfile.txt"
      chdir: "/root/"

  - name: Create file 2
    file:
      path: "/root/"
      owner: root
      group: root
      mode: "0644"
      state: touch

  - name: Change file text
    lineinfile:
      path: /root/mynewfile.txt
      regexp: '^hello'
      line: Helllo2

  - name: install tuned
    apt:
      name: tuned
```
# roles
    galaxy.ansible.com
    ansible-galaxy install geerlingguy.apache -p ./
```
    ---
- name: Install packeges
  hosts: webservers
  gather_facts: false
  become: yes
  tags: 
    - files
  roles:
    - myrole
<<<<<<< HEAD

or

- name: Current ip
  debug:
    msg: "https://{{ ansible_facts.default_ipv4.address }}:80"

- name: Install packeges
  hosts: all
  gather_facts: false
  become: yes
  tags: 
    - packages
  roles:
    - name: "test"
      role: myrole
      vars:
        username: "Aziz2"
```
![image](https://user-images.githubusercontent.com/118117183/222061281-9f5a913d-40bb-4849-95a9-fcbab163adaa.png)
