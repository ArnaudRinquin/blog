There are cases where you have to use several [GitHub](https://github.com) or [Bitbucket](https://bitbucket.com) accounts on the same computer. The problem occured to me as I often need to work on a professional project from my personnal computer (and vice versa).

Switching between them requires 2 steps:

1. change the `ssh` key
2. change the git `user.email` setting

There are efficient solutions for both steps. For clarity reasons, this article will show you a setup for switching between two GitHub accounts. However, everything is adaptable to bitbucket or any git repository.

# What you'll get
My solution takes the form of a single git alias. Once executed, the current project user will be attached to another account:

```
# default github user is ArnaudRinquin, rinquin.arnaud@gmail.com (personnal account)

git clone git@github.com:employer/project.git
cd project
git gopro

# project github user is ArnaudWopata, arnaud.rinquin@wopata.com (pro account)
```

# Prerequisites

## Generate ssh keys

First things first, you'll have to generate as many keys as the number of accounts you have. Just specify a different email and name for every key:

```
ssh-keygen -t rsa -C "rinquin.arnaud@gmail.com" -f '/Users/arnaudrinquin/.ssh/id_rsa'

[...]

ssh-keygen -t rsa -C "arnaud.rinquin@wopata.com" -f '/Users/arnaudrinquin/.ssh/id_rsa_pro'
```

## Link them to your GitHub / Bitbucket accounts
This step is fast if you are well organized:

1. copy default public key `pbcopy < ~/.ssh/id_rsa.pub`
1. login to your [GitHub](https://github.com) acount
1. paste the key in the [`add SSH key` github page](https://github.com/settings/ssh)

1. copy other public key `pbcopy < ~/.ssh/id_rsa_pro.pub`
1. repeat and adapt steps 2 to 4 for every other account

# Step 1. Automatic ssh key switching.
We can configure `ssh` to send a use a specific encryption key depending on the `host`. The nice thing is that you can have several aliases for the same `hostname`.

See this example `~/.ssh/config` file:

```
# Default GitHub
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa

# Professional github alias
Host github_pro
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_pro
```

## git remote configuration
You can now use these aliases in the git remotes by changing `git@github.com` by `git@github_pro`.

You can either change your existing projects remotes (using something like `git remote origin set-url git@github_pro:foo/bar.git`) or adapt them directly when cloning them.

```
git clone git@github.com:ArnaudRinquin/atom-zentabs.git
```

using alias, it become:

```
git clone git@github_pro:ArnaudRinquin/atom-zentabs.git`
```

# Step 2. Changing git user.email

Git config settings can be global or per project. In our case, we want a per project settings. It is very easy to change it:

```
git config user.email 'arnaud.rinquin@wopata.com'
```

While this is easy, it takes way to long for the developers we are. We can write a very simple git alias for that.

We are going to add it to the `~/.gitconfig` file.

```
[user]
	name = Arnaud Rinquin
	email = rinquin.arnaud@gmail.com

...

[alias]
	setpromail = "config user.email 'arnaud.rinquin@wopata.com'"
```

Then, all we have to do is `git setpromail` to have our email changed for this project only.

# Step 3. One command switch please?!
Wouldn't it be nice to switch from default account to a specified one with a single parameter-less command? This is definitely possible. This command will have two steps:

* change current project remotes to the chosen aliases
* change current project user.email config

We already have a one command solution for the second step, but the first one is way harder.

## One command remote host change
Here comes the solution in the form of another git alias command to add to your `~/.gitconfig`:

```
[alias]
  changeremotehost = !sh -c \"git remote -v | grep '$1.*fetch' | sed s/..fetch.// | sed s/$1/$2/ | xargs git remote set-url\"
```

This allows changing all remotes from one host to another (the alias). See the example:

```
$ > git remote -v
origin	git@github.com:ArnaudRinquin/arnaudrinquin.github.io.git (fetch)
origin	git@github.com:ArnaudRinquin/arnaudrinquin.github.io.git (push)

$ > git changeremotehost github.com github_pro

$ > git remote -v
origin	git@github_pro:ArnaudRinquin/arnaudrinquin.github.io.git (fetch)
origin	git@github_pro:ArnaudRinquin/arnaudrinquin.github.io.git (push)
```

## Combine them all
We now just have to combine the two commands into one, this is quite easy. See how I also integrate bitbucket host switching.

```
[alias]
  changeremotehost = !sh -c \"git remote -v | grep '$1.*fetch' | sed s/..fetch.// | sed s/$1/$2/ | xargs git remote set-url\"
  setpromail = "config user.email 'arnaud.rinquin@wopata.com'"
  gopro = !sh -c \"git changeremotehost github.com github_pro && git changeremotehost bitbucket.com bitbucket_pro && git setpromail\"
```
