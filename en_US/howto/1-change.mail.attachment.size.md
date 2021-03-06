# Change mail attachment size

[TOC]

To change mail attachment size, we have to change 3 settings.

## Change message size limit in postfix

Postfix is MTA, so we have to change its setting to transfer mail with large
attachment.

For example, to allow mail with 100Mb attachment, please change both
`message_size_limit` and `mailbox_size_limit` settings like below:

```
# postconf -e message_size_limit='104857600'
# postconf -e mailbox_size_limit='104857600'
```

Restart postfix to make it work:

```
# /etc/init.d/postfix restart
```

__NOTES__:

* `104857600` is 100 (MB) x 1024 (KB) x 1024 (Bit).
* Mail will be encoded by mail user agent (Outlook, Thunderbird, etc) before
  transferred, the actual message size will be larger than 100MB, you can
  simplily increase above setting to 110Mb or 120Mb to make it work as expected.
* If `mailbox_size_limit` is smaller than `message_size_limit`, you will get
  error message in Postfix log file like this: `fatal: main.cf configuration
  error: mailbox_size_limit is smaller than message_size_limit`.

If you use mail clients such as Outlook, thunderbird to send mails, it's now
ok to sent large attachment with above setting.

## Change upload file size in Roundcube webmail

If you have Roundcube webmail, please change two more settings:

### Change PHP setting to allow to upload large attachment

You should change `memory_limit`, `upload_max_filesize` and `post_max_size` in
PHP config file `/etc/php.ini`

* on RHEL/CentOS: it's `/etc/php.ini`
* on Debian/Ubuntu, it's `/etc/php5/apache2/php.ini`
* on FreeBSD, it's `/usr/local/etc/php.ini` for Apache, or
  `/etc/php5/fpm/php.ini` for Nginx.
* on OpenBSD, it's `/etc/php-5.4.ini`. If you're running different PHP release,
  the version number `5.4` will be different.

```
memory_limit = 200M;
upload_max_filesize = 100M;
post_max_size = 100M;
```

### Change Roundcube webmail settings to allow large attachment

Change same settings in file `.htaccess` under roundcube root directory:

* on RHEL/CentOS, it's `/var/www/roundcubemail/.htaccess`
* on Debian/Ubuntu, it's `/usr/share/apache2/roundcubemail/.htaccess` or
  `/opt/www/roundcubemail/.htaccess`.
* on FreeBSD, it's `/usr/local/www/roundcubemail/.htaccess`
* on OpenBSD, it's `/var/www/roundcubemail/.htaccess`

Note: this `.htaccess` file may not exist on some Linux/BSD distributions,
if it doesn't exist, you can skip this step.

```
php_value    memory_limit   200M
php_value    upload_max_filesize    100M
php_value    post_max_size  100M
```

Restart Apache or php-fpm service to make it work.

## Change upload file size in Nginx

Find setting `client_max_body_size` in Nginx config file
`/etc/nginx/nginx.conf`, change it to a proper value to match your need.

```
http {
    ...
    client_max_body_size 100m;
    ...
}
```

## Change upload file size in SOGo-3.x

SOGo-3.x introduces parameter `WOMaxUploadSize` to limit upload file size, you
can add it in SOGo config file `/etc/sogo/sogo.conf` with a proper attachment
size.

```
// set the maximum allowed size for content being sent to SOGo using a PUT or
// a POST call. This can also limit the file attachment size being uploaded
// to SOGo when composing a mail.
//
//  - The value is in kilobyte.
//  - By default, the value is 0, or disabled so no limit will be set.
WOMaxUploadSize = 102400;
```

Restarting SOGo service is required.

## Change attachment size limit in Microsoft Outlook

Outlook has its own attachment size limit, and will raise error like `The
attachment size exceeds the allowable limit.`

To modify the default attachment limit size in Outlook on Windows system,
follow these steps:

* Exit Outlook.
* Start Registry Editor.
* Locate and then select one of the following registry subkeys:
    * Manually create the path in the registry if it does not currently exist.
    * `14.01 means Outlook version number, it may be different on your system.

```
HKEY_CURRENT_USER\Software\Microsoft\Office\14.0\Outlook\Preferences
HKEY_CURRENT_USER\Software\Policies\Microsoft\Office\14.0\Outlook\Preferences
```

* Add the following registry data under this subkey:

    * Value type: DWORD
    * Value name: MaximumAttachmentSize
    * Value data: An integer that specifies the total maximum allowable
      attachment size. For example, specify 30720 (Decimal) to configure a
      30-MB limit.

    Notes:

    * Specify a value of zero (0) if you want to configure no limit for attachments.
    * Specify a value that is less than 20 MB if you want to configure a limit
      that is less than the default 20 MB.

* Exit Registry Editor
* Start Outlook.

Reference: <https://support.microsoft.com/en-us/kb/2222370>
