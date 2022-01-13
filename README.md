# Informix
This is an easy install guide of the informix database on
raspberry-pi. It is mostly a 1:1 from [here](https://www.raspberrypi.org/forums/viewtopic.php?f=37&t=97199)

## Overview 
- [download informix](#Step-1)



## Step 1
- Download `Informix Developer Edition for Linux ARM v7 32 (Raspberry PI)`. Must be
that version and must be ARM enabled.
You can use [this](https://www.ibm.com/products/informix/editions) link.

## Step 2
- Copy the tar file for installation to your `raspberry` temporary folder.

  #`
  scp examplefile yourusername@yourserver:/tmp
  #`
I.e. in my case:

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
sudo ln -s /opt/IBM/informix1210UC4DE /opt/IBM/informix
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
   
