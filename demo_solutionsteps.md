### Linux Demo

1. Install 3 VM to Acloud Guru playground
    - web1.devops.local, CentOs7
    - web2.devops.local, Ubuntu 18.04
    - bastion.devops.local, Ubuntu 18.04

---

2. Users and SSH config

    Create user on every server called: Devops \
        - `sudo useradd -m Devops`

    Home should be /home/devops \
        - `sudo usermod -h /home/devops Devops`
    
    Shell: zsh \
        - `chsh -l` \
        - `yum install zsh`

    It should have root privileges without password \
        - `grep Devops /etc/passwd` \
        - `echo "Devops  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/Devops` \
            ```
            Devops  ALL=(ALL)       NOPASSWD: ALL
            ```

    Create an ssh key and copy (authorize) on the two web server \
        - `ssh-keygen` \
        - `cat /home/Devops/.ssh/id_rsa` \
        - `ls ~/.ssh/id*` \
        - `file .ssh/id_rsa.pub` \
        - `ssh-copy-id -i ~/.ssh/id_rsa.pub Devops@35.178.40.13` \
        - `ssh -i ~/.ssh/id_rsa.pub Devops@35.178.40.13` \
    
    Create user whit your username on bastion \
        - `sudo adduser Andor` \
        - `sudo adduser --force-badname Andor` \
        - `pwd` \
        - `sudo -l` \
        - `logout` \
        - `ssh-keygen` \
        - `ls -l ~/.ssh/` \
        - `ssh-copy-id -i ~/.ssh/id_rsa.pub Andor@18.170.115.86` \
        - `ssh Andor@18.170.115.86` \
        - `sudo visudo` \
            ```
            Andor ALL=NOPASSWD: /bin/su - Devops \
            ```

    SSH Config \
        - `sudo vi /etc/ssh/sshd_config` \
            ```
                PermitRootLogin no
                PasswordAuthentication no
                X11Forwarding no
                AllowTcpForwarding no
                UsePAM yes
            ``` \
        - `systemctl restart sshd`

    Test the bastion host funcionality \
        - `ssh Andor@UBUNTU IP` \
        - `sudo su - Devops`

---

3. Nameserver config and BIND \
    Install and configure BIND name server on bastion host \
        - `sudo apt-get install bind9 bind9utils bind9-doc` \
        - `sudo nano /etc/default/bind9` \
        - `sudo systemctl restart bind9` \
        - `sudo nano /etc/bind/named.conf.options` \
        - `forwarders {8.8.8.8; 8.8.4.4;};` \
        - `vim /etc/bind/named.conf.local` \  
            ```
                zone "devops.local" {
                    type master;
                    file "/etc/bind/db.devops.local";
                    allow-update { none; };
                };
            ``` \
        - `vim /etc/bind/forward.devops.local` \
            ```
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
            ``` \

    Configure name server on web1 with systems-resolver \
        - `sudo vim /etc/systemd/resolved.conf` \
            ```
            [Resolve]
            DNS=172.31.46.58
            FallbackDNS=1.1.1.1
            Domain=devops.local
            ``` \
        - `sudo systemctl restart systems-resolved` \
        - `sudo vim /etc/resolv.conf` \
            ```
            search devops.local
            nameserver 172.31.46.58
            ```
    
    Configure name server on web2 with dnsmasq \
        - `sudo systemctl disable systemd-resolved` \
        - `sudo systemctl stop systemd-resolved` \
        - `sudo unlink /etc/resolv.conf` \
        - `echo nameserver 8.8.8.8 | sudo tee /etc/resolv.conf` \
        - `sudo apt install dnsmasq` \
        - `sudo systemctl start dnsmasq` \ 
        - `cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak` \ 
        - `sudo vim  /etc/dnsmasq.conf` \
            ```
            port=53
            domain-needed
            bogus-priv
            dnssec
            strict-order
            expand-hosts
            domain=devops.local
            listen-address=127.0.0.1
            ``` \
        - `sudo systemctl restart dnsmasq` \
        - `sudom vim /etc/resolv.conf` \
            ```
            search devops.local
            nameserver 172.31.46.58
            nameserver 1.1.1.1
            ``` \
        - `netplan generate` \
        - `netplan apply`

---

4. NTP \
    Install NTP on all 3 servers \
        - `sudo apt-get install chrony`    [On Ubuntu] \
        - `sudo yum install chrony`       [On CentOS] \
        - `systemctl enable --now chronyd` \
        - `systemctl status chronyd` \
        - `chronyc sources -v` \
        - `vim /etc/chrony/chrony.conf` \
            ```
            server 0.europe.pool.ntp.org iburst
            server 1.europe.pool.ntp.org iburst
            server 2.europe.pool.ntp.org ibusrt
            server 3.europe.pool.ntp.org ibusrt
            ``` \
        - `systemctl restart chronyd` \
        - `chronyc sources` \
        - `timedatectl` \
        - `vim /etc/chrony.conf` \
            ```
            pool 172.31.46.58 burst
            ```
---

