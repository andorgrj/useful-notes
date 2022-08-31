### Demo

1. Install 3 VM to Acloud Guru playground
    1. CentOS -  web1.devops.local
    2. Ubuntu - web2.devops.local
    3. Ubuntu - bastion.devops.local
2. Users and SSH config
    1. Create user on every server called: Devops
        1. Home should be /home/devops
        2. shell: zsh
        3. It should have root privileges without password
        4. Create an ssh key and copy (authorize) on the two web server
        5. Steps for the creation:
            ## CENTOS7 (WEB1):
            
            sudo -i -> become a root
            usermod -g wheel <username> -> superuser 
            1 sudo adduser Devops
            2 sudo passwd Devops
            4 su - Devops
            6 pwd
            7 logou t
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
        forwarders

        1. It should serve the devops.local zone
            1. devops.local zone should contain the A record of web1,web2,bastion.devops.local

        

        2. It should forward requests to 8.8.8.8 and 8.8.4.4 if not known

            forwarders
            
    2. Configure name server on web1 with systems-resolver
        1. It should go to bastion host for DNS queries or to 1.1.1.1
        2. It should be able to resolv web1,web2,bastion without specifying the rest of FQDN.
    3. Configure name server on web2 with dnsmasq
        1. It should go to bastion host for DNS queries or to 1.1.1.1
        2. It should be able to resolv web1,web2,bastion without specifying the rest of FQDN.



4. NTP
    1. Install NTP on all 3 servers
    2. Bastion should be the NTP source for web1 and web2
    3. Check NTP sync state
5. Security
    1. Setup firewall on web1 and web2 so that only from bastion host you are able to SSH and reach the HTTP port.
    2. Setup auto patching for all 3 server.
6. Monitoring
   1. Create script that collects metrics to the following file /var/log/system_metrics.log
      1. load1 load5 load15
      2. free mem
      3. free swap
      4. disk usage
   2. Setup cron job on every server to run the script every 30 seconds.
   3. Setup logrotate on the file
      1. Maximum 100kB size
      2. Keeping 5 historical file in .tar.gz
7. Web service
   1. Install NGINX on every server
   2. Create an A record called www.devops.local
   3. Create a static website which been hosted on web1 and web2 with an index.html
   4. Setup bastion host to be a load balancer of www.devops.local zone in front of the web1 an web2.
   5. Check load balancing with shutting down web2.
8. Backup and NFS
    1. Install and configure NFS server on the bastion host
    2. Creat NFS share at /mnt/shared/web1 /mnt/shared/web2 for the another 2 server.
    3. Ensure that only web1 server is able to mount the /mnt/shared/web1 and same for web2
    4. The web1 and web2 servers should boot these NFS shared during the boot at /mnt/backup folder
    5. Create script that will backup the directories that are important from the web1 and web2 servers into the NFS shared.
    6. Ensure that the backups are beign rotated.
    7. Delete the web1 and restore from backup