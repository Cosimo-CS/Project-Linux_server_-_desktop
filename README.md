# Project Linux server and desktop
1st linux project - Kamkar 04.24

## Project context 
The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation.
To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).

You may propose any additional functionality you consider interesting.

## Must Have

Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    - DHCP (one scope serving the local internal network)  isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on  separate disk, this partition must be mounted for the backup, then unmounted

2. One workstation running a desktop environment and the following apps:
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required** 
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk 
    - **Optional**
        1. Propose and implement a solution to remotely help a user

----------------------------------------------------------------------------------------------

In this project you can find a setup with a desktop (Ubuntu Linux) and a server (Linux server) within a virtualized environment on Oracle virtual box

In this repository you can find the configuration of two virtual machines: 

- Server: Ubuntu 22.04.4 TLS
- Desktop: Ubuntu server 22.04

The server is installed without GUI.

![alt text](assets/no-gui.png)

Let's discover the differents configurations made for this project:

- Server configuration --> linux-server-config.md
- Desktop configuration --> linux-desktop-config.md


# Server Ubuntu VM:

Settings:
- 4096 Mo of RAM
- 3 CPU cores
- 20 GB of storage
- Adaptater 1 : Bridged network adapter
- Adaptater 2 : Private Host 
- Network manager CFG (Virtual box) : ipv4 : 192.168.52.2 DHCP: Disabled
- Ubuntu 22.04 LTS Server Edition
- Hostname: `ubuntu-server`

### __1. Creating the New VM__

First of all you need to download your .iso file. 

You can get one in the following link: [ubuntu-22.04-iso](https://ubuntu.com/download/server)

Once the ISO downloaded, we can set up the new VM on VirtualBox.

Click on New button, fill out the name, select the ISO you just downloaded.

![alt-text](assets/iso-vm.png)

Press "Next" and there you can set up your username with password and the hostname of the server.

Allocate 4096MB of RAM;
Allocate 20GB of hard drive space or more. (Depending on the space in your host computer)

Finish.

------------------------------------------------------------------------------------

### __2. Installing Linux on the VM__

Select your language: I choosed English by default.

Configure the keyboard: 
- Layout: Belgian
- Variant: Belgian

Done.

Choose the base for the installation:
- Ubuntu server

Done.

## Network connections

Don't touch nothing and just press done.

Same for "Configure proxy", "Configure ubuntu archive mirror" just press done, we don't need to set up one for this project.

## Guided storage configuration

You can leave it by default also:
- Use an entire disk
- Set up this disk as an LVM group

Done.

## Storage configuration

I left the configuration by default also.

Done and you can confirm destructive action.

## Profile setup

- Your name: Totti K
- Your servers name: ubuntu-server
- Pick a username: totti
- choose a password: ******
- Confirm it.

Done.

## SSH setup

I didn't Installed open SSH server we will configure it later, same for Feature Server Snaps.

Congrats! You can take a coffee during the installation of your server. x-)

When it's done you can press on "Reboot now"

### __3. Setting Up the VM__

Now let's get down to business and start configuring the server!

As you can see, it's quite annoying to 'not be able' to do what you want because you're only faced with a terminal.

