# Full-Stack-Course Linux Configuration Project

This is to help other udacity students to complete the last project 
of full stack course to deployment the catalog project on the server

## Step 1
### Setting Up Amazon Lightsail:
1- Go to the Amazon website using this page [Amazon Lightsail](https://lightsail.aws.amazon.com)

2- Log in! to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.

3- *sign up* then enter your method payment info.

4- Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. 
   A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.  

5- Choose an instance image: Ubuntu
   Lightsail supports a lot of different instance types. 
   An instance image is a particular software setup, including an operating system and optionally built-in applications.
   For this project, you'll want a plain Ubuntu Linux image. 
   There are two settings to make here. 
   First, choose "OS Only" (rather than "Apps + OS"). 
   Second, choose Ubuntu as the operating system.

6- Choose your instance plan.
   The instance plan controls how powerful of a server you get. 
   It also controls how much money they want to charge you. 
   For this project, the lowest tier of instance is just fine. 
   And as long as you complete the project within a month and shut your instance down, the price will be zero.

7- Give your instance a hostname.
   Every instance needs a unique hostname. 
   You can use any name you like, as long as it doesn't have spaces or unusual characters in it. 
   Your instance's name will be visible to you and to the project reviewer.

8- Wait for it to start up.
   It may take a few minutes for your instance to start up.
  
9- It's running; let's use it!
   Once running click on it and click "Account Page" at the bottom so you can download your private SSH key.
   You can log into it with SSH from your browser.
   The public IP address of the instance is displayed along with its name. In the above picture it's  like 54.84.49.254.

10- Now click download to get your private key. The file type is .pem and will be used to SSH into the server.

11- The last thing we will need to do is configure the ports Amazon Lightsail will allow. 
    By default the firewall is set to only allow connects from port 22 and port 80. 
    We need to set up port 2200 and 123.

12- Click the networking tab

13- From this tab click add another under "Firewall" and choose **Custom** for application, **TCP** for protocol, and the port number under **Port Range**. 
    Then click save.  

14- That should be all that needs to be done with the Lightsail website.


## Step 2
### SSH into the server:
1- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.pem`.

2- To make our key secure type $ chmod 600 ~/.ssh/lightsail_key.pem into the terminal.

3- From here we will log into the server as the user ubuntu with our key. 
   From the terminal type $ ssh -i ~/.ssh/lightsail_key.pem ubuntu@54.93.214.14,where `54.93.214.14` is the public IP address of the instance.
4- You will log into the server as the user `ubuntu@54.93.214.14`, switch to the root user by typing `sudo su -`

5- As Udacity requires we need to create a user called grader. 
   From the command line type $ sudo adduser grader. 
   It will ask for 2 passwords and then a few other fields which you can leave blank
   I enter a password (123456) and fill out information for this new user.

6- We must create a file to give the user grader superuser privileges. 
   To do this type $ sudo nano /etc/sudoers.d/grader. 
   This will create a new file that will be the superuser configuration for grader. 
   When nano opens type grader ALL=(ALL:ALL) ALL, to save the file hit Ctrl-X on your keyboard, type 'Y' to save, and press enter save the file.

7- One of the first things you should always do when configuring a Linux server is updating it's package list, upgrading the current packages, and install new updates with these three commands:

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
```

8- We will also install a useful tool called **Finge**r with the command $ sudo apt-get install finger. This tool will allow us to see the users on this server

9- Now we must create an SSH Key for our new user **grade**r. From a new terminal run the command: $ ssh-keygen -f ~/.ssh/udacity.rsa

10- In the same terminal we need to read and copy the public key using the command: $ cat ~/.ssh/udacity.rsa.pub. Copy the key from the terminal.

11- Back in the server terminal locate the folder for the user **grade**r, it should be /home/grader. Run the command $ cd /home/grader to move to the folder.

12- Create a directory called .ssh with the command $ mkdir .ssh

13- Create a file to store the public key with the command $ touch .ssh/authorized_keys

14- Edit that file using $ nano .ssh/authorized_keys

15- Now paste in the public key

16- We must change the permissions of the file and its folder by running

    $ sudo chmod 700 /home/grader/.ssh
    $ sudo chmod 644 /home/grader/.ssh/authorized_keys 

17- Change the owner of the .ssh directory from root to grader by using the command $ sudo chown -R grader:grader /home/grader/.ssh

18- The last thing we need to do for the SSH configuration is restart its service with $ sudo service ssh restart


## Step 3
### Changing the SSH port from 22 to 2200
**on SSH Server**
1- Now we need to login with the grader account using ssh. From your local terminal type $ ssh -i ~/.ssh/udacity.rsa grader@54.93.214.14

2- you should now be logged into your server via SSH

3- Lets enforce key authentication from the ssh configuration file by editing $ sudo nano /etc/ssh/sshd_config. 
   Find the line that says PasswordAuthentication and change it to no. Also find the line that says Port 22 and change it to Port 2200. 
   Lastly change PermitRootLogin to no.
   Save and exit using CTRL+X and confirm with Y.

4- Restart ssh again: $ sudo service ssh restart

5- Disconnect from the server and try step "1-" again BUT also adding -p 2200 at the end this time. You should be connected.
   run: `ssh -i ~/.ssh/udacity.rsa -p 2200 grader@54.93.214.14`, where `54.93.214.14` is the public IP address of the instance.


## Step 4
### Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.


# Application Deployment


Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
------------------

## Step 5
### Configure the local timezone to UTC

- While logged in as grader, Check the timezone with the `date` command. This will display the current timezone after the time.
  ```
  Wed Jan  9 22:11:15 UTC 2019
  ```
- If it's not UTC change it with this command:
  `sudo timedatectl set-timezone UTC`


## Step 6
While logged in as grader

1- Start by installing the required software
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
```
2- Enable mod_wsgi with the command $ sudo a2enmod wsgi and restart Apache using $ sudo service apache2 restart

3- If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page

4- We now have to create a directory for our catalog application and make the user **grader** the owner.
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
```

5- In this directory we will have our catalog.wsgi file var/www/catalog/catalog.wsgi, our virtual environment directory which we will create soon and call venv /var/www/catalog/venv, and also our application which will sit inside of another directory called catalog /var/www/catalog/catalog. 

6- First lets start by cloning our Catalog Application repository by $ git clone [repository url] catalog

7- Create the .wsgi file by $ sudo nano catalog.wsgi and make sure your secret key matches with your project secret key

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

8- Rename your application.py, project.py, or whatever you called it in your catalog application folder to __init__.py by $ mv project.py __init__.py

9- Now lets create our virtual environment, make sure you are in /var/www/catalog.

```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
This is what your command line should look like enter image description here **(venv) grader@54.93.214.14:/var/www/catalog**

10- While our virtual environment is activated we need to install all packages required for our Flask application. Here are some defaults but you may have more to install.

```
 $ sudo apt-get install python-pip
 $ sudo pip install flask
 $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2
 $ sudo pip install requests
 $ sudo pip install --upgrade oauth2client
 $ sudo apt-get install libpq-dev
 $ sudo pip install sqlalchemy_utils
```

11- Now for our application to properly run we must do some change to the __init__.py file.

12- Anywhere in the file where Python tries to open client_secrets.json or fb_client_secrets.json must be changed to its complete path ex: /var/www/catalog/catalog/client_secrets.json 
    actualy there is two place in my project i just use google i did not use facebook

13- Time to configure and enable our virtual host to run the site
    ```$ sudo nano /etc/apache2/sites-available/catalog.conf```

      Paste in the following:

```
<VirtualHost *:80>
    ServerName [Public IP]
    ServerAlias [Hostname]
    ServerAdmin admin@54.93.214.14
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
If you need help finding your servers hostname go [find hostname](https://whatismyipaddress.com/ip-hostname) and paste the IP address. Save and quit nano

14- Enable to virtual host: $ sudo a2ensite catalog.conf and DISABLE the default host $ a2dissite 000-default.conf otherwise your site will not load with the hostname.

15- Finally - Deactivate the virtual environment: `deactivate`

## Step 7:
### The final step is setting up the database Install and configure PostgreSQL

1- install the following 

```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
```

Create a database user and password

2- The final step is setting up the database

```
postgres=# CREATE USER catalog WITH PASSWORD '123456';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
Your command line should now be back to grader.

3- Now use nano again to edit your __init__.py, database_setup.py, and createitems.py files to change the database engine from sqlite://catalog.db to postgresql://catalog:123456@localhost/catalog

4- Restart your apache server $ sudo service apache2 restart and now your IP address and hostname should both load your application.


## Step 8
### Authenticate login through Google
  - Go to [Google Cloud Plateform](https://console.cloud.google.com/).
  - Click `APIs & services` on left menu.
  - Click `Credentials`.
  - Create an OAuth Client ID (under the Credentials tab), and add:
    - http://54.93.214.14	
    - http://ec2-54-93-214-14.eu-central-1.compute.amazonaws.com / as authorized JavaScript origins.
    - Add http://ec2-54-93-214-14.eu-central-1.compute.amazonaws.com/oauth2callback as authorized redirect URI.
  - Download the corresponding JSON file, open it and copy the contents.
  - go to catalog application file using the command `cd /var/www/catalog/catalog` edit client_secret.json using the command `$sudo nano client_secret.json` and paste the previous contents into the this file.
  - Replace the client ID to line 25 of the `templates/login.html` file in the project directory.


## Do not forget to do the following 

- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.
- In `__init__.py`, change the following:
  ```
  //catalog.db")
   engine = create_engine('postgrengine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:123456@localhost/catalog')
  ```
  Update path of client_secrets.json file:

  ```
  '/var/www/catalog/catalog/client_secrets.json'
  ```
  and:
  ```
  app.run(host="0.0.0.0", port=5000, debug=True)
  TO
  app.run()
  ```

- In database_setup.py, and lotsofperfumes.py change the following by using the command`$ sudo nano database_setup.py`, `$ sudo nano lotsofperfumes.py` :
   ```
   engine = create_engine("sqlite:/esql://catalog:123456@localhost/catalog')
   ```
- Run the files by using python:
    ```
    python database_setup.py
    python lotsofperfumes.py
    ```


## References
- Udacity course: [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299-nd)
- GitHub Repositories
  - https://github.com/boisalai/udacity-linux-server-configuration
  - https://github.com/mulligan121/Udacity-Linux-Configuration
  - https://githu

## NOTES 
### This the RSA Private Key

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,0B36FF47DF03E1185998F6BF3E9DAEFF

AtWWpOiuyXOcc+Gd/wXTeG4bww8b7b8hIeyEKpntVb2htN81FxBQ/s8TB5f3tPkA
vsI++LwneCB/fbbJoMgVuSP2wc7Ad4MKXX87hEWm5B3/L+myLtPbNk9T1kq7M0Pb
nO6CNujMJ0UtV3WYOaop6DwY2btPwIIc5QvOa4tVEnXVhBQS7v8Bi0Hs8TntFVRH
hOdurjtO83AozKoQYzS0oc55Ayvt2wv3xzXf3YANkDZf7aVRtwMvcWnL13rdWpu0
ckxlrLyp38cm5Wf6m2Ixus62PdxAjbYQ6MJ6cQMzN9NFWqHHYCkS5y1CPTU5LjHu
5uX8kmh4rgj7xsMo1S0sdN+i42Vv9leoKhY4Zjo1dWXU3USi9bEiWQcJyMH/KSkR
NyMS0udkQrOqCak18GiEhnqytKRFxqqxNAOpIz3UIGqJbEWlAQE9rRZ/EowVZByY
DrWEAaajH0GDMzB8t7MJoi3F1h09YCumN8qz+wsAS6L0lwNmq5lNCcg4TBvAmCah
4MH53CxRvlNanqj+z1IK1U34zpOinocZ7aefIAsxpePSzUio2P+Lv+wLc1OgD6hG
gvu3E5MI9iMnAUA/HGnwXC+ybTqH48QIQ0cqTu2UZGL5Dzy2i+Hn8oxnfKIldIf2
hwD3SioA9KyKamabwSQNEmyq21kJtyeyBrctZg7E/QNMYAoCRRStMGtoIctDHkmp
JuOEpuBI5WutHq2xfLghEI6DeTNIQ3ZZ+Fx+eTsv5f3pJqKMbdKVDJvYdzNdrRc/
dHMhmkMlawv8DAbluqRQUdJplZw6mjTDNyVi2cHf/hU1PfPnqSzgv5z/zhEHWCwF
ygkD1WrFgFYlVYekduwrez/ETFlLS8ovATh3LlbHeeiN3CLzZbll6IDZ4tQ4aCd/
9jqwWTa8dXNXEf771V9eR6BBmnMFpwAy0hOAfHYQjp9pOFqkf1H2G1GSM4omx6wf
3k8CR1WyvILhXPM9JtGWMkSEnEjkBYxV48JKwR8wkOk1ON+eSspzu49lRIQYPy0Y
E0MeLsxJnxU7toEshsH0zJ7CBYxMWEU3KgDXiDCt3Kd18HBqri2U9PppQf60hK0x
WERMdoKbfwuysNq27cc6/V6zG04qgkB9abaq8HK7V3hrsaFvLTiaPQImStD61WQ3
S1WhDZzHb7p6TZONJ/BG5oJiJcUrVc3IG/u1PR4RX9WIEfxPXoKAozw2WTzcWcoq
1YHu1eT6QT2sHpe6M1jl632VFVEA/EZtNrSCtEwovOSZz9MOsdHfstKA44b5qdbk
JZxAAW8vy39qexMmfY37+oyfeAYriXh0mIOrG8tuJfwxnQ53VMPzCVoxQTAzOm60
sKMOSbD7jOz7zSE0OSQRenLlKL9qZF47CeWbLLuMkm7m5eJ9aggV2FrNf1CcR67g
1XENK6YLG26R8YlzpI6jtgjFkifYud/3p1fLsREmJjHBPbNyfHBb/n8G0MTO8u57
Ulmikp8lDjlbf0raOLf1KKPx9T6uHIxSee3ksnLV8Il+nNoP0TSXzpsb1WC7YBSQ
AbWRPOnHsaQ1wJUkjVHPcJu+0WVnHR6ohgnG/D4sizz29/2qF8ysNHJCQg2ezrfy
-----END RSA PRIVATE KEY-----

### This is the password of grader user 
123456
