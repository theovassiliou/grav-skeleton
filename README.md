# Installing a default grav powered website

To start exploring the possibilities of [grav](http://getgrav.com) I recommend to install

- start from a grav skeleton
- to user ```docker``` for defining a webserver
- user ```docker-compose``` to link both together

For this example walkthrough I used the grav skeleton [Woo Site](https://github.com/getgrav/grav-skeleton-woo-site) as a starting point. But actually you could take any skeleton as presented at [Grav Skeletons](https://getgrav.org/downloads/skeletons)

## Create the local environment

You have unpacked the environment, and you are readinng this file. Well done. Nearlly there. In the following I will address the folder where this README is located as ```$PRJ```.

In ```$PRJ``` run

```sh
unzip grav-skeleton-woo-site-v1.0.1.zip
mv grav-skeleton-woo-site www
```

Voilà, you have your first grav installation. Althought not yet working, it is there. If you would have now a web-server, running the right version of php, etc. you could just use ```www``` as the webroot folder. We will create the webserver environment by using docker.

## Creating the webserver and linking to the required webroot

In the following I assume that you have a working docker installation running. Primarily the commands ```docker``` and ```docker-compose``` are available from command line.

In ```$PRJ``` run

```sh
docker compose up &
```

The command ```docker-compose up``` basically "runs" the commands as shown in the file ```docker-compose.yaml```

```yaml
version: '3'
services:
  www:
     build: .
     container_name: decyphirwww
     restart: always
     ports:
        - "5555:80"
     volumes:
        - ./www:/var/www/html

```

Besides the fact, that is uses the available ```Dockerfile``` build the container images, it configures the launch and networking of the freshly created container.

```yaml
ports:
 - "5555:80"
```

binds the (container internal) apache port 80 to (the external port) 5555, so that you can reach the webserver via 5555. You can modify this as to your needs.

```yaml
volumes:
        - ./www:/var/www/html
```

mounts the local directory ```./wwww``` as seen from the place where the ```docker-compose.yaml``` file lives to the internal path ```/var/www/html```. The apacher web-server defines this place as the webroot.

```yaml
     build: .
```

says nothing else, that you are using the ```Dockerfile``` at this location to build the docker image.

Use your webbrowser and go to [http:///localhost:5555](http:///localhost:5555) and see your website, so far.

## Configuring grav with the admin plugin

So far we have a basic installation of a grav-based website. You have to edit in ```$PRJ/www/``` in order to produce the content for your website.

Actually you can use a text-editor (I am using Visual Studio Code) to edit the files located for example under ```$PRJ/www/user/pages/01.home/modular.md```
grav uses markdown pages to define the layout and content of the website.

For a good and convient maintenance of the website a visual admin interface might be beneficial that we will install in the following.

In ```$PRJ``` run

```sh
docker exec -it decyphirwww bash
```

With this command you literally jump in the running container anmed decyphirwww and you have a bash to perform the next steps

In the container perform the following steps:

```console
root@b12c50e6dba7:/var/www/html# cd /var/www/html
root@b12c50e6dba7:/var/www/html# bin/gpm install admin

GPM Releases Configuration: Stable



The following dependencies need to be installed...
  |- Package form
  |- Package login
  |- Package email

Install these packages? [Y|n] Y
Preparing to install Form [v4.0.0]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok
  '- Success!  

Preparing to install Login [v3.0.4]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok
  '- Success!  

Preparing to install Email [v3.0.3]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok
  '- Success!  


Dependencies are OK

Preparing to install Admin Panel [v1.9.11]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok
  '- Success!  


Clearing cache

Cleared:  /var/www/html/cache/twig/*
Cleared:  /var/www/html/cache/doctrine/*
Cleared:  /var/www/html/cache/compiled/*
Cleared:  /var/www/html/images/*

Touched: /var/www/html/user/config/system.yaml

root@b12c50e6dba7:/var/www/html#
```

This is installs the grav admin plugin which is now available at [http://localhost:5555/admin](http://localhost:5555/admin).
Jump there NOW and create your admin account.

### Creating you first backup

Running a website is a nice thing, running it safely is even better. Grav provides you different means on how to this. My favorite is to use git as version control system and to be able to deploy changes via a simple ```git push``` from your local installation.

However this requires some more configuration. Perhaps it is worth writting an own small tutorial. But for the time being let's cretae a full backup of your webiste the goold, via command line,

As said we will use the command line. So jump again into the container, if not already there via:

```sh
docker exec -it decyphirwww bash
```

(NOTE: You could also use the admin interface for this purpose. But I like the commandline a little bit more, as I know what I am doing and what I am getting as a result.)

```console
root@b12c50e6dba7:/var/www/html# bin/grav backup

Grav Backup
===========

Archiving 5113 files [====================================================================================================] 100%  1 min Done...


 [OK] Backup Successfully Created: /var/www/html/backup/default_site_backup--20191120110855.zip


root@b12c50e6dba7:/var/www/html#
```

In your local shell in ```$PRJ``` I suggest that you run now

```console
cp www/backup/default_site_backup--20191120110855.zip ./
$ ls -la
  rwxr-xr-x  the  staff   256 B    Wed Nov 20 12:11:59 2019    ./
  rwxr-xr-x  the  staff   512 B    Wed Nov 20 11:29:19 2019    ../
  rwxrwxr-x  the  staff   800 B    Thu Nov  7 01:56:27 2019    www/
  rw-r--r--  the  staff     1 KiB  Wed Nov 20 10:53:47 2019    Dockerfile
  rw-r--r--  the  staff     5 KiB  Wed Nov 20 12:12:44 2019    README.md
  rw-r--r--  the  staff    14 MiB  Wed Nov 20 12:11:59 2019    default_site_backup--20191120110855.zip
  rw-r--r--  the  staff   175 B    Wed Nov 20 10:59:33 2019    docker-compose.yaml
  rw-r--r--  the  staff    10 MiB  Wed Nov 20 10:33:30 2019    grav-skeleton-woo-site-v1.0.1.zip
```

## Restoring your first backup

Are you brave? If yes than do the following in ```$PRJ```

```console
$rm -rf www
```

Well, what you just did is to destroy the whole installation of the grav-based website.  Verify this by calling [http://localhost:5555](http://localhost:5555)

```html
Not Found

The requested URL was not found on this server.
Apache/2.4.38 (Debian) Server at localhost Port 5555
```

But hold-on, we have a backup. A fresh backup. So what about running

```console
$ unzip default_site_backup--20191120110855.zip -d www
[...]
$ docker-compose down
Stopping decyphirwww ... done
Removing decyphirwww ... done
Removing network decyphir-www_default
$ docker-compose up &
Creating network "decyphir-www_default" with the default driver
Creating decyphirwww ... done
Attaching to decyphirwww
```

Hurray. You destroy and reanimated your complete website.

To double check that everything is up and running, go to [http://localhost:5555/admin](http://localhost:5555/admin) and log in. Yes, you can log in, as the whole website has been backuped, included any users, etc.

That's is for now. As you see this is not so much of a tutorial on how to *write* your website with grav, but how to get the playground going. As you have also seen, it is very easy to migrate this to a production environment. If you have something like the git-sync up and running, you might conider removing the admin plugin from your site. No login, less security hassles. But this are just my personal considerations.  Given the little experience I have with it, you might come up with completely differently solutions. 

Have a lot of fun, and stay tuned for a second part of this tutorial.
