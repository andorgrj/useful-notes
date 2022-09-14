   ##### sudo visudo -f /etc/sudoers.d/web_admin

Enter in the following at the top of the file. This will create an alias command group that we can apply to any user or group that we add to this file. This group of commands will contain the necessary commands for restarting or reloading the web server:


   ##### Cmnd_Alias  WEB = /bin/systemctl restart httpd.service, /bin/systemctl reload httpd.service

Add another line in the file for gfreeman to be able to use the sudo command in conjunction with any commands listed in the WEB alias:

   ##### <user> ALL=WEB

----------------------------------------

The wheel group is a special user group used on some Unix systems, mostly BSD systems, to control access to the su or sudo command.

   ##### useradd -m -> creates the home directory if it is not existing

Set up the web administrator:

    Now we need to configure gfreeman's account to have the ability to restart or reload the web server when needed. Since he will be the webmaster, he needs sudo permissions to restart the service.

First, create a new sudoers file in the /etc/sudoers.d directory that will contain a standalone entry for webmasters. Use the -f option with the visudo command to create this new file:
    
   ##### sudo visudo -f /etc/sudoers.d/web_admin

Enter in the following at the top of the file. This will create an alias command group that we can apply to any user or group that we add to this file. This group of commands will contain the necessary commands for restarting or reloading the web server:

   ##### Cmnd_Alias  WEB = /bin/systemctl restart httpd.service, /bin/systemctl reload httpd.service

Add another line in the file for gfreeman to be able to use the sudo command in conjunction with any commands listed in the WEB alias:
    
   ##### gfreeman ALL=WEB

Next, log into the gfreeman account:

   ##### sudo su - gfreeman

Attempt to restart the web service:
    
   ##### sudo systemctl restart httpd.service

Now gfreeman can restart the web server. As the gfreeman user, try to read the new web_admin sudoers file with the sudo command:
    
   ##### sudo cat /etc/sudoers.d/web_admin

Since the cat command is not listed in the command alias group for WEB, gfreeman cannot use sudo to read this file.

--------------------------------------------
    tar -xvf deploy_content.tar.gz >> tar-output.log
    tar -xvf deploy_script.tar.gz >> tar-output.log
    ll
    cat tar-output.log
    ll
    history
    scp *.gz 54.210.198.84:~/
    ssh-copy-id <server2 IP>

    Copy All tar Files from `/home/dev/` on `server1` to `/home/dev/` on `server2`, Extract Them, and Redirect Output to `/home/dev/tar-output.log`

    We need to use a method of copying files over a network. scp is the best tool, like this:

        [dev@server1]$ scp *.gz <server2 IP>:~/
        Then connect to server2 using ssh:

        [dev@server1]$ ssh <server2_IP>
        Then we can extract the files:

        [dev@server2]$ tar -xvf deploy_content.tar.gz >> tar-output.log
        [dev@server2]$ tar -xvf deploy_script.tar.gz >> tar-output.log
        Make sure to use >>, so that the output is appended rather than overwritten.

        Set the Umask So That New Files Are Only Readable and Writeable by the Owner

        The task is asking to make new files with the following permission: 0600.

        So we can do subtraction to figure out what our umask should be.

        0666 <-- Default

        0600 <-- Desired

        ----

        0066 <-- What we need to set

        So we run:

        [dev@server2]$ umask 0066

        Verify the `/home/dev/deploy.sh` Script Is Executable and Run It

        First, we check permissions on deploy.sh:

        [dev@server2]$ ls -l deploy.sh
        There's no execute bit. Let's add one:

        [dev@server2]$ chmod +x deploy.sh
        And then run it:

        [dev@server2]$ ./deploy.sh

