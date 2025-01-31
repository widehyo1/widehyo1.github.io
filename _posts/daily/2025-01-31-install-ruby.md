---
layout: post
title: install ruby
subtitle: install ruby with rvm
tags: [commandline, ruby]
comments: true
author: widehyo
---

```bash
widehyo1.github.io on  main via 💎 v3.0.2 ❯ which ruby
/usr/bin/ruby

widehyo1.github.io on  main via 💎 v3.0.2 ❯ ls -l /usr/bin/ruby
lrwxrwxrwx 1 root root 7 Dec  3  2021 /usr/bin/ruby -> ruby3.0

widehyo1.github.io on  main via 💎 v3.0.2 ❯ ls -l /usr/bin/ruby
ruby     ruby3.0

widehyo1.github.io on  main via 💎 v3.0.2 ❯ ls -l /usr/bin/ruby3.0
-rwxr-xr-x 1 root root 14488 Oct 25 14:43 /usr/bin/ruby3.0

widehyo1.github.io on  main via 💎 v3.0.2 ❯ rvm
Command 'rvm' not found, but there are 19 similar ones.

widehyo1.github.io on  main via 💎 v3.0.2 ❯ rvm
Command 'rvm' not found, but there are 19 similar ones.

widehyo1.github.io on  main via 💎 v3.0.2 ❯ curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.29.12.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc
gpg: directory '$HOME/.gnupg' created
gpg: keybox '$HOME/.gnupg/pubring.kbx' created
gpg: Signature made Sat Jan 16 03:46:22 2021 KST
gpg:                using RSA key 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
gpg: Can't check signature: No public key
GPG signature verification failed for '$HOME/.rvm/archives/rvm-1.29.12.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --keyserver hkp://keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
    command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -

In case of further problems with validation please refer to https://rvm.io/rvm/security


widehyo1.github.io on  main via 💎 v3.0.2 took 5s ❯ command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
gpg: key 3804BB82D39DC0E3: 47 signatures not checked due to missing keys
gpg: $HOME/.gnupg/trustdb.gpg: trustdb created
gpg: key 3804BB82D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no ultimately trusted keys found

widehyo1.github.io on  main via 💎 v3.0.2 ❯ command curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
gpg: key 105BD0E739499BDB: public key "Piotr Kuczynski <piotr.kuczynski@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1

widehyo1.github.io on  main via 💎 v3.0.2 ❯ curl -sSL https://get.rvm.io | bash -s stable
Downloading https://github.com/rvm/rvm/archive/1.29.12.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc
gpg: Signature made Sat Jan 16 03:46:22 2021 KST
gpg:                using RSA key 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
gpg: Good signature from "Piotr Kuczynski <piotr.kuczynski@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 7D2B AF1C F37B 13E2 069D  6956 105B D0E7 3949 9BDB
GPG verified '$HOME/.rvm/archives/rvm-1.29.12.tgz'
Installing RVM to $HOME/.rvm/
    Adding rvm PATH line to $HOME/.profile $HOME/.mkshrc $HOME/.bashrc $HOME/.zshrc.
    Adding rvm loading line to $HOME/.profile $HOME/.bash_profile $HOME/.zlogin.
Installation of RVM in $HOME/.rvm/ is almost complete:

  * To start using RVM you need to run `source $HOME/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
Thanks for installing RVM 🙏
Please consider donating to our open collective to help us maintain RVM.

👉  Donate: https://opencollective.com/rvm/donate
```
