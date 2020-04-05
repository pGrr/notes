Code versioning
=============== 

* Centralized (old): CVS (Concurrent Versions System), SVN (Subversion)
* Decentralized (modern) - DVCS (Decentralized Versioning Control System) - Git, Mercurial, Bazaar

Git (github.com)
----------------

* Install and config
  * `sudo apt-get install git`
  * `git config --global user.name "Name Surname"`
  * `git config --global user.email "myemail@mymail.com"`
* `git init` - initialize repository into the current directory
* `git status` - see current repository status (staging area)
* Add/remove file from the staging area
  * `git add <file>` - add file to the staging area
  * `git reset <file>` - remove staged file from the staging area
  * `git rm --cached <file>` - remove committed file from the tree and the staging area
* `git commit -m "Commit comment"` - commit
  * each commit is identified by a 40-hex-chars unique hash code computed upon the commit informations using SHA1 algorithm 
* `git log` - see commit history
* branches
  * `git branch feature` - create a branch named "feature"
  * `git checkout feature` - switch to branch feature
  * `git checkout -b feature` - create a branch named "feature" and switch to it
  * `git branch` - print branches (`*` = current branch)
* `git remote add origin https://github.com/name/project` - add a remote repository named origin (which is the conventional name)
* `git push origin master` - push master branch to origin remote repository
* Authentication
  * credentials
    * `git config --global credential.helper cache` - cache credentials for 15 minutes
    * `git config --global credential.helper 'cache --timeout=3600'` - cache credentials for 1h
  * ssh
    * `git remote add origin git@github.com:name/project.git` - cloning using `.git` url is necessary
    * `ssh-keygen -t rsa -b 4096 -C "yourMail@example.com"` - create a private ssh key (a password to protect it should be choosed)
    * `eval $(ssh-agent -s)` - verify ssh-agent (service which manages authentication of ssh connections) is running
    * `ssh-add ~/path/to/id_rsa` - add the private ssh key to ssh-agent (usually located in `~/.ssh/`)
    * Add the public ssh key `id_rsa.pub` (usually located in `~/.ssh/`) to the remote repository (in github it's "SSH and GPG keys")
* `git fetch origin` - update local files synching with origin
* `git merge origin/master` - merge remote repository `origin/master` with the local one in the current branch
* `git pull origin/master` - executes both fetch and merge of the remote repository into the local one
* Conflicts - must be resolved modifying the conflicted text, which is shown by git as below:
  ```git
  <<<<<<< HEAD
  text in local branch
  =======
  text in branch-a
  >>>>>>> branch-a
  ```
* Pull requests
  * fork (e.g on github) the repository
  * create a new branch and modify
  * send a pull request (e.g. on github) with a significant title and description that clarify the reason of the request
  * the administrator of the repository reviews (usually, with all of its team) and can leave comments (that can be replied)
  * When finished, it's possible to merge (via github or cli) 
