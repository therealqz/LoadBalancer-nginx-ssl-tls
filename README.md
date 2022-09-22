#### LB-SOLUTION-WITH-NGINX-SSL-TLS 
#

![LbArchitecureNginx](/LB-Architecture.png)


####
CONFIGURE NGINX AS A Loadbalancer
#
#### 
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB 

Open ports 80 & 443 for HTTP & HTTPS connections

Update /etc/hosts file for local DNS with Web Servers’ names and their private ip addresses.
*webserver1* 
*webserver2*

![LB dns](/LB-hosts-file.png)

2. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers.

`sudo apt update`
`sudo apt install nginx`
Configure Nginx LB using Web Servers’ names defined in `/etc/hosts`




3. Rename the default nginx configuration file so nginx will not read it.(We're not using it). Instead create a new LB config file for our website. 

`sudo vi /etc/nginx/sites-available/load-balancer.conf`

Add this code 
````
 upstream myproject {
    server webserver1 weight=5;
    server Webserver2 weight=5;
  }

server {
    listen 80;
    server_name www.trooblebox.com;
    location / {
      proxy_pass http://myproject;
    }
  }

````
![LB config](/LB.conf.png)

#
####
REGISTER A DOMAIN NAME & CONFIGURE SECURE CONNECTION USING SSL/TLS
#

1. Register a domain Name
2. Assign an Elastic IP to the LB instance & associate the Elastic IP to the domain name for better access to the website.
3. Update A record in your Domain Name registrar to point the **root** address to Nginx LB using Elastic IP address (44.210.46.126)

4. To configure Nginx to recognize the new domain name,
update nginx OR load-balancer.conf created earlier with 

server_name www.<domain-name.com> instead of server_name www.domain.com

5. Check that the webservers  can be reached via a browser through the domain name through the HTTP protocol
http://trooblebox.com

![HTTP](/website-HTTP.png)

6. Install certbot and request for an SSL/TLS certificate.
Make sure snapd service is active and running

`sudo systemctl status snapd`
Install certbot

`sudo snap install --classic certbot`
Request your certificate ( follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step ).

`sudo ln -s /snap/bin/certbot /usr/bin/certbot`

`sudo certbot --nginx`

**_CERTBOT best practices_** :
Have a scheduled cronjob to renew the certificates twice daily

`sudo crontab -e`
Add this line 

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`



![](/installcertbot.png)

![](/ssl-confirmed.png)

Test secured access to your Web Solution:

 https://<your-domain-name.com>
https://www.trooblebox.com


You should be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.

![website-cert](/ssl-done.png)

