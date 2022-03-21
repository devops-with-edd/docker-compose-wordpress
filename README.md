If you are planning to host your blog or create a website, Wordpress is the solution of choice. It is the #1 CMS, with a very active community around it. Whatever features you want to add, you will certainly find the plugin you need.
WordPress is a bit slow at startup. This is dommageable for your users’ experience and SEO. Caching with Redis will help in that matter.
There are many ways to install WordPress. The one we speak about is primarily for those who know what Docker containers are and can use Linux trough SSH. If this is not your case, it might be simpler to go through a host that offers WordPress pre-installed.
If Docker or Docker Compose is not installed, it is relatively simple. Just follow the instructions in the documentation for Docker, then follow the instructions for Docker Compose.
I/ Installing Traefik
Traefik is a reverse proxy, load balancer and SSL manager. It is a reference tool in its field. At the level of a blogger, it is very useful if you have several sites on your server for example. If this is not your case and you don’t see how it can help you, skip this section. I will provide you with the necessary information to adapt the configuration.
First of all, let’s start by creating an external network that will allow all environments to communicate with Traefik :
docker network create traefik
Create a Traefik folder on your server :
mkdir traefik
cd traefik
In this folder, we will only add a docker-compose-.yml file. This file will contain all the configuration options for Traefik. Personally I prefer this approach, among the many Traefik offers, because it allows you to have everything in one place.

What this file does is configure Traefik to provide a free Let’s Encrypt SSH certificate and open the necessary ports.
Launch it with :
docker-compose up -d
That’s all. Traefik is now running in the background and ready to receive traffic…
II/ The docker-compose.yml of WordPress
It’s time to move on to the environment that will run WordPress.
Let’s create a “mysite” folder on the server and a WordPress folder inside :
mkdir mysite
cd mysite
mkdir wordpress
Add a Dockerfile to the WordPress folder:
FROM wordpress
# printf statement mocks answering the prompts from the pecl install
RUN printf "\n \n" | pecl install redis && docker-php-ext-enable redis
RUN /etc/init.d/apache2 restart
This file will simply take the official WordPress image and add a PHP module capable of handling Redis.
We can now go back to the parent folder “mysite” and add the file docker-compose.yml :

This Redis configuration allows you to store a certain volume of keys and then, once the limit is reached, delete the oldest ones. Here, Redis behaves similarly to MemCached.
We can launch the whole thing with:
docker-compose up -d
What if I don’t want to use Traefik?
If you don’t use Traefik you can remove all labels under wordpress and add a port instead to expose the WordPress container directly:
ports:
  - 8080:80
In our example, the SSL certificate was provided by Traefik. You will no longer have access to it and this is important for referencing and security.
The solution that seems to be the most used is to set up NGINX instead of Traefik. Not that it makes things simpler. If you have several sites on the server it will even be much more complicated.
Some hosts provides a certificate for free, like OVH with its SSL Gateway service. If your provider offers such services, this is often the easiest option.
III/ Configure WordPress
At this point if all goes well we can go to https://monsite.com/wp-admin and create an account for the administrator, then access WordPress.
Oh, that’s nice… That doesn’t mean it’s over.
For Redis to work, you’ll have to associate it with a plugin. I recommend W3 Total Cache which is the most popular and most complete.
For W3 Total Cache to discover Redis, it is necessary to fill in some variables in the mysite/wordpress/data/wp_config.php file, just before configuring the SQL database.
//
// redis config cache for total cache
//
define( 'W3TC_CONFIG_CACHE_ENGINE', 'redis');
define( 'W3TC_CONFIG_CACHE_REDIS_SERVERS', 'redis::6379' );
// optional redis settings
define( 'W3TC_CONFIG_CACHE_REDIS_PERSISTENT', true );
define( 'W3TC_CONFIG_CACHE_REDIS_DBID', 0 );
define( 'W3TC_CONFIG_CACHE_REDIS_PASSWORD', '' );
In Performance > General Settings, enable Redis for page (highly efficient), database, and object caching.
You will also find that sending emails does not work with the official WordPress container. This can be resolved by adding sendmail to the container. The limitation is that if your users report your mail as spam, it can damage the reputation of your domain. Many WordPress plugins allow you to use external mail services, such as WP Mail SMTP.
On the theme side, the leaders of the moment, such as Astra, Hello, OceanWP or Neve are making lightness a major selling point. Added to Redis, this allows you to take full advantage of the load time saving on SEO and user experience.
Several services offer to improve loading times by delivering media faster. There are CDNs such as StackPath or Cloudflare. Alternatively, you can use JetPack and enable options in Jetpack > Settings > Performance.
To help manage SEO, Yoast is a reference and is compatible with W3 Total Cache. Think about going to Performance > Extensions to allow it to integrate with the cache.