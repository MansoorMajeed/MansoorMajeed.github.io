---
title: 'SuperFast WordPress Part 2 : Setting up FastCGI microcaching + Load tests'
date: 2016-05-28T19:57:40+00:00
author: Mansoor
layout: post
permalink: /superfast-wordpress-part-2-nginx-fastcgi-microcaching/
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
This is the second part of a post series. Please read the first one, if you haven't:   
[Part 1 : Setting up Nginx + PHP-FPM + WordPress](https://digitz.org/blog/superfast-wordpress-part-1-nginx-php-fpm/)

Alright. So now we have our WordPress site up and running. But things aren't running as smooth as you'd wished. If you did a load test right now, you will probably get similar results as if you had Apache serving the pages. The reason is that each time a page is requested, the php-fpm process has to use the CPU cycles to execute the script, retrieve the post from the database and render a page that can be served back to the user.

Enter FastCGI microcaching. 

Unlike Apache, nginx does not have any way to execute dynamic files by itself. This is why we use php-fpm to execute php files. Nginx is really really good at serving static content though ( far better than Apache on any day ).
So, how do we make use of that ability of Nginx to serve static contents really well? Well, we can use Microcaching. 

### What's Microcaching?

Microcaching means caching for a small amount of time. For example, cache the files for 5 seconds. Well, you may ask, how does it make a difference?

Imagine the situation where your site ( let's say it's a WordPress site) is getting 50 concurrent users per second. Since you are not caching dynamic contents, your php-fpm process will have to hit the database "at least" 50 times in that second ( the actual number is way more than that though). This isn't good.  

So, instead of that, what if you hit the datbase once, and cache that dynamic content for 5 seconds? You don't have to hit the database in those 5 seconds anymore, and this makes a huuugee difference.  

Hope you get the idea.

### Setting up Fastcgi microcaching

Open `/etc/nginx/sites-available/default` using your favourite text editor

Make sure to backup the conf file before doing any modification
{% highlight bash %}
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
vim /etc/nginx/sites-available/default
{% endhighlight %}

Add the following to the top of the file. Make sure you add this outside the "server" block
{% highlight nginx %}
fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=MYSITE:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";
{% endhighlight %}

Now, inside the `location ~ \.php$ {` block, add the following

{% highlight nginx %}
fastcgi_cache MYSITE;
fastcgi_cache_valid 200 60m;
{% endhighlight %}

The whole configuration file should look something like below

{% highlight nginx %}
# Change 1 : FastCGI cache setup
fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=MYSITE:100m inactive=60m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";

server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.php index.html index.htm;

    server_name server_domain_name_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;

	# Change 2 
	fastcgi_cache MYSITE;
	fastcgi_cache_valid 200 60m;
    }
}
{% endhighlight %}

Now run `nginx -t` and see if there is any synatx error in your configuration. 
If all good, go ahead and reload nginx `service nginx reload`

You're all set with Nginx FastCGI microcaching. Your Ubuntu server running your WordPress site should now be able to
handle a couple hundred users at the same time ( or even a 1000 concurrent users, if you do it right ;) )

### Load test result
Now, this is the result of the load test I did using [Loader.io](http://loader.io) 

![Nginx Load test]({{ site.url }}/blog/assets/nginx/nginx-3000.png)

Did you notice the number "176990" that's the number of successful responses over 1 minute.
Also, 3 gigs of traffic sent in a minute. And, not even one failed request. This is pretty good, isn't?

This is not a 512MB ram VPS. It's a bit more powerful than that. But I was able to get around
2000 clients per second on a digitalocean 512MB RAM VPS, on my WordPress blog. 

So, let's do the math for 2000 clients per second instead of 3000 ( just give a margin of 1000 - oh yeah )

    2000 clients per second
    2000 * 60 = 120000 clients per minute
    120000 * 60 = 7200000 clients per hour
    7200000 * 24 = 172,800,000 clients per day

Well, damn! That's a big number. Now I am skeptical about this load test, tbh. :D
So, there you go. That's it for now. See you around. Leave a comment if you have a question. 

--
<div class="synved-social-container synved-social-container-share" style="text-align: center">
  <a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-facebook nolightbox" data-provider="facebook" target="_blank" rel="nofollow" title="Share on Facebook" href="http://www.facebook.com/sharer.php?u=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;t=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress&#038;s=100&#038;p&#091;url&#093;=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;p&#091;images&#093;&#091;0&#093;=&#038;p&#091;title&#093;=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="Facebook" title="Share on Facebook" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i0.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/facebook.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-twitter nolightbox" data-provider="twitter" target="_blank" rel="nofollow" title="Share on Twitter" href="http://twitter.com/share?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;text=Hey%20check%20this%20out" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="twitter" title="Share on Twitter" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/twitter.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-google_plus nolightbox" data-provider="google_plus" target="_blank" rel="nofollow" title="Share on Google+" href="https://plus.google.com/share?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="google_plus" title="Share on Google+" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/google_plus.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-reddit nolightbox" data-provider="reddit" target="_blank" rel="nofollow" title="Share on Reddit" href="http://www.reddit.com/submit?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;title=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="reddit" title="Share on Reddit" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i2.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/reddit.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-pinterest nolightbox" data-provider="pinterest" target="_blank" rel="nofollow" title="Pin it with Pinterest" href="http://pinterest.com/pin/create/button/?url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;media=&#038;description=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="pinterest" title="Pin it with Pinterest" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i2.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/pinterest.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-linkedin nolightbox" data-provider="linkedin" target="_blank" rel="nofollow" title="Share on Linkedin" href="http://www.linkedin.com/shareArticle?mini=true&#038;url=https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll&#038;title=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;margin-right:5px;"><img alt="linkedin" title="Share on Linkedin" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/linkedin.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a><a class="synved-social-button synved-social-button-share synved-social-size-48 synved-social-resolution-single synved-social-provider-mail nolightbox" data-provider="mail" rel="nofollow" title="Share by email" href="mailto:?subject=SuperFast%20WordPress%20Part%201%20%3A%20Setting%20up%20Nginx%20%2B%20PHP-FPM%20%2B%20WordPress&#038;body=Hey%20check%20this%20out:%20https%3A%2F%2Fdigitz.org%2Fblog%2Fwp-admin%2Fexport.php%3Ftype%3Djekyll" style="font-size: 0px; width:48px;height:48px;margin:0;margin-bottom:5px;"><img alt="mail" title="Share by email" class="synved-share-image synved-social-image synved-social-image-share" style="display: inline; width:48px;height:48px; margin: 0; padding: 0; border: none; box-shadow: none;" src="https://i1.wp.com/digitz.org/blog/wp-content/plugins/social-media-feather/synved-social/image/social/regular/96x96/mail.png?resize=48%2C48&#038;ssl=1" data-recalc-dims="1" /></a>
</div>
