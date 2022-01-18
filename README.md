# Informix
This is an easy install guide of the informix database on
raspberry-pi. It is mostly a 1:1 from [here](https://www.raspberrypi.org/forums/viewtopic.php?f=37&t=97199)

## Overview 
- [download informix](#Installation-of-Informix)
- [Configuration and Initialization of a New Informix Instance](#Configuration-and-Initialization-of-a-New-Informix-Instance)
- [Create an Informix demo database](#Create-an-Informix-Demo-Database)



## Installation of Informix
- Download `Informix Developer Edition for Linux ARM v7 32 (Raspberry PI)`. Must be
that version and must be ARM enabled.
You can use [this](https://www.ibm.com/products/informix/editions) link.

- Copy the tar file for installation to your `raspberry` temporary folder: **scp examplefile yourusername@yourserver:/tmp** I.e. in my case:

```
scp ids.12.10.UC9DE.Linux-ARM7.tar pi@yipAdress:/tmp
```
- Then login into your raspberry and do the following:

- Create a temporary folder for the Informix install files

```
mkdir /tmp/ifxinstall
```
- Change directory to that folder:

```
cd /tmp/ifxinstall
```
-  Extract the Informix tar file - wait a bit until everything is
   unpacked and do not interrupt:

```
tar xvf /tmp/ids.12.10.UC9DE.Linux-ARM7.tar 
```
- create a new group informix:

```
sudo addgroup informix
```
-  (Ref:1) Create a new user informix (with the primary group
    informix). During this step you will be asked for a password for
    informix. Please take a note of that password. You will need it
    later.
```
 sudo adduser --ingroup informix informix
```
- Make sure to add the user informix to the /etc/sudoers file by adding the following line by using the ‘sudo visudo’ command:

```
informix ALL=(ALL) NOPASSWD: ALL
```
- Run the installation script

```
sudo ./ids_install
```
- Follow the UI and enter the following:

     + accept -> 1 RET

     + RET

     + specify directory where to install the products:
       `/opt/IBM/informix1210UC9DE`

     + accept -> 1 (wait a long bit and do not interrupt)

- Optional: As soon as the installation has successfully finished,
  you can delete the ifxinstall folder and the Informix install tar
  file if you want.

```
rm -rf /tmp/ifxinstall
rm /tmp/ds.12.10.UC9DE.Linux-ARM7.tar 
```
- Optional, but highly recommended: create the following symbolic link:

```
sudo ln -s /opt/IBM/informix1210UC9DE /opt/IBM/informix
```
- Create the folder which will later contain the Informix database files:

```
sudo mkdir /opt/IBM/ifxdata
```
- Set its ownership and permissions:

```
sudo chown informix:informix /opt/IBM/ifxdata
sudo chmod 770 /opt/IBM/ifxdata
```
## Configuration and Initialization of a New Informix Instance

- Login as the informix user
```
  su informix 
    
```
- And enter your password of (Ref:1).
Set the $INFORMIXDIR environment variable to point to the Informix install directory (actually to the symbolic link):

```
- export INFORMIXDIR=/opt/IBM/informix
    
```
- Extend the $PATH environment variable:

```
  export PATH=$PATH:$INFORMIXDIR/bin
    
```
- Set the $INFORMIXSERVER environment variable (you can choose any name here, but let’s use ol_informix1210 for now to keep it simple):

```
  export INFORMIXSERVER=ol_informix1210
    
```
- Create a new Informix configuration file:

```
  cp $INFORMIXDIR/etc/onconfig.std $INFORMIXDIR/etc/onconfig
    
```
- Create a new Informix hosts definition file:

```
  cp $INFORMIXDIR/etc/sqlhosts.demo $INFORMIXDIR/etc/sqlhosts
    
```
- Edit the file $INFORMIXDIR/etc/onconfig (with nano, vi or any other editor)
I.e. enter the file via the nano editor (or any other editor)

```
 nano $INFORMIXDIR/etc/onconfig

```
- And apply the following changes (use the search option ^W (control-W)):

```
  ROOTPATH /opt/IBM/ifxdata/rootdbs
  DBSERVERNAME	ol_informix1210
  LTAPEDEV	/dev/null
  TAPEDEV	  /dev/null
  LOGFILES	10

```
Save the file and exit the editor (Control-O RET; Control-X).

- Edit the file $INFORMIXDIR/etc/sqlhosts

```
  nano $INFORMIXDIR/etc/sqlhosts
    
```

- And add the following line:

```
  ol_informix1210	onsoctcp	localhost	9088
    
```
Note: 9088 is the port which will be used by Informix for the client/server communication. You can choose any available port you want. Save the file and exit the editor. 

- Create an empty database file and set the correct access mode:

```
  touch /opt/IBM/ifxdata/rootdbs
  chmod 660 /opt/IBM/ifxdata/rootdbs

```
- Now we are ready to initialize Informix for the first time:

```
  oninit -iv
    
```
- The first initialization will take a few minutes and it will create a few system databases automatically. You can monitor the pogress by doing the following:

```
  tail -f /opt/IBM/informix/tmp/online.log
    
```
Please wait until you see the following entry in the `online.log` file before you continue: **‘sysadmin’ database built successfully**


## Create an Informix Demo Database

