# Ansible でサーバメンテナンス

- 目的  
  Ansibleを使用したPC管理について手を動かしてみる
- 手段  
  EC2で複数のUbunutuサーバ(数個)を作成  
  下記を参考にAnsibleで apt-get update、apt-get upgradeを実施    
  http://blog.serverworks.co.jp/tech/2019/09/09/post-73961/

1. EC2インスタンス作成  
   省略  
1. Andibleのインストール  
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip  
ubuntu 23以前は pipxが使用できないので pipでインストールする  
sudo apt-get update  
sudo apt-get upgrade  
sudo apt install ansible-core  
( sudo apt install python3.12-venv  
( python3 -m venv myenv   
( source myenv/bin/activate  
( python3 -m pip install ansible  
ansible --version  
1.  クライアントインスタンスの秘密鍵をマスタインスタンスの.ssh/配下に配置  
  .ssh/mnr_key.pem  
1. マスタインスタンス内に作業ディレクトリを作成  
  mkdir -p hands-on-01/inventory  
  cd hands-on-01  
  vim inventory/hosts  
  ```text:hands-on-01/hosts
  [client_node]
Clinet01 ansible_ssh_host=57.180.44.225

[client_node:vars]
ansible_ssh_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/mnr_key.pem  
  ```  
  cd ~/hands-on-01  
  vim hands-on-01/site.yml  
  ```yaml:hands-on-01/site.yml
  - hosts: client_node
  become: True
  roles:
    - hands-on
  ```
  mkdir -p roles/hands-on/tasks  
  vim roles/hands-on/tasks/main.yml  
  ```yaml:hands-on-01/roles/hands-on/tasks/main.yml
#- name: create user
#  user:
#    name: hands-on-user01
#    createhome: yes
#    state: present
#    password: "{{ 'hands-on' | password_hash('sha512') }}"
#
#- name: modify sshd_config
#  lineinfile:
#    dest: /etc/ssh/sshd_config
#    state: present
#    backrefs: yes
#    regexp: '^PasswordAuthentication no'
#    line: 'PasswordAuthentication yes'
#    backup: yes
#  notify:
#    - restart sshd
- name: update apt cache
  command: apt-get update -y

- name: upgrade all packages
  command: apt-get upgrade -y
  ```
  mkdir -p roles/hands-on/handlers  
  vim roles/hands-on/handlers/main.yml  
  ```yaml:hands-on-01/roles/hands-on/handlers/main.yml
  - name: restart sshd
  service:
    name: sshd
    state: restarted
    enabled: yes
  ```
1. playbook を実行  
   ansible-playbook -i inventory/hosts site.yml  

