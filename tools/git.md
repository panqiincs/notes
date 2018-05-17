# Git

### 1: Git does not tell how many commits ahead of origin

In Git version 1.8 or later, make sure you are on the local branch and then run:

```bash
git branch --set-upstream-to origin/remote
```

Reference: [stackoverflow](http://stackoverflow.com/questions/5341077/git-doesnt-show-how-many-commits-ahead-of-origin-i-am-and-i-want-it-to)

### Awesome log configuration:

Very great:

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

