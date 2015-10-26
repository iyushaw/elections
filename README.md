# Elections DemTool - elections module
###ANALYZE ELECTION INFORMATION

The Elections tool can convincingly demonstrate the veracity of official election results. NDI’s election data management system aggregates and manages reports submitted by text or telephone from trained observers at polling stations across a country.  By analyzing the data instantaneously, election experts can spot trends, flag potential problems, and maintain easy, direct communication with observers in the field. Sophisticated automated analysis and visualization tools empower civic groups to document for political leaders, public officials and fellow citizens whether an election was, in fact, free and fair.

NDItech DemTools is funded under a grant by the National Endowment for Democracy.

The Elections is Python Flask platform that is configured to run on Docker. The full user and administrator documentation can be found at https://www.nditech.org/demtools/elections

## Install Instructions

####Audience
This documentation is designed for someone with basic Linux system administration skills.  By following the steps outlined below, someone with basic system administration skills should have no problems deploying a new Election instance.

####Getting Started
The preferred platform for easy distribution is Ubuntu Server 14.04 LTS.  This version of Ubuntu is a Long Term Support version and will remain the most recent LTS version until April, 2016 and will continue to be supported until it’s end of life date of April, 2019.  This guide uses digitalocean.com as an example for deploying to a virtual environment.  However, this application can also be installed in other environments such as Amazon’s EC2, which is the way NDI does it.  If you choose to use a different virtualization environment, you will need to install the Docker application manually.  Please visit http://www.docker.com for more information on how to install Docker on your platform of choice.

####Installation Steps

1.The first step is creating a new droplet using digitalocean.com.  A droplet is simply another name for a virtualized instance.  Choose the size of the instance that is appropriate for your deployment.  If you anticipate heavy traffic and utilization of the Elections software package, you may want to use a more powerful instance.


2.After firing up your new system, immediately do an update and upgrade on the system using the following command

    apt-get update && apt-get upgrade -y

3.Next, install nginx on the system.  Nginx is a powerful web server that is easy to configure and maintain.  Use the following command to install nginx:

    apt-get install -y nginx-full

4.Clone the Elections repo from Github using the following command:

    git clone https://github.com/nditech/elections.git

5.Once the repo has been cloned, change into the directory of the repo and build the docker image using:

    cd elections
    docker build -t apollo .

6.Pull the docker images for mongodb and redis with the following command:

    docker pull library/mongo
    docker pull library/redis

7.Create and run containers for redis and mongodb

    docker run -d --name database library/mongo
    docker run -d --name jobqueue library/redis

8.Next, we’ll want to create an environment configuration file for the Elections instance.  You can use the template at http://git.io/TAc4jg as a starting point and then edit as desired:
   
    wget -O ~/.env http://git.io/TAc4jg

9.We’re now ready to launch the Elections instance by linking the mongodb and redis containers we created earlier to it.  The following command will do that:

    cd $HOME; docker run -d --hostname apollo --name apollo --link jobqueue:redis --link database:mongodb -p 8000:5000 --env-file=.env apollo honcho start

10.Setup nginx using this template as a starting point -     :

    wget -O /etc/nginx/sites-available/elections http://git.io/6mRRMA
    rm /etc/nginx/sites-enabled/default
    ln -s /etc/nginx/sites-available/elections /etc/nginx/sites-enabled/elections
    service nginx restart

The following steps require that you start a container that (rather than spawn a running instance of the app) spawns a shell that you can use in interacting with the application’s backend via management commands

    cd $HOME; docker run --rm -t -i --link jobqueue:redis --link database:mongodb --env-file=.env apollo /bin/bash

11.Create Deployment

    ./manage.py create_deployment

Choose a name for the deployment (e.g. demo) and then also specify the hostname. The hostname must match the hostname of the server (e.g. www.apollo.la). This is a crucial step as Apollo is able to find related data only based on lookups of the hostname. If you have two deployments on the same server (same IP address) only the hostname will be used to distinguish between them.

13.Create Event

    ./manage.py create_event

Choose the deployment you want to create the event in, the name you want to call the event and the start and end dates for the event.

14.Create User Account

Once you’ve created a deployment and event, you can then create the default roles for the users in the deployment. You do this by visiting the home page of the application. Next, you create users by typing the command:

    ./manage.py create_user

Where you specify the deployment, email address and password (and confirmation password) of the account you wish to create. The username is the email address to the account that will be used.

