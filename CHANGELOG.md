# CHANGLOG for 'safe' script

## 2.0.1 Bug fix: Create mount points

* When a mount point in the config file doesn't exist, create it. Used the wrong variable name previously and didn't test this rare condition, so it wouldn't actually create them. Just tested now and appears to work.

## 2.0.0 Github Update

* Completely rewrote it. Not what I was expecting to do tonight, but it needed to be done.

* The old script didn't use a config file: it had some nasty case statements that mixed code and data to identify what safe file to use.  This was the reason for the whole rewrite. I'm big on separating code and data these days.

* Now tested for use with sudo, so users can maintain $HOME/.safe.conf.  Previously I'd just been using 'sudo -i' to run this script.

* Nice new config file

* New code base is cleaner, more mature than when I wrote this in 2014.

* Rewrote README.md with up to date documentation.

## 1.0.0 Github Update

* Adding Semantic Version for automated releases

* Adding install.sh script to allow automated installs
