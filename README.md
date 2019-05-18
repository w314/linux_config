# Server Information
## IP Address
3.215.39.37
## SSH Port
2200
## Application URL
http://3.215.39.37.xip.io

# Summary of Software Installed
- apache2
- libapache2-mod-wsgi-py3
- python3-dev
- python3-pip
- virtualenv
- Flask
- httplib2
- oauth2client
- sqlalchemy
- sqlalchemy_utils
- psycopg2
- requests
- libpq-dev
- python3-dev
- postgresql
- postgresql-contrib

# Configuration Changes on Server
## Configuring Grader User SSH & Firewall
1. Attach static IP in
1. Configure Firewall
    1. Set ssh port to 2200
        - Edit ssh config file: ```sudo nano /etc/ssh/sshd_config  ```
        - Change row ```Port 22``` to Port ```2200```
        - Restart ssh service: ```sudo service sshd restart```
        - Edit server in online consol 
            - Add Custom row with Protocol:TCP Port:2200 to your Firewall settings in the Networking tab
            - Add Custom row with Protocol:UDP Port:123 to your Firewall settings in the Networking tab
            - Remove SSH line with Port:22
    1. Create firewall rules        
    ```sudo ufw status```
    ```sudo ufw default deny incoming```
    ```sudo ufw default allow outgoing```
    ```sudo ufw allow 2200/tcp```
    ```sudo ufw allow www```
    ```sudo ufw allow 123/udp```
    ```sudo ufw enable```
    ```sudo ufw status``` 


1. Add user (grader) with sudo rights
    1. Generate ssh keypair on local machine with ```ssh-keygen```
    1. Create new user on server
        1. Create user with: ```sudo adduser grader```
        1. You can see user added in /etc/passwd file
        ```sudo cat /etc/passwd```
        1. You can also see its home directory with ```ls /home```
        1. Create ssh directory to grader
        sudo mkdir /home/grader/.ssh
        1. Check .ssh direcotry created with ```ls -al /home/grader```
        1. Add authorized_keys file under grader's .ssh directory
        sudo touch /home/grader/.ssh/authorized_keys
        1. Edit the authorized_keys file, copy grader's public key to file
        ```sudo nano /home/grader/.ssh/authorized_keys```
        1. Change authorized_keys file permission to 600
        ```sudo chmod 600 /home/grader/.ssh/authorized_keys```
        1. Check new permission with ```ls -al /home/grader/.ssh```
        1. Change authorized_keys file ownership to grader:grader
        ```sudo chown grader:grader /home/grader/.ssh/authorized_keys```
        1. Check new ownership with ```ls -al /home/grader/.ssh```
        1. Change grader's .ssh folder permission to 700
        ```sudo chmod 700 /home/grader/.ssh```
        1. Check new permission with ```ls -al /home/grader```
        1. Change .ssh folder ownership to grader:grader
        ```sudo chown grader:grader /home/grader/.ssh```
        1. Check new ownership with ```ls -al /home/grader```
        1. Check your setup by signing in with new user
    1. Add sudo rights to grader
        1. Sing in with user who has sudo rights
        1. Create file grader in sudoers.d directory
        ```sudo touch /etc/sudoers.d/grader```
        1. Edit grader file under /etc/sudoers.d
        ```sudo nano /etc/sudoers.d/grader```
        Add the following code:
        ```grader ALL=(ALL) NOPASSWD:ALL```
        1. Check your configuration by executing a sudo command with user grader with command like
        ```sudo ls /etc/sudoers.d```
1. Disable remote login of root user and force SSH login
    - Edit sshd_config with: ```sudo nano /etc/ssh/sshd_config```.
    - Set ```PermitRootLogin no```.
    - Make sure config file has ```PasswordAuthentication no```.
## Update all system packages
    - ```sudo nano apt-get update```
    - ```sudo nano apt-get upgrade```