5. Steps for the creation:
            ## CENTOS7 (WEB1):
            
            sudo -i -> become a root
            usermod -g wheel <username> -> superuser 
            1 sudo adduser Devops
            2 sudo passwd Devops
            4 su - Devops
            6 pwd
            7 logout
            5 pwd
            9 echo $HOME
            10 grep Devops /etc/passwd
            12 echo "Devops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/Devops
            13 chsh -l
            14 yum install zsh
            15 nano /etc/ssh/sshd_config.
            16 ls -la
            17 vi /etc/ssh/sshd_config.
            18 sudo cp /etc/sudoers /root/sudoers.bak
            19 sudo visudo
            20 yum install zsh
            21 sudo yum install zsh
            23 echo $SHELL
            24 chsh -l
            25 chsh -s /bin/zsh
            26 logout
            27 history
            28 ssh-keygen
            29 cat /home/Devops/.ssh/id_rsa
            30 ls ~/.ssh/id*
            31 file .ssh/id_rsa.pub
            32 ssh-copy-id -i ~/.ssh/id_rsa.pub Devops@35.178.40.13
            33 ssh -i ~/.ssh/id_rsa.pub Devops@35.178.40.13
            34 ssh Devops@35.178.40.13

            ## UBUNTU (WEB2):

            1 sudo apt update
            2 sudo adduser Devops
            3 sudo adduser --force-badname Devops
            4 pwd
            5 history
            16 sudo -l
            21 sudo visudo -> Devops ALL=(ALL:ALL) ALL
            22 chsh -l
            23 chsh -s /bin/zsh
            24 ssh-copy-id -i ~/.ssh/id_rsa.pub Devops@13.40.113.137
            25 ssh Devops@13.40.113.137
            26 history

            ## UBUNTU (BASTION)

            sudo apt update
            sudo adduser Andor
            sudo adduser --force-badname Andor
            pwd
            sudo -l
            logout
            ssh-keygen
            ls -l ~/.ssh/
            ssh-copy-id -i ~/.ssh/id_rsa.pub Andor@18.170.115.86
            ssh Andor@18.170.115.86


            
            
    
PERSONAL NOTES DURING THE DEMO:
    
    1. on centos 7 everything is done / ssh copy to web2 is also done

    2. on web2 zsh is to be done / ssh copy to web1 is also done

    3. SSH config file is found, need to be set the things -> sudo vi /etc/ssh/sshd_config


    2. Create user with your username on bastion
        1. Home should be /home/$username
        2. Shell: bash
        3. Create key for your self and authorize it
        4. Set so the user is able to become devops user

## BASTION UBUNTU

            sudo apt update
            sudo adduser Andor
            sudo adduser --force-badname Andor
            pwd
            sudo -l
            logout
            ssh-keygen
            ls -l ~/.ssh/
            ssh-copy-id -i ~/.ssh/id_rsa.pub Andor@18.170.115.86 /on local computer
            ssh Andor@18.170.115.86
            sudo visudo -> Andor ALL=NOPASSWD: /bin/su - Devops



    3. SSH config
        1. Root login is prohibited

            A compromised root account will give an attacker full access to your server. The compromise could be caused by bots that would normally brute force root SSH account or by the leakage of the password or private key of the root user. Therefore, it is good practice to disable direct access by root user to SSH servers and only allow standard users to log in. These users will then use sudo or other methods to perform their administrative tasks.

            sudo vi /etc/ssh/sshd_config -> PermitRootLogin no

        2. No password auth

            sudo vi /etc/ssh/sshd_config -> NoPassWord

        3. Port forwarding and x11 disabled

            sudo vi /etc/ssh/sshd_config -> Port forwarding / X11

        4. PAM enabled

            sudo vi /etc/ssh/sshd_config -> PAM
            systemctl restart sshd
        
    4. Test the bastion host funcionality
        1. Login to the bastion with your user
        2. Become devops user
        3. Login to web1 and web2 without password auth
        4. Test password auth (it shouldn’t work)

            whoami
            su - Andor
            exit
            logout


