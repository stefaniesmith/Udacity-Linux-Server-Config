# Linux Configuration Project

For this project, a web application was setup and configured on a secure Linux server. The web application is a book catalog, which can be used for itemizing books by category. It was setup on an Amazon Lightsail instance of Ubuntu.

## Accessing the web application

The book catalog web application can be accessed here:  
http://34.222.122.101.xip.io

**Note: The web application is no longer available for access.**

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
$ sudo apt-get --with-new-pkgs upgrade
$ sudo apt-get autoremove
```

### SSH configuration

The ssh port was changed to 2200 in the `sshd_config` file, `PermitRootLogin` was set to `no`, and key-based authentication was enforced by ensuring that the `PasswordAuthentication` option was set to `no`. The ssh service was then restarted.
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

A grader user was created and given sudo privileges (by copying an existing sudo user's file, and modifying the file to contain the correct user name). 
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

### Apache configuration

Apache was installed and the `000-default.conf` file was modified to include the following line:  
```WSGIScriptAlias / /var/www/catalog/catalog.wsgi```

```sh
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo vi /etc/apache2/sites-enabled/000-default.conf
```

### Setup and deploy of web application

PostgreSQL, git and required python packages were installed.
```sh
$ sudo apt-get install postgresql postgresql-contrib git-core python-pip
$ sudo pip install sqlalchemy flask oauth2client requests itsdangerous click jinja2 
$ sudo pip install werkzeug cli json psycopg2 six httplib2 urllib3 chardet certifi idna
```

The web application was downloaded from GitHub and copied to the `/var/www/catalog/catalog` folder.
```sh
$ cd ~
$ git clone https://github.com/svsmith/Udacity-Flask-Catalog-App.git
$ sudo cp -rf ~/Udacity-Flask-Catalog-App/* /var/www/catalog/catalog
```

An empty `__init__.py` file was created in `/var/www/catalog/catalog/`.
```sh
$ sudo touch /var/www/catalog/catalog/__init__.py
```

The file `catalog.wsgi` was created in `/var/www/catalog`, and contains:  
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog")

from catalog.app import app as application
application.secret_key = 'super_secret_key'
```

The web application was switched over to PostgreSQL, by replacing all occurrences of `sqlite:///catalog.db` with `postgresql://catalog:catalog@localhost/catalog` in:
- `database_setup.py`
- `add_books.py`
- `app.py` 

The OAuth credentials file, `client_secrets.json`, was created as detailed here:  
https://github.com/svsmith/Udacity-Flask-Catalog-App/blob/master/README.md#obtaining-oauth-credentials-from-google  
using `34.222.122.101.xip.io` instead of `localhost:5000` for the JavaScript origins and redirect URIs. The json file was then added to the `/var/www/catalog/catalog` folder. 

The location of `client_secrets.json` was updated in `app.py` to include the full path instead of the relative path:  
`/var/www/catalog/catalog/client_secrets.json`

A `catalog` database user was created for PostgreSQL:
```sh
$ sudo su - postgres
$ createuser --pwprompt catalog
$ createdb -O catalog catalog
```

The `catalog` database was setup and some sample data was added to the database:
```sh
$ cd /var/www/catalog/catalog
$ python database_setup.py
$ python add_books.py
```

Finally, the Apache service was restarted to apply the changes made:
```sh
$ sudo apache2ctl restart
```

## References

Sources of information used to complete this project:
- Udacity Knowledge Forum: https://knowledge.udacity.com
- A2 Hosting Knowledge Base: https://www.a2hosting.com/kb/developer-corner/postgresql/managing-postgresql-databases-and-users-from-the-command-line

