# How to fuck GFW under linux

## Step 1: Install shadowsocks

Run the following commands:

```bash
$ sudo zypper install python-pip
$ pip install shadowsocks
```

## Step 2: Configure shadowsocks client

Create a new file named /etc/shadowsocks/config.json, add the following content:

```
{
    "server": "45.78.30.72",
    "server_port": 443,
    "local_port": 1080,
    "password": "ZTk2Y2RkZD",
    "method": "aes-256-cfb"
}
```

run shadowsocks client in a terminal:

```bash
$ sslocal -c /etc/shadowsocks/config.json
```

## Step 3: Configure firefox

Firstly, install the add-on named foxyproxy, in the Preference of this add-on, do as following:
Add new proxy --> Proxy Details --> Manual Proxy Configuration
Host or IP Address: 127.0.0.1, Port: 1080
Select Automatic Proxy Configuration, by PAC, Automatic proxy configuration URL is: https://bstrill.com/proxy.pac

Enjoy fucking GFW :)
