# Digital Ocean and R

## Security

You'll want to generate a public/private rsa key pair to ensure beef up security for people logging in to your server via ssh.  This can be done at the command line of any unix system:

```
ssh-keygen -t rsa
```

The output will be something like

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/garthtarr/.ssh/id_rsa): digitalocean
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in digitalocean.
Your public key has been saved in digitalocean.pub.
```

Where I specified the name `digitalocean` and provided a basic passphrase.  On my computer I stored these files in the folder `~/sshKeys`.   The public key file (`digitalocean.pub`) will be given to Digital Ocean when you initiate the droplet (server).

You keep the private key file (`digitalocean`) private (i.e. only given to people who need to access the server).  The private key needs to be registered on the computer you're trying to connect from.  This can be achieved using:

```
eval `ssh-agent -s`
ssh-add ~/sshKeys/digitalocean
```

Note that `~/sshKeys/digitalocean` is the path to the private key location, which may need adjusting depending on where you store your private keys.

## Spinning up a server

Super simple, just head over to digitalocean.com and get a server going. At the end of this process, you will be given the IP address of your new server (we'll use the example, 123.456.789.01).  You should then be able to ssh in using

```
ssh root@123.456.789.01
```

You'll want to set the root password

```
passwd
```

You'll want to create a non-root user

```
adduser username
```

You'll want to update the OS

```
sudo apt-get update
sudo apt-get upgrade
```

## R


Add R package source to ubuntu (http://cran.rstudio.com/bin/linux/ubuntu/README.html)

```
nano /etc/apt/sources.list
```

then add an entry `deb http://cran.rstudio.com/bin/linux/ubuntu trusty/`

Save and exit. Try to update the sources:

```
sudo apt-get update
```

You'll get an error message:

```
W: GPG error: http://cran.rstudio.com trusty/ Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 51716619E084DAB9
```

From the CRAN project README:

> The Ubuntu archives on CRAN are signed with the key of
> Michael Rutter marutter@gmail.com with key ID E084DAB9. To add
> the key to your system with one command use:

```
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
```

Now update the sources and install R:

```
sudo apt-get update
sudo apt-get install r-base
sudo apt-get install r-base-dev
```


## RStudio server

Follow the guide at http://www.rstudio.com/ide/download/server.html

```
sudo apt-get install gdebi-core
wget http://download2.rstudio.org/rstudio-server-0.99.441-amd64.deb
sudo gdebi rstudio-server-0.99.441-amd64.deb
```

All being well, we should be able to access RStudio remotely at:

http://:8787/

using the log in credentials of the non-root user we created earlier (you cannot log in to RStudio server as root).

## Shiny server

Now to set up shiny server (http://www.rstudio.com/shiny/server/install-opensource).

Install the shiny package as root

```
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
```

This opens R, and asks for shiny to be installed.  It will also install dependencies (including Rcpp which can take a while, and take some RAM - might need to increase the swap file size, though that wasn't necessary on the 1GB droplet I just fired up).

At this point you might as well install the rmarkdown package too:

```
sudo su - -c "R -e \"install.packages('rmarkdown', repos='http://cran.rstudio.com/')\""
```

Now we're ready to install shiny server:

```
sudo apt-get install gdebi-core
wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.3.0.403-amd64.deb
sudo gdebi shiny-server-1.3.0.403-amd64.deb
```

If all's well you should get a message:

```
shiny-server start/running, process 20874
```

and you should see the test page here: http://123.456.789.01:3838/


At which point we need to think about how our server is running and any security issues.  For a complete overview see the Shiny Server Administrators Guide: http://rstudio.github.io/shiny-server/latest/.


To guaranteed packages are available to the shiny server, I've found it's best to install them as root,

```
sudo su - -c "R -e \"install.packages('mplot', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('ggplot2', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('huge', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('pairsD3', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('networkD3', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('Hmisc', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('shinydashboard', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('mvoutlier', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('dplyr', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('tidyr', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('data.table', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('lubridate', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('readxl', repos='http://cran.rstudio.com/')\""
sudo su - -c "R -e \"install.packages('pacman', repos='http://cran.rstudio.com/')\""
```

## Install git

I find the best way to move shiny apps between my computer and the server is to use git (specifically github).  

```
sudo apt-get install git
```

Any apps that are going to be made public I write in a private github repository (RStudio project).  The first stage on the server is to clone the repository, then after that you can just use the `pull` command to update your apps.

## Authentication (optional)

Apache log in server needs to sit "in front" of the shiny server.

I adapted the instructions found here:
https://support.rstudio.com/hc/en-us/articles/200552326-Running-with-a-Proxy

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-proxy-html
sudo apt-get install libxml2-dev
```

Then, to update the Apache configuration files to activate mod_proxy you execute the following commands:

```
sudo a2enmod proxy
service apache2 restart
sudo a2enmod proxy_http
service apache2 restart
```

Edit the apache settings:

```
cd /etc/apache2/sites-available
nano 000-default.conf

	# ServerAdmin webmaster@localhost
	# DocumentRoot /var/www/html

	Redirect / /
        ProxyPass / http://localhost:3838/
        ProxyPassReverse / http://localhost:3838/
        ProxyPreserveHost On

        <Location />
             AuthType Basic
             AuthName "Restricted Access - Authenticate"
             AuthUserFile /etc/httpd/htpasswd.users
             Require valid-user
        </Location>

```

Edit the shiny server config file to work with this

```
sudo restart shiny-server
sudo start shiny-server
status shiny-server
```

Add users with htpasswd

```
apt-get install apache2-utils
cd /etc/
mkdir httpd
nano htpasswd.users
# save and exit
htpasswd /etc/httpd/htpasswd.users user1
htpasswd /etc/httpd/htpasswd.users user2
```

## Adding swap (optional)

Adding swap (sort of like extra RAM).  Check for swap:

```
swapon -s
```

Check the current storage state:

```
df
```

Let's add a gig of swap space (http://en.wikipedia.org/wiki/Dd_%28Unix%29#Block_size).

```
dd if=/dev/zero of=/swapfile bs=1024 count=1024k
```

Output:

```
1048576+0 records in
1048576+0 records out
1073741824 bytes (1.1 GB) copied, 3.65791 s, 294 MB/s
```

Create a swap area and activate:

```
mkswap /swapfile
swapon /swapfile
swapon -s
```

This swap file will hang around on the server until the machine reboots. To ensure it returns on restart, we need to add an entry to `/etc/fstab`

```
nano /etc/fstab
```

Immediately below the UUID=... line add the line:

```
/swapfile       none    swap    sw      0       0
```

The swap area has an attribute `swappiness` (http://en.wikipedia.org/wiki/Swappiness) which we would like to set to 10.

```
echo 10 | sudo tee /proc/sys/vm/swappiness
echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
```

Set permission of the swap area so only root has access:

```
chown root:root /swapfile
chmod 0600 /swapfile
```
