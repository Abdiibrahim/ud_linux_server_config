# Udacity - Linux Server Configuration Project

Udacity Full Stack Web Developer Nanodegree

## Notes for Instructor

## About

For this project, I have set up an Ubuntu 18.04 image on a DigitalOcean droplet.

### Technical Information About the Project

- **Server IP Address:** 206.189.113.158
- **SSH server access port:** 2200
- **SSH login username:** grader
- **password:** grader
- **public IP:** 206.189.113.158
- **Application URL:** http://206.189.113.158.xip.io

## Tasks

### 1. Create the RSA Key Pair

On your local machine, set up the public and private key pair.
To generate a key pair:

   ```console
   $ ssh-keygen
   ```

When it asks to enter a passphrase, you may either leave it empty or enter some passphrase. A passphrase adds an additional layer of security to prevent unauthorized users from logging in.

The whole process would look like this:

You now have a public and private key that you can use to authenticate.

### 2. Setup DigitalOcean Droplet

1. Log in or create an account on [DigtalOcean](https://cloud.digitalocean.com/login).

2. Go to the Dashboard, and click "Create Droplet".

3. In the section *Add Your SSH Keys*, paste the content of your public key

   This step will automatically create the file `~/.ssh/authorized_keys` with appropriate permissions and add your public key to it. It also adds the following rule in the `/etc/ssh/sshd_config` file:

   ```
   PasswordAuthentication no
   ```

4. Click *Create* to create the droplet. This will take some time to complete. After the droplet has been created successfully, a public IP address will be assigned. In this project, the public IPv4 address that I have been assigned is `206.189.151.124`.

5. Logging In as `root` via SSH

 Now log in to the server as `root` user:

 ```
    $ ssh root@206.189.113.158
 ```

 Run the following command to update the virtual server:

```
 # apt update
 # apt upgrade
```

### 4. Changing the SSH Port from 22 to 2200

1. Open the `/etc/ssh/sshd_config` file

   ```
   # nano /etc/ssh/sshd_config
   ```

2. Find the line `#Port 22` and change it to `Port 2200`

3. Restart the SSH server:
   ```
   # service ssh restart
   ```

4. Confirm the changes:
   ```
   # exit
   ```

   Run in the local machine:

   ```
   $ ssh root@206.189.113.158 -p 2200
   ```

### 5. Configure Timezone to Use UTC

To configure the timezone to use UTC, run the following command:

```
# sudo dpkg-reconfigure tzdata
```
Choose ``None of the Above`` and ``UTC``.

You should now see an output like this:

### 6. Setup the Firewall

Configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
# ufw enable
```

Confirm new rules:
```
# ufw status
```

### 7. Create the User `grader` and add grant `sudo`

1. As root, run:

   ```
   # adduser grader
   # usermod -aG sudo grader
   # su - grader
```

Enter the following to allow SSH access to `grader`:

```
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
```

Copy the content of the public key file and paste the public key to the virtual server's `authorized_keys`

```
$ nano authorized_keys
```

Exit and log in as `grader` via ssh

### 9. Disabling Root Login

1. Log in as `root` from your local machine:
   ```
   $ ssh root@206.189.113.158 -p 2200
   ```

2. After you are logged in, open the file `/etc/ssh/sshd_config` and change `PermitRootLogin yes` to `PermitRootLogin no`:
   ```
   # nano /etc/ssh/sshd_config
   ```

3. Restart the SSH server and end connection:
   ```
   # service ssh restart
   # exit
   ```

### 10. Installing Apache Web Server

To install the Apache Web Server, run the following command after logging in as `grader` user via SSH:

```
$ sudo apt update
$ sudo apt install apache2
```

To confirm whether it successfully installed or not, enter the URL `http://206.189.113.158` in your Web browser:

### 11. Installing Python 3.6 and pip3

1. Install Python 3.6 and `pip3`:

   ```
   $ sudo apt install python3
   $ sudo apt install python3-pip
   ```

### 12. Install and Configure Git

Install `git`:
```
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt update
$ sudo apt install git
```

Configure a username and an email:
```
$ git config --global user.name "Abdi Ibrahim"
$ git config --global user.email "contact@abdiibrahim.com"
```

### 13. Install Required `pip` Packages

Install all the required packages:

```
pip3 install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
```

### 14. Setup Apache to Run the Flask Application

#### 14.1. Install `mod_wsgi`
 
```
$ sudo apt install libapache2-mod-wsgi-py3
$ sudo a2enmod wsgi
$ sudo service apache2 restart
```

#### 14.2. Clone the Item Catalog Flask application

1. Change directory to `/var/www/`:

   ```
   $ cd /var/www/
   ```

2. Create a directory called `FlaskApp` and change directory to it:

   ```
   $ sudo mkdir FlaskApp
   $ cd FlaskApp/
   ```

3. Clone the repo as the directory `FlaskApp`:

   ```
   $ sudo [repo] FlaskApp
   ```

4. Move inside the newly created directory:

   ```
   $ cd FlaskApp/
   ```
   The directory tree should now look like this:

   ```
   FlaskApp
    └── FlaskApp
        ├── LICENSE
        ├── README.md
        ├── __init__.py
        ├── client_secrets.json
        ├── database_setup.py
        ├── fake_db_populator.py
        ├── static
        │   └── style.css
        └── templates
            ├── delete.html
            ├── delete_category.html
            ├── edit_category.html
            ├── index.html
            ├── items.html
            ├── layout.html
            ├── login.html
            ├── new-category.html
            ├── new-item-2.html
            ├── new-item.html
            ├── update-item.html
            └── view-item.html
    ```

#### 14.3. Setting Up Virtual Hosts

1. Run the following command in terminal to set up a file called `FlaskApp.conf` to configure the virtual hosts:

   ```
   $ sudo nano /etc/apache2/sites-available/FlaskApp.conf
   ```

2. Add the following lines to it:

   ```

   <VirtualHost *:80>
      ServerName 206.189.113.158
      ServerAdmin contact@abdiibrahim.com
      WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
      <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny
        Allow from all
      </Directory>
      Alias /static /var/www/FlaskApp/FlaskApp/static
      <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   ```

3. Enable the virtual host and disable default:

   ```
   $ sudo a2ensite FlaskApp
   $ sudo a2dissite 000-default.conf
   ```

4. Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```

5. Creating the .wsgi File

   Apache uses the `.wsgi` file to serve the Flask app. Move to the `/var/www/FlaskApp/` directory and create a file named `flaskapp.wsgi` with following commands:

   ```
   $ cd /var/www/FlaskApp/
   $ sudo nano flaskapp.wsgi
   ```

   Add the following lines to the `flaskapp.wsgi` file:

   ```python
   #!/usr/bin/python3
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/FlaskApp/")

   from FlaskApp import app as application
   application.secret_key = 'super_secret_key'
   ```

### 15. Install and Configure PostgreSQL

#### 15.1. Installing PostgreSQL

1. Create the file `/etc/apt/sources.list.d/pgdg.list` and add a line for the repository:

   ```
   $ deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
   ```

2. Import the repository signing key, and update the package lists

   ```
   $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   $ sudo apt update
   ```

3. Install PostgreSQL:

   ```
   $ sudo apt install postgresql-10
   ```

#### 15.2. Configuring PostgreSQL

1. Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:

   ```
   $ sudo su - postgres
   ```

2. Open the `psql` shell:

   ```
   $ psql
   ```

3. This will open the `psql` shell. Now type the following commands one-by-one:

   ```sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```

   Then exit from the terminal by running `\q` followed by `exit`.

### 16. Install virtualenv and run the database files


## References

[3] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>