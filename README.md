# spinalcordmri.github.io

**THIS README IS A WORK IN PROGRESS**

## How to set up Jekyll

Nice tutorials here:
https://www.taniarascia.com/make-a-static-website-with-jekyll/

## Add domain
- Purchase domain, e.g. from Namecheap
- configure it to the GH page: https://help.github.com/articles/quick-start-setting-up-a-custom-domain/
- Link domain to GitHub Pages:  https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages

## Set up Discourse Forum

### Set up Compute Canada instance

- Create instance on Compute Canada
  - https://graham.cloud.computecanada.ca/dashboard/project/
  - HD space: 20GB min --> TO VALIDATE

- SSH permissions: add the "SSH" ingress rule to allow ssh into the instances:
https://drive.google.com/open?id=1QHcMx4N8a3HGs5quRsaUJZ4_FzO67RIX
add rules:
~~~
Ingress	IPv4	TCP	22 (SSH)	0.0.0.0/0
Ingress	IPv4	TCP	80 (HTTP)	0.0.0.0/0
Ingress	IPv4	TCP	443 (HTTPS)	0.0.0.0/0
~~~

To connect to server:
~~~
ssh ubuntu@X.X.X.X  # connect to floating IP
~~~

### Set up discourse
https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md

Connect to server, then do:
~~~
mkdir /var/discourse
git clone https://github.com/discourse/discourse_docker.git /var/discourse
cd /var/discourse
~~~

- add valid SPF and DKIM records in your DNS

- Create a DNS A record for the `forum.spinalcordmri.org` subdomain in your DNS control panel, pointing to the IP address of your cloud instance where you are installing Discourse.
  - Domain List > Manage > Advanced DNS > Add new record
  - Type=A record, Host=forum, IP=floatingIP, TTL=automatic
TODO: maybe add rule (DNS, HTTP...) to have discourse access server.
TODO: remove DNS or HTTP rule (added them for checking...)

Setup Discourse install:
~~~
./discourse-setup
Hostname      : forum.spinalcordmri.org
Email         : xxx@spinalcordmri.org
SMTP address  : smtp.elasticemail.com
SMTP port     : 2525
SMTP username : xxx  # get this from Elasticmail > Settings > SMTP/API
SMTP password : pa$$word  # get this from Elasticmail > Settings > SMTP/API
Let's Encrypt : [press Enter]
~~~

Launch
~~~
sudo ./launcher bootstrap app
~~~

## Debugging

Check what IP are associated with the URL:
```host spinalcordmri.org
```

Check that domain exists, and get info about registrar:
```whois spinalcordmri.org
```

Here we will be using Compute Canada cloud for running, but most of the explanations could be applied to another cloud instance.



-
