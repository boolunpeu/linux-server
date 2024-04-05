
==**CLIENT**==
### Install Ubuntu

1. **Mount the Installation ISO on Hyper V**
    
2. **Install Ubuntu**: I've installed Ubuntu 20.04 LTS Desktop Edition. The installation process is straightforward, and you can follow the on-screen instructions to complete the installation.
    
    As for user creation, I've created a user named "library" with the password "library1234" .

![VM](https://github.com/boolunpeu/linux-server/assets/131985567/5f019b98-3ed6-4adc-9217-0b6efddc1d68)

### Configuration

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

![[SSH-client 1.png]]

1. **Configure UFW Firewall**:
    
    ```shell
    sudo ufw allow ssh
    sudo ufw enable
    ```

1. **Install Additional Software**:
    
    - **LibreOffice** :
        
        ```shell
        sudo apt install libreoffice
        ```
        
    - **GIMP**:
        
        ```shell
        sudo apt install gimp
        ```


	**Mullvad Browser**

1. **Download the archive:** Make sure you have downloaded the Mullvad archive in .tar.xz format from the official website or the appropriate source.
    
2. **Extract the archive:** Open a terminal and navigate to the directory where you downloaded the Mullvad archive. Use the following command to extract the archive:

    
    `tar -xf mullvad-*.tar.xz`
    
    Be sure to replace `mullvad-*.tar.xz` with the name of the downloaded archive.
    
3. **Access the extracted directory:** Use the `cd` command to access the extracted directory:
    
    
    `cd mullvad-*`
    
4. **Installation:** At this point, you'll need to consult the documentation supplied with Mullvad to find out how to install the software. Usually, an installation script is supplied with the archive. You can run it using the following command:
    
    `./install.sh`

![[mullvad 1.png]]
## Create a partition for /home

Before creating a partition, make sure you have space available on your hard disk for the new partition. You can use tools such as `fdisk` or `parted`. Here's an example of how to use `fdisk` :
    
    
    `sudo fdisk /dev/sda`
    
    Then use the commands `n` to create a new partition, `p` for the primary partition, specify the partition size, and `w` to save changes and exit.
    
2. **Format the partition:** Once the partition has been created, you need to format it with a file system. For example, to format the partition with ext4, you can use the following command:

    `sudo mkfs.ext4 /dev/sdaX`
    
    Replace `/dev/sdaX` with the path to the partition you've created.
    
3. **Mount the partition:** Before copying the data, you need to mount the new partition on a temporary location. You can do this using the `mount` command:
    
    
    `sudo mount /dev/sdaX /mnt`
	Here, `/mnt` is the temporary location where you mount the partition.


1. **Copy data from /home:** Now you need to copy the current data from `/home` to the new mounted partition. You can do this using the `rsync` command:

    
    `sudo rsync -av /home/ /mnt/`
    
    This command recursively (`-a`) copies all files and directories from `/home` to `/mnt`.
    
5. **Define /home partition in /etc/fstab:** To automatically mount the new `/home` partition at boot time, you need to add an entry to the `/etc/fstab` file. Open the file with a text editor and add a line similar to the following:
    
    
    `/dev/sdaX    /home    ext4    defaults    0    2`
    
    This line specifies that the `/dev/sdaX` partition should be mounted on `/home` with the `ext4` file system and default options.
    
6. **Unmount /mnt:** Once you've added the entry to `/etc/fstab`, you can unmount the temporarily mounted partition:
    
    
    `sudo umount /mnt`
    
7. **Reboot the system:** Reboot your system to apply the changes and check that everything is working correctly.
    

By following these steps, you will have correctly configured a separate partition for the `/home` directory on your Linux server. This will separate user data from other system data, and ensure better management of disk space and security.

![[PARTITION home.png]]


### Installing xrdp on the Client:

1. Install `xrdp`:
    
    ```shell
    sudo apt install xrdp
    ```
    
2. After installation, the `xrdp` service should start automatically. To check the status of the service:
    
    ```shell
    sudo systemctl status xrdp
```
   
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

Thats it for the client side. Now let's move on to the server side.


==**SERVER**==

### Install Ubuntu Server

1. **Mount the Installation ISO**
    
2. **Install Ubuntu Server**: I've installed Ubuntu 22.04 Live server The installation process is straightforward, and you can follow the on-screen instructions to complete the installation.
    
    As for user creation, I've created a user named "library" with the password "library1234" .

### Configuring DHCP Server


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
    subnet 172.21.25.0 netmask 255.255.255.0 {
        range 172.21.25.100 172.21.25.200;
        option domain-name-servers 172.21.25.1; 
        option routers 172.21.25.1; 
        option broadcast-address 172.21.25.255;
        default-lease-time 600;
        max-lease-time 7200;
    }
    ```
	**Specify the Network Interface**:

- Edit `/etc/default/isc-dhcp-server` to specify the interface DHCP listens on.

```shell
 sudo nano /etc/default/isc-dhcp-server
```

- and once inside we set the `INTERFACESv4` variable to our server's network interface (`eth0`) in my case.

	**Start and Enable DHCP Service**:

```shell
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

![[DHCP status.png]]

### Setting Up HTTP and MariaDB for GLPI

1. **Install Apache and MariaDB**:
    
    ```shell
    sudo apt install apache2 mariadb-server
    ```
    
2. **Secure MariaDB Installation**:
    
    ```shell
    sudo mysql_secure_installation
    ```
    
    And here we follow the on-screen instructions to set a root password, remove anonymous users, disallow remote root login, and remove the test database.
    
    In this case scenario I set the password to `kamkar` for simplicity, we remove the anonymous users, remove the test database, allowed root remote login and reload privileges.

	**Create a Database for GLPI**: Access MariaDB prompt:

```shell
sudo mysql -u root -p
```

```sql
CREATE DATABASE glpi;
GRANT ALL PRIVILEGES ON glpi.* TO 'admin@localhost' IDENTIFIED BY 'kamkar';
FLUSH PRIVILEGES;
EXIT;
```

### Installing and configuring a web server :

1. **Install Apache (or another web server):** Install Apache (or another web server of your choice) on your system. On Ubuntu/Debian, you can run :
    
    `sudo apt install apache2`
    
2. **PHP installation:** Make sure you install PHP and the modules required for Apache. On Ubuntu/Debian, you can run :

    
    `sudo apt install php libapache2-mod-php php-mysql`
    

### 4. Downloading and configuring GLPI :

1. **Downloading GLPI:** Download the latest version of GLPI from the official website or using wget :

    
    `wget -O glpi.tar.gz https://github.com/glpi-project/glpi/releases/download/9.5.5/glpi-9.5.5.tgz`
    
2. **Extract the archive:** Extract the archive into your web server directory:

    
    `sudo tar -zxvf glpi.tar.gz -C /var/www/html/`
    
3. **Assign permissions:** Ensure that the GLPI directory belongs to the appropriate user and group:

    `sudo chown -R www-data:www-data /var/www/html/glpi`
    
4. **GLPI configuration via web interface:** Go to the URL of your web server where GLPI is installed (e.g. http://localhost/glpi) and follow the instructions to configure GLPI using the MariaDB database information you configured earlier.

	(MariaDB = database name : libraryDB
	user = library@localhost , password : 1234)

![[GLPI.png]]
### Setting Up SSH

1. **Install Updates and SSH**
    
    ```shell
    sudo apt update
    sudo apt upgrade
    sudo apt install openssh-server
    ```
    
    Verify it's running:
    
    ```shell
    sudo systemctl status ssh
    ```

![[Status-SSH.png]]
![[Connexion-SSH.png]]

### Configuring Firewall

```shell
sudo ufw allow ...
sudo ufw enable
sudo ufw status
```

I use the TCP (Transmission Control Protocol) and I have the following ports open :

- 22/tcp: Standard port for SSH (Secure Shell) - remote administration.
    
- 2222/tcp: alternative SSH port.
    
- 80/tcp: Standard port for HTTP (HyperText Transfer Protocol) unsecured web traffic.
    
- 443/tcp: Standard port for HTTPS (HTTP Secure) - secured web traffic.

![[ufw.png]]
### Backup Configuration Files

1. Let's create our script in `/usr/local/bin/backup_glpi.sh`:
    
    ```shell
    sudo nano /usr/local/bin/backup_glpi.sh
    ```
    
    Inside the script, we have the following content:
    
    ```shell
    #!/bin/bash
    # Define backup paths
    BACKUP_DIR="/var/backups/glpi"
    GLPI_DIR="/var/www/html/glpi"
    DB_NAME="libraryDB"
    DB_USER="library@localhost"
    DB_PASSWORD="1234" 
    DATE=$(date +%Y-%m-%d)
    
    # Create a new directory for today's backup
    mkdir -p "${BACKUP_DIR}/${DATE}"
    
    # Backup GLPI files
    tar -czf "${BACKUP_DIR}/${DATE}/glpi_files_${DATE}.tar.gz" -C "${GLPI_DIR}" .
    
    # Backup GLPI database
    mysqldump -u "${DB_USER}" -p"${DB_PASSWORD}" "${DB_NAME}" > "${BACKUP_DIR}/${DATE}/glpi_db_${DATE}.sql"
    
    # Remove backups older than 30 days
    find "${BACKUP_DIR}" -type f -mtime +30 -exec rm {} \;
    ```

1. Make the script executable:
    
    ```shell
    sudo chmod +x /usr/local/bin/backup_glpi.sh
    ```
    
    We can schedule the backup script to run weekly using a cron job.
    
    ```shell
    sudo crontab -e
    ```
    
    Add this line to execute the script every Sunday at 2 AM:
    
    ```
        0 2 * * Sun /usr/local/bin/backup_glpi.sh
    ```
    
![[crontab.png]]
