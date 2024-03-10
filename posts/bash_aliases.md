# Bash Aliases: Take Them With You

## *nix Aliases
We all know how aliases on the *nix system help to create short commands for the long ones. E.g. ```kga``` can stand for ```kubectl get all``` or ```gl``` for ```git log --name-status --decorate```. Software associates create such aliases and write them in the ```~/.bashrc``` file. So aliases are available whenever they logged in to their machine.

## The Challenge
However, associates also need to log in to other *nix machines for various purposes. Say a developer needs to log in to the SV system for live troubleshooting of an issue. Or a DevOps Engineer requires to log in to various cloud VMs. One has become very used to his/her alias s/he has created. It's very difficult to copy/paste the aliases every time s/he logs in to the another machine.

## The Solution
Here is one way to address this challenge. Create your aliases, categorize and put them in different files. Say [General Aliases](https://github.com/simplyatul/bin/blob/master/gen_export_and_aliases), [Git Aliases](https://github.com/simplyatul/bin/blob/master/git_aliases), [Docker Aliases](https://github.com/simplyatul/bin/blob/master/docker_aliases) etc. Then create a [setaliases.sh](https://github.com/simplyatul/bin/blob/master/setaliases.sh) file. Download it on the machine you have logged in.

```Shell
wget https://raw.githubusercontent.com/simplyatul/bin/master/setaliases.sh
```

and then source it
```Shell
source setaliases.sh
```

That's it. All your aliases are set :slightly_smiling_face:

If you are frequently logging in to a particular machine, then just copy the above source command in the ```~/.bashrc``` file.
