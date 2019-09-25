## lemonitor

Bash script that monitor TLS endpoints and sends email if Letsencrypt certs are about to expire.

### Install

- Copy `lemonitor` to `/opt/lemonitor/lemonitor`
- Copy `lemonitor.conf` to `/opt/lemonitor/lemonitor.conf`
- Copy `lemonitor.hosts` to `/opt/lemonitor/lemonitor.hosts`
- Copy `lemonitor.service` and `lemonitor.timer` to`/etc/systemd/system`. 

### Configure and run

- At a minimum, edit `EMAIL_TO` in `/opt/lemonitor/lemonitor.conf`. Local mail delivery must be configured. 
- Edit `/opt/lemonitor/lemonitor.hosts` and add your hosts. Run `lemonitor -f` to see the output. 
- Enable the timer: `systemctl enable lemonitor.timer --now` 

## Email setup

To have `mail` deliver email to a remote SMTP server such as Gmail, configure an MTA. OpenSMTPD is simple: 

Install it: 

    $ pacman -S opensmtpd s-nail         # Config in /etc/smtpd/smtpd.conf
    $ yum install opensmtpd mailx        # Config in /etc/opensmtpd/smtpd.conf
    $ apt install opensmtpd mailutils    # Config in /etc/smtpd.conf

Configure: 

- Forward all email to foo@example.com

    $ mkdir -p /etc/smtpd

    $ cat /etc/smtpd/aliases
    @ foo@example.com

- Configure 

    $ cat smtpd.conf
    listen on localhost
    
    table aliases file:/etc/smtpd/aliases
    table secrets file:/etc/smtpd/secrets
    
    accept from local for any relay via secure+auth://<USERNAME>@<DOMAIN> auth <secrets> as "root@<HOSTNAME>"

    $ cat /etc/smtpd/secrets
    <USERNAME>      <SMTP_SUBMISSION_USER>@<DOMAIN>:<PASSWORD>

    $ systemctl enable smtpd --now
    $ smtpctl monitor

- Test

    $ echo "Email body" | mailx -s "Email subject" foo@example.com

