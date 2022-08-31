The Dockerfile can be created in a number of ways, so long as it meets expectations. The following nine steps are suggested:
Images without parent are called base images

Define the desired container.
Update the packages on the container.
Install Nginx.
Update the Nginx configuration.
Create the location of the website files.
Set the working directory to that of the website.
Copy over website files, ensuring that Nginx is the owner.
Expose port 80.
Run Nginx at /tmp/nginx.pid; ensure Nginx is not set to run as a daemon.

# FROM alpine:3.13
# RUN apk upgrade
# RUN apk add nginx
# COPY files/default.conf /etc/nginx/conf.d/default.conf # in files website and config files are stored, replacing, copy them into nginx
# RUN mkdir -p /var/www/html                             # make sure our website files end up at /var/www/html
# WORKDIR /var/www/html                                  # desired directory to work as a root
# COPY --chown=nginx:nginx /files/html/ .                # copying files/html to the workdir, correct owner(ngninx) has access
# EXPOSE 80
# CMD [ "nginx", "-g", "pid /tmp/nginx.pid; daemon off;"]

Use docker build to generate an image based on the provided Dockerfile
Create a web01 container based on the image, ensuring port 80 on the container maps to 80 on the host and the container is launched in the background.

# docker build . -t web
# docker image ls
# docker run -dt -p 80:80 --name web01 web
# curl localhost


# WHAT DOCKER BUILD DOES STEP BY STEP:

Sending build context to Docker daemon  638.5kB
Step 1/9 : FROM alpine:3.13
3.13: Pulling from library/alpine
72cfd02ff4d0: Pull complete
Digest: sha256:100448e45467d4f3838fc8d95faab2965e22711b6edf67bbd8ec9c07f612b553
Status: Downloaded newer image for alpine:3.13
 ---> 6b5c5e00213a
Step 2/9 : RUN apk upgrade
 ---> Running in 09170f374ffc
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/community/x86_64/APKINDEX.tar.gz
OK: 6 MiB in 14 packages
Removing intermediate container 09170f374ffc
 ---> b383d268a48b
Step 3/9 : RUN apk add nginx
 ---> Running in 9ea0bca2e578
(1/2) Installing pcre (8.44-r0)
(2/2) Installing nginx (1.18.0-r15)
Executing nginx-1.18.0-r15.pre-install
Executing nginx-1.18.0-r15.post-install
Executing busybox-1.32.1-r9.trigger
OK: 7 MiB in 16 packages
Removing intermediate container 9ea0bca2e578
 ---> 45b7abd49683
Step 4/9 : COPY files/default.conf etc/nginx/conf.d/default.conf
 ---> ef02e88fddea
Step 5/9 : RUN mkdir -p /var/www/html
 ---> Running in 2ae509b90f53
Removing intermediate container 2ae509b90f53
 ---> 1e2f8b3d87d1
Step 6/9 : WORKDIR /var/www/html
 ---> Running in 1b2474998899
Removing intermediate container 1b2474998899
 ---> 4adcb7d5d69d
Step 7/9 : COPY --chown=nginx:nginx /files/html/ .
 ---> 37c4f467c1f0
Step 8/9 : EXPOSE 80
 ---> Running in bfd34e7dfdfa
Removing intermediate container bfd34e7dfdfa
 ---> 00a487b7b321
Step 9/9 : CMD [ "nginx", "-g", "pid /tmp/nginx.pid; daemon off;" ]
 ---> Running in 97c3961468a1
Removing intermediate container 97c3961468a1
 ---> 189f2a2c81f8
Successfully built 189f2a2c81f8
Successfully tagged web:latest

--------------------------------------------------------------------------------------------------------------

Creating a container image by hand allows you to quickly make and test images using Linux commands you already know. It will allow you to automate part of the process of creating a purpose-built container from an existing image. This technique is also useful for troubleshooting problems in existing containers. You will be able to save the state of an existing container as an image, similar to a Virtual Machine snapshot, then run containers from that image to debug your problem.


Get and Run the Base Image
1. Retrieve the httpd 2.4 image from Docker hub.
2. Start a container from the httpd image.
Install Tools and Code in the Container
1. Log in to the container.
2. Update the base image and install git.
3. Get website code from GitHub.
4. Remove the default index page and copy the website files to httpd's web serving directory.
5. Log out of the container.
Clean up the Template for a Second Version
1. Log in to the container.
2. Remove temporary files and installed utilities.
3. Log out of the container.
4. Find the template container's ID.
5. Create a new image named widgetfactory with version v2 from the container
6. View the image information.
7. Delete the v1 image since it is obsolete.
Run Multiple Containers from the Image
1. Start three containers from the widgetfactory:v2 image with different published web ports. The exposed ports should be in the 8000 to 8999 range.
2. View the running containers in Docker.
3. View the website from each container in a browser, using the three exposed web ports.





