# Shadowsocks + Chromium + SwitchyOmega

## Install and configure shadowsocks

See more detail in the file `shadowsocks-firefox.md`

## Forcing chromium to use socks5 proxy

You can install the **SwitchyOmega** extension, but the problem is you can not open Chrome Web Store. So you should force chromium to use socks5 proxy on startup, please see [here](https://github.com/shadowsocks/shadowsocks/wiki/Forcing-Chrome-to-Use-Socks5-Proxy). 

``` bash
$ /usr/bin/chromium --proxy-server="socks5://127.0.0.1:1080" --host-resolver-rules="MAP * 0.0.0.0 , EXCLUDE localhost" 
```

It is similar to global mode. Now you can finish installing SwitchyOmega.

## Configure SwitchyOmega

Please see [this page](https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList). Then you can enjoy the auto switchy mode. Enjoy fucking GFW with shadowsocks and chromium.
