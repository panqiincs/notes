# How to use shadowsocks to fuck GFW under CLI?

## Step 0: Install and configure shadowsocks

## Step 1: Install proxychains

Run:

```bash
$ sudo zypper install proxychains
```

## Step 2: Configure proxychains

Open file `/etc/proxychains.conf`, modify the last line under `[ProxyList]` to:

```
socks5 127.0.0.1 1080
```

## Step 3: How to use

Just add `proxychains` before your commands, for example:

```
$ proxychains curl https://twitter.com
```

Enjoy fucking GFW under CLI :)

