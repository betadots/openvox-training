# Sample Solution

## Create a Git Repository

* Change to your working working directory

```bash
    $ cd ~/puppetcode
```

* Create a git repository `git init`

```bash
    $ git init
    Initialized empty Git repository in /root/puppetcode/.git/
```

* Add a file *.gitignore* with lines: `modules` and `*.swp` to explude the modules directory from git

```bash
    *.swp
    modules
```

* Check `git status`

```bash
    $ git status
    On branch main

    No commits yet
    
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
    	.gitignore
    	Puppetfile
    	data/
    	environment.conf
    	hiera.yaml
    	manifests/
    	site-modules/
    
    nothing added to commit but untracked files present (use "git add" to track)
```

* Rename the current branch `git branch -m student\<N\>`

```bash
    $ git branch -m student<N>
    $ git status
    On branch student<N>

    No commits yet
    
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
    	.gitignore
    	Puppetfile
    	data/
    	environment.conf
    	hiera.yaml
    	manifests/
    	site-modules/
    
    nothing added to commit but untracked files present (use "git add" to track)
```

* Create a connection to the remote repository `git remote add origin git@\<git url\>:control.git`

```bash
    $ git remote add origin git@<git url>:control.git
    $ git remote -v
    origin	git@<git url>:ontrol.git (fetch)
    origin	git@<git url>:control.git (push)
```

* Add files and directories to your new repository `git add .`

```bash
    $ git add .
```

* Check `git status`

```bash
    $ git status
    On branch student<N>
    
    No commits yet
    
    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
    	new file:   .gitignore
    	new file:   Puppetfile
    	new file:   data/common.yaml
    	new file:   data/os/RedHat.yaml
    	new file:   environment.conf
    	new file:   hiera.yaml
    	new file:   manifests/site.pp
    	new file:   site-modules/profile/manifests/base.pp
    	new file:   site-modules/profile/templates/motd.epp
```

* Do yout first commit `git commit -m 'Initial commit'`

```bash
    $ git commit -m 'Initial commit'
    [student<N> (root-commit) 21f7047] Initial commit
     9 files changed, 56 insertions(+)
     create mode 100644 .gitignore
     create mode 100644 Puppetfile
     create mode 100644 data/common.yaml
     create mode 100644 data/os/RedHat.yaml
     create mode 100644 environment.conf
     create mode 100644 hiera.yaml
     create mode 100644 manifests/site.pp
     create mode 100644 site-modules/profile/manifests/base.pp
     create mode 100644 site-modules/profile/templates/motd.epp
```

* Synchronize the local repository with the remote `git push origin student\<N\>`

```bash
    $ git push origin lbetz
    Enumerating objects: 18, done.
    Counting objects: 100% (18/18), done.
    Delta compression using up to 10 threads
    Compressing objects: 100% (10/10), done.
    Writing objects: 100% (18/18), 2.23 KiB | 2.23 MiB/s, done.
    Total 18 (delta 0), reused 0 (delta 0), pack-reused 0
    To git@<git url>:control.git
     * [new branch]      student<N> -> student<N>
```
