## How to install chrome in OpenSUSE Leap

### Installation steps

Add repository:

```
$ sudo zypper ar http://dl.google.com/linux/chrome/rpm/stable/x86_64 Google-Chrome
```

it will show:

```
Adding repository 'Google-Chrome' ..............................[done]
Repository 'Google-Chrome' successfully added
Enabled     : Yes                                                
Autorefresh : No                                                 
GPG Check   : Yes                                                
URI         : http://dl.google.com/linux/chrome/rpm/stable/x86_64
```

Now refresh repository:

```
$ sudo zypper ref
```

Now download and import Google's public signing key:

```
$ wget https://dl.google.com/linux/linux_signing_key.pub
$ sudo rpm --import linux_signing_key.pub
```

We can install it now:

```
$ sudo zypper in google-chrome-stable
```

### Disable kwallet popups

When you startup chrome in KDE, it always appears a kwallet popup, asking you to input a password.

Edit `/usr/bin/google-chrome-stable`, add `"--password-store=basic"` option in the bottom, it looks like:

```bash
else
  exec -a "$0" "$HERE/chrome"  "$@" "--password-store=basic"
fi
```

Reboot and try, the popup will never occur again.

### Reference

1. [How to Install Google Chrome On OpenSUSE Leap](https://www.linuxbabe.com/desktop-linux/how-to-install-google-chrome-on-opensuse-leap-42-1)