3. Nameserver config and BIND
    1. Install and configure BIND name server on bastion host

        sudo apt-get install bind9 bind9utils bind9-doc
        sudo nano /etc/default/bind9 -> Add -4 to the end of the OPTIONS parameter.
        sudo systemctl restart bind9
        sudo nano /etc/bind/named.conf.options -> Configuring the Options File
        forwarders {8.8.8.8; 8.8.4.4;};

        1. It should serve the devops.local zone
            1. devops.local zone should contain the A record of web1,web2,bastion.devops.local

            vim /etc/bind/named.conf.local

                zone "devops.local" {
                    type master;
                    file "/etc/bind/db.devops.local";
                    allow-update { none; };
                };


        2. It should forward requests to 8.8.8.8 and 8.8.4.4 if not known

            vim /etc/bind/db.devops.local

             $TTL    86400
            @       IN      SOA     bastion.devops.local. admin.devops.local. (
                             2         ; Serial
                         604800         ; Refresh
                         86400         ; Retry
                         2419200         ; Expire
                         604800 )       ; Negative Cache TTL
            ;
            @       IN      NS      bastion.devops.local.
            bastion IN      A       172.31.46.58
            web1    IN      A       172.31.43.210
            web2    IN      A       172.31.42.152

            forwarders
            
    2. Configure name server on web1 with systems-resolver
        1. It should go to bastion host for DNS queries or to 1.1.1.1
        2. It should be able to resolv web1,web2,bastion without specifying the rest of FQDN.

        nmtui # :D

    3. Configure name server on web2 with dnsmasq
        1. It should go to bastion host for DNS queries or to 1.1.1.1
        2. It should be able to resolv web1,web2,bastion without specifying the rest of FQDN.

        vim /etc/netplan/00-installer-config.yaml

         nameservers:
            addresses: [192.168.1.10]
            search: [devops.local]

        netplan generate
        netplan apply

-------------------------------------------

sudo su - become root user

-------------------------------------------

ssh create

ssh-keygen

/.ssh/authorized_keys

copy the public key into this file

-------------------------------------------

ssh tunnel create

ssh-keygen
ssh-copy-id cloud_user@10.0.1.100
ssh -f cloud_user@10.0.1.100 -L 2000:10.0.1.100:80 -N

-------------------------------------------

systemctl status sshd.socket
sudo at now + 3 minutes
at> systemctl stop sshd.service
at> systemctl start sshd.socket
at> <EOT>
systemctl status sshd.socket
sudo systemctl enable sshd.socket
sudo systemctl disable sshd.service

ldd /usr/sbin/sshd | grep libwrap
/etc/hosts.allow
sshd2 sshd : ALL
/etc/hosts.deny
ALL : ALL

------------------------------------------
Installing and managing packages Debian/Ubuntu

wget http://localhost > local_index.response
ls -la
rm local_index.response
mv index.html local_index.response
sudo systemctl status apache2

------------------------------------------

Installing and managing packages Centos/Red Hat

ls -la
cd Downloads/
ls
sudo rpm -i elinks-0.12-0.37.pre6.el7.0.1.x86_64.rpm
yum provides libmozjs185*
sudo yum install js
sudo rpm -i elinks-0.12-0.37.pre6.el7.0.1.x86_64.rpm
yum provides libnss_compat_ossl*
sudo yum install nss_compat_ossl
sudo rpm -i elinks-0.12-0.37.pre6.el7.0.1.x86_64.rpm
elinks
history

-------------------------------------------
Managing RPM

First, let's become root:

sudo -i
Install the telnet package:

yum install -y telnet
Verify the integrity of the RPM database:

cd /var/lib/rpm/
/usr/lib/rpm/rpmdb_verify Packages
Move Packages to Packages.bad and create a new RPM database from Packages.bad:

mv Packages Packages.bad
/usr/lib/rpm/rpmdb_dump Packages.bad | /usr/lib/rpm/rpmdb_load Packages
Verify the integrity of the new RPM database:

/usr/lib/rpm/rpmdb_verify Packages
Query installed packages for errors:

rpm -qa > /dev/null
Rebuild the RPM database:

rpm -vv --rebuilddb
Install telnet:

yum install -y telnet

cd /var/lib/rpm
/usr/lib/rpm/rpmdb_verify Packages
mv Packages Packages.bad
/usr/lib/rpm/rpmdb_dump Packages.bad | /usr/lib/rpm/rpmdb_load Packages
/usr/lib/rpm/rpmdb_verify Packages
rpm -qa > /dev/null
rpm -vv --rebuilddb
yum install -y telnet
history

yum install -y httpd
grep 'httpd' /etc/yum.conf -> exclude:httpd
vim /etc/yum.conf -> comment out exclude:httpd in the conf file
yum install -y httpd

-------------------------------------------


