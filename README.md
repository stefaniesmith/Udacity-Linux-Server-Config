# Linux Configuration Project

For this project, a web application was setup and configured on a secure Linux server. The web application is a book catalog, which can be used for tracking books by category. It was setup on an Amazon Lightsail instance of Ubuntu.

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

### Update packages

Currently installed packages were updated using:

```sh
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```
### SSH configuration
