# Installing Asterisk SIP Server

Asterisk is a free and open source framework for building communications applications. This blog post explains how you can install Asterisk SIP Server. In the next blog post, I will explain how can you make simple VoIP call using this Asterisk setup.

## Prerequisites
A VM or a Machine having Ubuntu 20.04.6 LTS. Although you can take other Linux based distributions as well. Login to Ubuntu and install some useful and required packages.

```Shell
$ sudo apt install -y net-tools vim \
        curl git tree traceroute make \ 
        dos2unix build-essential
```

## Asterisk Installation
Create a directory and download Asterisk source code
```Shell
$ mkdir /usr/local/src/
$ cd /usr/local/src/
$ wget https://downloads.asterisk.org/pub/telephony/asterisk/releases/asterisk-20.3.0.tar.gz
$ tar -xzvf asterisk-20.3.0.tar.gz
```
Go into the source code and install prereqs.
```Shell
$ cd ./asterisk-20.3.0/contrib/scripts
$ ./install_prereq install
```
Above command may take some time.

Now do configure.
```Shell
$ cd /usr/local/src/asterisk-20.3.0
$ sudo ./configure
```

Now do menuselect. For now, just do ```Save & Exit``` to go with default options.
```Shell
$ make menuselect
```

And finally build the code and install it
```Shell
$ sudo make 
$ sudo make install
```

Create sample files with default configurations.
```Shell
$ sudo make samples
```
Above command creates samples files at ```/etc/asterisk/``` directory

Now install the initialization script or ```initscript```. This script starts Asterisk when your server starts, will monitor the Asterisk process in case anything bad happens to it, and can be used to stop or restart Asterisk as well.
```Shell
$ sudo make config
```

It is also recommended to install the ```logrotation``` script in order to compress and rotate those files to save disk space.
```Shell
$ sudo make install-logrotate
```

Last step, check if asterisk is running
```Shell
$ /etc/init.d/asterisk status
```

Gr8 😀. We are done with the installation.

## References
https://docs.asterisk.org/Getting-Started/Beginning-Asterisk/

