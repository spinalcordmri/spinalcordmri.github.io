# `spinalcordmri.org`

This repo contains the source code and documentation for the https://spinalcordmri.org/ website.

The main website is a simple landing page written using the Jekyll static site generator, and is deployed and hosted using GitHub Pages. The website uses a custom domain name purchased from NameCheap, then linked with GitHub Pages. (This way, instead of https://spinalcordmri.github.io, we get to use https://spinalcordmri.org.)

## Setting up the GitHub Pages site

Configuration settings can be found here:

- **GitHub Pages**: https://github.com/spinalcordmri/spinalcordmri.github.io/settings/pages
- **NameCheap**: https://ap.www.namecheap.com/Domains/DomainControlPanel/spinalcordmri.org/domain/

Configuration is relatively simple, and was done using instructions from the following pages:

- **GitHub Pages**: https://help.github.com/articles/quick-start-setting-up-a-custom-domain/
- **NameCheap**: https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages

# `forum.spinalcordmri.org`

The webforum ["forum.spinalcordmri.org"](https://forum.spinalcordmri.org/) is a subdomain of the spinalcordmri.org website. The forum was [originally envisioned](https://github.com/neuropoly/onboarding/issues/71#issuecomment-1008948573) as a general web forum and community for discussing MRI processing, acquisition, etc. That being said, the spinalcordmri.org forum is often colloquially referred to as the "SCT Forum", because the most active part of the forum is the ["SCT"](https://forum.spinalcordmri.org/c/sct/8) subsection. 

The forum runs using the open-source forum software Discourse, and operates within a VM provided by DigitalOcean. This means that it is an entirely separate site from the main website, and so the setup, configuration, and administration of the forum is a bit more involved than the Jekyll part of the site.

## Setting up the SCT Forum

If you need to remake the forum's VM from scratch (e.g. to debug an issue without impacting the production server), then follow the instructions below. 

> _**NB**: The following instructions are heavily based off of Discourse's official Cloud Installation instructions found [here](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md). If anything is unclear, it may be helpful to refer to those instructions for further guidance._

----

### 0. Domain name

Before you begin, first choose an [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (i.e. a domain name). This can be `forum.spinalcordmri.org` if you're starting completely from scratch (i.e. there is no currently-running forum instance) or something like `forum.dev.spinalcordmri.org` (if you're setting up a separate test server).

> _**NB**: For the rest of these instructions, we will use the `forum.dev.spinalcordmri.org` hostname, but make sure you substitute in whichever domain name you decided on._

----

### 1. DigitalOcean

First, make sure you have an account with DigitalOcean. Next, contact someone on the admin team; they will add you to the [NeuroPoly DigitalOcean team](https://cloud.digitalocean.com/account/team?i=fff1bd). This will grant you access to the [Spinal Cord MRI project](https://cloud.digitalocean.com/projects/12ec554b-d344-44bc-9973-c956f11d032b/resources?i=fff1bd). 


#### 1.1 Creating a new droplet

From the Spinal Cord MRI project page, you can create a new "droplet", which is DigitalOcean's name for cloud servers.

![image](https://user-images.githubusercontent.com/16181459/207157185-6cc381a1-762c-4db5-8350-e128aec93962.png)

Next, create a droplet using the following settings:

  - **Choose an image**: Distribution -> Ubuntu -> Version: 22.10 x64
  - **Choose a plan**: 
    - Shared CPU: Basic
    - CPU Options: Regular
    - Price: $6/mo (Specs: 1 GB/1 CPU, 25 GB SSD Disk, 1000 GB Transfer)
  - **Add block storage**: Skip this step
  - **Choose a datacenter region**: Toronto 1
  - **VPC Network**: Skip this step
  - **Authentication**: If your key isn't present yet, press "New SSH Key" and follow the instructions to authenticate the device you're currently using.
  - **Select additional options**: Check "Monitoring" and leave the rest blank.
  - **How many Droplets?**: 1 Droplet
  - **Choose a hostname**: This is a very critical step! Make sure you enter the same domain name you chose earlier (e.g. `forum.dev.spinalcordmri.org`)
  - **Add tags**: Skip this step

Once the droplet has finished creating, you should see a public IP address on the droplet's page that looks something like `ipv4: 142.93.152.255`. You can then use this IP address to test that you've set the hostname correctly by running the following command on any linux machine:

```console
user@device:~$ dig +short -x 142.93.152.255
forum.dev.spinalcordmri.org.
```

Once you've confirmed that it returns the domain name you chose, you're all set to connect to the droplet.

#### 1.2 Connecting to the droplet for the first time

You can connect to the server either through SSH (assuming your local device contains the SSH key you added earlier):

```console
user@device:~$ ssh root@forum.dev.spinalcordmri.org
Welcome to Ubuntu 22.10 (GNU/Linux 5.19.0-26-generic x86_64)

Last login: Fri Dec  9 23:06:11 2022 from 162.243.188.66
root@forum:~# 
```

Or, you can navigate to your droplet's main page, then click the "Console" button in the top-left corner:

![image](https://user-images.githubusercontent.com/16181459/207170763-68db34b0-4f1d-4e6d-a7b6-72340994c7ae.png)

From here, you can make changes to the server itself. 

----

### 2. Namecheap

Before we make any changes to the server, we first have to configure some DNS settings.

First, make sure you have an account with Namecheap. Again, you will need to contact someone on the admin team; they will grant you permissions for specific domains on a case-by-case basis. In this case, you will want access to the spinalcordmri.org domain, which will grant you access to the [Spinalcordmri.org Dashboard](https://ap.www.namecheap.com/domains/domaincontrolpanel/spinalcordmri.org/domain).

Next, on the dashboard, navigate to the [Advanced DNS tab](https://ap.www.namecheap.com/Domains/DomainControlPanel/spinalcordmri.org/advancedns). Then, modify the following sections:

#### 2.1 Host Records

First you will want to make note the subdomain label of your domain name. For `forum.dev.spinalcordmri.org`, you will use `forum.dev` to identify your subdomain.

Next, you will want to mark down the IP address you noted in Step 1. For `forum.dev.spinalcordmri.org`, the IP address was `142.93.152.255`.

You can now use the red "Add New Record" button to add 3 new records in the following form:

Type | Host | Value | TTL
-- | -- | -- | --
A Record | forum.dev | 142.93.152.255 | Automatic
TXT Record | forum.dev | v=spf1 a mx ip4:142.93.152.255 include:spf.efwd.registrar-servers.com ~all | Automatic
TXT Record | _dmarc.forum.dev | v=DMARC1; p=none | Automatic

These entries accomplish the following:

1. [A Record](https://support.dnsimple.com/articles/a-record/): Maps the `forum.dev.spinalcordmri.org` subdomain to the DigitalOcean droplet's IP address.
2. [SPF Record](https://spfrecord.io/syntax/): This will help later on when setting up the forum's mail server. It authorizes the DigitalOcean droplet as a valid sender of mail, which helps prevent the forum's notification emails from being caught as spam.
3. [DMARC Record](https://mxtoolbox.com/dmarc/details/what-is-a-dmarc-record): This will help later on when setting up the forum's mail server. It defines what should happen in case a message sent by the forum fails to be authenticated.

You can double-check the current values by running the following commands:

```console
user@device:~$ dig +short forum.spinalcordmri.org
142.93.152.255
user@device:~$ dig +short TXT forum.spinalcordmri.org
"v=spf1 a mx ip4:159.89.119.65 include:spf.efwd.registrar-servers.com ~all"
user@device:~$ dig +short TXT _dmarc.forum.dev.spinalcordmri.org
"v=DMARC1; p=none"
```

> _**NB**: If you make a mistake and want to update the values of these records, please note that it may take [up to a few hours](https://ns1.com/resources/dns-propagation) for changes to these records to propagate. So, if you're running `dig` and see no change, that's why!_

#### 2.2 Mail Settings (i.e. MX Records)

Next, scroll down to the "Mail Settings" section and find the table containing MX records. Then, add the following entry:

Type | Host | Mail Server | Priority | TTL
-- | -- | -- | -- | --
MX Record | forum.dev | forum.dev.spinalcordmri.org. | 0 | Automatic

Again, this setting will help us later when we set up our mail server. (There's not much here to talk about, since details such as 'priority' are only really relevant for setups with [multiple mail servers](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/).)

You can double-check the current value of this record by running the following command:

```console
user@device:~$ dig +short MX forum.dev.spinalcordmri.org
0 forum.dev.spinalcordmri.org.
```

----

### 3. Server Setup: Hostnames

Now that NameCheap is configured, we can set up the server itself, starting with its hostname.

First, connect to the server using either SSH or the built-in DigitalOcean console. Next, edit two files using the text editor of your choice:

```console
root@forum:~# sudo nano /etc/hostname
forum.dev.spinalcordmri.org
root@forum:~# sudo nano /etc/hostname
forum.dev.spinalcordmri.org
```

Make sure that both of these files contain only the domain name you chose. Additionally, ensure that the following command also returns the same domain name:

```console
root@forum:~# hostname
forum.dev.spinalcordmri.org
```

----

### 4. Server Setup: Discourse

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
Email         : [initial administrator's email address]
SMTP address  : [press Enter]
SMTP port     : [press Enter]
SMTP username : [press Enter]
SMTP password : [press Enter]
Let's Encrypt : [press Enter]
~~~

Note that this skips SMTP (email). We run a mail server on the same machine as Discourse, so there is a circular dependency that we need to side-step: the mail server relies on Discourse to generate a SSL certificate, but Discourse needs a mail server to operate.

Other ways to side-step this:
1. Run letsencrypt ourselves, outside of the Discourse container; make sure that works and then say "No" to the Let's Encrypt prompt.
1. Run a second letsencrypt account for the same domain outside of the Discourse container?
1. Run the mail server on a separate server e.g. `mail.spinalcordmri.org` with its own independent subdomain and certificates.


----

### 5. Setup Email

We run a small mail server on the same server as Discourse for it to send notifcations and password resets. Discourse recommends using a cloud service like MailGun or Amazon SES or SendGrid, but our usage is so small that the overhead (and risk) of outsourcing is high. Mail servers are something of an arcane art now, but never fear, these instructions will make it work.


Before continuing, make sure that Discourse has generated the SSL cert. It is in `/var/discourse/shared/standalone/ssl/`:

```
root@forum:~# ls -l /var/discourse/shared/standalone/ssl/forum.spinalcordmri.org.{cer,key}
-rw-r--r-- 1 root root 3799 Dec  5 08:33 /var/discourse/shared/standalone/ssl/forum.spinalcordmri.org.cer
-rw------- 1 root root 3247 Dec  5 08:33 /var/discourse/shared/standalone/ssl/forum.spinalcordmri.org.key
```


#### 5.1 Install mail server

Install [`opensmtpd`](https://www.opensmtpd.org/):

```
sudo apt-get install opensmtpd
```

The installer might(TODO: check on this) prompt you to name the system; make sure to tell it "forum.spinalcordmri.org".
Afterwards, make sure that `/etc/mailname` contains "forum.spinalcordmri.org".

**There is a bug** in the OpenSMTPd packaged for Ubuntu 18.04: https://bugs.launchpad.net/ubuntu/+source/opensmtpd/+bug/1840586. To work around it, apply this patch:

```
--- /lib/systemd/system/opensmtpd.service.old	2020-11-05 01:20:51.164473166 +0000
+++ /lib/systemd/system/opensmtpd.service	2020-11-03 21:22:34.309085523 +0000
@@ -6,7 +6,8 @@
 [Service]
 Type=forking
 ExecStart=/usr/sbin/smtpd
-ExecStop=/usr/sbin/smtpctl stop # backported fix for https://bugs.launchpad.net/ubuntu/+source/opensmtpd/+bug/1840586
+ExecStop=/bin/kill -15 $MAINPID
 
 [Install]
 WantedBy=multi-user.target
```

Despite this bug, setting up `opensmtpd` is still leagues simpler and more reliable than `postfix` or `sendmail`.

#### 5.2 Configure mail server

Put this into `/etc/smtpd.conf`:

```
pki forum.spinalcordmri.org certificate "/var/discourse/shared/standalone/ssl/forum.spinalcordmri.org.cer"
pki forum.spinalcordmri.org key "/var/discourse/shared/standalone/ssl/forum.spinalcordmri.org.key"

listen on eth0 tls-require pki forum.spinalcordmri.org auth-optional
listen on eth0 tls-require pki forum.spinalcordmri.org auth port 587
table aliases file:/etc/aliases
# incoming mail disabled until if/when we want https://meta.discourse.org/t/set-up-reply-via-email-support/14003
#accept from any for domain "forum.spinalcordmri.org" alias <aliases> deliver to maildir "~/.mail" 
accept for local alias <aliases> deliver to maildir "~/.mail" 
accept for any relay hostname "forum.spinalcordmri.org"
```

Enable the server with

```
systemctl enable --now opensmtpd
```

View the logs -- especially to look for configuration errors -- with

```
journalctl -f -u opensmtpd
```

(it helps to run this in a separate tab while doing the rest of the configuration and testing)


#### 5.3 Test mail delivery

At this point the mail server *should* be a member of the internet email community. To test, use:

```
echo "Test Message" | mail -s "This is a message" you@example.org
```

If you can, test with a few major email servers that we care about: "someone@polymtl.ca", "someoneelse@gmail.com", "antoinethethird@hotmail.com", "thefourthliest@yahoo.com". Generally, if the above has been done right, your message should get past the spam filters, and if it was done wrong it either won't send or will be caught by the spam filters.

https://www.mail-tester.com/ is very helpful for finding issues missed above, especially around spamminess. Email is very very complicated and this helps a lot.
Go to 
https://www.mail-tester.com/ and copy the email address it gives you, then run the same test but with it as a target:


```
echo "Test Message" | mail -s "This is a message" somethingsomething@mail-tester.com
```

then click the "View My Results" button.

Review until you have a good score and mails are getting accepted.

#### 5.4 Configure Discourse's email account

We need an SMTP account Discourse can send via. `opensmtpd` simply uses the OS's users by default, so we will make an OS user for outgoing emails. This username is *not* the same as what's on the email headers: `opensmtpd` allows authenticated users to spoof their identities, and we need actually want that because we want to send as `noreply@forum.spinalcordtoolbox.org`.

1. Run a password generator and save the result temporarily. If you have a password manager, see if it has a password generator built in. Otherwise there's [Diceware](https://www.rempe.us/diceware/#eff) and [xkpasswd](https://xkpasswd.net/s/) and [xkcdpass](https://pypi.org/project/xkcdpass/) and [pwgen](https://github.com/tytso/pwgen)
2. Create the user `forum@forum.spinalcordmri.org`;`useradd -s /usr/sbin/nologin forum && passwd forum`, inputting the saved password
3. Test:
    - install [`swaks`](https://www.jetmore.org/john/code/swaks/): `sudo apt-get install swaks`
    - 
      ```
      swaks --to me@example.com --from noreply@forum.spinalcordmri.org --server forum.spinalcordmri.org -p 25 --auth-user forum --tls-verify --tls
      Password: xxxxxxxxxxxxxxxxxxxxxxx
      === Trying forum.spinalcordmri.org:25...
      === Connected to forum.spinalcordmri.org.
      [...]
      <~  250 2.0.0: 8ccb62c7 Message accepted for delivery
      ~> QUIT
      <~  221 2.0.0: Bye
      ```
    - try varying `--to` and `--from` to see how various servers react
    - if you do not see "accepted" stop and debug until it works
4. Give Discourse the new SMTP credentials; this [is done](https://github.com/discourse/discourse/blob/dc49581344a9f0400a4b77e74331e269e995f648/docs/INSTALL-cloud.md#edit-discourse-configuration) by re-running the installer. The original values will be saved and prompted; add in the new credentials:
    -
      ```
      (cd /var/discourse; ./discourse-set
      Hostname      : forum.spinalcordmri.org
      Email         : [press enter]
      SMTP address  : forum.spinalcordmri.org
      SMTP port     : 587
      SMTP username : forum
      SMTP password : xxxxxxxxxxxxxxxxxxxxxxx
      Let's Encrypt : [press Enter]
      ```
5. Test: make a post on the forum, and have someone else reply to it. Watch the mail log (`journalctl -u -f opensmtpd`!) and check if you receive the notification in your inbox.

```

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
