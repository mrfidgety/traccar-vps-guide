# Traccar VPS Guide

This guide will instruct you on how to install [Traccar](https://www.traccar.org/) on a new Amazon Lightsail instance. We will set up the required database directly on the instance. This will allow you to have the most cost-effective setup, since you are not paying for a separate (dedicated) database instance. However, if you intend to use this for commerical purposes I highly recommend using a dedicated database.

## How to install Traccar server on an Amazon Lightsail VPS

You will need:
- An Amazon Web Services (AWS) account. Be aware that this requires your name/address/mobile/credit-card details, as
  you are creating a publicly available webserver that will cost a small monthly fee.
- A Traccar account. The signup is not obvious on their home page, but is available at https://www.traccar.org/register/


#### Part One - Creating a VPS instance

When logged in to AWS, navigate to the 'Lightsail' service by using the search on the top left.
From there you should be able to create a new instance.

When creating an instance select
- the Linux/Unix platform
- the 'OS Only' Amazon Linux 2 blueprint

Scroll down and choose the lowest cost instance plan.
I recommend renaming your instance to something like "TraccarServer".
Click 'Create instance'.

Give it a minute or two for your web server to come to life.

#### Part Two - Setting up the database

Open a terminal on your VPS instance. There will be a button in the Lightsail interface that allows you to do this.
It will be labelled something like 'connect' or 'terminal'.
Enter the following commands one at a time. Press enter after pasting a command.
Do not enter the lines starting with `#` as those are comments explaining what the command below does.
You might have trouble using keyboard shortcuts (Ctrl+V) to paste, but right-clicking and selecting 'paste' should work.

```
# Enable postgres. Postgres is our database engine.
sudo amazon-linux-extras enable postgresql13

# Install postgres. You will be prompted to type "y" to continue.
sudo yum install postgresql postgresql-server

# Run progres setup.
sudo postgresql-setup initdb

# Start postgres.
sudo systemctl start postgresql

# Tell the server to start postgres when it boots.
sudo systemctl enable postgresql

# Set a password for the 'postgres' linux user.
# When entering the password it will be hidden, and look like you havent typed anything.
# Press enter after you have typed your password, and you will be prompted to type it again to confirm.
sudo passwd postgres

# Change the current user to the 'postgres' linux user.
# You will need to enter the password you just set in the previous command.
su - postgres

# Enter the postgres terminal.
# This is how we 'talk' to the database.
psql

# Set a password for the 'postgres' database user (yes, there is a linux user and a database user with the same name).
# You should change yourpasswordhere in the command below to your own password.
# It is okay to use the same password as the one you set earlier for the postgres linux user.
# This is a password you need to remember for later on.
ALTER USER postgres WITH PASSWORD 'yourpasswordhere';

# Create a database for traccar to store its data.
CREATE DATABASE traccar;

# Exit the postgres terminal.
exit

# Open the postgres config in vim.
# Vim can be scary, but the main things to remember are:
# - you cannot use your mouse to select or click, everything is done via the keyboard
# - when in vim you will either be in 'command' or 'insert' mode
# - you can only type new things in insert mode
# - you start off in command mode
# - press 'i' to enter insert mode
# - press 'esc' to return to command mode
vi /var/lib/pgsql/data/pg_hba.conf

# Using the down arrow, scroll past the comments.
# Find the first line that begins with 'host'.
# Press 'i' to enter insert mode.
# Use the right arrow to move to the right, then delete the word 'ident' and type 'password'.
# The line should end up looking like:
host    all all 127.0.0.1/32  password

# Exit insert mode by pressing the esc key.

# Save the changes by typing the command below. This writes the changes (w) and quits (q) vim.
# There is no need to move your cursor since we are in command mode.
:wq

# Return to the default linux user.
exit

# Restart postgres.
sudo systemctl restart postgresql
```

#### Part Three - Installing the Traccar server
```
# Download the traccar zip file to your server.
sudo wget https://www.traccar.org/download/traccar-linux-64-latest.zip

# Unzip the traccar installer file.
sudo unzip traccar-linux-*.zip

# Run the traccar installer.
sudo ./traccar.run

# Open the traccar config in vim.
# Remember the instructions above about navigating in vim.
sudo vi /opt/traccar/conf/traccar.xml

# Replace the <entry> lines with the following.
# Replace yourpasswordhere in the command below with the postgres database password you set earlier.
# Replace yournotificationkeyhere in the command below with the 'Push notifications API key' found on your traccar.org account.
# You can delete an entire line under your text cursor while in command mode by pressing 'dd'.
# Once all entry lines have been deleted, press 'i' to enter insert mode.
# You can paste this into vim while in insert mode by right clicking and selecting 'paste'.
# If you think you have made a mistake enter command mode and type :q! for forcibly quit without making changes.
# You can then re-open the unmodified file to try again.

<entry key='database.driver'>org.postgresql.Driver</entry>
<entry key='database.url'>jdbc:postgresql://localhost/traccar</entry>
<entry key='database.user'>postgres</entry>
<entry key='database.password'>yourpasswordhere</entry>
<entry key='notificator.types'>traccar</entry>
<entry key='notificator.traccar.key'>yournotificationkeyhere</entry>

# Exit 'insert' mode by pressing the Esc key

# Save the changes by typing the command below. This writes the changes (w) and quits (q) vim.
# There is no need to move your cursor since we are in 'command' mode.
:wq

# Start the traccar service.
sudo service traccar start
```

#### Part Four - Accessing your Traccar web interface

Traccar is now installed and running on your VPS.
There are is one last thing to do before we can access it in the browser.

The Traccar interface is accessible on port 8082. See more details about this in the [Traccar quick-start-guide](https://www.traccar.org/quick-start/)
By default, your VPS will not permit connections on that port, so we need to enable it.

1. Click on your VPS instance (specifically the name) in Lightsail to open its configuration page.
2. Visit the 'Networking tab'.
3. Scroll to the 'IPv4 Firewall' section and click 'Add Rule'.
4. Leave the Application as 'Custom' and the Protocol as 'TCP', then enter 8082 in the port.
5. Click 'Create'.

Now you can visit your server!

Your server IP address is listed in a few places:
- on both tile on the Lightsail home page
- on the 'Connect' page under the heading 'Connect To'
- on the 'Networking' page under the heading 'Public IP'

Copy the server IP address and paste it into your browser URL.
Type :8082 at the end of the IP address and hit enter.
Your full address should look something like 12.34.56.789:8082

Your browser might complain that the site is not secure.
You can happily tell your browser to ignore/continue as this is YOUR site.

The default login will be username: admin, password: admin
The first thing you should do is change this account to have a username that is yours and password that is secure.