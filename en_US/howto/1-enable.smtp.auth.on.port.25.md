# Enable SMTP SASL AUTH on port 25

Since iRedMail-0.9.5, SMTP auth on port 25 is disabled by default, all end
users are forced to send email through port 587 (SMTP over TLS). If you need
to allow insecure SMTP auth on port 25 for some reason, please follow steps
below to enable it.

!!! note

    If you have just few clients need to send email through port 25, e.g.
    network printer, old network devices which don't support secure
    connection, you may try another tutorial instead:
    [Allow internal network devices to send email with insecure connection](./additional.smtp.port.html)

* Find comment out settings in Postfix config file `/etc/postfix/main.cf`
  (Linux/OpenBSD) or `/usr/local/etc/postfix/main.cf` (FreeBSD):

```
#
# Enable SASL authentication on port 25 and force TLS-encrypted SASL authentication.
# WARNING: NOT RECOMMENDED to enable smtp auth on port 25, all end users should
#          be forced to submit email through port 587 instead.
#
#smtpd_sasl_auth_enable = yes
#smtpd_tls_auth_only = yes
#smtpd_sasl_security_options = noanonymous
```

!!! note

    With `smtpd_tls_auth_only = yes`, it requires clients to enable STARTTLS
    for secure connection, if you don't want this for some reason, please
    comment it out.

* uncomment the last 4 lines:

```
smtpd_sasl_auth_enable = yes
smtpd_tls_auth_only = yes
smtpd_sasl_security_options = noanonymous
```

* Restart or reload Postfix service.

That's all.
