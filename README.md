__WARNING__: THIS IS A BETA EDITION, DO NOT TRY IT IN PRODUCTION.

Report issues to [iRedMail GitHub repo](https://github.com/iredmail/iRedMail/issues).

# Quick start

Create a docker environment file used to store custom settings:

```
touch iredmail-docker.conf
echo HOSTNAME=mail.mydomain.com >> iredmail-docker.conf
echo FIRST_MAIL_DOMAIN=mydomain.com >> iredmail-docker.conf
echo FIRST_MAIL_DOMAIN_ADMIN_PASSWORD=my-secret-password >> iredmail-docker.conf
echo MLMMJADMIN_API_TOKEN=$(openssl rand -base64 32) >> iredmail-docker.conf
echo ROUNDCUBE_DES_KEY=$(openssl rand -base64 24) >> iredmail-docker.conf
```

* Create required volumes:

```
docker volume create iredmail_data_backup           # Backup copies.
docker volume create iredmail_data_mailboxes        # All users' mailboxes
docker volume create iredmail_data_mlmmj            # mailing list data
docker volume create iredmail_data_mlmmj_archive    # mailing list archive
docker volume create iredmail_data_imapsieve_copy   # Used by Dovecot plugin 'imapsieve'
docker volume create iredmail_data_custom           # custom config files
docker volume create iredmail_data_ssl              # SSL cert/key files
docker volume create iredmail_data_mysql            # MySQL databases
docker volume create iredmail_data_clamav           # ClamAV database
docker volume create iredmail_data_sa_rules         # SpamAssassin rules
```

* Run:

```
docker run \
    --rm \
    --name iredmail \
    --env-file iredmail-docker.conf \
    --hostname mail.mydomain.com \
    -p 80:80 \
    -p 443:443 \
    -p 110:110 \
    -p 995:995 \
    -p 143:143 \
    -p 993:993 \
    -p 25:25 \
    -p 465:465 \
    -p 587:587 \
    -v iredmail_data_backup:/var/vmail/backup \
    -v iredmail_data_mailboxes:/var/vmail/vmail1 \
    -v iredmail_data_mlmmj:/var/vmail/mlmmj \
    -v iredmail_data_mlmmj-archive:/var/vmail/mlmmj-archive \
    -v iredmail_data_imapsieve_copy:/var/vmail/imapsieve_copy \
    -v iredmail_data_custom:/opt/iredmail/custom \
    -v iredmail_data_ssl:/opt/iredmail/ssl \
    -v iredmail_data_mysql:/var/lib/mysql \
    -v iredmail_data_clamav:/var/lib/clamav \
    -v iredmail_data_sa_rules:/var/lib/spamassassin \
    iredmail/mariadb:nightly
```

On first run, it will generate a self-signed ssl cert (if no cert exists under
`data/ssl/`). This may take a long time, please be patient.

Each time you run the container, few tasks will be ran:

- Update SpamAssassin rules.
- Update ClamAV virus signature database.

# Overview

Only one config file `iredmail-docker.conf` on Docker host.

This file is optional if you prefer overwriting parameters with the `-e`
argument while launching container. For example:

`docker run -e HOSTNAME=mail.mydomain.com FIRST_MAIL_DOMAIN=mydomain.com ...`

We recommend storing them in an env file (`iredmail-docker.conf` in our
example) to save some typing each time you launch the container.

# Required parameters

There're few __REQUIRED__ parameters you __MUST__ set in `iredmail-docker.conf`:

```
# Server hostname. Must be a FQDN. For example, mail.mydomain.com
HOSTNAME=

# First mail domain name. For example, mydomain.com.
FIRST_MAIL_DOMAIN=

# (Plain) password of mail user `postmaster@<FIRST_MAIL_DOMAIN>`.
FIRST_MAIL_DOMAIN_ADMIN_PASSWORD=

# A secret token used for accessing mlmmjadmin API.
MLMMJADMIN_API_TOKEN=

# The secret string used to encrypt/decrypt Roundcube session data.
# Required if you need to run Roundcube webmail.
# You can generate random string with command `openssl rand -base64 24` as the
# des key.
# Every time this key changed, all Roundcube session data becomes invalid and
# users will be forced to re-login.
ROUNDCUBE_DES_KEY=
```

Notes:

- `iredmail-docker.conf` will be read by Docker as an environment file,
  any single quote or double quote will be treated as part of the value.
  Do not use any whitespace, tab in value, and no single or double quotes.
- It will be imported as bash shell script too.

There're many OPTIONAL settings defined in file
`/docker/entrypoints/settings.conf` inside docker container,
you'd like to change any of them, please write the same parameter name with
your custom value in `iredmail-docker.conf` to override it.

# Installed softwares

- Postfix: SMTP server.
- Dovecot: POP3/IMAP/LMTP/Sieve server, also offers SASL AUTH service for Postfix.
- mlmmj: mailing list manager.
- mlmmjadmin: RESTful API server used to manage (mlmmj) mailing lists.
- Amavisd-new + ClamAV + SpamAssassin: anti-spam and anti-virus, DKIM signing and verification, etc.
- iRedAPD: Postfix policy server. Developed by iRedMail team.
- Fail2ban: scans log files and bans bad clients.
- Roundcube: webmail.
- iRedAdmin: web-based admin panel, open source edition.

# Exposed network ports

- 80: HTTP
- 443: HTTPS
- 25: SMTP
- 465: SMTPS (SMTP over SSL)
- 587: SUBMISSION (SMTP over TLS)
- 143: IMAP over TLS
- 993: IMAP over SSL
- 110: POP3 over TLS
- 995: POP3 over SSL
- 4190: Managesieve service
