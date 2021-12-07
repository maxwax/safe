# CHANGLOG for 'safe' script

## 2.0.2

* Bug fix: When selecting a single line in the config file that uniquely defines the config for a safe, you must scan the config file for the combintation of the "safe_file" keyword at the start of the line plus the unique name of the safe file itself.  Previously it would scan for lines that start with "safe_file" then filter them to only process the line for the unique safe name.  In cases where the safe name, such as 'maxwell', might also appear on multiple other safe_file lines, the program would fail to identify a single unique safe configuration and fail to work.

Ex:
safe_file personal maxwell LUKS_personal /mnt/secure/personal personal.luks
safe_file maxwell maxwell LUKS_maxwell /mnt/secure/maxwell maxwell.luks

Opening the safe 'personal' would work because that keyword is only found on one line, while a safe 'maxwell' would fail because it is not unique.

The fix searches for 'safe_file maxwell' and now both lines are unique for selection purposes.

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
