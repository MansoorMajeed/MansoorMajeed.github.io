---
id: 729
title: 'SuperFast WordPress Part 1 : Setting up Nginx + PHP-FPM + WordPress'
date: 2016-04-12T19:57:40+00:00
author: Mansoor
layout: post
guid: https://digitz.org/blog/?p=729
permalink: /superfast-wordpress-part-1-nginx-php-fpm/
dsq_thread_id:
  - 4741136035
categories:
  - Blogging
  - Ubuntu
  - Wordpress
tags:
  - high speed
  - microcache
  - nginx
  - php-fpm
  - redis
  - Wordpress
---
Nginx is fast. How fast? It&#8217;s crazy fast, if you configure it properly. In this post, I will help you guys to set up a WordPress site powered by Nginx that can server 1000s of users at the same time ( Oh, I&#8217;m not even close to exaggerating, You&#8217;ll see 😉 )

What if I tell you that you can serve 3000 users per second ( or **250,000,000 &#8211; 250 Million clients per day** ) a WordPress site, without crashing the server, what would you say? Sure, you can, if you have a huge server powering your site. Hold that thought.

What if I tell you that you can achieve this with a 512MB VPS from Digital Ocean ( for 5 $ ) ?, you will probably call me crazy. 😀

Follow me, and I&#8217;ll show you exactly how to do that. 😉

#### Why Nginx and Not Apache?

In case you did not know, I hate to break it out to you. Apache is a RAM eating maniac. I&#8217;m sure Apache is great and all at what it can do. But, when it comes to resource usage, Apache will eat you alive. On the other hand, Nginx is the new kid around the block ( Not a Kid anymore though ). Nginx was designed to address the issues Apache has. For every request the client makes to the server, Apache either spawns a process or a thread ( depending on the MPM being used &#8211; more on that some other time ) and this costs so much resources and apache will die a slow and painful death in case of a traffic spike.

On the other hand, Nginx is an event driven web server. It excels at serving static files. It can handle 1000s of concurrent users without crashing the server. But, static files you say. Hmm. What about dynamic files like PHP files?. Well, Nginx cannot execute it. So we need a way to execute the PHP script and feed the result back to Nginx so that Nginx can serve the user with it. We use PHP-FPM for that. This works pretty well.

&nbsp;

#### The VPS I will be using:

  * Ubuntu 14.04
  * 1 CPU
  * 512MB RAM
  * 20GB SSD

This is the cheapest VPS from Digital Ocean. <a href="https://m.do.co/c/b63c500f6bcd" target="_blank">HERE</a> is my referral link (Sharing shamelessly 😉 ). Sign up using the link and you&#8217;ll get 10$ free credit, and I&#8217;ll get some credit too 😉

&nbsp;

With that said, let&#8217;s get to the matter.

#### Setting Up WordPress with Ngnix + PHP-FPM

##### 1. Install Nginx

Login to your server as root or sudo user via SSH. And execute the following commands. I recommend you to use a new test VM to test it first before you implement on production.

```bash
sudo apt-get update
sudo apt-get install nginx
sudo service nginx start
```

Visit your server&#8217;s IP address and you should see the Nginx welcome message.

##### 2. Install MySQL

<pre class="toolbar:2 lang:default decode:true">sudo apt-get install mysql-server

sudo mysql_install_db

sudo mysql_secure_installation

# And follow the instructions</pre>

##### 3. Install PHP-FPM

<pre class="toolbar:2 lang:default decode:true">sudo apt-get install php5-fpm php5-mysql php5-gd php5-cli
# Open /etc/php5/fpm/php.ini using your favorite text editor
# I'm gonna use "nano" for the sake of simplicity
nano /etc/php5/fpm/php.ini

# Add the following line
cgi.fix_pathinfo=0

# Save the file ( Ctrl+X and then type "Y" and press enter in case of nano )
# Restart PHP-FPM service
sudo service php5-fpm restart</pre>

At this point, you should have all the components required to run a WordPress server

##### 4. Configure Nginx to use PHP-FPM to process php files

Now that we have installed everything we need, now is the time to tell nginx to what to do with requests for php files.

Open the file &#8220;/etc/nginx/sites-available/default&#8221; using any text editor

<pre class="toolbar:2 lang:default decode:true"># use any text editor to open the conf file

nano /etc/nginx/sites-available/default

# Add the following content to the "server" block in the conf file

location ~ \.php$ {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

# The whole "Server" block should look something like below.

###########
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.php index.html index.htm;

    server_name server_domain_name_or_IP;

    location / {
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri $uri/ /index.php?q=$uri&$args;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
###############</pre>

Make sure that you replace &#8220;server\_domain\_name\_or\_IP&#8221; with your domain name.

Restart Nginx so that the configuration changes take effect

<pre class="toolbar:2 lang:default decode:true">sudo service nginx restart</pre>

##### 5. Make sure PHP is installed and configured properly

Now that we have installed everything, we have to make sure that everything is in place. Issue the following command

<pre class="toolbar:2 lang:default decode:true ">echo "&lt;?php phpinfo(); ?&gt;" &gt; /usr/share/nginx/html/info.php</pre>

Now, open &#8220;http://your-domain/info.php&#8221; in your browser and you should see the info page of PHP. Delete the info.php after that.

##### 6. Install WordPress in this

So you have php working. Now you can install WordPress. I am not going to tell you how you can do that. There are already a ton of tutorials out there on how to do that. Use [THIS digital ocean link](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-nginx-on-ubuntu-14-04) and install WordPress.

Alright, that&#8217;s it. Now you have Nginx running which can serve both static and dynamic content. And no Apache in the vicinity, Yayy!! 😀

Go ahead and check the RAM usage. I use the &#8220;htop&#8221; utility (sudo apt-get install htop) to check the usage. Now, that&#8217;s under 200MB of RAM used. Happy enough? 😉

In the next part of this series, we will add some caching and we will start our speed/stress testing to find out if my claims are actually true ( 250 million 😉 )

Read the Part 2 here : [Part 2 : Setting up Microcaching](https://digitz.org/blog/superfast-wordpress-part-2-nginx-fastcgi-microcaching/)

<div class="synved-social-container synved-social-container-share" style="text-align: center">
  <a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-facebook nolightbox" data-provider="facebook" target="_blank" rel="nofollow" title="Share on Facebook" href="http://www.facebook.com/sharer.php?u=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;t=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress&#038;s=100&#038;p&#091;url&#093;=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;p&#091;images&#093;&#091;0&#093;=&#038;p&#091;title&#093;=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="Facebook" title="Share on Facebook" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i0.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/facebook.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-twitter nolightbox" data-provider="twitter" target="_blank" rel="nofollow" title="Share on Twitter" href="http://twitter.com/share?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;text=Hey%20check%20this%20out" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="twitter" title="Share on Twitter" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/twitter.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-google_plus nolightbox" data-provider="google_plus" target="_blank" rel="nofollow" title="Share on Google+" href="https://plus.google.com/share?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="google_plus" title="Share on Google+" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/google_plus.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-reddit nolightbox" data-provider="reddit" target="_blank" rel="nofollow" title="Share on Reddit" href="http://www.reddit.com/submit?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;title=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="reddit" title="Share on Reddit" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i2.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/reddit.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-pinterest nolightbox" data-provider="pinterest" target="_blank" rel="nofollow" title="Pin it with Pinterest" href="http://pinterest.com/pin/create/button/?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;media=&#038;description=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="pinterest" title="Pin it with Pinterest" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i2.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/pinterest.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-linkedin nolightbox" data-provider="linkedin" target="_blank" rel="nofollow" title="Share on Linkedin" href="http://www.linkedin.com/shareArticle?mini=true&#038;url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;title=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="linkedin" title="Share on Linkedin" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/linkedin.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-mail nolightbox" data-provider="mail" rel="nofollow" title="Share by email" href="mailto:?subject=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress&#038;body=Hey%20check%20this%20out:%20https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;"><img alt="mail" title="Share by email" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/mail.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a>
</div>
