# Let's Encrypt

[Let's Encrypt](https://letsencrypt.org) (LE) is an automated certificate authority that provides free SSL certificates that are trusted by all major browsers.  LE certificates are used by Hegemon instead of purchased certificates from authorities like RapidSSL in order to reduce the out-of-pocket cost of deploying Hegemon and avoid end-user problems with self-signed certificates.

### Design approach

The Let's Encrypt service uses DNS to look up domains being registered and then contact the client to verify. For this to work, DNS records must be configured before the playbook is run the first time.

A single certificate is created using Let's Encrypt with SANs used for the sub-domains.  At deploy-time, a script is used to query DNS for known sub-domains, build a list of the subset that is registered, and use it when making the certificate request of Let's Encrypt.

Several packages need access to the private key. Not all are run as root. An example is Prosody (XMPP). Such users are added to the ssl-cert group, and /etc/letsencrypt is set up to allow keys to be read by ssl-cert.

Certificates and private keys are backed up using tarsnap.

Certificate renewal is done automatically using cron. The cron script must be aware of private key copies and update them as well. Services that depend on new keys must also be bounced. It is up to roles that rely on keys to modify the cron script (preferably using `lineinfile` or something similar) to accomplish this.

### Testing support

An isolated VM deployed with Vagrant is used for testing. The Let's Encrypt service cannot be used to get keys for it, since it is not bound with DNS. A self-signed wildcard key is therefore used for testing. The wildcard key, certificate, and chain are installed in the same way that Let's Encrypt keys are installed.

### Alternative approaches

Another way to generate certificates is to generate one certificate per domain and expect each module that uses a subdomain to generate its own certificate for the subdomain.

This was prototyped. The common role included a parameterized task list that could be invoked by modules that needed to generate a key. The certificate renewal script run by cron could be modified to update all the certificates in the `live` directory.

This approach was rejected due to complexity. This would have been the first time modules needed to invoke a task list from another module. Managing multiple certificates is also more complicated.

# Webmail

This role installs the Roundcube package to enable browser-based email handling. This is useful if you need to access email with something other than your own computer or smart phone.  It is also the only friendly way in Hegemon to edit sieve filters for email.

Roundcube is stable and continues to be actively developed.

### Install

The role installs roundcube from the source package released by the Roundcube team.  The version is pinned.  Old versions of this role installed Roundcube from apt packages, but the packages for Debian 8 do not install unattended correctly unless mysql is used at the backend.  We want to use only one database server (postgres) to save on RAM, so using packages is not an option for now.

Roundcube is installed with sqlite3 for its persistence layer.  This eliminates dependency on a database server and likely improves performance given how little persistet data Roundcube keeps.  Roundcube automatically looks for the database file and initializes it if it is missing.  The file is kept on `/decrypted` since it contains user data, and the database will be backed up automatically if the tarsnap role is used.

PHP composer is used for downloading and installing plugins.  Configuration files are kept with Hegemon.  The configuration files for `twofactor_gauthentication` and `carddav` are not modified from their defaults.  I chose to do this so that maintainers could recognize when configuration files change in future plugin versions and decide whether or not to change new defaults.

### Upgrade

It's unknown how upgrades will be handled.  The best case is that an update can be installed over the current version, and code will automatically update the database the first time it connects.  This needs to be considered for plugins that store data also.
