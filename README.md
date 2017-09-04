# Introduction

Hegemon is an [Ansible](https://www.ansible.com/) based Debian server configuration tool.  This started as a fork of the absolutely wonderful [Soverign](https://github.com/sovereign/sovereign) playbooks for a personal cloud.

These playbooks strive to be reasonably secure, full featured, and low on maintenance.

## Services Provided

-   Common package installation with useful programs, including nice-to-have tools like [mosh](http://mosh.mit.edu) and [htop](http://htop.sourceforge.net).
-   User configuration with a remote git dotfiles directory, [Stow](https://www.gnu.org/software/stow/) linking, and Vim, Vundle, and Vim plugins.
-   [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) over SSL via [Dovecot](http://dovecot.org/), complete with full text search provided by [Solr](https://lucene.apache.org/solr/).
-   [POP3](https://en.wikipedia.org/wiki/Post_Office_Protocol) over SSL, also via Dovecot.
-   [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) over SSL via [Postfix](http://www.postfix.org/)
-   [DNSBLs](https://en.wikipedia.org/wiki/DNSBL) to discard spam before it ever hits mail filters.
-   Webmail via [Roundcube](http://www.roundcube.net/), with Carddav, [ManageSieve](https://wiki1.dovecot.org/ManageSieve), and 2-factor authentication plugins.
-   Mobile push notifications via [Z-Push](http://z-push.org/).
-   Email client [automatic configuration](https://developer.mozilla.org/en-US/docs/Mozilla/Thunderbird/Autoconfiguration).
-   Mail server verification via [OpenDKIM](http://www.opendkim.org/) and [OpenDMARC](http://www.trusteddomain.org/opendmarc/) so the Internet knows the mailserver is legit.
-   Spam fighting via [Rspamd](https://www.rspamd.com/) and [Postgrey](http://postgrey.schweikert.ch/).
-   Virtual domains for email, backed by [PostgreSQL](http://www.postgresql.org/).
-   Private storage cloud via [nextCloud](https://nextcloud.com/).
-   [CalDAV](https://en.wikipedia.org/wiki/CalDAV) and [CardDAV](https://en.wikipedia.org/wiki/CardDAV) to keep calendars and contacts in sync, via nextCloud.
-   Jabber/[XMPP](http://xmpp.org/) instant messaging via [Prosody](http://prosody.im/).
-   An RSS Reader via [Selfoss](http://selfoss.aditu.de/).
-   An IRC bouncer via [ZNC](http://wiki.znc.in/ZNC).
-   Git hosting via [cgit](http://git.zx2c4.com/cgit/about/) and [gitolite](https://github.com/sitaramc/gitolite).
-   Read-it-later via [Wallabag](https://www.wallabag.org/)
-   Web hosting via [Apache](https://www.apache.org/).
-   SSL certificates obtained from [Let's Encrypt](https://letsencrypt.org/) automatically refreshed monthly.  Used by Apache2, Dovecot, Postfix, and Prosody.
-   [RFC6238](http://tools.ietf.org/html/rfc6238) two-factor authentication compatible with [Google Authenticator](http://en.wikipedia.org/wiki/Google_Authenticator) and various hardware tokens.
-   Secure on-disk storage for email and more via [EncFS](http://www.arg0.net/encfs).
-   VPN server via [OpenVPN](http://openvpn.net/index.php/open-source.html).
-   [Monit](http://mmonit.com/monit/) to keep everything running smoothly (and alert you when it’s not).
-   [collectd](http://collectd.org/) to collect system statistics, with optional integration to [librato](https://www.librato.com/), an hosted monitoring solution.
-   Locally hosted [Carbon](https://github.com/graphite-project/carbon) metric processing daemon and [Whisper](https://github.com/graphite-project/whisper) time-series database library funneling collectd data into a [Grafana](https://grafana.com/) metrics dashboard.
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

---
# Usage

## Remote Server

-   A VPS (or bare-metal server). [Linode](http://www.linode.com), [Scaleway](https://www.scaleway.com/), and [DigitalOcean](https://www.digitalocean.com/) are fantastic options. Make sure to have at least 512 MB of RAM for the system.
-   [64-bit Debian 9.1](http://www.debian.org/) or an equivalent Linux distribution. (Deviating from Debian will require more tweaks to the playbooks. See Ansible’s different [packaging](http://docs.ansible.com/ansible/list_of_packaging_modules.html) modules.)
-   A [Tarsnap](http://www.tarsnap.com) account with credit.  Tarsnap is an optional backup service; consider paying the hosting provider for backups or using an additional backup service for redundancy.

## Home Server

-   Just about any computer will do.  Consider an older desktop, a laptop with a broken screen, an [Intel NUC](https://www.intel.com/content/www/us/en/products/boards-kits/nuc.html), or a [Qotom](http://www.qotom.net/) thin client.  Media and general purpose local servers need few resources to be effective.

---
# Installation

### Configure Server

If using Scaleway, create the instance and allow it to boot fully.  Once booted, change the kernel to the Apparmor kernel and reboot using SSH or the console.  This will allow the kernel modules to be fetched.  Do not switch over to Apparmor during the first boot cycle as it will break the process.

Log in through the remote console, or as part of the local install process.

Install Required Packages:

```
apt install sudo
```

Change the root password:

```
passwd
```

Create a deploy user account for Ansible:

```
adduser deploy
```

Authorize an ssh key for the deploy account:

```
mkdir /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
cp /root/.ssh/authorized_keys /home/deploy/.ssh/authorized_keys
chmod 400 /home/deploy/.ssh/authorized_keys
chown deploy:deploy /home/deploy -R
adduser deploy sudo
```

If the host provider doesn't copy ssh keys into the root user's account, keys can be added to the deploy user's authorized_keys file with this command, instead of the `cp` command above:

```
nano /home/deploy/.ssh/authorized_keys
```

### Configure Installation

Make sure [Ansible](http://docs.ansible.com/intro_installation.html#getting-ansible) 2.2+ is installed.

Choose a vault password option by uncommenting the setting in the 'ansible.cfg' file.

Modify the settings in the `group_vars` folder.  All passwords and sensitive data is contained in [ansible-vault](http://docs.ansible.com/ansible/latest/playbooks_vault.html) encrypted files, ending with _vault.yml.  The `hosts` file is also encrypted.  This allows for forking this repo and keeping sensitive passwords in a personal git account, available to multiple computers.  All '_vault'.yml' files are referenced by their counterparts without '_vault' for ease of searching.  Vault files can be edited, encrypted, and decrypted with this command:

```
ansible-vault [create|decrypt|edit|encrypt|encrypt_string|rekey|view] all_vault.yml

```

Python 3.3 or higher on Linux will generate the appropriate mail password hash:

```
python3 -c 'import crypt; print(crypt.crypt("password", salt=crypt.METHOD_SHA512))'
```

On OS X and other platforms the [passlib](https://pythonhosted.org/passlib/) package may be used to generate the required string:

```
python -c 'import passlib.hash; print(passlib.hash.sha512_crypt.encrypt("password", rounds=5000))'
```

To create an IRC password hash:

```
python3 -c 'import crypt; print("irc_password_salt: {}\nirc_password_hash: {}".format(*crypt.crypt("password", salt=crypt.METHOD_SHA256).split("$")[2:]))'
```

Insert the values for `irc_password_salt` and `irc_password_hash` into ircbouncer_vault.yml.

On OS X and other platforms the [passlib](https://pythonhosted.org/passlib/) package may be used to generate the required string:

```
python -c 'import passlib.hash; print("irc_password_salt: {}\nirc_password_hash: {}".format(*passlib.hash.sha256_crypt.encrypt("password", rounds=5000).split("$")[2:]))'
```

For Git hosting, create a key and copy the public key into place:

```
ssh-keygen -t rsa -b 4096 -C "$(whoami)@$(hostname)-$(date -I)" -f gitolite
cp ~/.ssh/gitolite.pub ~/hegemon/roles/git/files/
```

If the SSH daemon listens on a non-standard port, add a colon and the port number after the IP address. In that case you also need to add the custom port to the task `Set firewall rules for S` in the file `roles/remote/tasks/ufw.yml`.

If using Tarsnap, download and install from [www.tarsnap.com/download.html](https://www.tarsnap.com/download.html), or use `brew install tarsnap` if using [Homebrew](http://brew.sh).

Create a new machine key for the server (while in the hegemon/ directory):

```
tarsnap-keygen --keyfile roles/tarsnap/files/decrypted_tarsnap.key --user me@example.com --machine example.com
```

### Set up DNS

Create `A` or `CNAME` records which point to the server's IP address:

*   `example.com`
*   `mail.example.com` (for IMAP/POP/SMTP)
*   `www.example.com` (for Web hosting)
*   `autoconfig.example.com` (for email client automatic configuration)
*   `autodiscover.example.com` (for active-sync autodiscovery)
*   `sync.example.com` (for z-push syncronization)
*   `read.example.com` (for Wallabag)
*   `news.example.com` (for Selfoss)
*   `cloud.example.com` (for nextCloud)
*   `git.example.com` (for cgit)

Create an `MX` record for `example.com` which assigns `mail.example.com` as the domain’s mail server.

### Run the Ansible Playbooks

To run the remote playbook:

```
ansible-playbook remote.yml
```

To run one or more tasks, use tags:

```
ansible-playbook remote.yml -t ufw,ntp,swap
```

The `dependencies` tag just installs dependencies, performing no other operations. The tasks associated with the `dependencies` tag do not rely on the user-provided settings that live in `group_vars`. Running the playbook with the `dependencies` tag is particularly convenient for working with Docker images.

### Finish DNS set-up

To ensure emails pass [DKIM](http://www.dkim.org/) checks, add a `txt` record. The name field will be `default._domainkey.example.com.` The `p=` field contains the public key used by OpenDKIM. The exact value can be found in the file `/etc/opendkim/keys/example.com/default.txt`, which is also copied locally when the DKIM task runs.  The `v=` and `p=` keys are required. The `h=` key is optional, and defaults to all hash algorithms.  The `k=` key is optional, and defaults to 'rsa'.  The `s=` key is optional, and defaults to '*', for all services.  See the DKIM reference [here](http://dkim.org/specs/rfc4871-dkimbase.html) for any questions.

Example:

> v=DKIM1; h=sha256; k=rsa; s=*; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDKKAQfMwKVx+oJripQI+Ag4uTwYnsXKjgBGtl7Tk6UMTUwhMqnitqbR/ZQEZjcNolTkNDtyKZY2Z6LqvM4KsrITpiMbkV1eX6GKczT8Lws5KXn+6BHCKULGdireTAUr3Id7mtjLrbi/E3248Pq0Zs39hkDxsDcve12WccjafJVwIDAQAB

For DMARC, add a `txt` record. The name field should be `_dmarc.example.com` and the value should be `v=DMARC1; p=none`. More info on DMARC can be found [here](https://dmarc.org).

Add an SPF `txt` record.  The name should be `v=spf1 mx -all`.

Set the reverse DNS with the server provider to `mail.example.com`.

For reference, see [this post](http://sealedabstract.com/code/nsa-proof-your-e-mail-in-2-hours/).

### Verification and Checking

Make sure to validate functionality by sending an email to <a href="mailto:check-auth@verifier.port25.com">check-auth@verifier.port25.com</a> and review the report that will be emailed back.

Also, visit [DKIMValidator.com](http://dkimvalidator.com/) for another test.

Check certificate issuance at [crt.sh](https://crt.sh).

Check SSL setup with [SSLlabs.com/ssltest](https://www.ssllabs.com/ssltest/).

Be careful with Let's Encrypt SSL issuance, as there is a rate limit of 5 duplicate certificates / week.  Please see [letsencrypt.org/docs/rate-limits](https://letsencrypt.org/docs/rate-limits/).

Make sure to allow SMTP outbound mail with the server provider.  Most providers will have this turned off by default.  Scaleway requires a hard boot of the server through their console to change firewall rules and allow SMTP traffic.

### Miscellaneous Configuration

Sign into the ZNC web interface, which isn’t exposed through the firewall, and requires an SSH tunnel:

```
ssh deploy@example.com -L 6643:localhost:6643
```

Then proceed to [localhost:6643](http://localhost:6643).

Similarly, to access the monit server monitoring page, use another SSH tunnel:

```
ssh deploy@example.com -L 2812:localhost:2812
```

Again, proceed to [localhost:2812](http://localhost:2812).

---
# Troubleshooting

Please create an [issue](https://github.com/hegemon/hegemon/issues/new).

### Reboots

To reboot the machine, the reboot.yml playbook needs run.  This will restart the machine, decrypt the encfs, and restart all services that use the encrypted system.  Optionally, the task can be used to just re-mount and restart services without a reboot, which is useful if encfs has a hiccup:

```
ansible-playbook reboot.yml
```

The other option is to enter the encfs password manually.  This requires SSH into the machine after reboot, or accessing via a console interface:

```
encfs /encrypted /decrypted --public
```

After this, services dependent on the decrypted folder will need restarted:

```
sudo systemctl restart postgresql dovecot tomcat8 apache2 prosody
```

### Inspirations

These programs and references have been invaluable in this project:

[Sovereign](https://github.com/sovereign/sovereign)
-   The inspiration and original fork for this project.

[DebOps](https://github.com/debops/debops-playbooks)
-   An endless wealth of Debian server knowledge.  This project doesn't come close to the vastness of DebOps and doens't try to.

[Mail-in-a-Box](https://github.com/mail-in-a-box/mailinabox)
-   A fantastic single service setup for personally hosted e-mail.
