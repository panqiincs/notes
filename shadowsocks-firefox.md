# How to fuck GFW under linux

## Step 1: Install shadowsocks

Run the following commands:

```bash
$ sudo zypper install python-pip
$ pip install shadowsocks
```

## Step 2: Configure shadowsocks client

Create a new configuration file named `/etc/shadowsocks/config.json`, add the following content:

```
{
    "server": "xx.xx.xx.xx",
    "server_port": xxx,
    "local_port": xxx,
    "password": "xxxxxxxx",
    "method": "xxxxxxxx"
}
```

run shadowsocks client in a terminal:

```bash
$ sslocal -c /etc/shadowsocks/config.json
```

## Step 3: Configure firefox

Firstly, install the add-on named foxyproxy, modify the Preference of this add-on, do as following:
Add new proxy --> Proxy Details --> Manual Proxy Configuration
Host or IP Address: 127.0.0.1, Port: 1080
Select Automatic Proxy Configuration, by PAC, Automatic proxy configuration URL is: https://bstrill.com/proxy.pac

Enjoy fucking GFW :)
