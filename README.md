# Project 8: Configure A Linux Server
by Abdulrahman Elsharqawi, This project is supervised by Udacity's **[Full Stack Web Developer Nanodegree](https://www.udacity.com/course/nd004)**

This Project is an application of the skills earned in the last part of The Full-Stack web developement Nanodegree, which is:
- Installing a virtual linux distribution either on your device or on a VPS (virtual private server).
- Configuring a linux distribution and customize it for your needs, which includes:
    * Creating new users.
    * giving admin permissions to any of the users.
    * prevent users from logging in with their normal passwords.
    * securing the system using Firewall.
    * updating, installing, and removing packages and programs.
    * using ssh public keys to have a secure access to the system.
    * file permissions and chmod numbers.
- Deploying Python web applications via apache server and WSGI.

### The server information:
    **IP Address:** 35.196.100.215
    **SSH Port:** 2200
    **URL:** http://35.196.100.215.xip.io
    Note: all private details, passwords, and publick keys are provided in a text file given only to udacity review team
----------------------

## Server Configurations:
**Note: In this project I used GCP (Google Cloud Platform) instead of Amazon Lightsail**
**Note: Make sure you selected "Allow HTTP traffic" when you created the vm instance**
* First, access your server by clicking the ssh button in your vm instance page.
* Update all of your packages using the command: `sudo apt-get update`
* Upgrade all of your packages using the command: `sudo apt-get updgrade`
* Install finger using the command: `sudo apt-get install finger`
* Install git using the command: `sudo apt-get install git`

    ### Changing the SSH port:
    * Run the following command: `sudo nano /etc/ssh/sshd_config`
    * Locate the following line: "# port 22"
    * Remove # and change 22 to 2200
    * Restart the sshd service by running the following command: `sudo service sshd restart`
    * In your vm instance google cloud page, under "Network interfaces" click on "nic0"
    * Scroll down and click on "default-allow-ssh"
    * Click on Edit
    * Under "Protocols and ports" change the port to 2200 and click save
        * **Note: Be aware that your server will no longer be accessible through the web app 'SSH' button. The button assumes the default port is being used, instead click the small arrow besides the button and choose 'Open in Browser window on custom port', and write 2200, and then click OPEN.**

    ### Configuring Ubuntu Firewall (ufw):
    * Run the following command: `sudo ufw default deny incoming`
    * Run the following command: `sudo ufw default allow outgoing`
    * Allow SSH connection by running the command: `sudo ufw allow 2200/tcp`
    * Allow HTTP connection by running the command: `sudo ufw allow http`
    * Allow NTP connection by running the command: `sudo ufw allow ntp`
        * **Note: Double check by running the command: `sudo ufw show added` to make sure that SSH port is included in the allowed rules, otherwise your server will be dead after enabling the firewall and you won't be able to access it**
    * Enable Ubuntu Firewall by running the command: `sudo ufw enable`
    * Check the firewall status by running the command: `sudo ufw status`

    ### Creating grader user:
    * Run the command: `sudo adduser grader`
    * Enter a password for the new user
    * Enter the additional details like name, telephone, ... etc
    * Answer the message 'Is the information correct?' by typing 'y'
    * Confirm this process by running the command: `finger grader`

    ### Giving grader user sudo access:
    * **Don't** use the new account unless you finished the following process, so you can access root files and have the full access to the system
    * Run the command: `sudo ls /etc/sudoers.d/`
    * Usually, you will find three files: '90-cloud-init-users', 'README', 'your admin user given from the cloud'
        * **Note: Usually it will be 'gxxx' where xxx is the first part of your google email, or sometimes it will be 'google_sudoers', anyway, we will just call it 'udacity'**
    * Run the command: `sudo cp /etc/sudoers.d/udacity /etc/sudoers.d/grader`
    * Run the command: `sudo nano /etc/sudoers.d/grader`
    * Change the line **%udacity ALL=(ALL) NOPASSWD:ALL** to be **%grader ALL=(ALL) NOPASSWD:ALL**

    ### Generating public key for the user grader:
    * **On your local machine**, open terminal or command prompt and run the command: `ssh-keygen`
    * Write the following path: `C:\Users\XXX/.ssh/***` where XXX is the name of the current user logged in, and *** is the name of the ssh key pairs files, we will call it "secret"
    * Enter the passphrase
    * Now you have two files inside .ssh folder: `secret` and `secret.pup`
    * With your favorite text editor, open secret.pup and copy everything in the file
    * Using the "SSH" button on your vm instance google page, login to your vm and then do the following:
        * Run the command: `su grader`
        * enter the password for this account
        * Run `cd` to redirect to the home directory
        * Run the command: `mkdir .ssh`
        * Run the command: `nano .ssh/authorized_keys`
        * Paste the key in this file and replace the `user@DESKTOP-something` at the end with `grader`
        * Run the command: `cd ..`
        * Run the command: `chmod 700 .ssh`
        * Run the command: `chmod 644 .ssh/authorized_keys`
        * In your local machine `cd` to the .ssh folder
        * **Now you can login to grader account using the private key in your local machine by running the command: `ssh grader@your.vm.public.ip -p 2200 -i secret` enter the passphrase and you will be logged in**
        * **Note: Forcing key based authentication is enabled by default in this VPS, so you don't need any additional steps to enable it**

    ### Configure the local timezone to UTC:
    * Run the command: `sudo dpkg-reconfigure tzdata`
    * Choose `None of the above`
    * Choose `UTC`

    ### Install and configure Apache to serve a Python application:
    * Run the command: `sudo apt-get install apache2`
    * Run the command: `sudo apt-get install libapache2-mod-wsgi`
    * Run the command: `cd /var/www`
    * Run the command: `git clone <your-project-github-url>
    * **We will assume that your project called `Items` and the main python file in the project's folder called `run.py`**
    * Run the command: `cd Items`
    * Run the command: `sudo nano run.wsgi`
    * Type the following:
        *`import sys
        sys.path.insert(0, "/var/www/Items")
        from run import app as application`
