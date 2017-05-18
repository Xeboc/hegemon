Services Provided
-----------------

-   Common package installation with useful programs, including nice-to-have tools like [mosh](http://mosh.mit.edu) and [htop](http://htop.sourceforge.net) that make life with a server a little easier.
-   User configuration with a remote git dotfiles directory, [Stow](https://www.gnu.org/software/stow/) linking, and Vim, Vundle, and Vim plugins.
-   [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) over SSL via [Dovecot](http://dovecot.org/), complete with full text search provided by [Solr](https://lucene.apache.org/solr/).
-   [POP3](https://en.wikipedia.org/wiki/Post_Office_Protocol) over SSL, also via Dovecot.
-   [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) over SSL via [Postfix](http://www.postfix.org/)
-   [DNSBLs](https://en.wikipedia.org/wiki/DNSBL) to discard spam before it ever hits your filters.
-   Webmail via [Roundcube](http://www.roundcube.net/), with Carddav, managesieve, and 2-factor authenitication plugins.
-   Mobile push notifications via [Z-Push](http://z-push.org/).
-   Email client [automatic configuration](https://developer.mozilla.org/en-US/docs/Mozilla/Thunderbird/Autoconfiguration).
-   Mail server verification via [OpenDKIM](http://www.opendkim.org/) and [OpenDMARC](http://www.trusteddomain.org/opendmarc/) so the Internet knows your mailserver is legit.
-   Spam fighting via [Rspamd](https://www.rspamd.com/) and [Postgrey](http://postgrey.schweikert.ch/).
-   Virtual domains for your email, backed by [PostgreSQL](http://www.postgresql.org/).
-   Private storage cloud via [ownCloud](http://owncloud.org/) or [nextCloud](https://nextcloud.com/).
-   [CalDAV](https://en.wikipedia.org/wiki/CalDAV) and [CardDAV](https://en.wikipedia.org/wiki/CardDAV) to keep your calendars and contacts in sync, via [ownCloud](http://owncloud.org/) or [nextCloud](https://nextcloud.com/).
-   Jabber/[XMPP](http://xmpp.org/) instant messaging via [Prosody](http://prosody.im/).
-   An RSS Reader via [Selfoss](http://selfoss.aditu.de/).
-   An IRC bouncer via [ZNC](http://wiki.znc.in/ZNC).
-   Git hosting via [cgit](http://git.zx2c4.com/cgit/about/) and [gitolite](https://github.com/sitaramc/gitolite).
-   Read-it-later via [Wallabag](https://www.wallabag.org/)
-   Web hosting via [Apache](https://www.apache.org/).
-   SSL certificates obtained from [Let's Encrypt](https://letsencrypt.org/) automatically and refreshed daily.
-   [RFC6238](http://tools.ietf.org/html/rfc6238) two-factor authentication compatible with [Google Authenticator](http://en.wikipedia.org/wiki/Google_Authenticator) and various hardware tokens.
-   Secure on-disk storage for email and more via [EncFS](http://www.arg0.net/encfs).
-   VPN server via [OpenVPN](http://openvpn.net/index.php/open-source.html).
-   [Monit](http://mmonit.com/monit/) to keep everything running smoothly (and alert you when it’s not).
-   [collectd](http://collectd.org/) to collect system statistics.
-   Validating, recursive, and caching DNS resolver provided by [unbound](https://www.unbound.net/).
-   Firewall management via [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall).
-   Intrusion prevention via [fail2ban](http://www.fail2ban.org/).
-   GeoIP blocking using [Xtables-addons](http://xtables-addons.sourceforge.net/).
-   Malware detection via [rkhunter](http://rkhunter.sourceforge.net), [ClamAV](https://www.clamav.net/), and [LMD](https://www.rfxn.com/projects/linux-malware-detect/).
-   SSH configuration preventing root login and insecure password authentication with improved defaults.
-   File integrity monitoring with [Samhain](http://www.la-samhna.de/samhain/index.html).
-   Improved security with weekly [lynis](https://cisofy.com/lynis/) checks, and [apparmor](http://wiki.apparmor.net) security.
-   NTP, Apticron, and unattended-upgrades for server maintenance.
-   Nightly backups to [Tarsnap](https://www.tarsnap.com/).

Usage
=====

What's Needed:
----------------

1.  A VPS (or bare-metal server). [Linode](http://www.linode.com) and [Scaleway](https://www.scaleway.com/) are two great options. Get at least 512 MB of RAM between Apache, Solr, and PostgreSQL.
2.  [64-bit Debian 8.3](http://www.debian.org/) or an equivalent Linux distribution. (Deviating from Debian will require more tweaks to the playbooks. See Ansible’s different [packaging](http://docs.ansible.com/ansible/list_of_packaging_modules.html) modules.)
3.  A [Tarsnap](http://www.tarsnap.com) account with some credit in it. You could comment this out if you want to use a different backup service. Consider paying your hosting provider for backups or using an additional backup service for redundancy.


Installation
------------

## On the remote server

### 1. Install required packages

    apt-get install sudo

### 2. Get a Tarsnap machine key

If you haven’t already, [download and install Tarsnap](https://www.tarsnap.com/download.html), or use `brew install tarsnap` if you use [Homebrew](http://brew.sh).

Create a new machine key for your server:

    tarsnap-keygen --keyfile roles/tarsnap/files/decrypted_tarsnap.key --user me@example.com --machine example.com

### 3. Prep the server

Change the root password:

    passwd

Create a deploy user account for Ansible:

    adduser deploy

Authorize an ssh key for the deploy account:

    mkdir /home/deploy/.ssh
    chmod 700 /home/deploy/.ssh
    cp /root/.ssh/authorized_keys /home/deploy/.ssh/authorized_keys
    chmod 400 /home/deploy/.ssh/authorized_keys
    chown deploy:deploy /home/deploy -R
    adduser deploy sudo

If the host provide doesn't copy ssh keys into the root user's account (Scaleway), keys can be added to the deploy user's authorized_keys file with this command:

    nano /home/deploy/.ssh/authorized_keys

### 4. Configure your installation

Modify the settings in the `group_vars`.

Setting `password_hash` for your mail users is a bit tricky. You can generate one using [doveadm-pw](http://wiki2.dovecot.org/Tools/Doveadm/Pw).

    # doveadm pw -p'YOUR_PASSWORD' -s SHA512-CRYPT | sed -e 's/{.*}//'
    $6$drlIN9fx7Aj7/iLu$XvjeuQh5tlzNpNfs4NwxN7.HGRLglTKism0hxs2C1OvD02d3x8OBN9KQTueTr53nTJwVShtCYiW80SGXAjSyM0

`sed` is used here to truncate the hash type from the beginning of the `doveadm pw` output.

Alternatively, if you don’t already have `doveadm` installed, Python 3.3 or higher on Linux will generate the appropriate string for you (assuming your password is `password`):

    python3 -c 'import crypt; print(crypt.crypt("password", salt=crypt.METHOD_SHA512))'

On OS X and other platforms the [passlib](https://pythonhosted.org/passlib/) package may be used to generate the required string:

    python -c 'import passlib.hash; print(passlib.hash.sha512_crypt.encrypt("password", rounds=5000))'

Same for the IRC password hash…

    # znc --makepass
    [ ** ] Type your new password.
    [ ?? ] Enter Password: foo
    [ ?? ] Confirm Password: foo
    [ ** ] Kill ZNC process, if it's running.
    [ ** ] Then replace password in the <User> section of your config with this:
    <Pass password>
            Method = sha256
            Hash = 310c5f99825e80d5b1d663a0a993b8701255f16b2f6056f335ba6e3e720e57ed
            Salt = YdlPM5yjBmc/;JO6cfL5
    </Pass>
    [ ** ] After that start ZNC again, and you should be able to login with the new password.

Take the strings after `Hash =` and `Salt =` and insert them as the value for `irc_password_hash` and `irc_password_salt` respectively.

Alternatively, if you don’t already have `znc` installed, Python 3.3 or higher on Linux will generate the appropriate string for you (assuming your password is `password`):

    python3 -c 'import crypt; print("irc_password_salt: {}\nirc_password_hash: {}".format(*crypt.crypt("password", salt=crypt.METHOD_SHA256).split("$")[2:]))'

On OS X and other platforms the passlib:https://pythonhosted.org/passlib/ package may be used to generate the required string:

    python -c 'import passlib.hash; print("irc_password_salt: {}\nirc_password_hash: {}".format(*passlib.hash.sha256_crypt.encrypt("password", rounds=5000).split("$")[2:]))'

For Git hosting, copy your public key into place:

	cp ~/.ssh/id_rsa.pub roles/git/files/gitolite.pub

If your SSH daemon listens on a non-standard port, add a colon and the port number after the IP address. In that case you also need to add your custom port to the task `Set firewall rules for web traffic and SSH` in the file `roles/common/tasks/ufw.yml`.

### 5. Set up DNS

Create `A` or `CNAME` records which point to your server's IP address:

* `example.com`
* `mail.example.com` (for IMAP/POP/SMTP)
* `www.example.com` (for Web hosting)
* `autoconfig.example.com` (for email client automatic configuration)
* `autodiscover.example.com` (for active-sync autodiscovery)
* `sync.example.com` (for z-push syncronization)
* `read.example.com` (for Wallabag)
* `news.example.com` (for Selfoss)
* `cloud.example.com` (for ownCloud/nextCloud)
* `git.example.com` (for cgit)

Create an `MX` record for `example.com` which assigns `mail.example.com` as the domain’s mail server.

### 6. Run the Ansible Playbooks

First, make sure [Ansible] (http://docs.ansible.com/intro_installation.html#getting-ansible) 2.2+ is installed.

To run the remote playbook:

    ansible-playbook remote.yml --ask-vault-pass -K

If you chose to make a passwordless sudo deploy user, you can omit the `-K` argument.

To run one or more piece, use tags. I try to tag all my includes for easy isolated development. For example, to focus in on your firewall setup:

    ansible-playbook remote.yml --ask-vault-pass -K -t uwf

The `dependencies` tag just installs dependencies, performing no other operations. The tasks associated with the `dependencies` tag do not rely on the user-provided settings that live in `group_vars`. Running the playbook with the `dependencies` tag is particularly convenient for working with Docker images.

### 7. Finish DNS set-up

To ensure your emails pass DKIM checks you need to add a `txt` record. The name field will be `default._domainkey.EXAMPLE.COM.` The value field contains the public key used by OpenDKIM. The exact value needed can be found in the file `/etc/opendkim/keys/EXAMPLE.COM/default.txt`, which is also copied locally.

Example:

    v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDKKAQfMwKVx+oJripQI+Ag4uTwYnsXKjgBGtl7Tk6UMTUwhMqnitqbR/ZQEZjcNolTkNDtyKZY2Z6LqvM4KsrITpiMbkV1eX6GKczT8Lws5KXn+6BHCKULGdireTAUr3Id7mtjLrbi/E3248Pq0Zs39hkDxsDcve12WccjafJVwIDAQAB

For DMARC you'll also need to add a `txt` record. The name field should be `_dmarc.EXAMPLE.COM` and the value should be `v=DMARC1; p=none`. More info on DMARC can be found [here](https://dmarc.org)

Add an SPF `txt` record.  The name should be `v=spf1 mx -all`.  

Set the reverse DNS with the server provider to `mail.example.com`.

For reference, see [this post](http://sealedabstract.com/code/nsa-proof-your-e-mail-in-2-hours/). Make sure to validate functionality by sending an email to <a href="mailto:check-auth@verifier.port25.com">check-auth@verifier.port25.com</a> and reviewing the report that will be emailed back to you.  Also, visit [DKIMValidator.com](http://dkimvalidator.com/) for another test.

### 8. Miscellaneous Configuration

Make sure to allow SMTP outbound mail with the server provider.  Most providers have this turned off by default.  Scaleway requires a hard boot of the server to change firewall rules and allow SMTP traffic.

Sign in to the ZNC web interface and set things up to your liking. It isn’t exposed through the firewall, so you must first set up an SSH tunnel:

	ssh deploy@example.com -L 6643:localhost:6643

Then proceed to http://localhost:6643 in your web browser.

Similarly, to access the monit server monitoring page, use another SSH tunnel:

    ssh deploy@example.com -L 2812:localhost:2812

Again proceeding to http://localhost:2812 in your web browser.

How To Use Your New Personal Cloud
----------------------------------

We’re collecting known-good client setups [on our wiki](https://github.com/sovereign/sovereign/wiki/Usage).

Troubleshooting
---------------

If you run into an errors, please check the [wiki page](https://github.com/sovereign/sovereign/wiki/Troubleshooting). If the problem you encountered, is not listed, please go ahead and [create an issue](https://github.com/sovereign/sovereign/issues/new). If you already have a bugfix and/or workaround, just put them in the issue and the wiki page.

### Reboots

You will need to manually enter the password for any encrypted volumes on reboot. This is not Sovereign-specific, but rather a function of how EncFS works. This will necessitate SSHing into your machine after reboot, or accessing it via a console interface if one is available to you. Once you're in, run this:

    encfs /encrypted /decrypted --public

It is possible that some daemons may need to be restarted after you enter your password for the encrypted volume(s). Some services may stall out while looking for resources that will only be available once the `/decrypted` volume is available and visible to daemon user accounts.

IRC
===

Ask questions and provide feedback in `#sovereign` on [Freenode](http://freenode.net).
