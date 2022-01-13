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