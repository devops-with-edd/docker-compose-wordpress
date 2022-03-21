If you are planning to host your blog or create a website, Wordpress is the solution of choice. It is the #1 CMS, with a very active community around it. Whatever features you want to add, you will certainly find the plugin you need.

WordPress is a bit slow at startup. This is dommageable for your users’ experience and SEO. Caching with Redis will help in that matter.
There are many ways to install WordPress. The one we speak about is primarily for those who know what Docker containers are and can use Linux trough SSH. If this is not your case, it might be simpler to go through a host that offers WordPress pre-installed.

If Docker or Docker Compose is not installed, it is relatively simple. Just follow the instructions in the documentation for Docker, then follow the instructions for Docker Compose.

I/ Installing Traefik

Traefik is a reverse proxy, load balancer and SSL manager. It is a reference tool in its field. At the level of a blogger, it is very useful if you have several sites on your server for example. If this is not your case and you don’t see how it can help you, skip this section. I will provide you with the necessary information to adapt the configuration.

First of all, let’s start by creating an external network that will allow all environments to communicate with Traefik :

        $ docker network create traefik

Create a Traefik folder on your server :

       $ mkdir traefik
       $ cd traefik

In this folder, we will only add a docker-compose-.yml file. This file will contain all the configuration options for Traefik. Personally I prefer this approach, among the many Traefik offers, because it allows you to have everything in one place.


