# How to Setup a Cloud Server for Data Science

**Author**: [Luca Valnegri](https://www.linkedin.com/in/lucavalnegri/) [@datamaps](https://twitter.com/datamaps)

**Last Updated**:  07-Mar-2019

  <a name="index"/>

---
  * [Motivations](#motivations)
  * [Create a Virtual Private Server](#create-a-virtual-private-server)
    + [Sign up to Digital Ocean](#sign-up-do)
    + [Login into Digital Ocean](#login-do)
    + [Secure Digital Ocean Account](#secure-do)
    + [Create *droplet*](#droplet-without-ssh-key)
    + [First Connection](#first-connection)
      - [Windows Users](#without-key-windows)
      - [Linux and macOS Users](#without-key-linux-macos)
  * [Customize Your New Server](#customize-your-new-server)
    + [Upgrade the System](#upgrade-system)
    + [Add Admin User](#add-admin-user)
    + [Add *public* Group](#add-public-group)
    + [Add *public* Repository](#add-public-repository)
    + [Add Security Layer](#add-security)
    + [Install *Webmin*](#install-webmin)
    + [Add Domain Name](#domain-name)
    + [Take Your First *Snapshot*](#first-snapshot)
  * [The *R* Stack](#r-stack)
    + [Install core *R*](#install-r)
      - [Packages Management](#r-packages-management)
    + [Install *RStudio Server*](#install-rstudio-server)
      - [Using Projects with Version Control](#rstudio-using-projects-with-version-control)
    + [Install *Shiny Server*](#install-shiny-server)
    + [Install *Ubuntu* Dependencies for *R* packages](#install-linux-dependencies-for-r-packages)
    + [Install *R* packages](#install-r-packages)
      - [Data Preparation](#r-packages-data-preparation)
      - [Data Processing](#r-packages-data-processing)
      - [Data Display and Visualization](#r-packages-data-display-and-visualization)
        * [*htmlwidgets*, *JS* wrappers](#r-packages-htmlwidgets--js-wrappers)
        * [*ggplot* extensions](#r-packages-ggplot-extensions)
      - [Data Modeling, Mining and Learning](#r-packages-data-modeling--mining-and-learning)
      - [Spatial Data](#r-packages-spatial-data)
      - [Graphs, Network](#r-packages-graphs--network)
      - [Data Presentation, *Shiny*](#r-packages-data-presentation--shiny)
      - [Applications](#r-packages-applications)
      - [Tools and Utilities](#r-packages-tools-and-utilities)
    + [Testing the *R* stack](#testing-the-r-stack)
  * [Ngnix](#ngnix)
    + [Install Nginx](#nginx-install)
    + [Nginx Configuration](#nginx-configuration)
    + [Make Pretty URLs for RStudio Server and Shiny Server](#pretty-url)
    + [Add SSL Certificate](#ssl-certificate)
    + [Install php preprocessor](#php)
    + [Add Authentication to Shiny Server](#shiny-auth)
  * [The *Python* Data Science Stack](#python-data-science-stack)
    + [Install *Python*](#inpython)
    + [Install the Data Science Stack](#python-stack)
    + [Install *Jupyterlab*](#jupyterlab)
  * [Storage engines](#storage-engines)
    + [MySQL Server](#mysql)
      - [MySQL Web Cient: *DbNinja*](#dbninja)
    + [Neo4j](#neo4j)
    + [MongoDB](#mongodb)
    + [Redis](#redis)
    + $ [PostgreSQL](#postgresql)
    + $ [MonetDBLite](#monetdblite)
    + $ [Hive / Hbase](#hive-hbase)
    + $ [Influx DB](#influx-db)
  * [Docker](#docker)
    + [Install Docker](#docker-install)
    + [Basic Commands](#docker-commands)
    + [Dockerfile](#dockerfile)
    + [Example: *Selenium* for Web Driving](#docker-selenium)
    + [Resources](#docker+resources)
  * [Additional Tools](#additional-tools)
    + [Fonts](#fonts)
    + [Spark](#spark)
    + [Add SSH Key Pair for Enhanced Security](#droplet-with-ssh-key)
      - [Windows users](#with-key-windows)
      - [Linux and macOS users](#with-key-linux-macos)
  * [Appendix: Linux Basics](#linux-basics)
    + [Files and Folders](#linux-files-folders)
      - [Root Directory Structure](#linux-directory-structure)
      - [File Compressing](#linux-file-compressing)
    + [Users, Groups, Permissions, Ownerships](#linux-users-groups-permissions-ownerships)
    + [Software Management](#linux-software-management)
    + [Scheduling Tasks](#linux-cronjobs)
    + [Bash Shell](#linux-shell)
  * [Resources](#resources)
---

  <a name="motivations"/>

## Motivations 



<br/>

:point_up_2:[Back to Index](#index)
<a name="create-a-virtual-private-server"/>

## Create a Virtual Private Server  

  <a name="sign-up-do"/>

### Sign up to Digital Ocean
  - go to https://m.do.co/c/ef1c7bc80083 (you'll be credited $100 lasting 60 days)
  - insert your email and a sufficiently strong password (you can generate one suitable [here](https://www.random.org/passwords/?num=1&len=15&format=html&rnd=new))
  - you'll be asked for a credit card, but no money will be taken from your account. Just remember to check in at the end of the grace period!
  - check your email and validate your new account

  <a name="login-do"/>

### Login into Digital Ocean
  - go to https://cloud.digitalocean.com/login
  - click *Login* top right
  - enter username and password

  <a name="secure-do"/>

### Secure Digital Ocean Account
  - go to `Account` > `Security` > `Secure your account` > `Enable Two-Factor Authentication`
  - choose which system you prefer, then follow the corresponding instructions
  - in both cases, remember to generate the backup codes, and save them in some secure place

  <a name="droplet-without-ssh-key"/>

### Create *droplet*
  - Click the green "Create" button in the top right
  - Click "Droplet" from the unfolding menu
  - For the installation step, you should create a VPS which is at least 2GB RAM, because a few packages require more than 1GB RAM to compile. You can always change up or down either number of CPUs or amount of RAM later.
     For the moment being, choose the following:
    - Distribution: `Ubuntu 18.04.x x64`
    - Size: RAM 2GBPower: 1vCPUs, Storage: 50GB, Bandwith: 4TB, Cost: $10/mo ($0.030/hr)
	- Region: `London`
	- Choose a memorable name. You can always change it later from inside the machine
	- Choose the reference project. I guess you only have the default one at the moment though. You can build more structure to your account later if you decide to stick with Digital Ocean.
    - Click `Create`
  - Wait for the email containing the IP public address of the server, and the password for the *root* user. The IP address could also be found in the *Resources* tab besides the name of the droplet. 

Notice that Digital Ocean highly discourage the creation of *swap space*, practice often used to keep down the size, and hence the cost, of the droplet. This is due to the fact that their system is all made up of SSD storage, that is highly degraded by the continous read/write access, typical when swapping. Besides, upgrading the droplet leads to much better results in general.

  <a name="first-connection"/>

### First Connection

**SSH** stands for ***S**ecure **SH**ell* which is a [cryptographic](https://en.wikipedia.org/wiki/Cryptography "Cryptography")  [network protocol](https://en.wikipedia.org/wiki/Network_protocol "Network protocol") that allows secure access over an otherwise unsecured network. SSH is encrypted with *Secure Sockets Layer* (SSL), which makes it difficult for these communications to be intercepted and read.

Any VPS could be accessed with a typical user/password exchange, but it's also possible to setup *SSH keys* that identify trusted computers without the need for passwords. For additional security, it's also possible to add a *passphrase* to the key pair, that act as a password to access the key itself.

The first time you connect to a droplet as *root*, the system asks you to change the password. You have first to paste again the password you've just used to login, and then enter (twice) a new (strong) password. I advise you to use a password manager to securely collect, store and organize all your credentials. My suggestion is the free open source [KeePass Password Safe](https://keepass.info/).


  <a name="without-key-windows"/>

#### Windows users
Windows has no embedded ssh client by default. Many software can be downloaded for free, one of the most famous is [PuTTY](https://www.putty.org/), but we are going to use the much enhanced [MobaXTerm](https://mobaxterm.mobatek.net/), which is free for personal use and allows, among other functionalities, multi-tabbing and saving sessions.
  - [Download](https://mobaxterm.mobatek.net/download-home-edition.html) the *Home* edition of *MobaXTerm*. You can use, if you so prefer, the *portable* edition that doesn't need any installation. Just unzip the downloaded file in some folder of your choice, then run the included executable.
  - Open *MobaXTerm*
  - For a more standard copy and paste behaviour, click `Settings` towards the far right of the button bar, then click the tab `Terminal`. Uncheck the option *Paste using right-click*, then click `OK`. Now you can paste content in any terminal window using the standard `SHIFT+INS` keys combination (but you can't *copy* and *paste* using the more frequent `CTRL+C` and `CTRL+V`). In addition, a right-click button of the mouse exposes a quite extensive actions menu.
  - Click `Session` in the upper left button bar, then `SSH` in the upper left button bar of the new window
  - Paste the IP address you received with the email into the *Remote host* textbox, then click OK
  - type in  `root` when asked to *login as*, then copy the password you received with the email and paste it into the terminal. _**Notice**_ that by default Linux systems do not give any feedback from the password field. So don't try to paste again and again only because you feel the need to see some feedback, just paste the password once and hit enter!

  <a name="without-key-linux-macos"/>

#### Linux and macOS users
Both Linux *distros* and *macOS* have a built-in SSH client called *Terminal* which can be used to connect to remote servers:
  - **macOS**. *Terminal.app* is located in the `Applications > Utilities` folder. Double-click on the icon to start the client.
  - **Linux**. A Terminal window can be easily open using the shortcut `CTRL+ALT+T`. 
 
At the prompt you would type in general:  `ssh usrname@ip_address`. At the moment there is no other user than *root* , so to connect to your droplet just type:
`ssh root@ip_address`
If the IP address and the user name are correctly recognized, the system then prompts to enter the password associated with the specified user.


<br/>

:point_up_2:[Back to Index](#index)
<a name="customize-your-new-server"/>

## Customize Your New Server

  <a name="upgrade-system"/>

### Upgrade the System
  - To enable monitoring from the DO dashboard enter the following command (or simply copy and paste, it doesn't hurt):
    ~~~
    curl -sSL https://agent.digitalocean.com/install.sh | sh
    ~~~
    After a few minutes, you'll see a bunch of graphs and KPIs populating your droplets dashboard.
  - Enter the command  `date` to test if the timezone is correct. If it doesn't show the correct time and/or desired timezone, run the following commands: 
    ~~~
    dpkg --configure -a
	dpkg-reconfigure tzdata
    ~~~
	then enter the correct zone for your location. Notice that if you leave the timezone as **UTC**, there will be no automatic passage between winter and summer time. 
  - Before proceeding any further, let's thouroughly upgrade the system:
    ~~~
	apt-get update
	apt-get -y upgrade
	apt-get -y dist-upgrade
	apt-get -y autoremove
    ~~~
    answering `y` everytime you're asked permission.
	If during the above upgrading session a window pops up and asks for any changes, be sure to accept the choice:
	`keep the local version currently installed`
  - install some needed *basic* libraries that could be missed from the system (this much depends on how your chosen provider has decided to install the OS):
    ~~~
    apt-get -y install apt-transport-https software-properties-common nano dos2unix man-db ufw git-core libgit2-dev libauthen-oath-perl openssh-server
    ~~~
  - restart the system: 
    ~~~
    shutdown -r now
    ~~~

If you're on a different service than Digital Ocean, it'd also a good idea to disable the boot menu, or reduce the time it shows up:
  - open the conf file for editing:
    ~~~
    nano /etc/default/grub
    ~~~
  - add or change the following lines (if `GRUB_TIMEOUT=0` then you don't need any chages):
    ~~~
    GRUB_TIMEOUT=3
    GRUB_RECORDFAIL_TIMEOUT=3
    ~~~
  - update the boot loader:
    ~~~
    update-grub
    ~~~  
  
  <a name="add-admin-user"/>

### Add admin user
The Linux system is well known for its strong management of users, file and directories permissions and ownerships. In particular, it's an absolute no-no to use the default admin user, called *root*, as it could be 
It's customary instead to use a group called *sudo* that will act as a temporary admin 

  - create new user (change *usrname* with the actual user name):
    ~~~
    adduser usrname
    ~~~
  - enter a password twice (generate one suitable [here](https://www.random.org/passwords/?num=1&len=15&format=html&rnd=new)), and then the required information (you can simply void all the fields)
  - add new user as *sudoer* to the *sudo* group:
    ~~~
    usermod -aG sudo usrname
    ~~~
  - switch control to the new user *usrname*:
    ~~~
    su - usrname
    ~~~
  - check if *usrname* can actually run admin commands:
    ~~~
    sudo su 
    ~~~
  - always remember to exit from sudo when finished (also `CTRL+D` as shortcut):
    ~~~
    exit
    ~~~
 
From now on you should forget there exists a user called *root*, and always use instead *usrname* to run admin commands.

  <a name="add-public-group"/>

### Add *public* group

~~~
sudo groupadd public
sudo usermod -aG public usrname
~~~

  <a name="add-public-repository"/>

### Add *public* repository
I decided to go for `/usr/local/share/public`, but feel free to change the location as you wish.
~~~
sudo mkdir -p /usr/local/share/public/
sudo chgrp -R public /usr/local/share/public/
sudo chmod -R 2775 /usr/local/share/public/
~~~
To add the above path to a system variable, run the following command:
~~~
export PUB_PATH="/usr/local/share/public"
~~~
after which the path can be retrieved issuing a simple `$PUB_PATH` command. It's worth noting that the above command is only a *temporary* solution, as with a reboot the content of `PUB_PATH` is lost. To add the path to a permanent system variable, first open for editing the file that stores the system-wide environment variables:
~~~
sudo nano /etc/environment
~~~
then add the following line at the end
~~~
PUB_PATH="/usr/local/share/public"
~~~
Reboot to make sure the above changes have been applied.

Once you've decided the actual location, you have to build some structure in it, and that mostly depends on your projects. Any of the subdir can been created with the generic command:
~~~
mkdir -p /usr/local/share/public/newsubdir
~~~
or, if you've included the public path in the system environment:
~~~
mkdir -p $PUB_PATH/newsubdir
~~~

A possible quicker way to build a complete structure at once is to create a loop over a list of subdirs conveniently saved in a text file, as in the following example:
  - first save a plain text file as `subdirs.lst` in some directory in your *home* folder with the list of subdirs to be installed, each on its own line
  - save the following commands as text file named `subdirs.sh` in the same directory as the previous `subdirs.lst`:
    ```
    #!/bin/sh
    while read SDIR
    do
        mkdir -p $PUB_PATH/$SDIR
    done < `dirname $0`/subdirs.lst
    ```
  - make the above file an executable script:
    `chmod +x ~/path/to/subdirs.sh`
  - run it as:
    `~/path/to/subdirs.sh`

  <a name="add-security"/>
  
### Add security layers
First, deny the `root` user direct access via SSH:
  - open the SSH configuration file (if `nano: command not found` then `sudo apt-get install nano`):
    `sudo nano /etc/ssh/sshd_config`
 - change and Insert the following lines, then save the file (CTRL+x ==> y ==> Enter) :
	`PermitRootLogin no`
  - restart the service:
	`sudo systemctl restart ssh`	
  - logout, and test that the *root* user is NOT capable to ssh into the machine

Login back again as the *new* user, and let's change the standard ssh port **22** to a random integer number `xxxx` between 1024 and 65535:
  - open the SSH configuration file:
    `sudo nano /etc/ssh/sshd_config`
  - change the following line as desired, then save the file (CTRL+x ==> y ==> Enter)
	`Port xxxx`
	If there is a *hashtag* **#** at the start of the line, which means that the line is a comment and so not to be read by the system, delete it.
  - restart the service 
	`sudo systemctl restart ssh`
  - **without logging out from the current session**, open another session besides the one already open, and test that the new user is capable to *ssh* into the machine using the new `xxxx` port, but not from the standard **22**. If anything does not sounds right, close this session and fix using the original session.

Lastly, enable the standard firewall *ufw* allowing at once the new above port (THIS IS IMPORTANT!!!)
  - enable firewall
	`sudo ufw enable`
  - allow the *ssh* port (the new `xxxx` if it's been changed, or the standard **22** if it's not been changed)
	`sudo ufw allow xxxx`
  - check if the rule has been correctly applied, check again the number is correct!
	`sudo ufw status`
	Now, using a different session as earlier, test that the new user is still capable to ssh into the machine.

The following table lists the default ports for the main services used in this document. For a more comprehensive list of default ports used by various well-known services see [this Wikipedia article](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) .

| service | default |
|-----------|-------| 
| HTTP      |    80 |  
| HTTPS     |   443 |
| SSH       |    22 |
| FTP       |    21 | 
| SFTP      |    22*| 
| WEBMIN    | 10000 |
| RSTUDIO   |  8787 |
| SHINY     |  3838 |
| JUPYTER   |  8888 |
| MYSQL     |  3306 |
| POSTGRES  |  5432 |
| SQLSERVER |  1433 |
| MONGODB   | 27017 |
| NEO4J     |  7474 |
| REDIS     |  6379*|
| HIVE      |  |
| HBASE     |  |
| INFLUXDB  |  8086 |
| NEXTCLOUD |  |
| CALIBRE   |  |

A short list of some of the most used commands for the standard firewall is the following:
  - `ufw enable` 
  - `ufw disable` 
  - `ufw status` 
  - `ufw status verbose` 
  - `ufw allow XXXX` 
  - `ufw allow app list` 
  - `ufw allow app_profile`  
  - `ufw allow XXXX/tcp` 
  - `ufw allow XXXX/udp` 
  - `ufw allow XXXX:YYYY` 
  - `ufw allow from www.xxx.yyy.zzz` 
  - `ufw delete allow XXXX` 
  - `ufw deny XXXX` 
  - `ufw reset` 

All the above commands must obviously be launched as a *sudoer*.

If anything happens, and you can't login anymore through remote SSH, most VPS and Cloud providers allow users to open a shell from the dashboard account. 
On Digital Ocean head for the droplet dashboard. At the top right, there is a **Console** button which allows to login directly using password authentication. You often need to actually click into it before it becomes active.
Moreover, if you forget the root password, or you've never set it, head again for the droplet dashboard, and from the left menu click on the **Access** item. There you can find the magic button to reset the root password. Once you log in, if not asked by the system itself, you should reset the password again using the following commands:
~~~
sudo -i
passwd
~~~
 
  <a name="install-webmin"/>

### Install Webmin
While powerful and efficient, sometimes it's just nicer to work with a simple and intuitive graphic interface to manage the system. Here comes [Webmin](http://www.webmin.com/).

  - add the Webmin address to the list of trusted packages repositories:
    ~~~
    echo -e "\n# WEBMIN\ndeb http://download.webmin.com/download/repository sarge contrib\n" | sudo tee -a /etc/apt/sources.list
    ~~~
  - download the *public key* of the Webmin developer [Jamie Cameron](https://github.com/jcameron) to secure the Ubuntu package manager:
    ~~~
    wget http://www.webmin.com/jcameron-key.asc
    ~~~
  - add the above key to the manager sources keyring: 
    ~~~
    sudo apt-key add jcameron-key.asc
    ~~~
  - update the package management system: 
    ~~~
    sudo apt-get update
    ~~~
  - install webmin:
    ~~~
    sudo apt-get install -y webmin
    ~~~
  - allow access to the default Webmin port: 
    ~~~
    sudo ufw allow 10000
    ~~~
  - navigate to the *secure* URL [https://server_ip:10000/](https://server_ip:10000/) (notice that the *default* protocol `http` does not work), don't worry for now about the warnings
  - enter your now usual username and password to log in into the Webmin console
  - redirect standard `http` calls to encrypted `https` protocol:
    ~~~
    Webmin > 
	  Webmin Configuration >
	    SSL Encryption >
    ~~~
    Check **Yes** for `Redirect non-SSL requests to SSL mode?`, then `Save`
  - change default port to some random number `XXXX` :
    ~~~
    Webmin > 
	  Webmin Configuration >
	    Ports and Addresses >
	      Listen on IPs and ports >
	        Listen on port
    ~~~
	Also check:
    - **NO** for `Accept IPv6 connections? `
    - **Don't listen** for `Listen for broadcasts on UDP port` 
  - after the last change has been saved, the website will go down as the port has changed and it can't reconnect to its server anymore
  - restart *Webmin* to load the new configuration:
    ~~~
    sudo service webmin restart
    ~~~
  - go back to the terminal, and allow access to the new `XXXX` port: 
    ~~~
    sudo ufw allow XXXX
    ~~~
  - delete the previous rule for the default **10000** port:
    ~~~
    sudo ufw delete allow 10000
    ~~~
  - check the software is now reachable at the new port even without specifying the secure protocol: [server_ip:XXXX/](http://server_ip:XXXX/)

It's a safer choice to add [Two-Factor Authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication) to all web services that offer it. To do it with your new *Webmin* system configuration manager:
  - after opening the Webmin page, go to:
    ~~~
    Webmin > 
	  Webmin Configuration >
	    Two-Factor Authentication
    ~~~
    and choose your preferred provider, then click `Save`
  - go to:
    ~~~
    Webmin > 
	  Webmin Users >
	    Two-Factor Authentication
    ~~~
    click `Enroll For Two-Factor Authentication`, and follow the instructions. If you choose the *Google* authenticator, you now have to open the app, click the *plus sign*, and scan the barcode. From now on, you need *user*, *password* and Google app temporary *token* to enter the Webmin manager.

<a name="domain-name"/>

### Add Domain Name
How boring and annoying is to always remember an IP address? Enter [domain names](https://en.wikipedia.org/wiki/Domain_name)! For the current purpose, there's no point though in spending lots of money to own a fancy domain. Head to [Freenom World](http://www.freenom.org) to grab a free one! The catch here is that the choice of [Top-Level Domain](https://en.wikipedia.org/wiki/Top-level_domain) is restricted in the set: *tk*, *ga*, *ml*, *cf* and *gq*. 

Anyway, once you're on the *Freenom* landing page, look for a domain name you like, and click Get it Now and then move to the checkout page. Once there, click first `Use DNS` then the tab `Use your own DNS`, and in the texboxes labelled with `Nameserver` insert respectively:
`ns1.digitalocean.com` 
`ns2.digitalocean.com`
Go on and complete the sign up and checkout processes.

Once you own a domain, head to the [Digital Ocean](https://cloud.digitalocean.com/) website. 
  - from the main menu on the left click `Networking`, then enter the tab `Domains`. 
  - in the textbox *Enter domain* under `Add a domain` write your hostname, 
  - from the listbox on the right choose the project that include the server you want to apply the domain to
  - finally click `Add Domain`, and the domain should appear in the list below. Click on it!
    - in the `HOSTNAME` textbox enter `@`, 
    - in the `WILL DIRECT TO` choose the server
    - finally click `Create Record`
  - repeat the last steps entering `www`  in the `HOSTNAME` textbox 

Now you should simply wait from a few seconds to a few hours, depending on how fast the global sytem will update your changes, and if you head to [http://hostname]() you should see the same content as [http://ip_address]()


<a name="first-snapshot"/>

### Take Your First Snapshot
At this point in time, it'd be useful to save the current state of the machine, called **snapshot**, so that if something happens in the future it's always possible to revert back to the current situation in a few minutes with a click from the droplet dashboard. Moreover, we could also build other similar droplets but slighlty different, and use this snapshot as a starting point, instead of going back to the entire droplet creation.

To snapshot a droplet:
  - shut down the droplet: `sudo shutdown now`
  - login into your DO account, head for the droplet dashboard, and from the left menu click  **Snapshots**, enter a memorable name in the textbox, then click **Take Snapshot**
  - Once finished,  start the droplet again using the switch on the upper right

To restore a snapshot on the droplet it was created from:
  - 

In case you want to create a new droplet from a snapshot:
  - open the  [droplet **Create** page](https://cloud.digitalocean.com/droplets/new), 
  - select the  **Snapshots** tab
  - choose the snapshot you’d like to create the droplet from
  - fill out the rest of the choices on the **Create** page as desired, then click  **Create**


<br/>

:point_up_2:[Back to Index](#index)
<a name="r-stack"/>

## The *R* Stack

  <a name="install-r"/>

### Install core *R*

  - add the CRAN address to the list of trusted packages repositories
    ~~~
    echo -e "\n# CRAN REPOSITORY\ndeb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/\n" | sudo tee -a /etc/apt/sources.list
    ~~~

    Notice that the above command:
    - presumes that the installed OS version is **18.04 LST**. For previous versions of the *Ubuntu* distribution, change the word `bionic` with the correct adjective using [this list](https://en.wikipedia.org/wiki/Ubuntu_version_history) as a reference. In particular, the previous *16.04* LTS version is named `xenial`.
    - connects to  `cran.rstudio.com`, which is the the generic redirection service from *RStudio*, but it's also possible to switch to a static closer location (according to the chosen VM region, not the user's location!) using [this list](https://cran.r-project.org/mirrors.html).
  - download and add the *public key* of the CRAN maintainer [Michael Rutter](https://launchpad.net/~marutter/+archive/ubuntu/rrutter) to the apt keyring:
    ~~~
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
    ~~~
  - update the package management system: 
    ~~~
    sudo apt-get update
    ~~~
  - install R:
    ~~~
    sudo apt-get install -y r-base r-base-dev
    ~~~
 
To add the *public* repository we define earlier to the *R* environment:
  - open the general *R* configuration file for editing:
    ~~~
    sudo nano $(R RHOME)/etc/Renviron
    ~~~    
  - add the following line (or a similar one) at the end:
    ~~~
    PUB_PATH = '/usr/local/share/public'
    ~~~
  - to recall the path from inside any *R* script:
    ~~~
    Sys.getenv('PUB_PATH')
    ~~~
 
  <a name="install-rstudio-server"/>

### Install RStudio Server
  - install first the *deb* installer:
    ~~~
    sudo apt-get install gdebi-core
    ~~~   
  - create a new folder `software` in your *home* directory and move into it:
    ~~~
    cd
	mkdir software
	cd software
    ~~~
  - download the package:
    ~~~
    wget -O rstudio https://s3.amazonaws.com/rstudio-ide-build/server/bionic/amd64/rstudio-server-1.2.1268-amd64.deb
    ~~~
    Please note that the above command presumes Ubuntu *Xenial* 18.04 LTS, and the *preview* 64bit version at the time of writing. It's worth verifying the newest version visiting [this page](http://www.rstudio.com/products/rstudio/download/preview/), and in case substitute where needed. 
    Moreover, if you prefer to stay on the safer side and want to install the *stable* release, check instead [this page](https://www.rstudio.com/products/rstudio/download-server/) for the correct link of the newest version. 
  - install Rstudio Server:
    ~~~
    sudo gdebi rstudio
    ~~~
  - add a rule to the firewall to allow the default port Rstudio Server is listening to:
    ~~~
    sudo ufw allow 8787
    ~~~
  - head for [http://ip_address:8787/]() to check the software is up and running 
  - Change the default port **8787** to some random integer number `XXXX`:
    - open the configuration file for editing:
    `sudo nano /etc/rstudio/rserver.conf`
    - add an entry corresponding to the port you want RStudio to listen on:
      `www-port=XXXX`
    - restart the server:
      `sudo service rstudio-server restart`
    - add a rule to the firewall:
      `sudo ufw allow XXXX`
    - head for [http://ip_address:XXXX/]() to check the software is up and running.
    - delete the previous rule for the default **8787** port:
      `sudo ufw delete allow 8787`

You should now give yourself some time to play around the configurations, that you can find under the menu: `Tools > Global Options`. In particular, the first four options:
   - `General`: 
   - `Code`: 
   - `Appearance`: 
   - `Pane Layout`: 
   - `Git/SVN`: here you should simply check that *RStudio* has correctly recognized the [git](https://git-scm.com/) software, with the textbox marked as *Git executable* containing the path, usually `/usr/bin/git`. If you plan to use another version control software, feel free to make amendents.

  <a name="rstudio-using-projects-with-version-control"/>

#### Using Projects with Version Control
Before using *git*, you first have to inform *git* about the user, running from the terminal the following commands:
~~~
git config --global user.name "name here"
git config --global user.email "email here"
~~~
You can check that the above changes have been applied running the following command:
~~~
git config --list
~~~

If you now want to create a new project, which will be connected to a new repository, it's much simpler to create the repository first, and then add it as a new project in *RStudio* as a clone from that hosted version control project. When creating a new repository, remember to:
  - initialize the repository including the `README` file
  - add the desired `LICENCE`
  - specify the *R* `.gitignore` file
 
To add an existing repository to *RStudio*:
  - go and grab the url of  the repository: 
    - [GitHub](https://github.com): on the GH website, go to the main source of the repository, then copy the entire URL of the repository
    - [Bitbucket](https://bitbucket.org/): on the BB website, go to the main source of the repository, click `Clone` on the upper right, then copy the address in the textbox (drop the initial `git clone` text)
  - in *Rstudio Server*, click the Project dropdown list in the upper right (if this is the first time that RSudio runs, it probably reads as `Project: (None)`), then `New Project`, `Version Control`, and finally `Git`  (this is going to work for both GitHub and BitBucket)
  - copy the address in the textbox called  *Repository URL*, and fill as desired the other two text boxes
  - click `Create Project`. Notice that if the repository is private, you have to insert your username and  password to start cloning the repo. If you're using *GitHub*, it's a smart choice not to use the access password when dealing with RStudio projects, but create a [GitHub token](https://github.com/settings/tokens) to use instead of the password. You should limit the token scope only to *Access public repositories* or *Full control of private repositories*, depending on your needs. You could also generate a specific *RSA key* from the `Git/SVN` section of the `Tools` > `Global Options` menu, and add it to your GitHub account using the `New SSH key` button under the `SSH and GPG keys` section of the `Settings` menu.


  <a name="install-shiny-server"/>

### Install Shiny Server
  - install first the *shiny* package from inside *R*, using admin privileges. While we are at it, let's also install the *rmarkdown* package just to ensure that the *Shiny Server* landing page loads completely correct:
    - *Linux* terminal:
      `sudo su`
      `R`
    - *R* console:
      `install.packages(c('rmarkdown', 'shiny'))`
      `q()`
    - *Linux* terminal:
      `exit`
      
    Please, don't run *R* as `sudo R`, because in that case packages would be installed in the user local directory, and couldn't be loaded by the `shiny` user. 
    If you decided to use instead a subfolder into the *public* repository, remember to specify the optional argument as  in:
    `install.packages(c('rmarkdown', 'shiny'), lib.loc = file.path(Sys.getenv('PUB_PATH'), 'R-library'))`
    Hereafter, I'll drop the reference to the initial `sudo su / R` and final `q() / exit` commands, and will only warn to *enter $R$ as sudo*  
  - move into the software repository we created in the previous RStudio installation step:
    `cd ~/software`
  - download the package (check [here](https://www.rstudio.com/products/shiny/download-server/) for latest version):
    `wget -O shiny https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.9.923-amd64.deb`
  - install *Shiny Server*:
    `sudo gdebi shiny`
  - add a rule to the firewall to allow the default port *Shiny Server* is listening to:
    `sudo ufw allow 3838`
  - head for [http://ip_address:3838/]() to check the software is up and running. Notice that the error in the second app is normal, as it needs the *RMarkdown* package to work, which is  not be installed yet.
  - Change the default **3838** port to the standard **80**, so that the user can access the server without an explicit port in the URL (but be sure that there are no other web servers listening to that port):
    - open the *Shiny Server* configuration file for editing:
    `sudo nano /etc/shiny-server/shiny-server.conf`
    - change the port from **3838** to **80**:
      `listen 80`
    - restart the server:
      `sudo service shiny-server restart`
    - add a rule to the firewall:
      `sudo ufw allow 80`
    - head for [http://ip_address/]() to check the software is up and running.
    - delete the previous rule for the default **3838** port:
      `sudo ufw delete allow 3838`
  - add the *shiny* user to the *public* group, so that it can share the *public* repository as well:
    ~~~
    sudo usermod -aG public shiny
    ~~~
  - add permission to the shiny server subdirectory to every user in the public group (you need to reboot the system for the changes to take effect):
    ~~~
    cd /srv/shiny-server
    sudo chown -R usrname:public .
    sudo chmod g+w .
    sudo chmod g+s .
    ~~~
  - to copy apps from any location in any *public* user's home folder to the appropriate shiny server subfolder directory:
    ~~~
    mkdir /srv/shiny-server/<APP-NAME>
    cp -R /home/usrname/<APP-PATH>/* /srv/shiny-server/<APP-NAME>/
    ~~~
      
 
   <a name="testing-the-r-stack"/>

### Testing the R stack
If you're eager to watch some beauty about what you can do with $Shiny$ in a few lines of code, spend the next hour ... Otherwise, jump to the next section and finish installing all the R goodies.

There is a repository on the [WeR GiHub](https://github.com/WeR-stats) website called [shinyapps](https://github.com/WeR-stats/shinyapps). At the time of writing these notes, there's at least one app (subfolder) called [uk_petitions](https://github.com/WeR-stats/shinyapps/tree/master/uk_petitions) that lets you easily download all data regarding any of  the [petitions](https://petition.parliament.uk/petitions) created under the current UK government, and then draw a [choropleth map](https://gisgeography.com/choropleth-maps-data-classification/) of the provenance of the corresponding subscribers using the [leaflet](http://rstudio.github.io/leaflet/) package. 

If you still haven't installed any package, besides *shiny* and *rmarkdown*, let's install the ones needed for the app to run correctly. We first need to install some system dependencies though.
~~~
sudo apt-get install curl libssl-dev libcurl4-gnutls-dev
sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable 
sudo apt-get update 
sudo apt-get install gdal-bin libgdal-dev libgeos++-dev libudunits2-dev libv8-dev libjq-dev libcairo2-dev libxt-dev
~~~
You can now enter $R$ as *sudo*, and actually install the required packages:
~~~
install.packages('devtools')
library(devtools)
pkgs <- c(
  'Cairo', 'classInt', 'colourpicker', 'data.table', 'DT', 'jsonlite', 'leaflet', 'leaflet.extras', 
  'RColorBrewer', 'readxl', 'rgdal', 'rgeos', 'shinyjs', 'shinyWidgets'
)
install.packages(pkgs, dep = TRUE)
~~~
The above is going to take some time, possibly half an hour, so go and grab a cup of cofee to keep you happy. 

When finished, let's create a directory for the app in the *Shiny Server* repository:
~~~
mkdir /srv/shiny-server/uk_petitions
~~~

To copy the app code into the above folder, we first create a new project in *RStudio Server*, cloning the repository directly in the user *home* folder. 

Once the repo has been pulled on the server, run the following simple command to actually copy the code:
~~~
cp ~/shinyapps/uk_petitions/* /srv/shiny-server/uk_petitions/
~~~

You can now open a browser and head to [http://ip_address/uk_petitions]() to see the app up and running!

  <a name="install-linux-dependencies-for-r-packages"/>

### Install Linux dependencies for R packages

  - **devtools**:
    ~~~
    sudo apt-get install curl libssl-dev libcurl4-gnutls-dev
    ~~~
  - **xml2**:
    ~~~
    sudo apt-get install libxml2-dev
    ~~~
  - **RMySQL**:
    ~~~
    sudo apt-get install -y libmysqlclient-dev
    ~~~
  - **mongolite**:
    ~~~
    sudo apt-get install libsasl2-dev
    ~~~
  - **redux**:
    ~~~
    sudo apt-get install -y libhiredis-dev
    ~~~
  - **rgdal**:
    ~~~
    sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
	sudo apt-get update 
	sudo apt-get install -y gdal-bin libgdal-dev
    ~~~
    But check [here](https://launchpad.net/~ubuntugis/+archive/ubuntu/ppa)  for the availability of the *stable*  release of the *UbuntuGIS* suite of spatial packages  for Ubuntu *Bionic* **18.04**, then substitute the first line in the above group with the following:
    ~~~
    sudo add-apt-repository ppa:ubuntugis/ppa
    ~~~
  - **rgeos**  (must be installed after previous dependencies):
    ~~~
    sudo apt-get install -y libgeos++-dev
    ~~~
  - **sf** (must be installed after previous dependencies):
    ~~~
    sudo apt-get install -y libudunits2-dev
    ~~~
  - **geojsonio** / **tmaptools**  (must be installed *after* previous dependencies):
    ~~~
    sudo apt-get install -y libprotobuf-dev protobuf-compiler libv8-dev libjq-dev
    ~~~
  - **gdtools** / **tmaps** / **mapview** (must be installed *after* previous dependencies):
    ~~~
    sudo apt-get install -y libcairo2-dev
    ~~~
  - **Cairo** (must be installed *after* previous dependencies):
    ~~~
    sudo apt-get install -y libxt-dev
    ~~~
  - **RcppGSL**:
    ~~~
    sudo apt-get install -y libgsl0-dev
    ~~~
  - **gmp**:
    ~~~
    sudo apt-get install -y libgmp3-dev
    ~~~
  - **rgl**:
    ~~~
    sudo apt-get install -y libcgal-dev libglu1-mesa-dev
    ~~~
  - **Rglpk**:
    ~~~
    sudo apt-get install -y libglpk-dev
    ~~~
  - **magick**:
    ~~~
    sudo apt-get install -y libmagick++-dev
    ~~~
  - **nloptr**:
    ~~~
    sudo apt-get install -y libnlopt-dev
    ~~~
  - **gganimate** (only if you need the *gifski* package to save animation, notice that altogether it's quite a lot of space):
    ~~~
    sudo apt-get install -y cargo
    ~~~
  - **rJava**:
    ~~~
    sudo apt-get install -y default-jdk
    sudo add-apt-repository ppa:marutter/c2d4u3.5
    sudo apt-get update
    sudo R CMD javareconf
    ~~~

  <a name="install-r-packages"/>

### Install R packages
One note of caution is how packages are installed, if you don't want to end up with lots of duplications, different versions and incompatibility
To overcome this situation, choose one of the following alternatives:
  - packages should be installed as **root**, (`sudo su` / `R`) so that they always end up in the system library that can be read by any user by default
  - define a common shared location for the packages to be stored, then include it explicitly in *any* call to the installation command:
    `install.packages('pkgname', lib.loc = '/path/to/library')`

Having said that, to install a single package as *root* there are two main ways:
  - a single linux line:
    `sudo su - -c "R -e \"install.packages('pkgname', repos='http://cran.rstudio.com/')\""`
  - a *normal* *R* installation, after having started *R* as *root*:
    - *Linux* terminal:
      `sudo su`
      `R`
    - *R* console:
      `install.packages('pkgname', repos='http://cran.rstudio.com/')`
      `q()`
    - *Linux* terminal:
      `exit`
    
When dealing with many packages, though, both the above approaches are tedious, and most important very prone to errors. A more efficient and safer way to install multiple packages as *super user* in the system library, is the following:
  - first save a plain text file as `r_packages.lst` in some directory with the list of packages to be installed, each on its own line
  - have your password as the only entry in a plain text file called as `pwd` and saved in the root of your home folder
  - save the following commands as text file named `install_r_packages.sh` in the same directory as the previous list `r_packages.lst`:
    ```
    #!/bin/sh
    while read PKG
    do
      cat $PWD/pwd | sudo -S su - -c "R -e \"install.packages('$PKG'')\""
    done < `dirname $0`/r_packages.lst
    ```
  - change executable
    `sudo chmod +x ~/path/to/install_r_packages.sh`

You can obviously rename the above files `r_packages.lst` and `install_r_packages.sh` in any other way, changing accordingly the stated commands, but the extension `.sh` must be preserved because that's the way Linux recognize executable files

While efficient, the above linux script lacks of a *general* summary output at the end of the script. Because it processes one package at a time, it gives back only a partial log for each package, that gets easily lost in the plethora of *R* output that can be generated. Moreover, you can't avoid to install packages that are already installed. A most sensible way would be the following: 
  - enter *R* as *super user* as usual:
    ~~~
    sudo su
    R
    ~~~
  - run the below commands one by one:
    ~~~
    pkgs <- readLines(file('/home/usrname/scripts/r_packages/r_packages.lst'))
    pkgs <- pkgs[!sapply(pkgs, require, char = TRUE)]
    install.packages(pkgs)
    ~~~
    or save them in a text file `install_r_packages.R`, and run it using the `source` command:
    ~~~
    source('/home/usrname/path/to/install_r_packages.R')
    ~~~
    In the above calls to external files, do not use the standard $UNIX$ *tilde* character `~` to denote the user *home* directory, because $R$ does not expand it and will therefore throw an error. Alternatively, you can `cd` into the `path` before the $R$ call, and then call the script file directly without adding any path.
  - when the installation comes to an end, read carefully if there are any warnings about packages having *non-zero exit status*. If that's the case, scroll back until you find the errors in the log, usually in red bold ink, and act accordingly. It is often a lack of Linux libraries that needs to be installed in the *Ubuntu* system, before installing the packages. 
If you can't get over the error(s), just *google* the entire feedback. Often, limiting the timespan of the search in the previous year only is a good move, as the same error could be related to different past situations.
  - at the end, exit $R$:
      `q()`
  - and then exit *sudo*:
    `exit`

If you try to install lots of packages in the same session, you'll probably get the following message:
`maximal number of DLLs reached...`
In that case, you need to tell $R$ that it can open more than the default 100 libraries in a single session:
  - open the general $R$ configuration file for editing:
    `sudo nano $(R RHOME)/etc/Renviron`
  - add the following string at the end:
    `R_MAX_NUM_DLLS = 1000`
    Notice that 1,000 is the biggest number that you can set the variable to without $R$ complaining.
    
Some packages (**iheatmapr**) need also dependencies from the [Bioconductor]('https://bioconductor.org/) repository, that have to be installed with their own package manager:
~~~
install.packages('BiocManager')
BiocManager::install('S4Vectors')
BiocManager::install('graph')
~~~
Notice that like any other $R$ package, the above should be installed as `sudo` as well.

Finally, take also note also that starting with version 3.5 some $R$ internals have changed so much that all packages need to be rebuilt to work properly, and some of them have even been removed from $CRAN$ because of issues that have to be fixed to pass all due checks. 

On the [WeR GitHub](https://github.com/WeR-stats) website you can find two lists of suggested packages, classified hereafter by their possible application in Data Science:
  - `r_packages.lst` relates to packages found directly on the *CRAN* website
  - `r_packages_gh.lst` lists packages that are still to be released on *CRAN*, and possibly never be, and have therefore to be installed using the functionalities provided by the *devtools* package
Feel free to delete any package from the list, or add anyone else you need for your job  

To install the packages in the lists: 
  - cd into `~/scripts/r_packages/`
  - download from the GitHub repository the lists of packages, and the R source files:
    ~~~
    wget -O r_packages.R https://raw.githubusercontent.com/WeR-stats/workshops/master/setup_cloud_machine_data_science/r_packages.R
    wget -O r_packages.lst https://raw.githubusercontent.com/WeR-stats/workshops/master/setup_cloud_machine_data_science/r_packages.lst
    wget -O r_packages_gh.R https://raw.githubusercontent.com/WeR-stats/workshops/master/setup_cloud_machine_data_science/r_packages_gh.R
    wget -O r_packages_gh.lst https://raw.githubusercontent.com/WeR-stats/workshops/master/setup_cloud_machine_data_science/r_packages_gh.lst
    ~~~
  - enter $R$ as *root*, then `source('r_packages.R')`. It'll take probably 3 hours to end. Don't look at the warnings (for now), but instead exit $R$ *without saving the  session*, re-enter and run again `source('r_packages.R')`. At the end, that's the right time to look at all the warnings that $R$ has probably thrown out.

You should try to keep the SSH connection open during the whole installation, to avoid the scripts to break. If anything happens and the script stops suddenly, before running the script again you should look into the library repository and delete, if present, any folder that starts with `00LOCK-`, together with the folders recalled by them. For example, if the script breaks while installing the `xxx` package, there will exist a (temporary) folder called `00-LOCK-xxx` and the actual package folder called `xxx`. Notice that you have to be admin to succeed in deleting the folders.
  - check first what to delete:
    ~~~
    sudo su
    cd /usr/local/lib/R/site-library/
    ls 00* -l
    ~~~
  - delete the folders:
    ~~~
    rm -rf 00LOCK-*
    rm -rf xxx
    ~~~
  - exit sudo:
    ~~~
    exit
    ~~~

It's important to note that if you want to install ALL the packages in the above CRAN list, you do need to up the RAM of your machine to a minimum of 4GB, at least for the time necessary to install all the packages and their dependencies (you actually need even 8GB if you want to install also the [prophet](https://github.com/facebook/prophet) package, that requires the [RStan](https://mc-stan.org/users/interfaces/rstan) probabilistic language).
To accomplish the resizing:
  - turn off your machine: `sudo shutdown now`
  - head for the dashboard of the machine, then click `Resize` from the left menu
  - you shouldn't need to change also the size of your disk, so you can check `CPU and RAM only`, then choose the **4GB/2vCPUs** size (or the **8GB/4vCPUs** depending on your needs)
  - click the big button `Resize` at the bottom, and wait for the task to finish
  - turn the machine back on acting on the switch on the upper right
  - when you're done with the installation, you can resize down the machine to your desired specs using the same procedure

The [included file](r_packages.md) contains a brief description of most of the packages contained in the above lists, divided by their ... in the Data Science Workflow. A star after the name indicates that the package is (still) not available on CRAN, and must be installed from its GitHub repository using the *devtools* package. When possible, some links to available resources have also been included.
  
<br/>

:point_up_2:[Back to Index](#index)
<a name="nginx"/>

## Ngnix 
[Nginx](https://www.nginx.com/) is a free, open-source, high-performance HTTP server software, that also works as a proxy, load balancer, and Reverse Proxy. It's been developed with the clear intention to run on small resources, yet with the capacity to handle a large volume of concurrent connections. For these reasons, it is a great alternative to the more commonly used [Apache](https://httpd.apache.org/) web server.

<a name="nginx-install"/>

### Install Nginx
  - Just in case Apache is already installed, stop the server and remove the package:
    `sudo systemctl stop apache2`
    `sudo apt-get remove -y apache*`
 - if you changed the *Shiny Server* listening port to **80**, edit its configuration file again to change the port to whatever else, as **80** has to be dedicated to the web server. 
 - if you haven't done it before, open port **80** for unencrypted traffic:
    `sudo ufw allow 80`
  - now we're ready to install the *nginx* server:
    `sudo apt-get install -y nginx`
  - after the completion of the installation process, the Nginx web server should start and run automatically. To ensure that the service is actually up and running, run the following command:
    `sudo systemctl status nginx`

To test that the service is actually working, enter the **server_ip** or **hostname** directly into the browser's address bar, and you should see the default Nginx landing page.
    
  <a name="pretty-url"/>
  
### Install script language
We're going to install [PHP-FPM](https://php-fpm.org/), a FastCGI implementation alternative to the more common [PHP](http://php.net/) usually installed besides the Apache Web Server

  - install php, *plus* its mysql extensions (we're going to need the latter later):
    ~~~
    sudo apt install -y php-fpm php-mysql
    ~~~
  - open the *ngnix* website configuration file for editing:
    ~~~
    sudo nano /etc/nginx/sites-available/default
    ~~~
  - add `index.php` to the following line:
    ~~~
    index index.html index.htm index.nginx-debian.html;
    ~~~
  - uncomment as shown the following block of lines:
    ~~~
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        # With php-cgi (or other tcp sockets):
        # fastcgi_pass 127.0.0.1:9000;
    }
    ~~~
    and:
    ~~~
    location ~ /\.ht {
        deny all;
    }
    ~~~
  - check the actual version of the php installation, listing the files in `/var/run/php/`, and then modify the above code accordingly. At the time of writing the version it is `7.2`, not 7.0
  - close the file saving the modifications
  - check the new configuration is correct:
    ~~~
    sudo nginx -t
    ~~~
  - restart the web server:
    ~~~
    sudo systemctl nginx reload
    ~~~
  - to test if the *php* interpreter  is actually working, create a new file in the webroot directory:
    ~~~
    sudo nano /var/www/html/info.php    
    ~~~
    with content:
    ~~~
    <?php phpinfo();
    ~~~
    After opening the page you should be greeting with a horrible php welcome page listing lots of stuff, and on top the *php* and *Ubuntu* versions running on the system:



### Dedicated URLs for RStudio Server and Shiny Server
  - open the *Nginx* configuration file for editing
    ~~~
    sudo nano /etc/nginx/sites-enabled/default
    ~~~
  - add the following lines for *Shiny Server*, substituting `XXXX` with the correct port:
    ~~~
    location /shiny/ {
        proxy_pass http://127.0.0.1:XXXX/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        rewrite ^(/shiny/[^/]+)$ $1/ permanent;
    }
    ~~~
  - add the following lines for *RStudio Server*, substituting `XXXX` with the correct port:
    ~~~
    location /rstudio/ {
        proxy_pass http://127.0.0.1:XXXX/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    ~~~
  - find the existing line:
    `server_name _;`
    and replace the underscore **_** with the domain name	of your choice
  - quit the editor, saving the file
  - verify that the syntax of the above configuration editings is actually correct:
    `sudo nginx -t`
	If you get any errors, reopen the file and check for typos, then test it again, until you get a succesful feedback.
  - once the configuration's syntax is correct, reload *Nginx* to load the new configuration:
    `sudo systemctl reload nginx`
  - you should now see the same results from the url http://hostname/shiny as from http://hostname:XXXX, http://hostname/rstudio as from http://hostname:XXXX. 
  - once you're happy with the changes, you can delete the rules from the firewall related to Shiny and RStudio.

  <a name="ssl-certificate"/>

### Add SSL Certificate for encrypted *https* connection
We will use [Let's Encrypt](https://letsencrypt.org/) to obtain a free SSL certificate.
  - open port **443** to allow SSL/TSL encrypted traffic through the firewall:
    ~~~
    sudo ufw allow 443
    ~~~
  - install the Certbot software:
    ~~~
    sudo add-apt-repository ppa:certbot/certbot
	sudo apt-get update
	sudo apt-get install -y python-certbot-nginx
    ~~~
  - ask for the certificate:
    ~~~
	sudo certbot --nginx -d hostname.tld -d www.hostname.tld
    ~~~
  - answer the few questions and wait for the *challenge* to be positively completed. I suggest you ask for redirection (answer number 2)
  - check finally that http://hostname/shiny and http://hostname/rstudio redirect correctly to https://hostname/shiny and respectively https://hostname/rstudio

Note that every certificate has an expiry date:
  - to obtain a new or tweaked version of any certificate in the future, simply run the above command again adding the `--certonly` option
  - to non-interactively renew *all* of your certificates, run `certbot renew`.


  <a name="shiny-auth"/>

### Add Authentication to Shiny Server apps
As we've seen before, from the server's point of view a shiny app is nothing more than a subfolder in the server base folder, which by default is `/srv/shiny-server`. Using *Nginx* capabilities, it's easy to add a *basic* form of authentication to any shiny app.
  -  we need a password file where users that should be able to log in are listed along with their passwords in encrypted form. To accomplish that, we'll use the Apache's `htpasswd` command from the `utils` library, that we need to install:
    ~~~
    sudo apt-get install -y apache2-utils
    ~~~
 
 **_TBD_**

  <a name="nginx-configuration"/>

### Nginx Configuration Structure
The following are the location and names of the configuration and log files:
  - `/etc/nginx` the Nginx parent directory that contain all the server configuration file
  - `/etc/nginx/nginx.conf` the main configuration file of Nginx
  - `/etc/nginx/sites-available/` you can store the *server blocks* in this directory. It has the configuration files which will not be used until they are linked with sites-enable directory.
  - `/etc/nginx/sites-enabled/` This directory stores the "server blocks". They link to the configuration file in the sites-available directory.
    - `/etc/nginx/snippets/` Here the configuration fragments are stored and they can be used anywhere in the Nginx Configuration. If you are using specific configuration segments repeatedly, then they can be added to this directory.
  - `/var/log/nginx/` the Nginx parent directory for the server log files
  - `/var/log/nginx/access.log` stores all the entry requests to the web server (it has to be configured to do that).
  - `/var/log/nginx/error.log` Nginx errors are recorded in this file
  - `/var/www/html/` the default directory for the content of the website(s)

The default Nginx installation will have only one default server block, enabled with a document root set to:
`/var/www/html/`
It is possible to add as many blocks as desired as follows:
  - create a new domain document root:
    `sudo mkdir -p /var/www/newdomain.com`
  - in the above folder, create a basic welcome web page:
    `sudo nano /var/www/newdomain.com/index.html`
	like the following:
	```
	<html>
  		<head>
      		<title>Welcome to the "newdomain.com>" nginx webserver!</title>
  		</head>
  		<body bgcolor="white" text="black">
    		<center><h1>newdomain.com is working!</h1></center>
		</body>
	</html>
	```
  - create a new server block:
    `sudo nano /etc/nginx/sites-available/newdomain.com.conf`
	and add the following content:
	```
	server {
		listen 80;
		listen [::]:80;
		server_name newdomain.com www.newdomain.com;
		root /var/www/newdomain.com;

		index index.html;

		location / {
			try_files $uri $uri/ =404;
		}
	}
	```
  - Activate the server block by creating a symbolic link in the list of available websites: 
    `sudo ln -s /etc/nginx/sites-available/newdomain.com.conf /etc/nginx/sites-enabled/newdomain.com.conf`
  - eventually, test that the above configuration is actually correct:
    `sudo nginx -t`
  - restart the nginx web server:
    `sudo systemctl restart nginx`
  - check with the browser that `newdomain.com` is working as desired.


<br/>

:point_up_2:[Back to Index](#index)
<a name="the-python-data-science-stack"/>
## The Python Data Science Stack


  <a name="install-python"/>

### Install Python 
Although Python is often automatically installed on Ubuntu, take a moment to confirm that version **3.4+** is already installed on the system, by issuing the following command: 
`python3 -V`

In a similar way, the `pip` package manager is usually installed on Ubuntu, but it is related to previous version 2.7. Take a moment though to confirm if version 8.1+ is installed, by issuing the command:
`pip3 -V`

In any case, run the following commands to install both of them:
~~~
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-pip
sudo -H pip3 install --upgrade pip
~~~

  <a name="install-the-data-science-stack"/>

### Install the Data Science Stack
We are now in a position to install all the top packages needed for a decent data science stack:
  - [Pandas](http://pandas.pydata.org/) for data manipulation and wrangling
  - [NumPy](http://www.numpy.org/), [SciPy](http://www.scipy.org/scipylib/index.html), and [SymPy](http://www.sympy.org/) for numerical computation
  - [matplotlib](http://matplotlib.org/), [seaborn](http://seaborn.pydata.org/), [Bokeh](https://bokeh.pydata.org/), [plotly](https://plot.ly/python/), and [folium](http://python-visualization.github.io/folium/) for data visualization
  - [NetworkX](https://github.com/networkx/networkx), [pydot](https://github.com/pydot/pydot) 
  - [Statsmodels](http://statsmodels.sourceforge.net/) for statistical inference and modeling
  - [scikit-learn](http://scikit-learn.org/) for generic machine learning
  - [XGBoost](http://xgboost.readthedocs.io/en/latest/),  [LightGBM](http://lightgbm.readthedocs.io/en/latest/Python-Intro.html) and [CatBoost](https://github.com/catboost/catboost) provide highly optimized, scalable and fast implementations of [gradient boosting](https://en.wikipedia.org/wiki/Gradient_boosting)
  - [Eli5](https://eli5.readthedocs.io/en/latest/) 
  - [Keras](https://github.com/keras-team/keras), [TensorFlow](https://github.com/tensorflow/tensorflow), [Theano](https://github.com/Theano/Theano), and [Chainer](https://chainer.org/) for Deep Learning
  - [NLTK](http://www.nltk.org/), [Gensim](https://radimrehurek.com/gensim/) and [spaCy](https://spacy.io/) for Natural Language Processing and Topic Modeling
  - [CNTK](https://www.microsoft.com/en-us/cognitive-toolkit/) 
  - [pytorch]()
  - [Apache MXNet](https://github.com/apache/incubator-mxnet)
  - [shogun](http://www.shogun-toolbox.org/) a machine learning toolbox with a focus on Support Vector Machines (SVM)
  - [Caffe]() 
  - [Beautiful Soup](https://launchpad.net/beautifulsoup), [scrapy](https://scrapy.org/) and [pattern](https://github.com/clips/pattern) for data wrangling, web scraping and mining
  - [Cython](http://cython.org/), [Dask](https://dask.pydata.org/) and [Numba](https://numba.pydata.org/) for high performace and distributed computiing
  - [pytest](https://pytest.org/) for quality assurance
  - [IPython](https://ipython.org/) and [Jupyter](jupyter.org/) Notebook, for interactive computing in multiple programming languages.

Notice that some libraries *seaborn* requires the following additional library to be installed beforehand: 
  - aaa
    ~~~
    ~~~
sudo apt-get install -y python3-tk
    ~~~
  - the Microsoft Cognitive Framework *CNTK* requires the *OpenMPI* library:
    ~~~
    sudo apt-get install -y openmpi-bin
    ~~~
  - if using *Theano* or *Keras* it's better to also install the *OpenBLAS* libraries to improve performance:
    ~~~
    sudo apt-get install -y libopenblas-dev
    ~~~
  - ,,,
    ~~~
    pip3 install --user https://download.pytorch.org/whl/cpu/torch-1.0.0-cp36-cp36m-linux_x86_64.whl
    ~~~
    If any error shows up, you should first ensure your version of Python is 3.6.x, as indicated in the filename. If your version of Python is different, try first to adjust the filename according to the version number. 

It's possible to install the above packages one by one when needed, but you can also install all of themat once, as follows:
  - move into the *scripts* folder, and download from the [WeR GitHub repository](https://github.com/WeR-stats/workshops/tree/master/setup_cloud_machine_data_science) the lists of packages:
    ~~~
    cd ~/scripts
    wget -O python_libraries.lst https://raw.githubusercontent.com/WeR-stats/workshops/master/setup_cloud_machine_data_science/python_libraries.lst
    ~~~
  - run the following command:
    ~~~
    python3 -m pip install --user -r python_libraries.lst
    ~~~

Some packages needs to be installed separately:
  - **shogun**:
    ~~~
    sudo add-apt-repository ppa:shogun-toolbox/stable
    sudo apt-get update
    sudo apt-get install libshogun18
    sudo apt-get install python-shogun
    ~~~
  - the installation of the **Caffe** framework is a bit more involved as it requires to be built from source.
    ~~~
    ~~~

  <a name="install-jupyter-notebook"/>

### Install Jupyter Notebook
[IPython](https://ipython.org/) is an interactive command-line interface to Python. [Jupyter](https://jupyter.org/) offers an interactive web interface to many languages, including IPython and R.
First, install Ipython:
`sudo apt-get -y install ipython ipython-notebook`
Next, move on to installing Jupyter Notebook:
`sudo -H pip3 install jupyter`

Once installed correctly, to run it, execute the following command:
`jupyter notebook --no-browser --port XXXX`
By default, a notebook server runs locally at `127.0.0.1:8888`, and is accessible only from *localhost*. Hence to connect to the Jupyter Notebook we need to use **SSH tunneling**.

  <a name="tunneling-windows"/>

#### Create an SSH Tunnel in Windows using MobaXTerm 
  - From the `Buttons` bar click *Tunneling*, or from the  `Tools` menu choose *MobaSSHTunnel* then *New SSH tunnel*.
  - BE sure that *local port forwarding* is the chosen radio button in the upper group 
  - Write down the following information anti-clockwise from the upper left:
    - **Forwarded Port** = whatever port number YYYY you want to connect from your *localhost*, but be careful not to interfere with other services already running on your system
    - **SSH Server** = the IP address of the droplet
    - **SshUsername** = the name of the user that started *Jupyter*
    - **SSH port** = 22 or the alternative port if the SSH service has been configured differently
    - **Remote port** = 8888 or a different port if *Jupyter* has been so instructed in the command line
    - **Remote server** = *localhost*
  - Click `Save`
  - Give a name to the new entry
  - Click the `play` icon
  - Open your browser and connect to [localhost:YYYY](localhost:YYYY)


<br/>

:point_up_2:[Back to Index](#index)
<a name="storage-engines"/>
## Storage engines

  <a name="mysql"/>

### MySQL Server (relational database)

  - install main program:
    ~~~
    sudo apt-get -y install mysql-server
    ~~~
  - secure root login:
    ~~~
    sudo mysql_secure_installation
    ~~~
    skip the first question, then insert a strong new password for *root*, and finally answer **Yes** to all the remaining questions.
  - login as root, when asked enter the password you choose in the step before:
    ~~~
    sudo mysql -u root -p
    ~~~
  - create at least two new *agnostic* users to be used in:
    - scripts, with all privileges and working only on localhost:
      ~~~
      CREATE USER 'devs'@'localhost' IDENTIFIED BY 'pwd';
      GRANT ALL PRIVILEGES ON *.* TO 'devs'@'localhost';
      FLUSH PRIVILEGES;
      ~~~
    - apps, with a read-only privilege, possibly working from remote if you decide to build separate machines for *data storage* and *production*:
      ~~~
      CREATE USER 'shiny'@'localhost' IDENTIFIED BY 'pwd';
      GRANT SELECT ON *.* TO 'shiny'@'localhost';
      CREATE USER 'shiny'@'%' IDENTIFIED BY 'pwd';
      GRANT SELECT ON *.* TO 'shiny'@'%';
      FLUSH PRIVILEGES;
      ~~~
      Notice that it is really necessary for the *shiny* user to have both the *localhost* and the *%* statements to be able to connect from *anywhere* as *shiny*. Moreover, if it is known beforehand the exact ip where the shiny user is going to query from, then that ip should be included in the above statements, instead of the percent sign.

    In a similar way, it is possible to create additional *personal* users. See [here](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html) for a list of all possible specifications for the privileges.
    
  -  `exit` MySQL server

We're now in a position to add credentials in a way that avoid people to see password in clear in scripts: 
  - open the MySQL configuration file for editing:
    `sudo nano /etc/mysql/my.cnf`
  - scroll at the end and add the desired credential(s):
    ```
    [groupname]
    host = ip_address
    user = usrname
    password = 'password'
    database = dbname
    ```
  - restart the server:
    `sudo service mysql restart`

  <a name="tweak-mysql"/>

#### Tweak MySQL Server

~~~
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
default-storage-engine=MYISAM
~~~


  <a name="rmysql"/>

#### RMySQL

  - `library(RMySQL)` 
  - `conn <- dbConnect(MySQL(), host = 'hostname', username = 'usrname', password = 'pwd', dbname = 'dbname')` 
  - `conn <- dbConnect(MySQL(), group = 'grpname')` 
  - `dbGetQuery(conn, 'strSQL')` 
  - `dbReadTable(conn, 'tblname')`  
  - `dbSendQuery(conn, 'strSQL')` 
  - `dbWriteTable(conn, 'tblname', dfname, row.names = FALSE, append = TRUE)` 
  - `dbRemoveTable(con, 'tblname)` 
  - `dbDisconnect(conn) ` 


  <a name="dbninja"/>

#### Install DbNinja, a web client to MySQL Server
This step requires to have a Web server, like *Apache* or *Nginx*, and a *php* processor already installed on the system. We already have , so we have to install php.
  - download the client software:
    ```
    cd ~/software
    wget http://dbninja.com/download/dbninja.tar.gz
    ```
  - create subdirectory in web root (not necessarily the one chosen below):
    ```
    sudo mkdir /var/www/html/sql
    ```
  - copy content of zip file in the above directory:
    ```
    sudo tar -xvzf dbninja.tar.gz -C /var/www/html/sql --strip-components=1
    ```
  - open the homepage of your new *DbNinja* *MySQL* client at [http://ip_address/sql](), and agree to T&C
  - check and insert the filename as requested:
    ```
     sudo ls /var/www/html/sql/_users/
    ```
  - insert a strong password
  - login as *admin* using the previous password (this is not either the *MySQL* or the *Ubuntu* credentials) 
  - open the top left menu *DbNinja*, then *Settings*, then the *Settings* tab, and check `Hide the ...`. Click *Save*.
  - to add any *MySQL* Server, open the top left menu *DbNinja*, and under *MySQL Hosts* tab click *Add Host* , complete with the desired *MySQL* username (don't save the password for better security), and finally click *Save*


  <a name="mongodb"/>

### MongoDB (document database)

  - add the repository key to the apt keychain:
    `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5`
  -  add the repository to the list of  `apt`  sources:
    `echo -e "\n# MONGODB\ndeb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse\n" | sudo tee -a /etc/apt/sources.list`
  - update as usual the repository information:
    `sudo apt-get update`
  - install the software:
    `sudo apt-get install mongodb-org`
  - to connect to the server:
    ` mongo --host 127.0.0.1:27017`
 If the feedback is negative, run the following command
 `sudo rm /var/lib/mongodb/mongod.lock`
then restart the server:
`sudo service mongod restart`
before trying to connect again.


  <a name="neo4j"/>

### Neo4j
[Neo4j](https://neo4j.com/) is an extremely popular [graph database](https://en.wikipedia.org/wiki/Graph_database) used to store and query connected data. Rather than having *foreign keys* and *select* statements, it uses *edges* and graph *traversals* to query the data. This method of querying data is extremely powerful in any situation where data is best represented as items that have relationships with other items in the dataset, such as social networks, biology, and chemistry.

Neo4j is implemented in Java, so you’ll need to have the Java Runtime Environment (JRE) installed. You can check it using the command: `java -version`. If the feedback is negative:
  -  add the java repository to the list of  `apt`  sources:
    `sudo add-apt-repository ppa:webupd8team/java`
  - update the repository information:
    `sudo apt-get update`
  - install the software:
    `sudo apt-get install oracle-java8-installer`

Once you've installed java, you can proceed with Neo4j:
  - add the repository key to the apt keychain:
    `wget --no-check-certificate -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -`
  -  add the repository to the list of  `apt`  sources:
    `echo -e "\n# NEO4J\ndeb http://debian.neo4j.org/repo stable/\n" | sudo tee -a /etc/apt/sources.list`
  - update as usual the repository information:
    `sudo apt-get update`
  - install the software:
    `sudo apt-get install neo4j`

You can now head to [http://ip_address:7474/browser/]() to access the *neo4j* dashboard, using the default username and password `neo4j` and `neo4j`. You will be prompted to set a new password. If you find yourself in trouble logging in the first time, try to delete the file `/var/lib/neo4j/data/dbms/auth` and restart the server before trying to access again.

If the browser refuse to connect:
  - open the configuration file for editing:
    `sudo nano /etc/neo4j/neo4j.conf`
  - uncomment the following line:
    `dbms.connectors.default_listen_address=0.0.0.0`
  - restart the service:
    `sudo service neo4j restart`

    
  <a name="redis"/>
 
### Redis

[Redis](https://redis.io/) ([GitHub repo](https://github.com/antirez/redis)) is a distributed in-memory [key-value](https://en.wikipedia.org/wiki/Key-value_database) storage engine that persists on disk, and supports different kinds of abstract data structures. You can walk through the most important features of the *Redis* engine at the [Try Redis](http://try.redis.io/) demonstration website.

We need to install *Redis* as non-root user, and to accomplish that task we must build and install the package from source.
 - install first the *build* and *test* dependencies:
    ~~~
    sudo apt update
    sudo apt install build-essential tcl
    ~~~
  - create a folder in the `software` directory we've already created above, and move into it:
    ~~~
    cd ~/software
    mkdir redis
    cd redis
    ~~~
  - download the latest stable copy of the source package:
    ~~~
    curl -O http://download.redis.io/redis-stable.tar.gz
    ~~~
  - extract the content from the downloaded archive file:
    ~~~
    tar xzvf redis-stable.tar.gz
    ~~~
  - move into the *redis-stable* source folder for the *Redis* server software, that should have been created from the *tar* unpacking:
    ~~~
    cd redis-stable
    ~~~
  - compile a few needed  dependencies:
    ~~~
    cd deps
    sudo make hiredis jemalloc linenoise lua geohash-int
    cd ..
    ~~~
  - compile the *Redis* binaries:
    ~~~
    make
    ~~~
  - run the *test* suite to make sure the above process resulted in a correct built::
    ~~~
    make test
    ~~~
  - install the binaries in the system:
    ~~~
    sudo make install
    ~~~
  - create a user, with no home directory, and a group having the same `redis` ID:
    ~~~
    sudo adduser --system --group --no-create-home redis   
    ~~~
  - create a folder for persistent storage:
    ~~~
    sudo mkdir /var/lib/redis
    ~~~
  - adjust ownership and permissions on the above folder, so that regular users cannot access this location:
    ~~~
    sudo chown redis:redis /var/lib/redis
    sudo chmod 770 /var/lib/redis
    ~~~
  - create a configuration directory:
    ~~~
    sudo mkdir /etc/redis
    ~~~
  - copy therein the sample configuration file that comes with the source code:
    ~~~
    sudo cp ~/software/redis/redis-stable/redis.conf /etc/redis
    ~~~  
  - open the configuration file for editing: 
    ~~~
    sudo nano /etc/redis/redis.conf
    ~~~
  - make the following two changes:
    - `supervised no` to `supervised systemd`
    - `dir ./` to `dir /var/lib/redis`
  - exit the file, saving the changes 
  - create a `systemd unit` file for the new *Redis* service:
    ~~~
    sudo nano /etc/systemd/system/redis.service
    ~~~
  - add the following text:
    ~~~
    [Unit]
    Description=Redis In-Memory Data Store
    After=network.target
    
    [Service]
    User=redis
    Group=redis
    ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
    ExecStop=/usr/local/bin/redis-cli shutdown
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
    ~~~
  - exit the file, saving the changes 
  - to start, stop, restart, enable at boot, or simply check its status run respectively:
    ~~~
    sudo systemctl start redis
    sudo systemctl stop redis
    sudo systemctl restart redis
    sudo systemctl enable redis
    systemctl status redis
    ~~~
  - to test the server, after starting it, try first to connect using the redis client
    ~~~
    redis-cli
    ~~~
  - test the connectivity at prompt:
    ~~~
    127.0.0.1:6379> ping
    ~~~
    with expected result `PONG`
  - check the ability to set and return keys:
    ~~~
    127.0.0.1:6379> set mykey "Hello, World!"
    127.0.0.1:6379> get mykey
    ~~~
    with expected results respectively `OK` and `Hello, World!`
  - exit the *Redis* client:
    ~~~
    127.0.0.1:6379 exit
    ~~~

 - to test the connectivity from inside *R*, let's first load the *redux* package (no need to enter *R* as `sudo` at this time):
    ~~~
    library(redux)
    ~~~
  - create a `redis_api` object:
    ~~~
    r <- redux::hiredis()
    ~~~
  - create a `redis_api` object:
    ~~~
    r$SET('mykey', 'Hello, World!')
    ~~~
    with expected result `[Redis: OK]`
  - explore the content of the above object:
    ~~~
    r$GET('mykey')
    ~~~
    with expected result `[1] "Hello, World!"`
  - exit *R*:
    ~~~
    q()
    ~~~


  <a name="postgresql"/>

### PostgreSQL (relational database)

TBD



  <a name="mssqlserver"/>

### MS SQL Server (relational database)

TBD



  <a name="monetdblite"/>

### MonetDBLite (columnar database)

TBD


  <a name="hive-hbase"/>

### Hive/Hbase (hadoop store)

TBD


  <a name="influxdb"/>

### Influx DB (time database) 

TBD


<br/>

:point_up_2:[Back to Index](#index)
<a name="docker"/>
## Docker

  <a name="docker-install"/>

### Install Docker
  - install the dependencies:
    `sudo apt-get install apt-transport-https ca-certificates curl software-properties-common`
  - add the docker repository in the *apt* source list:
    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
    `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
  - update the package management source:
    `sudo apt-get update `
  - let's make sure, you're going to install docker from Docker repo,
    `apt-cache policy docker-ce`
  - install Docker:
    `sudo apt-get install docker-ce`
  - check the status:
    `sudo systemctl status docker`
  - check the version of both client and server:
    `sudo docker version`
  - verify the installation with a basic image:
    `sudo docker run hello-world`
  - returns a bunch of info about the Docker daemon:
    `sudo system info`

  <a name="docker-commands"/>

### Basic Commands
  - download the image **imgname** from the cloud repository:
    `sudo docker pull imgname`
  - run a container with the image **imgname**:
    `sudo docker run imgname`
  - see running containers
    `sudo docker ps` 
  - stop a container (look up cont_name running previous command)
    `sudo docker stop cont_name`
  - stop every running container
    `sudo docker stop $(sudo docker ps -q)`
  - stop container with a specified label
    `sudo docker stop $(sudo docker ps -q -f )`

  <a name="dockerfile"/>

### Dockerfile
A **Dockerfile** is a script that contains a collection of dockerfile instructions and operating system commands (tipycally Linux commands), that will be automatically executed in sequence in the docker environment for building a new docker image.

Below are some of the most used dockerfile instructions:
  - **FROM**  *registry/image:tag* The base image for building a new image. This command must be on top of the dockerfile.
  - **MAINTAINER** Optional, it contains the name of the maintainer of the image.
  - **RUN** Used to execute a command during the build process of the docker image.
  - **COPY** Copy a file from the host machine to the new docker image. There is an option to use an URL for the file, docker will then download that file to the destination directory. Thre is an additional `ADD` command which is not suggested to be used.
  - **ENV** Define an environment variable.
  - **CMD** Used for executing commands when we build a new container from the docker image.
  - **ENTRYPOINT** Define the default command that will be executed when the container is running.
  - **WORKDIR** This is directive for CMD command to be executed.
  - **USER** Set the user or UID for the container created with the image.
  - **VOLUME** Enable access/linked directory between the container and the host machine.

The following is an example of Dockerfile that creates an image similar to the server we're currently building on Digital Ocean: 
```
# Download base image ubuntu 16.04
FROM ubuntu:16.04
 
RUN \
    # Update software repository
    apt-get update \
    # Install missing basic commands
    && apt-get install -y --no-install-recommends apt-utils  \
    && apt-get install -y sudo wget gdebi-core libapparmor1  \
    # Upgrade system
    && apt-get upgrade -y  \
    # Install R packages dependencies
    && apt-get install -y  \
        curl  \
		libssl-dev  \ 
        libcurl4-gnutls-dev  \
		libssl-dev  \
		libxml2-dev  \
        libcairo2-dev  \
        libxt-dev  \
		pandoc  \
        pandoc-citeproc  \
        xtail \
    # cleaning
    && apt-get clean  \
    && rm -rf /var/lib/apt/lists/  \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds
    
RUN \ 
    # add CRAN repository to apt
    echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" | sudo tee -a /etc/apt/sources.list  \
    # add public key of CRAN maintainer
    && gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9  \
    && gpg -a --export E084DAB9 | sudo apt-key add -  \
    # Update software repository
    && sudo apt-get update  \
    # install R
    && sudo apt-get install -y r-base r-base-dev  \
    # install shiny package
    && su - -c "R -e \"install.packages('shiny', repos='https://cran.rstudio.com/')\""

RUN \
    # download and install RStudio Server
    wget -O rstudio https://s3.amazonaws.com/rstudio-ide-build/server/trusty/amd64/rstudio-server-1.2.830-amd64.deb  \
    && gdebi rstudio  \
    && rm rstudio  \
    # download and install Shiny Server
    && wget -O shiny https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.7.907-amd64.deb  \
    && gdebi shiny  \
    && rm shiny
    
# Copy shiny configuration files into the Docker image (change the port ? the user ? the app directory ?)
# COPY shiny-server.conf /etc/shiny-server/shiny-server.conf

# install R packages using R script (plus cleaning)
# RUN Rscript -e "install.packages()" \
#    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

RUN \
    adduser shiny  \
    # add a new group "public"
    && groupadd public  \
    # add "shiny"" to the "public" group
    && usermod -aG public shiny  \
    # Create a new directory as a base for the shared directory with the host and modify permissions to be used by the "public" group
    && mkdir -p /usr/local/share/public  \
    && chgrp -R public /usr/local/share/public/  \
    && chmod -R 2775 /usr/local/share/public/

# Volume configuration
VOLUME ["/usr/local/share/public"]

# Add the "public" path to the R configuration file 
# RUN 
 
```
  <a name="docker-selenium"/>

### Example: Selenium for Web Driving
  - pull Selenium image:
    `sudo docker pull selenium/standalone-firefox`
  - start a "simple" container listening to the port 4445:
    `sudo docker run -d -p 4445:4444 selenium/standalone-firefox`
  - start a "simple" container listening to the port 4445:
    `sudo docker run -d -p 4445:4444 selenium/standalone-firefox`
  - start a container with a mapping between some host directory and the guest browser download directory
    `docker run -d -p 4445:4444 -v /home/usrname/some/path:/home/seluser/Downloads selenium/standalone-firefox`   
    - to ensure the above actually works, you have to correctly configure the browser: 

  <a name="docker+resources"/>

### Resources

  - [main website](https://www.docker.com/) 
  - [Docker Hub](https://hub.docker.com/), the default public image registry
  - join the [community](dockercommunity.slack.com)
  - [*official* documentation](https://docs.docker.com)
  - exercise with an [online interactive environment](http://labs.play-with-docker.com/)

<br/>

:point_up_2:[Back to Index](#index)
<a name="fonts"/>
 
## Additional Fonts

### Preliminaries

  - Change permissions to the font repository in `/usr/share/fonts/`
    ```
    sudo chown -R root:public /usr/share/fonts/
    sudo chmod -R 644 /usr/share/fonts/*
    sudo chmod 755 /usr/share/fonts/
    ```
  - install **fontconfig**
    `sudo apt-get install fontconfig`
  - install the *R* package **extrafont**
    ```
    sudo su
    R
    
    install.packages('extrafont')
    q()
    
    exit
    ```

### Register fonts

  - (re)build the font information cache file (avoid printing output)
    `sudo fc-cache -fv > /dev/null`
  - open *R* as sudoer, load the `extrafont` package and import the new installed fonts (takes time...):
    ```
    sudo su
    R
    
    library(extrafont)
    font_import()
    q()
    
    exit
    ```

### Google Fonts

  - Create a dedicated folder in the above font directory:
    ```
    cd /usr/share/fonts/  
    sudo mkdir google
    cd google
    ```
  - Download the complete google fonts archive, unzip and clean
    ```
    sudo wget https://github.com/google/fonts/archive/master.zip
    sudo unzip master.zip
    sudo rm master.zip
    ```
  - Rebuild the system font cache, and import the new fonts in *R* as described above

  
### Microsoft Core Fonts

  - download the fonts
    `sudo apt-get install ttf-mscorefonts-installer`
    All fonts are copied in:
    `/usr/share/fonts/truetype/msttcorefonts`
  - Rebuild the system font cache, and import the new fonts in *R* as described above

  
### Windows Fonts

  - create a dedicated folder in the usual font directory:
    ```
    cd /usr/share/fonts/
    sudo mkdir windows
    ```
  - open an ftp session in *MobaXterm*, and copy the content of `C:\Windows\fonts` to a temporary folder in the shared repository `/usr/local/share/public/fonts`
  - copy the above fonts in the previous directory:
    `cp /usr/local/share/public/fonts/*  /usr/share/fonts/windows/`
  - remove the temporary directory in the shared repository:
    `rm -rf /usr/local/share/public/fonts/*`
  - rebuild the system font cache, and import the new fonts in *R* as described above


 
  <a name="add-ssh-key"/>

### Add SSH Key Pair for Enhanced Security

  <a name="with-key-windows"/>

#### Windows Users
Open *MobaXTerm*, then follow these steps:

    Tools > 
	  MobaKeyGen > 
	    (leave parameters as default) > 
	    Generate > 
	    Move the mouse around in the big empty area over the **Generate** button >
		insert a password twice in the textboxes called **passphrase** >
	Save both public and private keys >
	Close

  <a name="with-key-linux-macos"/>

#### Linux and macOS Users
  - run the command `ssh-keygen`. 
    The keys are immediately created and stored in `/home/usrname/.ssh` with the displayed names (usually id_rsa.pub and id_rsa for the [public and respectively private key). Both the files should be copied somewhere safe, and the private key promptly deleted from the server. The public key is a simple text that can be shared with anyone, and can be easily read with a simple `cat` command if in need of pasting its content.
  - go to Account / Security / SSH keys / Add SSH key
  - Paste the Key in the big textbox, then give it a name in the small textbox below


<br/>

:point_up_2:[Back to Index](#index)
<a name="linux-basics"/>
## Appendix: Linux Basics

  - what is *Linux*
  - what are *Linux distributions* (*distros*)
  - why *Ubuntu*
  - what is the *terminal*
  - main differences between *Linux* and *Windows*
  - gettinmg help: 
    - `help`
    - `man cmdname`

  - `shutdown`
  - `reboot`


  - `free` 
  - `clear`  or `cls` 
 
  <a name="linux-files-folders"/>

### Files and Folders
  - `pwd` 
  - `ls` 
  - `cd` 
  - `mkdir` 
  - `rmdir` 
  - `cp /path/to/origin/fname /path/to/destination` 
  - `mv /path/to/origin/fname /path/to/destination` 
  - `rm /path/to/origin/fname` 
  - `rmdir /path/to/origin` 
  - `cat fname` 
  - `less fname` 
  - `more fname` 
  - `head fname` 
  - `tail fname` 
  - `touch fname` 
  - `nano fname` 
  - `find fname` 
  - `history` 
  - `df`

  <a name="linux-file-compression"/>

#### File compression


  <a name="linux-directory-structure"/>

#### Root Directory Structure



  <a name="linux-users-groups-permissions-ownerships"/>

### Users, Groups, Permissions, Ownerships

  - `` 
  - `whoami` 
  - `adduser usrname`  
  - `usermode -aG sudo usrname`  
  - `passwd usrname`
  - `su usrname`
  - `sudo` 
  - `exit`
  - `logout` 
  - `chmod`
  - `chown`

  <a name="linux-software-management"/>

### Software management 

   - `update`
   - `upgrade`
   - `dist-upgrade`
   - `autoremove`
  - `clean`
   - `install`
  - `/etc/apt/sources.list` Locations to fetch packages from
  - ``

  <a name="linux-processes"/>

### Process Management

  - `ps`
  - `top`
  - `kill`
  - dealing with services:
    - ` `

### Networking

  - `ifconfig`
  - `wget`
  - `hostname`

  <a name="linux-cronjobs"/>

### Scheduling Tasks



  <a name="linux-shell"/>

### Bash Shell




<br/>

:point_up_2:[Back to Index](#index)
<a name="resources"/>
## Resources 

 - [WeR Meetup](https://www.meetup.com/WeR-stats/)
 - [WeR GitHub Repository](https://github.com/WeR-stats)
 - [WeR Trello Board](https://trello.com/b/OrAZjOfx/01-set-up-cloud-machine-for-data-science)
 - [WeR Slack Channel](https://we-r-stats.slack.com/messages/CER9296DP/) To join this channel you have to send me a meetup message, including your email and permission to add you as a user.

---
<font size="10">**Disclaimer**</font>

I’m not a *devOps* or *sysAdmin*, and most of this document has been built over years of experience trying to overcome the problem of the hour. So it’s very possible that some steps here are not the very best way of performing the tasks they refer to. 

If anyone has any comments on anything in this document, [I’d love to hear about it!]()

---