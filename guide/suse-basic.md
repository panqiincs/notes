## Use USTC mirror

Use openSUSE Leap 42.2 as an example:

```bash
sudo zypper mr -da

sudo zypper ar -fc https://mirrors.ustc.edu.cn/opensuse/distribution/leap/42.2/repo/oss USTC:42.2:OSS
sudo zypper ar -fc https://mirrors.ustc.edu.cn/opensuse/distribution/leap/42.2/repo/non-oss USTC:42.2:NON-OSS
sudo zypper ar -fc https://mirrors.ustc.edu.cn/opensuse/update/leap/42.2/oss USTC:42.2:UPDATE-OSS
sudo zypper ar -fc https://mirrors.ustc.edu.cn/opensuse/update/leap/42.2/non-oss USTC:42.2:UPDATE-NON-OSS

sudo zypper ref
```

## Double-click to open files and folders

In mouse controls, Icons, select the right option, apply.

## Fix font rendering in openSUSE

Please see here: [How to Fix Font Rendering in OpenSUSE](http://sapiengames.com/2014/12/26/how-to-fix-font-rendering-in-opensuse/)