After creating the user, you want to assign a role to the account you just created.

    ./manage.py add_userrole

The roles that exist in the system are admin, analyst, manager and clerk.

Roles start out without having any special permissions besides the admin role which has all access.

15.(Optional) Add/remove permissions to roles.  You can explicitly specify which permissions your roles have.

    ./manage.py add_rolepermission

and

    ./manage.py remove_rolepermission

The permissions available are:

view_event, view_messages, view_analyses, add_submission, edit_forms, edit_locations, edit_submission, edit_participant, edit_location, import_participant, import_locations, export_participants, export_messages, export_submissions, export_locations, send_messages.

####Dockerless Installation Of Election: 

The first step is creating a new droplet using digitalocean.com.  A droplet is simply another name for a virtualized instance.  Choose the size of the instance that is appropriate for your deployment.  If you anticipate heavy traffic and utilization of the Elections software package, you may want to use a more powerful instance. After firing up your new system:

1.Start by updating and installing any upgrades

    sudo apt-get update && sudo apt-get upgrade -y

2.Install nginx, mongodb and redis

    sudo apt-get install -y nginx-full mongodb redis-server

3.Install other dependencies

    sudo apt-get install -y build-essential file libxml2-dev libxslt1-dev vim python-dev python-setuptools git-core libz-dev

4.Install pip and virtualenv

    sudo easy_install pip

    sudo easy_install virtualenv

5.Ensure you have enough RAM
If you don’t have at least 1GB in RAM available, you can create sufficient swap space using the snippet below.

    sudo swapon -s

    sudo dd if=/dev/zero of=/swapfile bs=1024 count=1024k

    sudo mkswap /swapfile

    sudo swapon /swapfile

    echo "/swapfile       none    swap    sw      0       0" | sudo tee -a /etc/fstab

    echo 10 | sudo tee /proc/sys/vm/swappiness

    echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf

    sudo chown root:root /swapfile

    sudo chmod 0600 /swapfile

6.Create a deployment key and add to the bitbucket repository

    ssh-keygen -t rsa

7.Clone the repo

    git clone git@bitbucket.org:nationaldemocraticinstitute/pvt-apollo-generic.git

    cd pvt-apollo-generic

8.Create virtual environment and activate it

    virtualenv .

    source bin/activate

9.Install Apollo dependencies

    pip install -r requirements.txt

10.Create environment file for apollo instance using the template http://git.io/TAc4jg and edit as desired

    wget -O .env http://git.io/TAc4jg

11.Run Apollo instance

    bin/honcho start

12.Setup nginx using the template - http://git.io/VCsVtw
The template might require some customization including things like the port (honcho will normally run the application server on port 5000 so you might have to change this in the template)

    sudo wget -O /etc/nginx/sites-available/apollo http://git.io/VCsVtw

    sudo rm /etc/nginx/sites-enabled/default

    sudo ln -s /etc/nginx/sites-available/apollo /etc/nginx/sites-enabled/apollo

    sudo service nginx restart

Please note that you also have to configure SSL (which is beyond the scope of this document).

13.Create Deployment
Choose a name for the deployment (e.g. demo) and then also specify the hostname. The hostname must match the hostname of the server (e.g. www.apollo.la). This is a crucial step as Apollo is able to find related data only based on lookups of the hostname. If you have two deployments on the same server (same IP address) only the hostname will be used to distinguish between them.

  ./manage.py create_deployment

14.Create Event
Choose the deployment you want to create the event in, the name you want to call the event and the start and end dates for the event.

    ./manage.py create_event

15.Create User account
Once you’ve created a deployment and event, you can then create the default roles for the users in the deployment. You do this by visiting the home page of the application. Next, you create users by typing the command:

    ./manage.py create_user

Where you specify the deployment, email address and password (and confirmation password) of the account you wish to create. The username is the email address to the account that will be used.
After creating the user, you want to assign a role to the account you just created.

    ./manage.py add_userrole

The roles that exist in the system are admin, analyst, manager and clerk.
Roles start out without having any special permissions besides the admin role which has all access.

16.(Optional) Add/remove permissions to roles
You can explicitly specify which permissions your roles have.

    ./manage.py add_rolepermission

and 

    ./manage.py remove_rolepermission

The permissions available are:
view_event, view_messages, view_analyses, add_submission, edit_forms, edit_locations, edit_submission, edit_participant, edit_location, import_participant, import_locations, export_participants, export_messages, export_submissions, export_locations, send_messages.

