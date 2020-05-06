## lemonitor

Bash script that monitor TLS endpoints and sends email if Letsencrypt certs are about to expire.

Test run: `lemonitor <domain>[:<port>]`

### Install

- Copy `lemonitor` to `/opt/lemonitor/lemonitor`
- Copy `lemonitor.conf` to `/etc/lemonitor.conf`
- Copy `lemonitor.hosts` to `/etc/lemonitor.hosts`
- Copy `lemonitor.service` and `lemonitor.timer` to `/etc/systemd/system`.

### Configure and run

- At a minimum, edit `EMAIL_TO` in `/etc/lemonitor.conf`. Local mail delivery must be configured (see below). 
- Edit `/etc/lemonitor.hosts` and add your hosts. Run `lemonitor -f` to see the output. 
- Enable the timer: `systemctl enable lemonitor.timer --now` 

## Email setup

To have email delivered to a remote SMTP server such as Gmail, configure an MTA such as OpenSMTPD.

Install it: 

    $ pacman -S opensmtpd s-nail         # Config in /etc/smtpd/smtpd.conf
    $ yum install opensmtpd mailx        # Config in /etc/opensmtpd/smtpd.conf
    $ apt install opensmtpd mailutils    # Config in /etc/smtpd.conf

Configure Email (User: foo@example.com, Server: smtp.example.com:587, Password: PASSWORD):

    $ mkdir -p /etc/smtpd
    
    $ cat /etc/smtpd/aliases
    @ foo@example.com
    
    $ cat smtpd.conf
    listen on localhost
    
    table aliases file:/etc/smtpd/aliases
    table secrets file:/etc/smtpd/secrets
    
    action "relay" relay host "tls+auth://lemonitor@example.com:587" auth <secrets> mail-from "root@lemonitor"
    match from any for local action "relay"
    
    $ cat /etc/smtpd/secrets
    lemonitor      foo@example.com:PASSWORD

    $ systemctl enable smtpd --now
    $ smtpctl monitor

Replace as needed, and test

    $ echo "Email body" | mailx -s "Email subject" foo@example.com
