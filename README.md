# Linux Server Configuration

---

## Introduction

This is Project 5 for Udacity's Full Stack Web Developer Nanodegree.

Main objectives of this project:

* Remotely access, secure and perform initial configuration of a bare-bones Linux server
* Install a web and database server to host a web application
* Properly secure the server and applications to ensure stability of system and safety of data

## Server Info

IP address: 54.200.104.8

SSH port: 2200

Web URL: [http://54.200.104.8/](http://54.200.104.8/)

## Software/Libraries Installed

These are the software/libraries that I have installed:

* git
* Apache
* PostgreSQL
* Flask
* SQLAlchemy
* PIP (for installing Python libraries)
* oauth2client
* psycopg2
* requests
* httplib2


## Steps

These are the steps that I have done to meet the requirements of this project:

1. **Create a new user named `grader`**

		root:~$ adduser grader

2. **Give the `grader` user permission to sudo**

		root:~$ echo "grader ALL=(ALL) ALL" > /etc/sudoers.d/grader

	Without "NOPASSWD" in the config, the user is prompted for password whenever sudo is used.

3. **Set up key-based authentication for `grader`**

		#did this on my local vagrant machine
		vagrant:~$ ssh-keygen

	A private key (.rsa) and a public key (.pub) were created.

	Then I got into the Udacity remote machine and copied contents of public key (.pub file) to `/home/grader/.ssh/authorized_keys`

		root:~$ sudo -i -u grader
		grader:~$ mkdir ~/.ssh
		grader:~$ vim ~/.ssh/authorized_keys
		(copied contents of .pub file into this file)
		grader:~$ chmod 700 ~/.ssh
		grader:~$ chmod 644 ~/.ssh/authorized_keys

4. **Disable remote SSH login as `root`**

		#set this in /etc/ssh/sshd_config
		PermitRootLogin no

5. **Disable password-based authentication for SSH**

		#set this in /etc/ssh/sshd_config
		PasswordAuthentication no

6. **Change the SSH port from 22 to 2200**

		#set this in /etc/ssh/sshd_config
		Port 2200

	With all the SSH config settings done, I restarted the SSH service:

		grader:~$ sudo service ssh restart

	
	I then logged out.	

	After all these changes so far, to login as `grader` with private key, I now use:

		vagrant:~$ ssh -i ~/.ssh/myprivatekey.rsa grader@54.200.104.8 -p 2200

	To check the current SSH port number, I used:

		grader:~$ netstat -an | grep ESTABLISHED

	There is a tcp entry with ESTABLISHED state that has local address with port 2200.

3. **Update all currently installed packages**

		grader:~$ sudo apt-get update
		grader:~$ sudo apt-get upgrade

	I got a "sudo: unable to resolve host ip-10-20-30-114" error every time I used sudo, so I added that hostname to `/etc/hosts`:

		#added this line to /etc/hosts
		127.0.1.1 ip-10-20-30-114

	After exiting and logging back in via ssh, I got a "\*\*\*system restart is required\*\*\*" message, so I rebooted the system:

		grader:~$ sudo reboot

7. **Configure the Uncomplicated Firewall (UWF) to only allow incoming connections for SSH (port 2200), HTTP (port 80) and NTP (port 123)**

		grader:~$ sudo ufw default deny incoming
		Default incoming policy changed to 'deny'
		grader:~$ sudo ufw default allow outgoing
		Default outgoing policy changed to 'allow'
		grader:~$ sudo ufw allow 2200
		Rules updated
		Rules updated (v6)
		grader:~$ sudo ufw allow www
		Rules updated
		Rules updated (v6)
		grader:~$ sudo ufw allow ntp
		Rules updated
		Rules updated (v6)
		grader:~$ sudo ufw enable

	I then checked that the desired ports are opened:

		grader:~$ sudo ufw status
		Status: active

		To				Action	From
		--				------	----
		80/tcp			ALLOW	Anywhere
		2200			ALLOW	Anywhere
		123				ALLOW	Anywhere
		80/tcp (v6)		ALLOW	Anywhere (v6)
		2200 (v6)		ALLOW	Anywhere (v6)
		123 (v6)		ALLOW	Anywhere (v6)	

	I also tried logging out of the Udacity remote machine and ssh using default port 22 but no connection can be made.


8. **Configure the local timezone to UTC**

	When I ran the `date` command, it shows the time in UTC, so I didn't make any changes here.

9. **Install and configure Apache to serve a Python mod_wsgi application**

	Installed Apache:

		grader:~$ sudo apt-get install apache2

	At this point of time, I was able to visit [http://54.200.104.8/](http://54.200.104.8/) to see the Apache2 Ubuntu Default Page.

	Installed mod_wsgi:
		
		grader:~$ sudo apt-get install libapache2-mod-wsgi

	Got an error message that says:  "apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message". So I added the line below to `/etc/apache2/apache2.conf`:

		ServerName localhost
	

10. **Install and configure PostgreSQL**

		grader:~$ sudo apt-get install postgresql

	Created a new user named `catalog` that has limited permissions to the catalog application database

		grader:~$ sudo adduser catalog	

		grader:~$ sudo -i -u postgres
		postgres:~$ createuser --interactive -P
		Enter name of role to add: catalog
		Enter password for new role:
		Enter it again:
		Shall the new role be a superuser? (y/n) n
		Shall the new role be allowed to create database? (y/n) n
		Shall the new role be allowed to create more new roles? (y/n) n

		postgres:~$ psql
		postgres=# CREATE DATABASE catalog;
		postgres=# \q

		postgres:~$ exit
		logout
		grader:~$   

11. **Install git, clone and setup your Catalog App project so that it functions correctly when visiting your server's IP address in a browser.**

	First, I installed Git:

		grader:~$ sudo apt-get install git

	I cloned the Item Catalog project using git:

		grader$ sudo mkdir -p /var/www/itemcatalog/ItemCatalog
		grader$ sudo git clone https://github.com/shikeyou/ItemCatalog.git /var/www/itemcatalog/ItemCatalog

	Protected .git folder:
 
		grader$ sudo chmod 700 /var/www/itemcatalog/ItemCatalog/.git

	I added http://54.200.104.8 to Authorized JavaScript Origins in  the project in Google Developer Console, then downloaded the JSON file. Copied the JSON file contents to a new file `/var/www/itemcatalog/ItemCatalog/client_secrets.json`.

	Installed additional packages and libraries necessary to host this application:

		grader:~$ sudo apt-get install python-psycopg2
		grader:~$ sudo apt-get install python-flask
		grader:~$ sudo apt-get install python-sqlalchemy
		grader:~$ sudo apt-get install python-pip
		grader:~$ sudo pip install oauth2client
		grader:~$ sudo pip install requests
		grader:~$ sudo pip install httplib2

	Changed the project to use PostgreSQL database (it was using SQLite originally):

	* Changed all references to SQLite database to reference PostgreSQL database:

			#change argument of all create_engine() calls to use postgresql instead of sqlite
			engine = create_engine('postgresql://catalog:password@localhost/catalog')

		The above step was done for
		* `/var/www/itemcatalog/ItemCatalog/application.py`
		* `/var/www/itemcatalog/ItemCatalog/db/db_populate_test_data.py`
		* `/var/www/itemcatalog/ItemCatalog/db/db_setup.py`
		* `/var/www/itemcatalog/ItemCatalog/db/db_show_all_data.py`  

	* Setup the new PostgreSQL database `catalog` and populate it with some test data
			
			grader$ python /var/www/itemcatalog/ItemCatalog/db/db_setup.py
			grader$ python /var/www/itemcatalog/ItemCatalog/db/db_populate_test_data.py 

	Created a wsgi file as the main entry point, for Flask to work with mod_wsgi:

		grader:~$ sudo vim /var/www/itemcatalog/itemcatalog.wsgi

	In `itemcatalog.wsgi`: 

		#insert application path into sys.path so that it can be found
		import sys
		applicationPath = '/var/www/itemcatalog/ItemCatalog'
		if applicationPath not in sys.path:
        	sys.path.insert(0, applicationPath)

		#also change current working directory to that path
		import os
		os.chdir(applicationPath)

		#import Flask's app as the WSGI-required application object
		from application import app as application
		
	Edited `/etc/apache2/sites-available/000-default.conf`:

		#update server admin email to my email so that it will show up in error pages		
		ServerAdmin shikeyou@gmail.com

		#change document root
		DocumentRoot /var/www/itemcatalog

		#create a daemon process for user catalog
		#this isolates execution env using a normal user account meant just for serving the app
		WSGIDaemonProcess itemcatalog user=catalog group=catalog threads=5

		#create script alias for wsgi main entry script
		WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi

		#create rules for itemcatalog directory
		<Directory /var/www/itemcatalog>
			WSGIProcessGroup itemcatalog
			WSGIApplicationGroup %{GLOBAL}
			Order deny,allow
			Allow from all
		</Directory>

	After changes were made to the config files, I restarted the Apache server:

		grader:~$ sudo apache2ctl restart 

	During this whole process, if any error pages were served (e.g. 500 Internal Server Error), I would check:

		grader:~$ sudo cat /var/log/apache2/error.log

	I also added a logging system to my application.py file so that Flask exceptions get logged (if not, Flask exceptions will just show up as 500 Internal Server Error with nothing showing up in Apache's error log!)

	Created the log file first for `catalog` user:

		grader:~$ sudo mkdir /var/log/itemcatalog
		grader:~$ sudo touch /var/log/itemcatalog/error.log
		grader:~$ sudo chown catalog:catalog /var/log/itemcatalog/error.log
		grader:~$ sudo chmod 640 /var/log/itemcatalog/error.log

	Then in `application.py`:

		#in /var/www/itemcatalog/ItemCatalog/application.py
		import logging
		fileHandler = logging.FileHandler('/var/log/itemcatalog/error.log')
		fileHandler.setLevel(logging.WARNING)
		formatter = logging.Formatter('%(asctime)s; %(levelname)s; %(message)s', '%Y-%m-%d %H:%M:%S')
		fileHandler.setFormatter(formatter)
		app.logger.addHandler(fileHandler)

	I also had to move the secret key variable out of the `if __name__ == '__main__'` part for Google+ login to work:

		#in /var/www/itemcatalog/ItemCatalog/application.py
		app = Flask(__name__)
		app.secret_key = os.urandom(123)  #move this out of the "if __name__ == '__main__'" block 

	With all these done, the server is up and running, and the application can be viewed at [http://54.200.104.8/](http://54.200.104.8/)

## Bonus Work
		
1. **Setup system emailing system (required to get feedback for the extra services that I am going to install)**

	The bonus features that I will be installing requires sending out email for alerts or logs. Thus I have installed `sendemail` to send out emails:

		grader:~$ sudo apt-get install sendmail
		grader:~$ sendmailconfig
		(answer yes to all the questions)

	Had to change the first line in `/etc/hosts` to make sending emails really fast:

		127.0.0.1 localhost localhost.localdomain ip-10-20-30-114

	The last bit (ip-10-20-30-114) is the hostname of the remote machine, which I got from typing `hostname` in the terminal.

		grader:~$ hostname
		ip-10-20-30-114

	Finally, I restarted the service:

		grader:~$ sudo /etc/init.d/networking restart

	To test that it works, I tried sending an email to myself:

		grader:~$ (echo "Subject:test"; echo "This is the message body") | sudo sendmail shikeyou@gmail.com

	I got an email in my inbox almost instantly, which meant that it works and it's fast!

	References: 
	* [http://collinhenderson.com/post/48046976172/a-fix-for-slow-sendmail-on-ubuntu](http://collinhenderson.com/post/48046976172/a-fix-for-slow-sendmail-on-ubuntu)
	* [https://holarails.wordpress.com/2013/11/17/configure-sendmail-in-ubuntu-12-04-and-make-it-fast/](https://holarails.wordpress.com/2013/11/17/configure-sendmail-in-ubuntu-12-04-and-make-it-fast/)



2. **Monitor for repeat unsuccessful login attempts and appropriately ban attackers**

	I installed `fail2ban` to monitor log files for attacks and automatically ban IP addresses after certain number of tries.

		grader:~$ sudo apt-get install fail2ban

	After installation, I made these changes to `/etc/fail2ban/jail.conf`:

		#change email destination to my email
		destemail = shikeyou@gmail.com

		#indicate that an email will be sent out
		action = $(action_mwl)s

		#enable ssh and change port number
		[ssh]
		enabled 	= true
		port		= 2200
		filter		= sshd
		logpath		= /var/log/auth.log
		maxretry	= 3

	I activated this service for SSH only but it could be used for other things like FTP as well.

	Then I restarted the service:
	
		grader:~$ sudo /etc/init.d/fail2ban restart
	
	I checked that the service is active:
		
		grader:~$ sudo fail2ban-client status
		Status
		|- Number of jail:	1
		`- Jail list:		ssh
		
	I then tried to SSH into the remote server using a non-existing user name `test`.
	
		vagrant:~$ ssh -i ~/.ssh/myprivatekey.rsa test@54.200.104.8 -p 2200 

	After 3 unsuccessful tries, I found that I was not able to connect anymore, even with a correct username and private key. This indicates that my IP address has been banned.

	After waiting for a while (10 minutes), I was able to SSH into the remote machine again. I checked the logs:

		grader:~$ sudo cat /var/log/auth.log		
		...		
		Invalid user test from XX.XX.XXX.XX
		input_userauth_request: invalid user test [preauth]
		...

		grader:~$ sudo cat /var/log/fail2ban.log		
		...		
		fail2ban.actions: WARNING [ssh] Ban XX.XX.XXX.XX
		...
		fail2ban.actions: WARNING [ssh] Unban XX.XX.XXX.XX
		...

	I also received emails from Fail2Ban concerning this "attack" and that a particular IP (which is mine) has been banned. 

	All these indicate that fail2ban has been setup properly and is working.

	Reference: [https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics](https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1204-lts-server-part-1-basics)

2. **Setup cron jobs to automatically manage package updates**

	I installed a package called `cron-apt` which does `apt-get` on regular intervals based on cron jobs.

		grader:~$ sudo apt-get install cron-apt 

	Then I added these lines in `/etc/cron-apt/config` to get an email everytime cron-apt runs (so that I know it is working):

		MAILON="always"
		MAILTO="shikeyou@gmail.com"

	Edited `/etc/cron.d/cron-apt` to change how often `cron-apt` runs:

		# Every Sunday at midnight
		0 0 * * 0 root test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt

	I ran `cron-apt` manually just to test and got an email once the job was done:

		grader:~$ sudo /usr/sbin/cron-apt

	To read the logs for cron-apt:

		grader:~$ sudo cat /var/log/cron-apt/log

	`cron-apt` only downloads the updates but will not install them  automatically (it runs `apt-get upgrade -d`). I then had to run this command to upgrade from the downloaded files:

		grader:~$ sudo apt-get dist-upgrade

	An alternative would be to just edit crontab and run apt-get commands in it. But `cron-apt` does logging and emailing automatically, so I decided to go with this simpler approach.

	Reference: [http://www.techrepublic.com/article/automatically-update-your-ubuntu-system-with-cron-apt/](http://www.techrepublic.com/article/automatically-update-your-ubuntu-system-with-cron-apt/)

