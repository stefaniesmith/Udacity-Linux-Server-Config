# Linux Configuration Project

For this project, a web application was setup and configured on a secure Linux server. The web application is a book catalog, which can be used for itemizing books by category. It was setup on an Amazon Lightsail instance of Ubuntu.

## Accessing the web application

The book catalog web application can be accessed here:
http://34.222.122.101.xip.io

Additional information on the book catalog can be found here:
https://github.com/svsmith/Udacity-Flask-Catalog-App

## Accessing the linux server

The linux server can be accessed over ssh (port 2200) as the grader user with a private key file (provided separately):
```sh
$ ssh grader@34.222.122.101 -p 2200 -i <private_key_file_path_and_name>
```

## Setup of linux server

Details on the linux server setup and configuration are summarized below.

### Updates to linux packages

Currently installed packages were updated using:
```sh
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```

### SSH configuration

The ssh port was changed to 2200 in the `sshd_config` file, and key-based authentication was enforced by ensuring that the `PasswordAuthentication` option was set to `no`. The ssh service was then restarted.
```sh
$ sudo vi /etc/ssh/sshd_config
$ sudo service ssh restart
```

### Uncomplicated firewall configuration

The Uncomplicated Firewall (UFW) was configured to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```sh
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow ntp
$ sudo ufw allow www
$ sudo ufw enable
```

### Configuration of grader user

A grader user was created and given sudo privileges (by copying an existing sudo users file, and modifying the file to contain the correct user name). 
```sh
$ sudo useradd grader
$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
$ sudo vi /etc/sudoers.d/grader
```

A public and private key pair was generated in a local linux environment, and the public key was added to the grader user's `authorized_keys` file.
```sh
$ sudo su grader
$ vi ~/.ssh/authorized_keys
```

### Configuration of apache

Apached was installed and the `000-default.conf` file was modified to include the following line:  
```WSGIScriptAlias / /var/www/catalog/catalog.wsgi```

```sh
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo vi /etc/apache2/sites-enabled/000-default.conf
$ sudo apache2ctl restart
```
