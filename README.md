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

## Local build

The page can be built locally after installing Jekyll. See more details [here](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll). 


# `forum.spinalcordmri.org`

The webforum ["forum.spinalcordmri.org"](https://forum.spinalcordmri.org/) is a subdomain of the spinalcordmri.org website. The forum was [originally envisioned](https://github.com/neuropoly/onboarding/issues/71#issuecomment-1008948573) as a general web forum and community for discussing MRI processing, acquisition, etc. That being said, the spinalcordmri.org forum is often colloquially referred to as the "SCT Forum", because the most active part of the forum is the ["SCT"](https://forum.spinalcordmri.org/c/sct/8) subsection. 

The forum runs using the open-source forum software Discourse, and operates within a VM provided by DigitalOcean. This means that it is an entirely separate site from the main website, and so the setup, configuration, and administration of the forum is a bit more involved than the Jekyll part of the site.

## Setting up the SCT Forum

If you need to remake the forum's VM from scratch (e.g. to debug an issue without impacting the production server), then follow the instructions below. 

> _**NB**: The following instructions are heavily based off of Discourse's official Cloud Installation instructions found [here](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md). If anything is unclear, it may be helpful to refer to those instructions for further guidance._

----

### Table of contents

0. [Domain name](#0-domain-name)
1. [Digital Ocean](#1-digitalocean)
2. [Namecheap](#2-namecheap)
3. [Hostnames](#3-server-setup-hostnames)
4. [Email](#4-server-setup-email)
5. [Discourse](#5-server-setup-discourse)
6. [Google/GitHub logins](#6-googlegithub-logins-for-discourse)

----

### 0. Domain name

Before you begin, first choose an [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (i.e. a domain name). This can be `forum.spinalcordmri.org` if you're starting completely from scratch (i.e. there is no currently-running forum instance) or something like `forum.dev.spinalcordmri.org` (if you're setting up a separate test server).

> _**NB**: For the rest of these instructions, we will use the placeholder `{subdomain}.spinalcordmri.org`, but make sure you substitute in whichever domain name you decided on._

----

### 1. DigitalOcean

First, make sure you have an account with DigitalOcean. Next, contact someone on the admin team; they will add you to the [NeuroPoly DigitalOcean team](https://cloud.digitalocean.com/account/team?i=fff1bd). This will grant you access to the [Spinal Cord MRI project](https://cloud.digitalocean.com/projects/12ec554b-d344-44bc-9973-c956f11d032b/resources?i=fff1bd). 


#### 1.1 Creating a new droplet

From the Spinal Cord MRI project page, you can create a new "droplet", which is DigitalOcean's name for cloud servers.

![image](https://user-images.githubusercontent.com/16181459/207157185-6cc381a1-762c-4db5-8350-e128aec93962.png)

Next, create a droplet using the following settings:

  - **Choose an image**: Distribution -> Ubuntu -> Version: 22.04 (LTS) x64
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
  - **Choose a hostname**: This is a very critical step! Make sure you enter the same domain name you chose earlier (`{subdomain}.spinalcordmri.org`)
  - **Add tags**: Skip this step

Once the droplet has finished creating, you should see a public IP address on the droplet's page that looks something like `ipv4: 142.93.152.255`. You can then use this IP address to test that you've set the hostname correctly by running the following command on any linux machine:

```console
user@device:~$ dig +short -x 142.93.152.255
{subdomain}.spinalcordmri.org.
```

Once you've confirmed that it returns the domain name you chose, you're all set to connect to the droplet.

----

### 2. Namecheap

Before we make any changes to the server, though, we first have to configure some DNS settings.

First, make sure you have an account with Namecheap. Again, you will need to contact someone on the admin team; they will grant you permissions for specific domains on a case-by-case basis. In this case, you will want access to the spinalcordmri.org domain, which will grant you access to the [Spinalcordmri.org Dashboard](https://ap.www.namecheap.com/domains/domaincontrolpanel/spinalcordmri.org/domain).

Next, on the dashboard, navigate to the [Advanced DNS tab](https://ap.www.namecheap.com/Domains/DomainControlPanel/spinalcordmri.org/advancedns). Then, modify the following sections:

#### 2.1 Host Records

First you will want to make note of two things:

1. The subdomain label of your domain name. For `{subdomain}.spinalcordmri.org`, you will use `{subdomain}` to identify your subdomain.
2. The IP address of the server (from step 1). For `{subdomain}.spinalcordmri.org`, the IP address was `142.93.152.255`.

You can now use the red "Add New Record" button to add 3 new records in the following form:

Type | Host | Value | TTL
-- | -- | -- | --
A Record | {subdomain} | 142.93.152.255 | Automatic
TXT Record | {subdomain} | v=spf1 a mx ip4:142.93.152.255 ~all | Automatic
TXT Record | _dmarc.{subdomain} | v=DMARC1; p=none | Automatic

These entries accomplish the following:

1. [A Record](https://support.dnsimple.com/articles/a-record/): Maps the `{subdomain}.spinalcordmri.org` subdomain to the DigitalOcean droplet's IP address.
2. [SPF Record](https://spfrecord.io/syntax/): This will help later on when setting up the forum's mail server. It authorizes the DigitalOcean droplet as a valid sender of mail, which helps prevent the forum's notification emails from being caught as spam.
3. [DMARC Record](https://mxtoolbox.com/dmarc/details/what-is-a-dmarc-record): This will help later on when setting up the forum's mail server. It defines what should happen in case a message sent by the forum fails to be authenticated.

You can double-check the current values by running the following `dig` commands **from an external device**, not the server itself:

```console
user@device:~$ dig +short {subdomain}.spinalcordmri.org
142.93.152.255
user@device:~$ dig +short TXT {subdomain}.spinalcordmri.org
"v=spf1 a mx ip4:159.89.119.65 include:spf.efwd.registrar-servers.com ~all"
user@device:~$ dig +short TXT _dmarc.{subdomain}.spinalcordmri.org
"v=DMARC1; p=none"
```

> _**NB**: Please note that it may take [up to a few hours](https://ns1.com/resources/dns-propagation) for changes to these records to propagate. So, if you're running `dig` and see no change, that's why!_

#### 2.2 Mail Settings (i.e. MX Records)

Next, scroll down to the "Mail Settings" section and find the table containing MX records. Then, add the following entry:

Type | Host | Mail Server | Priority | TTL
-- | -- | -- | -- | --
MX Record | {subdomain} | {subdomain}.spinalcordmri.org. | 0 | Automatic

Again, this setting will help us later when we set up our mail server. (There's not much here to talk about, since details such as 'priority' are only really relevant for setups with [multiple mail servers](https://www.cloudflare.com/learning/dns/dns-records/dns-mx-record/).)

You can double-check the current value of this record by running the following command:

```console
user@device:~$ dig +short MX {subdomain}.spinalcordmri.org
0 {subdomain}.spinalcordmri.org.
```

----

### 3. Server Setup: Hostnames

Now that NameCheap is configured, we can set up the server itself, starting with its hostname.

#### 3.1 Connecting to the droplet for the first time

You can connect to the server either through SSH (assuming your local device contains the SSH key you added earlier):

```console
user@device:~$ ssh root@{subdomain}.spinalcordmri.org
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-50-generic x86_64)

Last login: Fri Dec  9 23:06:11 2022 from 162.243.188.66
root@forum:~# 
```

Or, you can navigate to your droplet's main page, then click the "Console" button in the top-left corner:

![image](https://user-images.githubusercontent.com/16181459/207170763-68db34b0-4f1d-4e6d-a7b6-72340994c7ae.png)

From here, you can make changes to the server itself. 

> _**NB:** If the initial DNS records haven't propagated yet, you can also SSH directly into the server using its IP address (`ssh root@142.93.152.255`) while you wait._

#### 3.2 Updating the hosts and hostname files

First, connect to the server using either SSH or the built-in DigitalOcean console. Next, edit two files using the text editor of your choice:

```console
root@forum:~# sudo nano /etc/hostname
{subdomain}.spinalcordmri.org
root@forum:~# sudo nano /etc/mailname
{subdomain}.spinalcordmri.org
```

Additionally, edit the `/etc/hosts` file to ensure that the hostname of the server (`{subdomain}.spinalcordmri.org`) maps to the server's permanent IP address [instead of `127.0.1.1`](https://qref.sourceforge.net/quick/ch-gateway.en.html#s-net-dns).

```console
root@forum:~# sudo nano /etc/hosts
# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
142.93.152.255 {subdomain}.spinalcordmri.org
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Reboot the server by running `reboot`, then reconnect and ensure that the following command also returns the same domain name as above:

```console
root@forum:~# hostname
{subdomain}.spinalcordmri.org
```

----

### 4. Server Setup: Email

We run a small mail server on the same server as Discourse for it to send notifcations and password resets. Discourse recommends using a cloud service like MailGun or Amazon SES or SendGrid, but our usage is so small that the overhead (and risk) of outsourcing is high. Mail servers are something of an arcane art now, but never fear, these instructions will make it work.

#### 4.1 Install mail server

First, we must install our mail server software of choice, [`opensmtpd`](https://www.opensmtpd.org/). 

However, we cannot use `sudo apt-get install opensmtpd` because of [a buggy interaction with OpenSSL 3.0](https://github.com/OpenSMTPD/OpenSMTPD/issues/1171), which Ubuntu 22.04 installs by default. To get around this, we must build `openssl` and `opensmtpd` from their source code, to ensure that `opensmptd` uses OpenSSL v1.1.1, rather than v3.0. Thankfully, one of SCT's developers (@kousu) has created a script that will automate this procedure:

1. Run `cd ~`, then create a new file called `patch_opensmtpd_openssl.sh` and paste in the contents of [this comment](https://github.com/OpenSMTPD/OpenSMTPD/issues/1171#issuecomment-1218503481):
2. Make the script executable using `chmod +x patch_opensmtpd_openssl.sh`.
3. Run the script using `./patch_opensmtpd_openssl.sh`
4. Grab a coffee and find something interested to read. This step will take a while!

> _**NB:**: During the installation, a purple text UI will appear on the screen prompting you to answer some questions. For **Daemons using outdated libraries**, keep the default selections; Just press enter. For **Configuring opensmtpd**, enter in "{subdomain}.spinalcordmri.org" for the system mail name, then press enter._

You can verify that `opensmptd` was built and installed correctly by running:

```console
root@forum:~# ldd `which smtpd`
    [...]
	libcrypto.so.1.1 => /opt/openssl-1.1.1q/lib/libcrypto.so.1.1 (0x00007f5ed6eb2000)
	libssl.so.1.1 => /opt/openssl-1.1.1q/lib/libssl.so.1.1 (0x00007f5ed6e19000)
    [...]
```

If `libcrypto.so.1.1` and `libssl.so.1.1` are linked to `/opt/openssl-1.1.1q/`, then you've installed `opensmtpd` correctly.

#### 4.2 Configure mail server

Replace the existing contents of  `/etc/smtpd.conf` with the following:

```
pki {subdomain}.spinalcordmri.org cert "/var/discourse/shared/standalone/ssl/{subdomain}.spinalcordmri.org.cer"
pki {subdomain}.spinalcordmri.org key "/var/discourse/shared/standalone/ssl/{subdomain}.spinalcordmri.org.key"

listen on eth0 tls-require pki {subdomain}.spinalcordmri.org
listen on eth0 tls-require pki {subdomain}.spinalcordmri.org auth port 587
table aliases file:/etc/aliases

action "local_mail" maildir "~/.mail" alias <aliases>
match for local action "local_mail"

action "relay_mail" relay helo "{subdomain}.spinalcordmri.org"
match from auth for any action "relay_mail"
```

#### 4.3 Enable verbose debugging

Edit the following file:

```
sudo nano /etc/systemd/system/multi-user.target.wants/opensmtpd.service
```

And make the following change:

```diff
-ExecStart=/usr/sbin/smtpd
+ExecStart=/usr/sbin/smtpd -v
```

#### 4.4 Create a new mail-specific user account

We need an SMTP account that Discourse can send mail via. `opensmtpd` simply uses the OS's users by default, so we will make an OS user for outgoing emails.

1. Run a password generator and save the result somewhere secure. (You will need the result for the remainder of this tutorial.)
    - If you have a password manager, see if it has a password generator built in. Otherwise there's [Diceware](https://www.rempe.us/diceware/#eff) and [xkpasswd](https://xkpasswd.net/s/) and [xkcdpass](https://pypi.org/project/xkcdpass/) and [pwgen](https://github.com/tytso/pwgen)
    - Note: Do NOT use symbols in your password. It can cause [weird bugs](https://github.com/neuropoly/computers/issues/403#issuecomment-1341500294) in Discourse's YAML config.
2. Create the user `forum@{subdomain}.spinalcordmri.org` using the following commands:
    - `useradd -s /usr/sbin/nologin forum` : Creates the user account.
    - `passwd forum`: Sets the password for the account. Paste the password you generated earlier.

> _**NB**: This username is *not* the same as what's on the email headers, because `opensmtpd` allows authenticated users to spoof their identities, and we want to send as `noreply@{subdomain}.spinalcordmri.org`._

----

### 5. Server Setup: Discourse

#### 5.1 Initial Setup

Connect to the droplet server provided by Digital Ocean, then do:
- Install Docker:
    ```
    wget -qO- https://get.docker.com/ | sh
    ```
- Clone Discourse deploy
    ```
    mkdir /var/discourse
    git clone https://github.com/discourse/discourse_docker.git /var/discourse
    cd /var/discourse
    ```
- Increase timeout values
    
    Before we can install Discourse, we must make a small tweak to the default Discourse Docker container template.

    Open the file `/var/discourse/samples/standalone.yml` and add the following lines to the SMTP section:

    ```diff
      ## TODO: The SMTP mail server used to validate new accounts and send notifications
      # SMTP ADDRESS, username, and password are required
      # WARNING the char '#' in SMTP password can cause problems!
      DISCOURSE_SMTP_ADDRESS: smtp.example.com
      #DISCOURSE_SMTP_PORT: 587
      DISCOURSE_SMTP_USER_NAME: user@example.com
      DISCOURSE_SMTP_PASSWORD: pa$$word
    + DISCOURSE_SMTP_OPEN_TIMEOUT: 60
    + DISCOURSE_SMTP_READ_TIMEOUT: 60
      #DISCOURSE_SMTP_ENABLE_START_TLS: true           # (optional, default true)
      #DISCOURSE_SMTP_DOMAIN: discourse.example.com    # (required by some providers)
      #DISCOURSE_NOTIFICATION_EMAIL: noreply@discourse.example.com    # (address to send notifications from)
    ```
  
    The default values for these options are `5` seconds, which is too strict for our (slower) internal mail server. So, we must increase the timeout [to avoid `Net::ReadTimeout` errors](https://meta.discourse.org/t/smtp-net-readtimeout-without-relation-to-net-or-login-problems-smtp-host-is-just-slow/235727/1).
- Install Discourse
    ```
    ./discourse-setup
    Hostname        : {subdomain}.spinalcordmri.org
    Email           : [initial administrator's email address -- will be used for 1st admin account creation]
    SMTP address    : {subdomain}.spinalcordmri.org
    SMTP port       : 587
    SMTP username   : forum
    SMTP password   : xxxxxxxxxxxxxxxxxxxxxxx
    Notification    : noreply@{subdomain}.spinalcordmri.org
    Let's Encrypt   : neuropoly-admin@liste.polymtl.ca
    Maxmind License : [Enter]
    ```

This will download the Discourse base image, then build the image and initialize the Docker container.

#### 5.2 Check SSL certs

Before continuing, make sure that Discourse has generated its SSL certs. (They will be [generated automatically](https://meta.discourse.org/t/set-up-https-support-with-lets-encrypt/40709), so long as you provide an email address for Let's Encrypt.) 

The files exist in `/var/discourse/shared/standalone/ssl/`:

```
root@forum:~# ls -l /var/discourse/shared/standalone/ssl/{subdomain}.spinalcordmri.org.{cer,key}
-rw-r--r-- 1 root root 3799 Dec  5 08:33 /var/discourse/shared/standalone/ssl/{subdomain}.spinalcordmri.org.cer
-rw------- 1 root root 3247 Dec  5 08:33 /var/discourse/shared/standalone/ssl/{subdomain}.spinalcordmri.org.key
```

These files must be present for `opensmtpd` to be able to send mail correctly, as per the previous `/etc/smtpd.conf`  configuration we pasted in during a previous step. 

#### 5.3 Starting the mail server

Once you've confirmed that the SSL certs are present, you can start the mail server with the following command:

```
systemctl enable --now opensmtpd
```

#### 5.4 Test mail server connectivity

You should be able to connect to the mail server by running:

```console
root@forum:~# nc {subdomain}.spinalcordmri.org 587
220 {subdomain}.spinalcordmri.org ESMTP OpenSMTPD
```

#### 5.5 Test mail delivery

To create the first admin account on the newly-installed Discourse forum, email delivery must be working, because Discourse will need to send out a confirmation email.

To test email delivery, rather than sending messages to specific personal addresses, we use https://www.mail-tester.com/. This page is very helpful for finding issues missed during setup, especially around spamminess. Email is very very complicated and this helps a lot.

Go to https://www.mail-tester.com/ and copy the email address it gives you. You can use this address alongside one of three testing methods:

1. Simple test (`mail`)

    You will first need to install [Mailutils](https://mailutils.org/) using `sudo apt-get install mailutils`. Then, you can run:

    ```bash
    echo "Test Message" | mail -s "This is a message" somethingsomething@mail-tester.com
    ```

2. Complex test (`swaks`)

    You will first need to install the "Swiss Army Knife for SMTP" ([`swaks`](https://www.jetmore.org/john/code/swaks/)) using `sudo apt-get install swaks`. Then, you can run:

    ```bash
    # You will need to enter the password that you set during the previous "User Setup" section!!
    swaks --to me@example.com --from noreply@{subdomain}.spinalcordmri.org --server {subdomain}.spinalcordmri.org -p 587 --auth-user forum --tls-verify --tls
    ```

3. Discourse-specific test

    First, `cd` into `/var/discourse`. Then, run the following command:

    ```bash
    ./discourse-doctor
    ```
   
    It will run some automated tests, then it will prompt you for an email to send a test message to.

    > _**NB**: There is a possibility that you may encounter a `Net::ReadTimeout` error. In that case, edit the `/var/discourse/containers/app.yml` file to increase the values of `DISCOURSE_SMTP_OPEN_TIMEOUT` and `DISCOURSE_SMTP_READ_TIMEOUT`. Then, restart the docker container by running `cd /var/discourse`, `./launcher destroy app`, and `./launcher start app`._

With each of these tests, your goal here is to ensure that:

- The email message gets delivered.
- Your mail-tester score is relatively high (ideally at least 9/10, but at the very minimum, 7/10).

Once you send your test message, you can click "View my results" on the mail-tester webpage. It will give you feedback on things that need to be changed. So, try to iteratively address the feedback, send a new test message, and refresh the page until you get a high score.

#### 5.6 Create your first admin account

Navigate to the page https://{subdomain}.spinalcordmri.org/, and follow the instruction to create an admin email account. Discourse will send you a confirmation email. If you've set up mail correctly, you should receive the email and be able to finish making your account.

Once you log in for the first time, you will be presented with an onboarding flow that will ask for things like "Community name" and "Point of contact". Feel free to put some sensible defaults (e.g. "Spinalcordmri.org", "neuropoly-admin@liste.polymtl.ca", etc.). However, if you will be loading a previous backup of the forum, then these settings will be overwritten anyway, making them moot.

#### 5.7 Loading a previous backup of the forum

If there is a currently-running instance of the forum, you can download backups from [the Backups page](https://forum.spinalcordmri.org/admin/backups).

Next, you need to upload this backup to the forum. You can try using the Backups page on the dev forum, however sometimes uploads will fail inexplicably with (due to the large size of the backups). So, you may need to instead use [FileZilla to transfer files to the Digital Ocean Droplet](https://docs.digitalocean.com/products/droplets/how-to/transfer-files/). Specifically, the backup will need to be uploaded to the `/var/discourse/shared/standalone/backups/default` folder.

Finally, you will need to enable the "allow restore" setting on the dev server's Settings page. Then, you can return to the Backups page and restore the uploaded backup.

#### 5.8 Installing plugins

Currently, the only mandatory plugin in use by the forum is [`discourse-solved`](https://meta.discourse.org/t/discourse-solved/30155). To install it, please follow the instructions from the [Install Plugins in Discourse](https://meta.discourse.org/t/install-plugins-in-discourse/19157) documentation page.

----

### 6. Google/GitHub logins for Discourse

<details>
<summary> Note: These steps are not currently used in the production server for spinalcordmri.org </summary>

#### 6.1 Configuring Google login for Discourse ([reference](https://meta.discourse.org/t/configuring-google-login-for-discourse/15858))

Go to https://console.developers.google.com, click on Credentials and create a new Project.
- Project name `Forum spinalcordmri`
- Project id `forum-spinalcordmri`

Select Credentials in the left menu, Create credentials and OAuth client ID type for the credentials.
- Application type `Web application`
- Name `Forum spinalcordmri.org`
- Authorized JavaScript origins `http://{subdomain}.spinalcordmri.org`
- Authorized redirect URIs `http://{subdomain}.spinalcordmri.org/auth/google_oauth2/callback`

Configure your OAuth Consent Screen
  - Product name shown to users `Forum spinalcordmri.org`
  - Homepage URL `http://www.spinalcordmri.org/`
  - Privacy policy URL `http://www.spinalcordmri.org/`

Click Library in the left menu and you’ll see a huge list of Google API’s. Find Google+ API and enable them.

The API will create `google_client_id` and `google_client_secret` which you can add under http://{subdomain}.spinalcordmri.org/admin/site_settings/category/login, after checking `enable google oauth2 logins`

#### 6.2  Configure GitHub login for Discourse ([reference](https://meta.discourse.org/t/configuring-github-login-for-discourse/13745))

Under github.com/spinalcordmri, click Settings (the gear icon), then look for OAuth Applications in the left menu. Select Register new application.
  - Application name
  ```
  Forum spinalcordmri
  ```
  - Homepage URL
  ```
  http://{subdomain}.spinalcordmri.org/
  ```
  - Application description
  ```
  Forum spinalcordmri
  ```
  - Authorization callback URL
  ```
  http://{subdomain}.spinalcordmri.org//auth/github/callback
  ```
The app will create `github_client_id` and `github_client_secret`which you can add under http://{subdomain}.spinalcordmri.org/admin/site_settings/category/login, after checking `enable github logins`

</details>