5. Security \
    Setup firewall on web1 and web2 so that only from bastion host you are able to SSH and reach the HTTP port. \
        - `sudo ufw allow from 172.31.46.58 to any port 22` \ 
        - `sudo ufw allow from 172.31.46.58 to any port 80`

    Setup auto patching for all 3 servers. \
        - `sudo yum install dnf-automatic` \
        - `sudo vim /etc/dnf/automatic.conf` \
            ```
            apply_updates = yes
            ```
        - `systemctl enable --now dnf-automatic.timer` \
        - `sudo apt install unattended-upgrades` \
        - `sudo apt install update-notifier-common` \
        Default conf on ubuntu is already enabled.

6. Monitoring \
    Create script that collects metrics to the following file /var/log/system_metrics.log \
        -`vim_system_metrics_auto.sh` \
            ```
            #!/bin/sh
            {
            echo >> /var/log/system-metrics.log;
            date >> /var/log/system-metrics.log;
            uptime | grep -o load.* >> /var/log/system-metrics.log;
            cat /proc/meminfo | grep MemFree >> /var/log/system-metrics.log;
            cat /proc/meminfo | grep SwapFree >> /var/log/system-metrics.log;
            df -h >> /var/log/system-metrics.log
            }
            ```
            \
    Cron job \
        - `30 * * * * /home/andor/system_metrics_auto.sh` \
    
    Log rotate \
        -`sudo vim /etc/logrotate.d/system_metrics_auto.sh` \
            ```
            /var/log/system-metrics.log {
            su root root
            hourly
            rotate 5
            maxsize 100k
            compress
            compressext .tgz
            dateext
            dateformat -%Y-%m-%d
            extension .log
            }
            ```
            \
7. Webservice \
    Install Nginx on every server \
        -`sudo apt install nginx` \
        -`sudo yum -y install nginx` \
        -`systemctl start nginx` \
    Create an A record called devops.local \
        -`sudo vim /etc/bind/fw.devops.local` \
            ```
            wwww    A       172.31.46.58
            ```
            \
    Create a static website which been hosted on web1 and web2 with an index.html \
        -`echo "<h1> website1 on WEB1 </h1>" > /var/www/html/index.html` \
        -`echo "<h1>Test website2 on WEB2</h1>" > /usr/share/nginx/html/index.html` \
        -`vim /etc/nginx/conf.d/devops.local.conf` \
            ```
            server {
            listen 80;
            server_name www.devops.local;
            root /var/www/html/;
            }
            ``` \
            ```
            server {
            listen 80;
            server_name www.devops.local;
            root /usr/share/nginx/html/;
            }
            ``` \
        -`systemctl restart nginx` \
    Setup bastion host to be a load balancer of www.devops.local zone in front of the web1 an web2. \
                ```
                upstream bastion {
                server web1.devops.local;
                server web2.devops.local; 
                }

                server {
                location /{
                proxy_pass http://bastion;
                    }
                }
                ``` \
    
8. Backup and NFS \
    Install and configure NFS server on the bastion host \
        -`sudo apt install nfs-kernel-server` \
    Creat NFS share at /mnt/shared/web1 /mnt/shared/web2 for the another 2 server \
        -`sudo mkdir -p /mnt/shared/web1` \
        -`sudo mkdir -p /mnt/shared/web2` \
    Ensure that only web1 server is able to mount the /mnt/shared/web1 and same for web2 \
        -`sudo vim /etc/exports` \
                ```
                /mnt/shared/web1 172.31.43.210(rw,sync,no_subtree_check,no_root_check)
                /mnt/shared/web2 172.31.45.146(rw,sync,no_subtree_check,no_root_check)
                ``` \
        -`sudo exportfs -a` \
        -`sudo systemctl restart nfs-kernel-server` \
    The web1 and web2 servers should mount these NFS shares during boot at /mnt/backup folder \
        -`sudo apt install nfs-common` \
        -`sudo mkdir -p /mnt/backup` \
        -`sudo mount 172.31.43.210:/mnt/shared/web1  /mnt/backup` \
        -`sudo mount 172.31.43.146:/mnt/shared/web2  /mnt/backup` \
        -`sudo vim /etc/fstab` \
                ```
                172.31.18.210:/mnt/shared/web1   /mnt/backup     nfs     defaults        0       0
                172.31.18.210:/mnt/shared/web2   /mnt/backup     nfs     defaults        0       0
                ```
                \
    Create script that will backup the directories that are important from the web1 and web2 servers into the NFS shared. Ensure that the backups are beign rotated. \
                ```
                #!/bin/bash
                date=$(date '+%Y-%m-%d')
                tar czf /mnt/backup/backup$date.tar.gz /var/www/devops.local /etc/nginx /etc/fstab /etc 2 > /dev/null
                find /mnt/backup -mtime +30 -name "*.tar.gz" -delete
                ```
                \
        -`vim /etc/cron.d/backup` \
                ```
                SHELL=/bin/bash
                PATH=/sbin:/bin:/usr/sbin:/usr/bin
                MAILTO=
                * * * * * root /opt/backup.sh
                ``` 
                \
    Backup rotating \
                ```
                find /mnt/backup -mtime +30 -name "*.tar.gz" -delete
                ```
                \
    Delete the web1 and restore from backup \
        -`tar -xpf /mnt/backup/<backup-filename.tar.gz> -C` \
        