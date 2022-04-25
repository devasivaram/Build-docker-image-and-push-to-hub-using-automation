# Build-docker-image-and-push-to-hub-using-automation

## Description

Here we will build a docker image with different versions as we need and push it to docker hub using Ansible. This is performed in the local system, so we use "localhost" as host. We will also build a docker container from the image we pushed to docker hub and will run a template in that docker container.

## Prerequisites

1. Need Amazon LInux instances, installed with ansible
~~~sh
sudo amazon-linux-extras install ansible2 -y
~~~
2. Purchase a domain name or point you domain name to server IP. You can use this to purchase new [domain_name](http://www.freenom.com/en/index.html).
3.  Need to point A record for example.com pointing to your server’s public IP address.
4.  Docker hub accountn to push and pull image

## Steps

It will be great to create a directory for our project and we perform below steps in that directory, here I have crerated a direc tory with name "Project".

## Step 1 : Download template

Download an sample HTML template to the EC2 instance, you can use this link for getting [sample HTML](https://www.tooplate.com/) . 
Here we downloaded the template to /home/ec2-user/website/ location. This is used to creeate container with site template.

> To get sample HTML template

~~~sh
wget https://www.tooplate.com/zip-templates/2124_vertex.zip /home/ec2-user/project/website/
cd /home/ec2-user/project/website/
unzip 2124_vertex.zip
rm -rf 2124_vertex.zip
~~~

![image](https://user-images.githubusercontent.com/100773863/162551874-8a37fd2e-de7f-4737-8b57-0da2c3d47a3e.png)

So, we have the site template in /home/ec2-user/project/website/. Here I have changed the index of site template for specifying the version, I have add it under heading of home page.

~~~
vim website/index.html
~~~

>![image](https://user-images.githubusercontent.com/100773863/165044704-bec980f9-a934-49be-ac23-fda9948510d0.png)
 

## Step 2: Create Dockerfile

First we need to create a Dockerfile for building the docker image :

~~~sh
vim Dockerfile
~~~
Add:

~~~
FROM  httpd:alpine
    
COPY  ./website/  /usr/local/apache2/htdocs/

CMD   ["httpd-foreground"]
~~~

## Step 3: Create YML file for ansible

We need to write configuration file for ansible for performing docker image creation and image push to docker hub:

~~~sh
vim docker
~~~~
Added these:

~~~
---
- name: "creating docker image using ansible"
  become: true
  hosts: localhost
  connection: local
  vars:
    docker_user: ${your_docker_hub_username}
    docker_password: ${your_docker_hub_password}
    version: "1"                       
    hub_name: "devanandts/version"     
    packages:
       - docker
       - python2-pip
    user: ec2-user
  tasks:
    - name: "Installing Docker"
      yum:
        name: "{{ packages }}"
        state: present

    - name: "Installing docker client for python"
      pip:
        name: docker-py
        state: present

    - name: "Adding user {{ user }} to group docker"
      user:
        name: "{{ user }}"
        group: docker
        append: yes

    - name: "Restarting and enabling docker service"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "build image"
      docker_image:
        build:
          path: /home/ec2-user/project/    ------------------------------->> Your file path
        name: "{{hub_name}}{{version}}"
        tag: "{{ item }}"
        source: build
        push: yes
      with_items:
        - "{{version}}"
        - latest

    - name: Log into DockerHub
      docker_login:
        username: "{{ docker_user }}"
        password: "{{ docker_password }}"

    - name: "Adding a httpd container"     -------------------------------->> Creating docker container from pushed image
      docker_container:
        name: httpd-container1
        image: "{{hub_name}}{{ version }}"
        recreate: true
        state: started
        ports:
          - "80:80"
~~~

Here we used variables declared by "vars", The variables are:
  
  1. docker_user & docker_password: Need to substitute your docker hub credentials
  2. version: Need to specify your desired version number or tag
  3. hub_name: Need to put user docker hub username/(any variable)
  4. packages: Specified the necessary packgaes needed for our project
  5. User: specify your system user or desired user

## Step 4: Check syntax and execute playbook

Once the playbook is created, we need to check the syntax and execute the playbook:

~~~sh
# ansible-playbook docker --syntax-check
# ansible-playbook docker
~~~
Result:
> ![image](https://user-images.githubusercontent.com/100773863/165047613-e90c2548-f4f5-4c70-8500-da0baf742108.png)

Once those are successfull, we check check our docker hub to verify the images are pushed correctly and we can check the container is working by running domain name: example.com, Here my domain name is http://www.devanandts.tk

Result:
> ![image](https://user-images.githubusercontent.com/100773863/165048018-0cd73115-13d6-4eab-836b-751c9ce7ef6f.png)


## Conclusion

This is how an docker image is build and pushed to docker hub using ansible. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
