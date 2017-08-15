## How to install chrome in OpenSUSE Leap

### Steps

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

### Reference

1. [How to Install Google Chrome On OpenSUSE Leap](https://www.linuxbabe.com/desktop-linux/how-to-install-google-chrome-on-opensuse-leap-42-1)
