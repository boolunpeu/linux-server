# linux-server


## Project Context

The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation. To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).

You may propose any additional functionality you consider interesting.

## Must Have

Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    
    - DHCP (one scope serving the local internal network) isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on separate disk, this partition must be mounted for the backup, then unmounted
2. One workstation running a desktop environment and the following apps:
    
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required**
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk
    - **Optional**
        1. Propose and implement a solution to remotely help a user


 ----------------------------------------------------------

## Linux Server VM

I used Hyper V and installed an ISO file of Ubuntu Server :

(https://ubuntu.com/download/server)

Once you've downloaded the ISO, open the Hyper V program and install the VM.

![[VM.png]]

Click on "Nouveau" and follow the steps.

Allocate at least 2048MB of RAM and 2 CPU cores since we're gonna run a desktop environment on it.

Allocate 30GB of hard drive space or more.




## Linux Server Installation 

Follow the installation steps. 

If you have difficulties during this process check out : https://www.nakivo.com/blog/run-linux-hyper-v/


## Steps  : Server set-up 

### **Setting up the firewall**

```shell
sudo apt install ufw
```

```shell
sudo ufw enable
```

```shell
sudo ufw allow SSH,80,443,53
```

(80 for HTTP, 443 for HTTPS and 53 for DOMAIN)

### DHCP Server

1. **Install ISC DHCP Server**:
    
    ```shell
    sudo apt install isc-dhcp-server -y
    ```
    
2. **Configure DHCP Server**:
    
    - Edit the DHCP configuration file.
    
    ```shell
     sudo nano /etc/dhcp/dhcpd.conf
    ```
    
    - We add the following configuration to the file:
    
    ```shell
    subnet 172.28.171.9 netmask 255.255.255.0 {
        range 172.28.171.10 172.28.171.100; 
        option routers 172.28.171.254; 
        option broadcast-address 172.28.171.255;
        default-lease-time 600;
        max-lease-time 7200;
    }
    ```
    
3. **Specify the Network Interface**:
    
    - Edit `/etc/default/isc-dhcp-server` to specify the interface DHCP listens on.
    
    ```shell
     sudo nano /etc/default/isc-dhcp-server
    ```
    
    - and once inside we set the `INTERFACESv4` variable to our server's network interface ( in my case `eth0`).
    
    
4. **Start and Enable DHCP Service**:
    
    ```shell
    sudo systemctl restart isc-dhcp-server
    sudo systemctl enable isc-dhcp-server
    sudo systemctl status isc-dhcp-server
    ```

    

    
    The DHCP server is now running and ready to assign IP addresses to clients on the network
	
![[Capture.png]]
## Steps : Workstation VM 

This is the same procedure as for the Linux server, except that this time we'll be using the following ISO. : https://ubuntu.com/download/desktop (## Ubuntu 22.04.4 LTS)

## Steps : Workstation Installation  

Follow the installation steps. 

If you have difficulties during this process check out : https://ubuntu.com/server/docs/how-to-set-up-ubuntu-on-hyper-v


## Steps  : Workstation set-up 

1. **Install Updates and SSH**
    
    ```shell
    sudo apt update
    sudo apt upgrade
    sudo apt install openssh-server
    ```
    
2. **Enable SSH**:
    
    ```shell
    sudo systemctl enable ssh
    sudo systemctl start ssh
    ```
    
3. **Configure UFW Firewall**:
    
    ```shell
    sudo ufw allow ssh
    sudo ufw enable
    ```
    
4. **Install Additional Software**:
    
    - **LibreOffice** :
        
        ```shell
        sudo apt install libreoffice
        ```
        
    - **GIMP**:
        
        ```shell
        sudo apt install gimp
        ```
        
    - **Mullvad Browser** : I will install the Mullvad Browser on the Ubuntu Desktop VM.
        
        ```shell
        cd ~/Downloads
        wget https://mullvad.net/download/app/linux/latest/ -O mullvad-browser.tar.gz
        sudo tar -xzf mullvad-browser.tar.gz -C /opt
        cd /opt/mullvad-browser/
        sudo ln -s /opt/mullvad-browser/mullvad-browser /usr/local/bin/mullvad-browser
        mullvad-browser
        ```
        
5. **Set Up Remote Assistance**: To set up remote assistance on a Linux client,we can set up `xrdp` , which uses the RDP protocol and can be accessed by most RDP clients, including Remmina, which is often used on Linux systems.
    

### Installing xrdp on the Client:

[](https://github.com/Latteflo/linux_server?tab=readme-ov-file#installing-xrdp-on-the-client)

1. Install `xrdp`:
    
    ```shell
    sudo apt install xrdp
    ```
    
2. After installation, the `xrdp` service should start automatically. To check the status of the service:
    
    ```shell
    sudo systemctl status xrdp
    ```
    
    You should see that `xrdp` is active (running). Like so:
    
    ![[xrdp status.png]]
    To allow the RDP port through the firewall, run the following command:
    
    ```shell
    sudo ufw allow 3389/tcp
    ```
    
    We add an user to the `ssl-cert` group to allow the user to access the SSL certificate:
    
    ```shell
    sudo adduser library ssl-cert
    ```
    
	 To make sure all changes are applied, restart the xrdp service:
    
    ```shell
    sudo systemctl restart xrdp
    ```
    

### Using Remmina to Connect to the Client:

[](https://github.com/Latteflo/linux_server?tab=readme-ov-file#using-remmina-to-connect-to-the-client)

1. Install Remmina along with plugins for RDP and VNC, which are the most commonly used protocols: |
    
    ```shell
    sudo apt install remmina remmina-plugin-rdp remmina-plugin-vnc
    ```
    
    After installation, you can launch Remmina from your application menu or from the terminal:
    
    ```shell
    remmina
    ```
    
2. Configure Remmina to connect to the client:
    

- Click the '+' icon to add a new connection.
    
- Choose the RDP protocol from the drop-down menu.
    
- In the 'Server' field, input the client machine's IP address `192.168.0.131`
    
- Enter the username and password of the client machine: `library` and `2024kamkar`
    
- Save and Connect.
    ![[reminna - rdp connect.png]]
