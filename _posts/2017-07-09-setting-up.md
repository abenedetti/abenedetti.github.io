---
layout: post
title: "Setting up the whole thing..."
date: 2017-07-09 12:00:00 +0300
image: 2017-07-09-setting-up/main.jpg
tags: [domain, vps, ubuntu, shiny, R]
categories: website
author: Alessio Benedetti
published: true
---

In this opening post I'd like to share my experience in building up this website. The goal is to recap the steps I followed, in order to set up the various elements. Since I'm not a developer nor a system administrator, I do not know if that is the correct way to proceed. However it reached what were my objectives, so this was ok for me.

## My initial needs
Last year I made a simple app called [uCount](http://thelab.alessiobenedetti.com/shiny/uCountVis/) which I published in the [shinyapps.io](https://www.shinyapps.io/) platform. Unfortunately the free tier offered only 25 hours a month for all the apps hosted in it. When I started to work on my second app [uGeoTagger](http://thelab.alessiobenedetti.com/shiny/uGeoTagger/) it became clear that this tier was insufficient. So was the next one which do not gave enoughs hours/month to host an app 24/7.

## Dean's guide
For that reason I googled a little bit and found [Dean's article](http://deanattali.com/2015/05/09/setup-rstudio-shiny-server-digital-ocean/) which provided guidance to install the free version of shiny server.

Dean is a shiny expert and by following his guide you'll be able to:
- set up your [virtual private server](https://en.wikipedia.org/wiki/Virtual_private_server)
- install [R](https://cran.r-project.org/) and [R Studio server](https://www.rstudio.com/products/rstudio/download-server/)
- install your [Shiny server](https://www.rstudio.com/products/shiny/shiny-server/)

## Setting up the custom domain
As described in Dean's article you may set a custom domain for your shiny server. However in my case, my domain (alessiobenedetti.com) was already used to host this blog in the github platform, therefore I needed to set up subdomains to bind the shiny server (thelab.alessiobenedetti.com).

- [using a custom domain with GitHub Pages](https://help.github.com/articles/using-a-custom-domain-with-github-pages/)
- setting up subdomains
	- first, in your registrar, you should add a new NS Record with the host portion of the subdomain, in my case I added the string "thelab" since my subdomain is "thelab.alessiobenedetti.com". You should also add the DNS of your host, in my case I put ["ns1.digitalocean.com"](https://www.digitalocean.com/)
	- then you'd have to configure a new subdomain (I work with digital ocean and therefore I've made my setup in the "networking" section, as shown in the picture below)
	
	![do_networking](/images/2017-07-09-setting-up/do_networking.png)
	
	- then you need to add the subdomain to your vps, by configuring the subdomain files in the following paths `/etc/nginx/sites-available/` and `/etc/nginx/sites-enabled/`
		- inside the `sites-available` I created a file named `thelab` where I specified the path through the index file and the subdomain, as shown in the image below		

		![nginx_config](/images/2017-07-09-setting-up/setting-up_nginx_config.png)
		
		- then in order to enable the configuration made in the previous step, inside the `sites-enabled` I created a symbolic link name `thelab` that references the `thelab` file in the sites-available` directory. This can be done with the following statement `ln -s /etc/nginx/sites-available/thelab /etc/nginx/sites-enabled/thelab`
		For more details you can refer to the [following guide](https://www.digitalocean.com/community/questions/how-to-create-subdomain-with-nginx-server-in-the-same-droplet) on digital ocean and this [post](https://askubuntu.com/questions/56339/how-to-create-a-soft-or-symbolic-link), I found useful to create symbolic links. 
	- third you need to configure your web server, in my case nginx, by adding a couple of new folders named *thelab* and *thelab/html* inside `/var/www`. In the resulting path `/var/www/thelab/html` you can add your index.html (I personally used [start bootstrap](https://startbootstrap.com/) templates which are nice and easy to customize)
	- fourth you have to restart nginx with the `service nginx restart` statement, in order to make those cahnges effective 
	
## Installing mongoDB
When I started developing the app I used a google sheet page as persistent storage. Despite R offers nice packages to manage google's docs, I wasn't satisfied of the loading time. That's why I moved to a DB and chose Mongo. For simplicity I initially connected the app to an online istance of Mongo at the [mlab.com](https://mlab.com/) hosting site. Later on, since the free tier of mlab.com doesn't allow backups/restore, unless you pay for them, I decided to install my own istance of mongoDB. This [guide](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04) was useful in doing that.
	

## Installing RoboMongo
Browsing through mongoDB is made easier through a GUI browser. I found [Robo 3T (formerly RoboMongo)](https://robomongo.org/). Setting up the connection to the remote mongoDB was a little bit tricky because of the bad format of the private key. You need to create an Open SSH key (see [this thread](https://github.com/Studio3T/robomongo/issues/484) for support)


## Setting up scheduled jobs to backups/restore the app DB
Last difficulty was automating the backup/restore jobs for the database. My wish was to create one daily backup, while keeping the last 5 backups day by day. I've had to dig on:
1) changing the directory of the statements in an ubuntu distro
2) setting up crontab job

Point 1) came from the fact that "shell scripts are run inside a subshell, and each subshell has its own concept of what the current directory is." ([stack overflow reference)](https://stackoverflow.com/questions/255414/why-doesnt-cd-work-in-a-bash-shell-script). Initially I was working with aliases but then I wasn't able to set the cron job. At the end I simply created a function in a sh script that was easier to schedule on crontab.


End of this journey! I've made a quick recap of the main points, hope that you can eventually find some useful hints!
