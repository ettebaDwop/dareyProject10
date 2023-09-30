# PROJECT 10
## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

#### Overview
A DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. In the previous project we configures a load balancer using Apache 2 in this project we will do the same LB configuration by using an alternative solution: Nginx Load Balancer.

To ensure that connections to our Web solutions are secure and information is encrypted in transit – we will cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

There are different types of SSL/TLS certificates – you can learn more about them here. You can also watch a tutorial on how SSL works here or an additional resource here

In this project you will register your website with LetsEnrcypt Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – cetrbot.


Our target architecture will look like this:
![Screenshot (588)](https://github.com/ettebaDwop/dareyProject10/assets/7973831/f6e3ea18-6e14-421f-aac3-4e98b194ee3c)


#### Task
This project consists of two parts:

1. Configure Nginx as a Load Balancer
2. Register a new domain name and configure secured connection using SSL/TLS certificates

#### Prerequisite
The following applications from previuos projects  should be up and running:

a. NFS SERVER
b. Web 1 and Web 2
c. Database Server
d. EC2 instance (Ubuntu server on AWS )
e. Domain name with the ability to configure the DNS settings.

#### Implementation

CONFIGURE NGINX AS A LOAD BALANCER
You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx. We will do the later here.

1. Create an EC2 VM based on Ubuntu Server 22.04 LTS and name it Nginx LB (Open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections).
   
   
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```

Configure Nginx LB using Web Servers’ names defined in */etc/hosts*


Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

#insert following configuration into http section
```
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
```
#comment out this line
#       include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.
![Elastic IP](https://github.com/ettebaDwop/dareyProject10/assets/7973831/6749f402-ee99-44e4-84c0-1e8b2f6d0449)

Update A record in your registrar to point to Nginx LB using Elastic IP address
![DNS_Etteware](https://github.com/ettebaDwop/dareyProject10/assets/7973831/501689f2-f153-4f79-85f1-10247d0ff289)

Check that our Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>
![etteware NS](https://github.com/ettebaDwop/dareyProject10/assets/7973831/969eeec7-abdf-4b84-955b-01adf02e60af)

On starting  NFS server, DataBase, Web1 and Web 2:

![tooling_etteware](https://github.com/ettebaDwop/dareyProject10/assets/7973831/7a49c2f0-4085-43e0-8304-75e2edc858a7)


* Configure Nginx to recognize your new domain name
  
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com



* Install certbot and request for an SSL/TLS certificate
To Make sure snapd service is active and running, run the command:

                          `sudo systemctl status snapd`

![nginx-snapd](https://github.com/ettebaDwop/dareyProject10/assets/7973831/5cb79199-e601-48fc-98d2-07bfd3715b35)

* To install certbot run the command:

              `sudo snap install --classic certbot`
  

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

```
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   sudo certbot --nginx
```

![certCreated](https://github.com/ettebaDwop/dareyProject10/assets/7973831/9d2d91d2-0624-4630-992b-e5091103121e)

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>
![login_Lockedtoolingette](https://github.com/ettebaDwop/dareyProject10/assets/7973831/9fedd84d-b5d7-4f99-95ea-26a15bb96942)

Login as admin:

![LockedToolingEtte](https://github.com/ettebaDwop/dareyProject10/assets/7973831/b35d1aa8-a835-4b0a-9f31-3556ace56ff1)

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.     


Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

sudo certbot renew --dry-run
Best practice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

crontab -e
Add following line:

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

Side Self Study: Refresh your cron configuration knowledge by watching this video.

You can also use this handy online cron expression editor.


