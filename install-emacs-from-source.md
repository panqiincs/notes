# How to install emacs from source

## Step 1: Download source and extract

Run:

```
$ wget ftp://ftp.gnu.org/pub/gnu/emacs/emacs-VERSION.tar.gz
$ tar zxvf emacs-VERSION.tar.gz 
```

## Step 2: Compile and install

Configure:

```
$ cd emacs-VERSION
$ ./configure
$ # install missing dependencies
$ make
$ sudo make install
```

## Step 3: Create a launch for emacs (maybe unnecessary)

Create a new file `/usr/share/applications/Emacs-VERSION.desktop` and add the following:

```
[Desktop Entry]
Version=1.0
Name=Emacs-24.4
Exec=env /usr/local/bin/emacs
Terminal=false
Icon=emacs
Type=Application
Categories=IDE
X-Ayatana-Desktop-Shortcuts=NewWindow
[NewWindow Shortcut Group]
Name=New Window
TargetEnvironment=KDE
```

End.
