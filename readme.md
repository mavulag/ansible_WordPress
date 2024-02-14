How To Install, Setup, Deploy and Configure WordPress Website on AWS EC2 Ubuntu Linux Server Using Ansible

To begin with, install ansible in your control node
```
$ sudo apt install ansible -y
```

Then create a directory
```
mkdir -p ansible-demo
```
```
cd ansible-demo/
```
In that directory you are going to create all required files

```
touch hosts
```
```
nano hosts
```
Write the ip address of the machine that you are going to install WordPress in it(managed node)
```
[webapp]
192.168.X.X

[database]
```
Save and exit

Then check the public key of the machine
```
cat ~/.ssh/id_rsa.pub
```
copy them, then go to your managed node (the one you want to install WordPress)
```
nano ~/.ssh/authorized_keys
```
paste them (public keys from the control node) save and exit

Go back to the control node then try to ssh to that managed node (the one you want to install WordPress)
```
ssh nameOfManagedNode@192.168.X.X
```
then 
```
exit
```
Then ping the host to make sure ansible is able to reache to the managed node
```
ansible -i hosts all -m ping -u nameOfManagedNode
```

In addition, make another directory and file in ansible-demo directory
```
mkdir -p roles
```
```
touch playbook.yml
```
```
cd roles
```

Then install the setting roles
```
ansible-galaxy init server
```
```
ansible-galaxy init php
```
```
ansible-galaxy init mysql
```
```
ansible-galaxy init wordpress
```

after that
```
cd ..
```
```
nano playbook.yml
```
Copy and paste these
```
---
 - hosts all
   gather_facts: false
   bacome: yes

 - hosts: all
   roles:
     - server
     - php
     - mysql
     - wordpress
```
save and exit

Then
```
cd roles/server/tasks/
```
```
nano main.yml
```
copy and paste these
```
---
# tasks file for server
 - name: Update apt cache
   apt: update_cache=yes cache_valid_time=3600
   become: yes

 - name: Install required software
   apt: name={{ item }} state=present
   become: yes
   loop:
     - apache2
     - mysql-server
     - php-mysql
     - php
     - libapache2mod-php
     - python3-mysqldb
```
save and exit

Then 
```
cd ../..
```
```
cd php/tasks/
```
```
nano main.yml
```
copy and paste these
```
---
# tasks file for php
 - name: Install php extensions
   apt: name={{ item }} state=present
   become: yes
   with_items:
    - php-gd
    - php-ssh2
```
save and exit

Then 
```
cd ../..
```
```
cd mysql/defaults/
```
```
nano main.yml
```
copy and paste these
```
---
# defaults file for mysql
 wp_mysql_db: wordpress
 wp_mysql_user: wordpress
 wp_mysql_password: yourpassword
```
save and exit
```
cd ..
```
```
cd tasks/
```
```
nano main.yml
```
copy and paste these
```
---
# tasks file for mysql
 - name: Create mysql database
   mysql_db: name={{ wp_mysql }} state=present
   become: yes

 - name: Create mysql user
   mysql_user: 
    name={{ wp_mysql_user }}
    password={{ wp_mysql_password }}
    priv=*.*:ALL

   become: yes
```
save and exit
 
Then 
```
cd ../..
```
```
cd wordpress/tasks/
```
```
nano main.yml
```
copy and paste these
```
---
# tasks file for wordpress
 - name: Download WordPress
   get_url:
    url: https//wordpress.org/latest.tar.gz
    dest: /tmp/wordpress.tar.gz
    validate_certs: no

 - name: Extract WordPress
   unarchive:
    src: /tmp/wordpress.tar.gz
    dest: /var/www/
    copy: no
   become: yes

 - name Update default Apache site
   become: yes
   lineinfile:
    dest: /etc/apache2/sites-enabled/000-default.conf
    regexp: "DocumentRoot /var/www/html"
    line: "DocumentRoot /var/www/wordpress"
   notify: restart apache

 - name: Copy sample config file
   command: mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
   args:
     creates: /var/www/wordpress/wp-config.php
   become: yes

 - name: Update WordPress config file
   lineinfile:
    dest: /var/www/wordpress/wp-config.php
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
   loop:
    - {'regexp': "define\\('DB_NAME', '(.)+'\\);", 'line': "define('DB_NAME', '{{ wp_mysql_db}}');"}
    - {'regexp': "define\\('DB_USER', '(.)+'\\);", 'line': "define('DB_USER', '{{ wp_mysql_user}}');"} 
    - {'regexp': "define\\('DB_PASSWORD', '(.)+'\\);", 'line': "define('DB_PASSWORD', '{{ wp_mysql_password}}');"} 
   become: yes
````
save and exit

Then 
```
cd ..
```
```
cd handlers
```
```
nano main.yml
```
copy and paste these
```
---
# handlers file for wordpress
 - name: restart apache
   service: name=apache2 state=restarted
   become: yes
```
save and exit

Then 
```
cd ../../..
```
In the ansible-demo directory type
```
ansible-playbook playbook.yml -i hosts -u nameOfManagedNode
```

In conclusion, go to the browser and type
```
192.168.X.X
```
or
```
192.168.X.X/wp-admin/install.php
```
continue with the wordpress installation

Cheers!!