So I wanted to simplify my life and set up an SSH connection directly with my desktop (which I'd already installed upstream).

But first let's do some basic configuration, which will be necessary for the rest of the operation.

### __Steps__

### __1. Install sudo and add user to sudoers__

Install sudo and add your user to the sudoers group:
```sh
su root
```
```sh
apt install sudo
```
```sh
usermod -aG sudo YOUR_USERNAME
```
```sh
exit
```

Test sudo and privilege:
```sh
sudo apt update
```

I've also added few nice commands to get information from your new fresh server. They are in the commands file.

Here below you can see 1 nice example to get information that we will need to set up our SSH connection, DHCP, DNS,etc.

The df command displays information about total space and available space on a file system. The FileSystem parameter specifies the name of the device on which the file system resides, the directory on which the file system is mounted, or the relative path name of a file system.
```sh
df
```
![alt-text](assets/df-command.png)

### __2. Installation of SSH__

I will put the sudo command to have the full syntax but to make this configuration it's better that you take your root privilege directly to avoid using "sudo" in the syntax of your command.

Root privileges
```sh
su -
```

Install the ssh package via sudo
```sh
sudo apt install ssh
```

Check the systemctl status
```sh
sudo systemctl status ssh
```
If the service is active(running), then it's ok.

We want to check if we are really listening to port 22 so we will use this command.
```sh
ss -ltn
```
As you can see on the image we are listening by default on port 22.

![alt-text](assets/ssltn-command.png)

### __Let's enable the firewall to allow ssh port connection.__
```sh
sudo ufw enable
```
We will allow our firewall to accept those ports:
```sh
sudo ufw allow 22/tcp
```

We can check if it's well allowed by using
```sh
sudo ufw status
```

Now let's go the change the configuration of the interface ipv4 for the ssh connection
```sh
cd /etc/netplan
```

Make a copy of 00-installer-config.yaml and rename it.
```sh
cp 00-installer-config.yaml 01-enp0s8-config.yaml
```
Let's configure it!
```sh
sudo vim 01-enp0s8-config.yaml
```
  - Add the following configuration to the file:
    ```bash
      network:
        ethernets:
        enp0s8:
        addresses:
        - 192.168.52.1/24
    nameservers: {}
    version: 2
    ```

After that switch off the machine using

```sh
init 0
```

### __Why are we using the interface enp0s8 and the ip address 192.168.52.1/24 ?__

Because in our software Oracle Virtual box we gonna change the settings of the virtual machine and add another interface to configure a static ip address to allow the configuration for the SSH and also for the GLPI.

![alt-text](assets/settings-vm.png)

After adding the adaptater 2 you have also to go in Files --> Tools --> Network Manager and there under the tab "Host-Only Networks" you set up your static ip address (don't forget to disable dhcp server)

![alt-text](assets/settings-network.png)

You are now ready to establish your SSH connection! (You can even try it via your host computer)

Turn on your server and first let's check if the settings are correctly made

```sh
ip a
```
You should get this result, as you can see enp0s8 have a static ip address 192.168.52.1

![alt-text](assets/ipa-command.png)

Now just open your VM desktop (if already mounted) or just try with your host computer to enter in the server via ssh connection.
```sh
ssh YOUR_USERNAME@192.168.52.1
```

### __Configuring DHCP Server__

1. **Install ISC DHCP Server**:
   ```bash
   sudo apt install isc-dhcp-server -y
   ```

2. **Configure DHCP Server**:
   - First we will edit dhcpd.conf file.
    ```bash
     sudo vim /etc/dhcp/dhcpd.conf
    ```
   - We add the following configuration to the file:
    ```bash
    subnet 192.168.0.0 netmask 255.255.255.0 {
        range 192.168.0.100 192.168.0.200;
        option domain-name-servers 192.168.0.1; 
        option routers 192.168.0.1; 
        option broadcast-address 192.168.0.255;
        default-lease-time 600;
        max-lease-time 7200;
    }
    ```

3. **Specify the Network Interface**:
   - Now we will edit "isc-dhcp-server" file to specify the interface DHCP.
    ```bash
     sudo vim /etc/default/isc-dhcp-server
     ```
   - and once inside we set the `INTERFACESv4` variable to our server's network interface. Now it will be "enp0s3"
  
         ```bash
INTERFACESv4="enp0s3"
INTERFACESv6=""
    ```
  
  # Start and Enable DHCP Service
   ```bash
   sudo systemctl restart isc-dhcp-server
   sudo systemctl enable isc-dhcp-server
   sudo systemctl status isc-dhcp-server
    ```
    
 ![alt text](assets/dhcp-service.png) 

DHCP service is now active.

By doing the following command you can see that we get a:
- DHCP Discover
- DHCP REQUEST
- DHCP ACK

   ```bash
   sudo dhclient -v
   ```
 ![alt text](assets/dhclient-command.png) 

 ### Setting Up a DNS Server with BIND

1. ### __Install BIND__
   ```bash
   sudo apt install bind9
   ```

2. ### __Configure DNS Server:__
    - Edit the BIND configuration file.
     ```bash
      sudo vim /etc/bind/named.conf.options
     ```
    - Add the following configuration to the file:
     ```bash
     forwarders {
        1.1.1.1;
        8.8.8.8.;
    };
    ```
- 1.1.1.1 = Cloudflare DNS
- 8.8.8.8 = Google DNS
  After that you can try to ping them to check if the connectivity is working.

     /!\ Remove well the // in the front of each line !
  
   ![alt text](assets/conf-bind9.png)

There is also a service included with systemd called resolved and we have to disable this service in order to put Bind9 being used as our main DNS resolution service.

So let's stop first this service and we will also disable it.
   ```bash
   sudo systemctl stop systemd-resolved
   sudo systemctl disable systemd-resolved
   ```
Let's open another file in cd/run/systemd/resolve
   ```bash
sudo vim /run/systemd/resolve/stub-resolv.conf
   ```
   ![alt text](assets/conf-systemd.png)
   
 ### __Start and Enable BIND:__
   ```bash
   sudo systemctl restart bind9
   sudo systemctl enable bind9
   ```

 ### __Verify DNS Configuration:__
    - Check the status of the BIND service.
    ```bash
      sudo systemctl status bind9
      ```
    ![alt text](assets/DNS.png) 

    The DNS server is now running and ready to resolve domain names for clients on the network.

    Testing the DNS server...

    ```bash
    nslookup google.com
      ```

 # Setting Up HTTP and MariaDB for GLPI

 ### __Installing prerequisites__
 
 First let's update the full system to have latests updates.
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
It's mandatory to use this command to install all php packages extensions (it will be useful for later)
   ```bash
   sudo apt install -y vim wget tar php-curl php-zip php-gd php-intl php-intl php-pear php-imagick php-imap php-memcache php-pspell php-tidy php-xmlrpc php-xsl php-mbstring php-ldap php-ldap php-cas php-apcu libapache2-mod-php php-mysql php-bz2
   ```
Why ? because when I wanted for the first time to run the php script I had this kind of issues.

![alt text](assets/issuesphpinstallation.png)

 Check if the service "Apache2" is running (active)
   ```bash
   sudo systemctl status apache2
   ``` 

### __Install Apache and MariaDB:__
   ```bash
   sudo apt install apache2 mariadb-server
   ```

### __Create a Database for GLPI:__
  
   Access MariaDB prompt:
 ```bash
   sudo mysql -u root -p
 ```
        
    ```sql
    CREATE DATABASE glpi_kamkar;
    CREATE user glpi_admin@localhost IDENTIFIED BY 'YOUR_PASSWORD';
    GRANT ALL ON glpi_kamkar.* TO glpi_admin@localhost;
    FLUSH PRIVILEGES;
    EXIT;
    ```
    ![alt text](assets/cmd_mariadb_glpi.png)

### __Install GLPI via Command Line__

1. ### __Download GLPI:__
   ```bash
   wget -O /tmp/glpi.tgz https://github.com/glpi-project/glpi/releases/download/10.0.14/glpi-10.0.14.tgz
   ```

2. ### __Extract GLPI:__
   ```bash
   tar -xzf /tmp/glpi.tgz -C /var/www/html/
   ```

3. ### __Set Permissions:__
   ```bash
   chown -R www-data:www-data /var/www/html/glpi
   ```
   ```bash
   chmod -R 755 /var/www/html/glpi
   ```
   You can check if it worked by going in the folder glpi
   ```bash
   cd /var/www/html/glpi
   ```
   Using ll command
   ```bash
   ll
   ```
![alt text](assets/ll-command.png)

   After modifying the permissions let's restart apache2
   ```bash
   sudo systemctl restart apache2
   ```

4. ### __Run the Install Script:__
   ```bash
   php /var/www/html/glpi/bin/console db:install --db-host=localhost --db-name=glpi --db-user=YOUR_USER --db-password='YOUR_PASSWORD'
   ```

   If it doesn't work for any reason,don't worry there is another option to get into your db via interface but for this you will need to set up your desktop VM first and have SSH connection from your desktop to your server.

   ### __Check ip of the server__

   We will check the ip of the server so that we can connect via a browser from your desktop's VM
   ```bash
   ifconfig
   ```

   You can see in the example next to the interface enp0s8 that we configured before we still have the ip address 192.168.52.1.

   ![alt text](assets/ifconfig-command.png)

   ### __Desktop's Browser__

   As mentioned above, we are now on the desktop with an SSH connection to the server. (This gives us access to our database via the browser).

      ```bash
   ssh YOUR_USER@IP_ADDRESS_OF_THE_SERVER   
   ```
See example below
  ![alt text](assets/ssh-browser.png) 

  Search the address ip of the server and add /glpi at the end and you will get the connection to GLPI and MariaDB.

If you put only the ip address you will get the apache2 server default page.

![alt text](assets/apacheserver_ui.png) 

Result with /glpi

![alt text](assets/glpinstallation2.png) 

After that:

- Select the language
- Licenses: Continue
- Choose Install
- You can see that all PHP extensions are ok, press continue.

For Database connection setup:

- SQL Server (MariaDB or MySQL) : localhost
- SQL User : In this case "glpi_admin"
- SQL Password: Your password that you have set in your command at the point "Create a Database for GLPI"

  When you are connected don't create a new database (we did it via CLI before so just select the database already created.

  ![alt text](assets/glpinstallation.png)

  The database is initialized.
  We don't need send "usage statistics" and press continue 2x.

  /!\ Pay attention at the step6 you will receive your credentials for your server. (Take a picture of it!)

  You will need those credentials to connect to your database and after that you will be able to change the password to make it more secure.

  Press "use GLPI".

  So by default to login to your account in this case it will be.

  - Login : glpi
  - Password: glpi

  And sign in.  You are finally inside your DB. Congratulations!

### __Change password of your users:__

First step is to change the password of your users because they are all set by default. You even have a warning message on the top of the page.

For this you can go to Administration --> Users

![alt text](assets/glpiconnection.png)
  

Second step is to remove also the file glpi/install/install.php  

      ```bash
    cd /var/www/html/glpi/install   
    ```
      ```bash
    sudo rm -fr install.php  
    ```
Of course we will need to restart our service apache2

      ```bash
    sudo systemctl restart apache2 
    ```

Don't forget to refresh also your web page in the browser. Congratulations.


# Desktop Ubuntu VM:

Settings:
- 4096 Mo of RAM
- 3 CPU cores
- 20 GB of storage
- Adaptater 1 : Bridged network adapter
- Ubuntu 22.04 LTS  Edition
- Hostname: `ubuntu-server`



