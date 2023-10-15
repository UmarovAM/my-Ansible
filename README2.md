cat  /etc/os-release
python --version
apt install git python3-pip
update-alternatives --install /usr/bin/py
update-alternatives --install /usr/bin/python python /usr/bin/python3.10 2
python --version
pip3 install ansible
pwd

 nano ansible.cfg

[defaults]
Inventory=/root/ansible/hosts
Host_key_chechking=false
#:W+q Vim

mkdir ansible
touch hosts

[all] #группа
client1 ansible_host=192.168.31.106 ansible_user=aziz ansible_password=8808        
client2 ansible_host=192.168.31.106 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa 
192.168.31.106 ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa 

Client1 ansible__host=192.168.31.106 ansible_ssh_private_key_file=/root/.ssh/rsa
Client1 ansible__host=192.168.31.106 ansible_user=root ansible_password=ptest #на хосте нужно настроить ssh!
192.168.31.106 ansible_user=testuser ansible_password=ptest
passwd #изменить пароль текущего пользователя
useradd -m -s /bin/bash/utest
passwd utest
ssh-keygen #без пароля нужно на клbенте скопировать rsa.pub на /root/.ssh/authorized_keys
ansible all -m ping 
#Нас просят подтвердить, поскольку подключается по SSH, нужно подтверждение сертификата. 
#И мы увидим первую ошибку, что доступ закрыт. Стандартная ошибка, это значит, что нам нужно ввести пароль. 
#На этот случай у Ansible существует команда --ask-pass.
#ansible all -i hosts.ini -m ping --ask-pass
#Которая спрашивает с каким паролем мы заходим. 
#Следующая ошибка, с которой мы сталкиваемся: нельзя использовать пароли без установки команды sshpass. 
#Проблема решается просто. sudo apt install sshpass
cd #должны запускать там где .cfg
apt sshpass #где хост с ansible

# ansible all -i hosts -m ping  
   ansible all -i hosts -m setup             
   ansible all -i hosts -m shell -a "uptime" 
   ansible all -i hosts -m command -a "uptime"
   ansible all -i hosts -m file -a "path=/root/file.html state=touch" -b      
   ansible all -i hosts -m file -a "path=/root/file2.html state=touch mode=777" -b    
             
# запуск как рут для этого visudo
Utest ALL=(ALL:ALL) NOPASSWD:ALL    
    
   
touch filecopy.html                       
ansible all -i hosts -m copy -a "src=filecopy.html dest=/home mode=777" -b 


# make group_vars

mkdir group_vars        

cd group_vars/   

 nano test

ansible_host: 192.168.31.124     
ansible_user: root                   
ansible_password: 12345678  
dev: chel1                               

cd group_vars/ 
 nano all_groups 

ansible_user:root 
ansible_ssh_private_key_file: /root/.ssh/id_rsa
dev: chel2 

# ИЗМЕНИМ HOSTS после добавления group_vars

 nano hosts 

[test]    
client01
[all_groups]  
client02 ansible_host=192.168.31.124  

ansible all_groups -i hosts -m debug -a "var=dev" 
ansible client01 -i hosts -m debug -a "var=dev" 
ansible all -i /root/ansible/hosts -m debug -a "var=dev"  
ansible all -i /root/ansible/hosts -m debug -a "var=ansible_host" 

# Playbook
 nano ping.yml   

- name: Ping Servers                 
  hosts: all                                        
  become: yes      
  tasks:                                              
  - name: Task ping                                       
    ping:         

  - name: Update cache                         
    apt:                                                   
      update_cache: yes                             
                                                                  
  - name: Upgrade                                                 
    apt:                                                        
      upgrade: yes 

ansible-playbook -i hosts ping.yml                                 
#apt-get update если ошибка, сначала запустить эту команду

nano play.yml 

- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  - name: Task ping
    ping:

  - name: Update cache
    apt:
      update_cache: yes

  - name: Upgrade
    apt:
      upgrade: yes

  - name: Install Nginx
    apt:
      pkg: 
        - nginx #нужно с маленькой буквы
        - tree
        - htop
      state: present
  
  - name: Copy file html
      copy:
        src: ./index.html
        dest: /var/www/html
        mode: 0777

ansible-playbook -i hosts play.yml 


#packages

- name: Ping Servers
  hosts: all
  become: yes

  vars:
    packages:
             - nginx
             - htop 
             - tree
             - rsync
    file_src: ./index.html
    file_dest: /var/www/html 
  
  tasks:

  - name: 1. Task ping
    ping:

  - name: 2. Install Nginx
    apt:
      pkg: "{{packages}}"
      state: present

  - name: 3. Copy File
    copy:
      src: "{{file_src}}"
      dest: "{{file_dest}}"
      mode: 0777

#Loops and with_items

