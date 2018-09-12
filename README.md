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

### Reference
https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md

Create account & droplet in Digital Ocean. Droplet configure : 1GB RAM, 1 vCPU,25 GB HDD,	1 TB transfer.

### Setup email
  - Create an account on [SendGrid](https://sendgrid.com/)
  - Generate and copy to clipboard an API Key in SendGrid/Settings/API Keys

### Setup Discourse server
Connect to the droplet server provided by Digital Ocean, then do:
  * Install Docker:
~~~
wget -qO- https://get.docker.com/ | sh
~~~
- Clone Discourse deploy
~~~
mkdir /var/discourse
git clone https://github.com/discourse/discourse_docker.git /var/discourse
cd /var/discourse
~~~
- Install Discourse
~~~
./discourse-setup
Hostname      : forum.spinalcordmri.org
Email         : xxx@gmail.com
SMTP address  : smtp.sendgrid.net
SMTP port     : 587
SMTP username : apikey
SMTP password : [SendGrid API Key]
Let's Encrypt : [press Enter]
~~~
### Setup subdomain in namecheap
  - To create a subdomain, please do the following:
    - Go to your Domain List and click Manage next to the domain
    - Select the Advanced DNS tab
    - Find the Host Records section and click on the Add New Record button
    - Select A Record for Type and enter the Host `forum.spinalcordmri.org`  you would like to point to an IP address `DigitalOcean_Server_IP_Address`

### Configuring Google login for Discourse ([reference](https://meta.discourse.org/t/configuring-google-login-for-discourse/15858))

Go to https://console.developers.google.com, click on Credentials and create a new Project.
- Project name `Forum spinalcordmri`
- Project id `forum-spinalcordmri`

Select Credentials in the left menu, Create credentials and OAuth client ID type for the credentials.
- Application type `Web application`
- Name `Forum spinalcordmri.org`
- Authorized JavaScript origins `http://forum.spinalcordmri.org`
- Authorized redirect URIs `http://forum.spinalcordmri.org/auth/google_oauth2/callback`

Configure your OAuth Consent Screen
  - Product name shown to users `Forum spinalcordmri.org`
  - Homepage URL `http://www.spinalcordmri.org/`
  - Privacy policy URL `http://www.spinalcordmri.org/`

Click Library in the left menu and you’ll see a huge list of Google API’s. Find Google+ API and enable them.

The API will create `google_client_id` and `google_client_secret` which you can add under http://forum.spinalcordmri.org/admin/site_settings/category/login, after checking `enable google oauth2 logins`
### Configure GitHub login for Discourse ([reference](https://meta.discourse.org/t/configuring-github-login-for-discourse/13745))

Under github.com/spinalcordmri, click Settings (the gear icon), then look for OAuth Applications in the left menu. Select Register new application.
  - Application name
  ~~~
  Forum spinalcordmri
  ~~~
  - Homepage URL
  ~~~
  http://forum.spinalcordmri.org/
  ~~~
  - Application description
  ~~~
  Forum spinalcordmri
  ~~~
  - Authorization callback URL
  ~~~
  http://forum.spinalcordmri.org//auth/github/callback
  ~~~
The app will create `github_client_id` and `github_client_secret`which you can add under http://forum.spinalcordmri.org/admin/site_settings/category/login, after checking `enable github logins`

## Debugging

Check what IP are associated with the URL:
~~~
host spinalcordmri.org
~~~
Check that domain exists, and get info about registrar:
~~~
whois spinalcordmri.org
~~~