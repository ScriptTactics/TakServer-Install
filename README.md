# TAK Server Install

Instructions for installing TAK Server on CentOS 7

# CentOS
First you will need the [CentOS](http://isoredirect.centos.org/centos/7/isos/x86_64/) ISO (CentOS 7). Setup either a VM or install on baremetal.

Follow the prompts on the install, be sure to enable your networking on the install screen, and also set the install to be "infrastructure server".

Be sure to create an admin password and make the user you create an admin.

# TAKServer
Once your CentOS server is setup update the packages.

`sudo yum update && sudo yum upgrade -y`

Make sure git is installed

`sudo yum install git`

then clone the TakServer repo

`git clone https://github.com/TAK-Product-Center/Server.git`

You will also need to make sure Java 11 is installed. (JDK & JRE)

`sudo yum install java-11-openjdk-devel`

`sudo yum install java-11-openjdk`


You will also need to install patch

`sudo yum install patch`

As well as Postgres

`sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y`

Once those are installed you can build the project.


## Build Project

Navigate into the src directory and clean and build the project

```
cd src
./gradlew clean bootWar
```

If that completes you are ready to move on.

Run the following:

`./gradlew clean buildRpm`

This will generate the rpm image for you.


## Install RPM

Next navigate to the following directory

`cd Server/src/takserver-package/takserver/build/distributions`

In here you should see the server rpm:

`takserver-<version>-RELEASE<Number>.noarch.rpm`

At the time of writing this my rpm is `takserver-4.5-RELEASE72.noarch.rpm`

Run `sudo yum install takserver-4.5-RELEASE72.noarch.rpm` to install the server.


## Setup DB

There is a db install script pre-made that you will have to run.

`sudo /opt/tak/db-utils/takserver-setup-db.sh`


## Reload Service

After the db setup script is complete you can reload the services

`sudo systemctl daemon-reload`

At this point you can set TAK Server to start at boot

`sudo systemctl enable takeserver`


# Certificates

First you will have to become the `tak` user that is created.

`sudo su tak`

Then create env variables for the following:
```
export STATE=NY
export CITY=NYC
export ORGANIZATION=my-organizaton
export ORGANIZATIONAL_UNIT=my-unit
```

Navigate to 
`/opt/tak/certs/cert-metadata.sh`

Then run 
`./makeRootCa.sh`

Give a name for your CA: example-name


Create a server certificate:

`./makeCert.sh server takserver`


## Client Certs

For each client that you want on your network copy the following command and change the user.

`./makeCert.sh client user`

## Admin UI Cert

Generate an admin cert to gain access to the admin UI.

`./makeCert.sh client admin`


## Reload

After you have created the certs restart the TAK Server.

`sudo systemctl restart takserver`

Then authorize the admin cert.

`sudo java -jar /opt/tak/utils/UserManager.jar certmod -A /opt/tak/certs/files/admin.pem`


Also, the generated CA trustores and certs will be here:

`/opt/tak/certs/files`