- name: Loops
  hosts: all
  become: yes

  tasks:
  - name: Create Folder
    file:
      path: "/var/www/html/{{item}}"
      state: directory
    loop:
      - dir1
      - dir2

#items
- name: Create user
  hosts: all
  become: yes

  tasks:
  - name: 0. Create group
    group:
      name: "{{item}}"
      state: present
    loop:
      - dev
      - test

  - name: 1. Create user
    user: 
      name: test1
      shell: /bin/bash
      groups: dev,test
      append: yes
      home: /home/test1

#id test1

#[test]
#client01
[all_groups]
client02 ansible_host=192.168.31.124


nano hosts # for user_client

[test]    
client01 user_client=client01

[group2]  
client02 ansible_host=192.168.31.124 user_client=client02
[all_groups:children]
group2

#items2
- name: Create user
  hosts: all
  become: yes

  tasks:
  - name: 0. Create group
    group:
      name: "{{item}}"
      state: present
    loop:
      - dev
      - test

  - name: 1. Create user
    user: 
      name: "{{user_client}}"
      shell: /bin/bash
      groups: dev,test
      append: yes
      home: /home/test1

nano hosts # for user_client

[test]    
client01

[group2]  
client02 ansible_host=192.168.31.124
[all_groups:children]
group2

#items3 создать несколько пользователей
- name: Create user
  hosts: all
  become: yes

  tasks:
  - name: 0. Create group
    group:
      name: "{{item}}"
      state: present
    loop:
      - dev
      - test

  - name: 1. Create user
    user: 
      name: "{{item.clientname}}"
      shell: /bin/bash
      groups: dev,test
      append: yes
      home: "/home/{{item.homedir}}"
    with_items:
        - {clientname: client1, homedir: client1}
        - {clientname: client2, homedir: client2}

#debug message
- name: Message
  hosts: group2
  become: yes

  vars:
    slovo1: Dom
    slovo2: in
    mesto: RUS
    ip: "{{ansible_default_ipv4.address}}"

  tasks:

  - name: Print vars
    debug:
      var: slovo1

  - debug: 
      msg: "Moi dom v {{mesto}}"
  - debug: 
      msg: "Moi {{slovo1}} {{slovo2}} {{mesto}}" 
  - set_fact:
      message: "Moi {{slovo1}} {{slovo2}} {{mesto}}"
  - debug: 
      var: message
  - debug: 
      var: ansible_distribution_version
  - debug: 
      msg: "ip хоста {{ip}} version: {{ansible_distribution_version}}"
  - debug: 
      var: ansible_default_ipv4.address
  - debug: 
      msg: "ip adress {{user_client}}: {{ip}}" 
  - shell: id client1
    register: client_groups

  - debug:
      msg: "client in groups: {{client_groups}}"
  - debug:
      msg: "hostname: {{ansible_hostname}}"


#blocks 
- name: blocks
  hosts: all
  become: yes

  vars:

  tasks:

  - block:

    - name: Install Packages
      apt:
        pkg:
          - tree
        state: present
    - name: Create Folder
      file:
        path: /home/Folder1
        state: directory
    when: ansible_hostname == "aziz-VirtualBox"

  - name: Copy File
    copy:
      src: ./index.html
      dest: /var/www/html
      mode: 0777
    when: ansible_hostname == "client02"
    

#template
- name: Test blocks
  hosts: all
  become: yes

  vars:
    position: boss

  tasks:
    - name: Copy index.j2
      template:
        src: ./index.j2
        dest: /var/www/html/index.html
        mode: 0777


$nano file222

host: {{ansible_hostname}}
user: {{inv_user}}
position: {{position}}
address: {{ansible_default_ipv4.address}}

$nano hosts
[test]
client01 inv_user=client01
[group2]
client02 ansible_host=192.168.31.124 inv_user=client02
[all_groups:children]
group2


#roles

 ansible-galaxy init first_setup
 .
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

 
cd first_setup/
cd tasks/

nano main.yml

---
# tasks file for first_setup
  - name: 1. Task ping
    ping:

  - name: 2. Install Nginx
    apt:
      pkg: "{{packages}}"
      state: present

  - name: 3. Copy File
    template:
      src: "{{file_src}}"
      dest: "{{file_dest}}"
      mode: 0777
    with_items:
      - index.j2

cd ../

cd vars/

nano main.yml
---
# vars file for first_setup
    packages:
      - nginx
      - htop 
      - tree
      - rsync
    file_src: "{{ item }}" # этот файл д.б. в папке ansible
    file_dest: /var/www/html/index.html


cd ../
cp index.j2 first_setup/files/    


cd ansible

nano side.yml

- name: First setup
  hosts: client02
  gather_facts: yes
  roles:
          - first_setup

ansible-playbook -i hosts side.yml

