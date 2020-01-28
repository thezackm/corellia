# Setting up SSH for multiple accounts

### Initial Clone: Setup the local user for this repo

**WORK**
```
git config --global user.email "zmutchler@newrelic.com"
git config --global user.name "Zack Mutchler"
```

**PERSONAL**
```
git config --global user.email "zack.mutchler@gmail.com"
git config --global user.name "zackm"
```

### Configure the `~/.ssh/config` file with targeted logins for multiple keys

```
Host github.com-thezackm
  HostName github.com
  User git
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa_personal

Host source.datanerd.us-zmutchler
  HostName source.datanerd.us
  User git
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
```

### Update the local repo's `./.git/config` file to set the remote URL to your targeted user
ex: `url = git@github.com-<USER>:user/repo.git`
```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
	precomposeunicode = true
[remote "origin"]
	url = git@github.com-thezackm:thezackm/corellia.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[user]
	name = zackm
	email = zack.mutchler@gmail.com
```