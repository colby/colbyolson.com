# [Using Let's Encrypt](/using-letsencrypt)

Go use [Let's Encrypt][], it's an free and easy method of creating trusted certificates.

## Installation

`#TODO`

## Automatic Renewal

Here's a small `cron` script to automatically renew a certificate at 00:00 on the first day of every month.

```
$ cat /etc/cron.d/letsencrypt-renewal
0 0 1 * * root systemctl stop nginx && sleep 1 && letsencrypt certonly -t --standalone -d colbyolson.com --email colby@colbyolson.com --renew-by-default && systemctl start nginx
```

[Let's Encrypt]: https://letsencrypt.org
