# Udacity_Linux_Configuration
#Udacity Linux Server Configuration Project - Sundharesan R


## Amazon LightSail instance
Public IP: 35.178.50.92

SSH Port: 2200

Complete URL: http://35.178.50.92

### Steps of configuration

1. Create an account and select Ubuntu OS on [Amazon Lightsail] (https://lightsail.aws.amazon.com) and download the private key (PK).

Change the access permission of the Public Key running the following:
```
$ chmod 400 LightsailDefaultKey.pem
``` 

2. Follow the instructions provided to SSH into your server.

Access the machine with ssh:

```
$ ssh ubuntu@35.178.50.92 -p 22 -i LightsailDefaultKey.pem
```


3. Update all currently installed packages.

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install finger
```


### Creating grader access.

4. Create a new user account named grader and check using finger.

```
$ sudo adduser grader
$ finger grader
```

5. Give grader the permission to sudo.
Open the file using below command.

```
$ sudo nano /etc/sudoers.d/grader
```

6. Now add below content.
```
grader ALL=(ALL) NOPASSWD:ALL
```

7. Create an SSH key pair for grader using the ssh-keygen tool.
Generate on your machine the keys (private and public) (/home/ubuntu/.ssh/linuxCourse):

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): /home/ubuntu/.ssh/linuxCourse
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/linuxCourse.
Your public key has been saved in /home/ubuntu/.ssh/linuxCourse.pub.

```
8. Create the following directories:
```

$ su --login grader
$ sudo mkdir .ssh
$ sudo touch .ssh/authorized_keys
$ exit
$ cat .ssh/linuxCourse.pub (copy the ssh key)
$ su --login grader
$ sudo nano .ssh/authorized_keys (paste the key here)
$ sudo chmod 700 .ssh
$ sudo chmod 644 .ssh/authorized_keys
$ exit
```

Note: "linuxCourse"" - Private key needs to share with Udacity team. To download that key use winscp with your lightsail private key to connect as ubuntu user.

### Prepare to deploy your project.

9. Configure the time zone using below command:
```
$ sudo dpkg-reconfigure tzdata
```

10. Choose the option 'None of the Above' and then select UTC.

11. Now Install and configure Apache to serve a Python mod_wsgi application.
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
```

12. Install and configure PostgreSQL:
```
$ sudo apt-get install postgresql
````

* Do not allow remote connections
* Create a new database user named catalog that has limited permissions to access db.
```
$ sudo adduser catalog
$ sudo -u postgres -i

postgres:~$ createuser catalog
postgres:~$ createdb catalog

postgres:~$ psql

postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
postgres:~$ exit
```

13. Install git.
```
$ sudo apt-get install git
```

### Deploy the Item Catalog project.

14. Clone and setup your Item Catalog project from the Github Rep.
```
$ git clone https://github.com/sundharesan/Udacity-ItemCatalog.git ItemCatalog
```

15. Open *application.py* and *database_setup.py* and udpate the the create_engine for:
```
engine = create_engine('postgresql://catalog:catalog@localhost:5432/catalog')
```

16. Now open sampleitems.py and replace the the create_engine for:
```
engine = create_engine('postgresql://catalog:catalog@localhost:5432/catalog')
```

17. Install the dependencies:
```
$ sudo apt-get -y install python-pip
$ sudo pip install SQLAlchemy
$ sudo pip install psycopg2
$ sudo pip install flask
$ sudo pip install oauth2client
$ sudo pip install requests
```

18. Now you can run *database_setup.py* file to create tables for your ItemCatalog project.
```
python database_setup.py
```

19. To load sample items/categories run *sampleitems.py*. Note: If you get any error please install the missed modules using *sudo pip install MODULENAME*
```
python sampleitems.py
```

20. Modify the file /etc/apache2/sites-enabled/000-default.conf to add the following line (Right before the closing </VirtualHost>):
```
WSGIScriptAlias / /var/www/html/myapp.wsgi
```

21. Modify the file /var/www/html/myapp.wsgi to add the following content:
```
#!/usr/bin/python
import sys
import os
import logging
logging.basicConfig(stream=sys.stderr)
sys.stdout = sys.stderr
sys.path.insert(0,"/home/ubuntu/ItemCatalog/")
os.chdir("/home/ubuntu/ItemCatalog/")
from application import app as application   
```

22. Restart the server:
```
$ sudo apache2ctl restart
```

### Securing the System

23. Change the SSH port from 22 to 2200. Configure the Lightsail firewall to allow it.
Open the file /etc/ssh/sshd_config
```
$ sudo nano /etc/ssh/sshd_config
```

24. Now change the following data:
```
Port 2200
PermitRootLogin no
PasswordAuthentication no
```

25. Now resart the ssh service
```
$ sudo service ssh restart
```

26. Configure the UFW to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
$ sudo ufw default deny incoming

$ sudo ufw default allow outgoing

$ sudo ufw allow 2200/tcp

$ sudo ufw allow 80/tcp

$ sudo ufw allow 123/udp

$ sudo ufw enable
```

27. Also on Lightsail, click on the tab Networking:
```
Add port Custom TCP 123
Add port Custom TCP 2200
Remove port SSH TCP 22
```


28. Now update Facebook & Google Auth services to allow access for you public IP.


29. Your item catalog was ready on http://35.178.50.92


30. Use below command to check log if you are facing any errors.
```
cat /var/log/apache2/error.log
```
