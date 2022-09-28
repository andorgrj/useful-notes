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
        