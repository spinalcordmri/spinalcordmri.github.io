# https://spinalcordmri.org

This repo contains the source code and documentation for the https://spinalcordmri.org/ website. 

## Main website (`spinalcordmri.org`)

The main website is a simple landing page written using the Jekyll static site generator, and is deployed and hosted using GitHub Pages. The website uses a custom domain name purchased from NameCheap, then linked with GitHub Pages. (This way, instead of https://spinalcordmri.github.io, we get to use https://spinalcordmri.org.)

## Setting up the GitHub Pages site

Configuration settings can be found here:

- **GitHub Pages**: https://github.com/spinalcordmri/spinalcordmri.github.io/settings/pages
- **NameCheap**: https://ap.www.namecheap.com/Domains/DomainControlPanel/spinalcordmri.org/domain/

Configuration is relatively simple, and was done using instructions from the following pages:

- **GitHub Pages**: https://help.github.com/articles/quick-start-setting-up-a-custom-domain/
- **NameCheap**: https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages

## SCT Forum (`forum.spinalcordmri.org`)

The webforum ["forum.spinalcordmri.org"](https://forum.spinalcordmri.org/) is a subdomain of the spinalcordmri.org website. The forum was [originally envisioned](https://github.com/neuropoly/onboarding/issues/71#issuecomment-1008948573) as a general web forum and community for discussing MRI processing, acquisition, etc. That being said, the spinalcordmri.org forum is often colloquially referred to as the "SCT Forum", because the most active part of the forum is the ["SCT"](https://forum.spinalcordmri.org/c/sct/8) subsection. 

The forum runs using the open-source forum software Discourse, and operates within a VM provided by DigitalOcean. This means that it is an entirely separate site from the main website, and so the setup, configuration, and administration of the forum is a bit more involved than the Jekyll part of the site.

## Setting up the SCT Forum

If you need to remake the forum's VM from scratch (e.g. to debug an issue without impacting the production server), then follow the instructions below. 

> _**NB**: The following instructions are heavily based off of Discourse's official Cloud Installation instructions found [here](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md). If anything is unclear, it may be helpful to refer to those instructions for further guidance._

### 1. DigitalOcean

Create account & droplet in Digital Ocean. Droplet configure : 1GB RAM, 1 vCPU,25 GB HDD,	1 TB transfer, running Ubuntu 18.04-LTS.

### 2. Namecheap

  - To create a subdomain, please do the following:
    - Go to your Domain List and click Manage next to the domain
    - Select the Advanced DNS tab
    - Find the Host Records section and click on the Add New Record button
    - Select A Record for Type and enter the Host `forum.spinalcordmri.org`  you would like to point to an IP address `DigitalOcean_Server_IP_Address`

### 3. System Hostname

Make sure that the Droplet's `/etc/hostname` contains "forum.spinalcordmri.org".

### 4. Setup Discourse server

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

#### 5.2 Setup DNS for Email

1. Again, triple-check that `cat /etc/hostname` and `cat /etc/mailname` and `hostname` all return "forum.spinalcordmri.org"; **if not**, edit those two files manually, then **reboot** and check again.
1. In NameCheap, under the "forum.spinalcordmri.org" subdomain:
    1. Define the `MX` record: scroll to the email section, *set* it to "Custom MX" and write in `MX forum = forum.spinalcordmri.org, priority 0`.
        * to test: `dig MX forum.spinalcordmri.org` should return "forum.spinalcordmri.org"
    2. Set up [SPF](https://spfrecord.io/syntax/): again in namecheap, in the main records section, add `TXT forum. = "v=spf1 a mx ip4:159.89.119.65 ~all"`
        * to test: `dig TXT forum.spinalcordmri.org` should return the string above.
    3. DMARC: again in namecheap, add a record `TXT _dmarc.forum. = "v=DMARC1; p=none"`; but I'm not sure this achieves anything really.
        * to test: `dig TXT _dmarc.forum.spinalcordmri.org` should return the string above.
3. Reverse DNS: log in to the Droplet's control panel at DigitalOcean (DO) and *set the name of the Droplet* to "forum.spinalcordmri.org"; this [causes the reverse DNS to be defined](https://www.digitalocean.com/community/questions/how-do-i-set-up-reverse-dns-for-my-ip).
    * to test: `dig +short -x $(dig +short forum.spinalcordmri.org)` should return "forum.spinalcordmri.org".

#### 5.3 Configure mail server

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


#### 5.4 Test mail delivery

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

#### 5.5 Configure Discourse's email account

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
