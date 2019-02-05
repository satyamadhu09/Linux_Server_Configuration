# Linux_Server_Configuration

## Project OverView
This Project is a part of Nano Degree based on Full Stack Development by Udacity.
This Project mainly explains the installation of a Linux Server and used to host web applications. This includes securing our server from several attacks, installing and configuring a database server and deploying one of our existing applications into it.

For setting up this server, below are the steps to be followed:

- The Private Server used here is [Amazon EC2](https://console.aws.amazon.com/)
- The web application used to deploy in the server is the [Item Catalog Project](https://github.com/satyamadhu09/Item-Catalog-02) which was done earlier in the Nano Degree Program.
- The Database Server used is [PostgreSQL](https://www.postgresql.org/)

## Procedure to connect to Amazon EC2:
- Firstly, open your [AWS Educate](https://www.awseducate.com/) and login to your account.
- After logging in, go to your lab which is activated for you and Click on start lab. Now your lab is running, and it should run till the end of your deployment process.
- Click on Open Console and then go to *All Services*. Select *EC2.*
- Now Change the server to N. Virginia for creating an instance. 
- Click on *Launch Instance* and select your preferred instance. In this project, the instance used is **Ubuntu Server 18.04 LTS.**
- After selecting, Click on **Review and Launch** and again select **Launch.**
- Choose **“Create a new Pair”** and edit the key pair name and Download it. The downloaded file is of the format “.pem”.
- Now Click on Launch Instance and give it a name. It shows that instance is running.
- Click on Description which is at the end of the page and Copy your Public DNS and Public IP
- Click on view Launch log to check whether it is running successfully.
- Now, click on **launch wizard-2** and Click on Inbound. In this you must add the rules for incoming connections i.e., add port for SSH (2200), HTTP (80) and NTP (123) and save.

To use privatekey [link](https://github.com/satyamadhu09/Linux_Server_Configuration/blob/master/Private%20Key).

Public IP address is: 18.233.166.130

## Steps for Configuring Linux Server:

Step 1: Firstly, create a new folder and place the downloaded .pem file in it.
Step 2: Open it using Git Bash by the following command:

        ssh -i  filename(linux_server_config.pem) ubuntu@IP address
     
Step 3: Now, you need to update the packages.

      sudo apt-get update
      sudo apt-get upgrade

Step 4: After installing packages, open the file:
     
      sudo vi /etc/ssh/sshd_config
     
In this file, Change the port number from 22 to 2200 and save it (Esc  and :wq)

Step 5: Restarting the service using :

       sudo service ssh restart
    
Step6. Now open the file with ssh port 2200. It is to check whether the port 2200 is working or not.

        ssh -i linux_server_config.pem -p 2200 ubuntu@IP address

Step 7. Now add the following commands to configure the Firewall (UFW):

     sudo ufw default deny incoming
     sudo ufw default allow outgoing
     sudo ufw default allow 2200/TCP
     sudo ufw default allow www
     sudo ufw default allow 123/NTP
     sudo ufw default deny 22
     
     sudo ufw default enable
     
After Proceed Option Y/N - Y
     
To check status:

     sudo ufw status

```Status: active

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

After Completing the above steps, now you need to create a User.

## Creating a New User ‘grader’:

1.	Firstly, login using Ubuntu user.
Now, add user using : 
         
         sudo adduser grader

And give a password.

2.	Now give the grader premissions in the sudoers file:

              sudo visudo
              
3.	Now edit the file by adding below the line:  root ALL=(ALL:ALL) ALL

Add the following line to give sudo privileges to grader user:

      grader  ALL=(ALL:ALL) ALL
 
4.	Save and exit using ctrl+x and confirm with Y.
5.	To verify the grader as sudo permission:
           
           su -grader
           
   and enter password.
           
6.	Now, create SSH key-pair for grader. Cofigure keypairs for grader. Create .ssh folder by:

        mkdir /home/grader/.ssh
 
7.	Change it to user  grader:    
              
              su grader
              
8.	RUN this command:

          sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys.

9.	Now change ownership:

          chown grader.grader /home/grader/.ssh.
 
10.	Now add grader to sudogroup.
             sudo su
           usermod -aG sudo grader
           
11.	change permissions for .ssh folder :

        chmod 0700 /home/grader/.ssh/, 
for authorized_keys:
 
        chmod 644 authorized_keys
 
12.	Now go to vi /etc/ssh/ssh_config. In this you have to edit some authentications.

Change permit root login no   and   pubkey authentication yes

13.	Now save and Exit.
14.	Restart the service: 

        sudo service ssh restart.
        
15.	After restarting, open the server with grader:

         ssh -i filename.pem -p 2200 grader@IPaddress.
 
### Set the time zone for grader:

Configure the local time zone to UTC Logged on grader Account:

        sudo dpkg-reconfigure tzdata
        
## Installing Apache and PostgreSQL:

**Step 1** . Now install apache software as grader.

      sudo apt-get install apache2
      
**Step 2**. Enter public IP of the Amazon EC2 instance into browser (Installation process success or not) --- If success, it displays the APACHE PAGE.
**Step 3.** Now again install library functions of apache

           sudo apt-get install libapache2-mod-wsgi-py3
           
**Step 4.** Enable mod_wsgi using: 

            sudo a2enmod wsgi
            
**Step 5.** After enable of wsgi, install some libraries of python development:
 
            sudo apt-get install libpq-dev python-dev	

### Install and configure PostgreSQL:

**Step 6.** Now, install postgresql 
      
      sudo apt-get install postgresql postgresql-contrib.

**Step 7.** After installing postgresql, change to postgresql from grader user

    		      sudo su - postgres
   		        psql
              
**Step 8.** After entering to psql,

Create User Catalog :

      CREATE USER catalog WITH PASSWORD 'catalog';

**Step 9.** Now alter the user, 

    	ALTER USER catalog CREATEDB;
      
**Step 10.** Create a database:

      create database catalog with owner catalog;

**Step 11.** Now change to catalog database:

        \c catalog.

**Step 12.** Now revoke all the schemas:

        REVOKE ALL ON SCHEMA public FROM public;
        
**Step 13.** Now grant all the public schemas to catalog:

    		GRANT ALL ON SCHEMA public TO catalog;
        
**Step 14.** Now exit from the database and switch back to the grader user: exit

## Installing Git:

**Step 1.** Now install git

         sudo apt-get install git.

**Step 2.** Now change the directory to www:

       cd /var/www.

**Step 3.** Now clone our git project here:

     sudo git clone ‘path(https://github.com/satyamadhu09/catalog.git)’.

**Step 4.** Now change the owner permissions:

      sudo chown -R grader:grader catalog.

**Step 5.** Now rename the main project file:

      sudo mv projectflask.py __init__.py.

**Step 6.** Now in python files, change the database sqllite engine to postgres engine.

    engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

## Creating Google Oauth Credentials:

- Go to your Project page in the [Google APIs Console](http://console.developers.google.com)
- Choose credentials from the menu on the left and Create an OAuth CLIENT_ID
- Before applying some changes, it prompts to configure the Consent Screen. After configuring, now choose Web Application from the list and give a Application name.
- Set the Authorized JavaScript Origins as the following:

        http://ipaddress.xip.io
        
and set Redirect URI’s 

        http://ipaddress.xip.io/login

        http://ipaddress.xip.io/gconnect

        http://ipaddress.xip.io/callback 
        
- Click on Save and now a JSON file and CLIENT_ID is created. Download and place it in your project folder. 
- In login.html, edit the new CLIENT_ID. Also, replace the content in old client_secrets.json file with the content in new client_secrets.json

## Configure and Enable a New Virtual Host:

      sudo nano /etc/apache2/sites-available/Project.conf

```
<VirtualHost *:80>
    ServerName 18.233.166.130.xip.io
    ServerAlias ec2-18-233-166-130.compute-1.amazonaws.com
    ServerAdmin ubuntu@54.210.140.47
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv3/lib/python3.6/site-packages
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

- Now, Enable the virtual host by using the command:
  	 
      sudo a2ensite catalog
     
- After enabling site catalog, to activate the new configuration
  You need to run:	 
  
      sudo service apache2 reload

## Setting the Flask Application:

- Create /var/www/catalog/catalog.wsgi file:
   Add the following lines:
   
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/catalog/")
        from catalog import app as application
        application.secret_key = 'supersecretkey'

- Reload and restart Apache using:
        
        sudo service apache2 reload
        sudo service apache2 restart
        
- From the folder var/www/catalog/catalog/   Create a new virtual Environment.

        sudo virtualenv -p python venv
        
- Now change the ownership permissions :

        sudo chown -R grader:grader venv.

- Finally, activate virtual environment:

                     . venv/bin/activate
                     
- After activating Virtual environment, you need to install the following packages:

    sudo apt-get install pip3
    pip3 install flask
    pip3 install sqlalchemy
    pip3 install httplib2
    pip3 install requests
    pip3 install --upgrade oauth2client
    pip3 install psycopg2-binary
    sudo apt-get install libpq-dev

- Now all the packages are installed. If not installed with the above commands or got a error that no module found, install with this command:
  
      sudo -H pip3 install packagename

- Now disable the default Apache site:

   	sudo a2dissite 000-default.conf

The following prompt will be returned:

        `Site 000-default disabled`
        - To activate the new configuration, you need to run:
          `service apache2 reload`
        - Reload Apache: `sudo service apache2 reload`

 - You need to edit the python source file:
   
          sudo vi python __init__.py
          
``` At database import statement, add:
        from catalog.database import *

- Replace xrange() with range()

- At specification of client_secrets.json, replace with /var/www/catalog/catalog/client_secrets.json

- Now save the file and again reload apache.
- Now run database file, sample items file.
```

         python database_setup.py
    	 python lotsofmenus.py
       
- Again reload and restart the apache
- Security Updates and package updates use these commands.

   	    sudo apt-get update
   	    sudo apt-get upgrade
   	    sudo apt-get dist-upgrade

- To check any errors found while running the web application, check the error logs by using :

      sudo tail -f /var/log/apache2/error.log

- After resolving the errors, run the application in the web browser:

      (http://18.233.166.130.xip.io)
      (http://ec2-18-233-166-130.compute-1.amazonaws.com)

- Special thanks to our mentors. Without their help I could not done this project.
- References from Online resources:
Udacity, Stack Overflow, YouTube.