# dockeren belül -> docker exec -it webtemplate bash

    1  apt update && apt install git -y
    2  git clone https://github.com/linuxacademy/content-widget-factory-inc.git /tmp/widget-factory-inc
    3  ls -l /tmp/widget-factory-inc/
    4  ls -l htdocs/
    5  rm htdocs/index.html
    6  cp -r /tmp/widget-factory-inc/web/* htdocs/
    7  ls -l htdocs/
    8  rm -rf /tmp/widget-factory-inc/
    9  apt remove git -y && apt autoremove -y && apt clean

    1  docker pull httpd:2.4
    2  docker run --name webtemplate -d httpd:2.4
    3  docker ps
    4  docker exec -it webtemplate bash
    5  docker ps
    6  docker commit 43c2a89e9796 example/widgetfactory:v1
    7  docker images
    8  docker exec -it webtemplate bash
    9  docker ps
   10  docker commit 43c2a89e9796 example/widgetfactory:v2
   11  docker images
   12  docker rmi example/widgetfactory:v1
   13  docker images
   14  docker run -d --name web1 -p 8081:80 example/widgetfactory:v2
   16  docker run -d --name web2 -p 8082:80 example/widgetfactory:v2
   17  docker run -d --name web3 -p 8083:80 example/widgetfactory:v2
   18  docker ps
   19  docker stop webtemplate
   20  docker ps
   21  docker stop web1
   22  docker ps

-------------------------------------------------------

Only the instructions RUN, COPY, ADD create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.

Using RUN apt-get update && apt-get install -y ensures your Dockerfile installs the latest package versions with no further coding or manual intervention

COPY only supports the basic copying of local files into the container, while ADD has some features (like local-only tar extraction and remote URL support) that are not immediately obvious.


For example, you should avoid doing things like:

ADD https://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all

And instead, do something like:

RUN mkdir -p /usr/src/things \
    && curl -SL https://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all


-----------------------------------------------------------------

You are setting up a developer environment for the front end developers for your company's new shipping website using Docker. Website files have been provided (found at /home/cloud_user/webfiles/html), and you have been instructed to use Nginx as the underlying web server. Create a containered environment that runs the website on port 80. When finished, create an image of the container and ensure it will work as expected by launching a new container and mapping it to port 80 on the localhost.

Note that an nginx site configuration has been supplied at /home/cloud_user/webfiles/default.conf.

Create an Nginx Container:
Use the docker run command to launch a new container based on the nginx image with the name web. Ensure the container is running detached and has not been exited.

Configure Nginx:
Create the /var/www/ directory on the container, then copy the default nginx configuration in webfiles to /etc/nginx/conf.d. The websites files in webfiles/html must also be moved to /var/www/. Ensure the nginx user and group owns these directories, and that nginx is using the new configuration.

Test and Publish the Website to Port 80
Test that the configuration was successful by trying to access the website on the container. If successful, create an image based on the container and launch a new web01 that publishes to port 80 on the localhost.

# Commands:

   # 2   docker run --name web -dt nginx
   # 3   docker container ls
   # 4   cat webfiles/default.conf
   # 5   docker exec web mkdir /var/www
   # 6   docker cp webfiles/default.conf web:/etc/nginx/conf.d/default.conf
   # 7   docker cp webfiles/html/ web:/var/www/
   # 8   docker exec web ls /var/www/html
   # 9   docker exec web chown -R nginx:nginx /var/www/html
   # 10  docker exec web nginx -s reload
   # 11  docker inspect web | grep IPAddress
   # 12  curl 172.17.0.2
   # 13  docker commit web web-image
   # 14  docker run -dt --name web01 -p 80:80 web-image
   # 15  curl localhost



----------------------------------------------------------------------------------

        docker container ps
        docker pull httpd:2.4
        docker images
        docker run --name httpd -p 8080:80 -d httpd:2.4
        docker container ps
        git clone https://github.com/linuxacademy/content-widget-factory-inc
        docker ps
        docker stop httpd
        docker ps -a
        docker rm httpd
        docker ps -a
        docker run --name httpd -p 8080:80 -v $(pwd)/web:/usr/local/apache2/htdocs:ro -d httpd:2.4
        docker ps

        docker pull nginx
        docker images
        docker run --name nginx -p 8081:80 -d nginx
        docker ps
        docker stop nginx
        docker ps -a
        docker rm nginx
        docker ps -a
        docker run --name nginx -p 8081:80 -v $(pwd)/web:/usr/share/nginx/html:ro -d nginx
        docker ps

-------------------------------------------------------

    
        docker run --name web -dt nginx
        docker images
        docker container ls
        docker exec web mkdir /var/www
        docker cp webfiles/default.conf web:/etc/nginx/conf.d/default.conf
        docker cp webfiles/html/ web:/var/www/
        docker exec web ls /var/www/html
        docker exec web chown -R nginx:nginx /var/www/html
        docker exec web nginx -s reload
        docker inspect web | grep IPAddress
        curl 172.17.0.2
        docker commit web web-image
        docker run -dt --name web01 -p 80:80 web-image
        curl localhost
        