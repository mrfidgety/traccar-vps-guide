# Traccar VPS Guide

This guide will instruct you on how to install [Traccar](https://www.traccar.org/) on a new Amazon Lightsail instance. We will set up the required database directly on the instance. This will allow you to have the most cost-effective setup, since you are not paying for a separate (dedicated) database instance. However, if you intend to use this for commerical purposes I highly recommend using a dedicated database.

## How to install Traccar server on an Amazon Lightsail VPS

Open a terminal on your VPS instance. You can simply click 'connect' from the Lightsail browser interface to achieve this. Enter the following commands one at a time. Do not enter the lines starting with `#` as those are comments explaining what each command does.

#### Part One - Setting up the database
```
# "enable" postgres, our database engine
sudo amazon-linux-extras enable postgresql13

# install postgres, type "y" to continue
sudo yum install postgresql postgresql-server

# create the initial database
sudo postgresql-setup initdb

# start postgres
sudo systemctl start postgresql

# tell the system to start postgres when it boots
sudo systemctl enable postgresql

# set a password for the 'postgres' linux user
sudo passwd postgres

# change to the 'postgres' linux user
su - postgres

# enter the postgres terminal
psql

# set a password for the 'postgres' database user
ALTER USER postgres WITH PASSWORD 'thisisademo';

# create a database for traccar to store its data
CREATE DATABASE traccar;

# exit the postgres terminal
exit

# open the postgres config in vim
vi /var/lib/pgsql/data/pg_hba.conf

# scroll down to the 'host' section and then change 'ident' to 'password'
host    all all 127.0.0.1/32  password

# exit 'insert' mode
press Esc

# write and quit
:wq

# return to the default linux user
exit

# restart postgres
sudo systemctl restart postgresql
```