# Sample Solution

## Working with Remote Git Repositories

* Install the `chrony` module from VoxPupuli via `Puppetfile` and `r10k`

```yaml
    mod 'puppetlabs/motd'
    mod 'puppetlabs/stdlib'
    mod 'puppetlabs/registry'
    mod 'puppet/chrony'
```

```bash
    $ /opt/puppetlabs/bolt/bin/r10k puppetfile install -v
    INFO	 -> Using Puppetfile '/root/puppetcode/Puppetfile'
    INFO	 -> Deploying module to /root/puppetcode/modules/motd
    INFO	 -> Deploying module to /root/puppetcode/modules/stdlib
    INFO	 -> Deploying module to /root/puppetcode/modules/registry
    INFO	 -> Deploying module to /root/puppetcode/modules/chrony
```

* Add a parameter `time_servers` for setting time servers to `profile:base`

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content = 'Have a nice day!',
    ) {
      include motd
    }
```

* Pass this parameter to class `chrony` in your `profile::base` class

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content = 'Have a nice day!',
    ) {
      include motd

      class { 'chrony':
        servers => $time_servers,
      }
    }
```

* Set time server `ptbtime1.ptb.de`, `ptbtime2.ptb.de` and `ptbtime3.ptb.de` in `common.yaml` hiera

```yaml
    ---
    profile::base::time_servers:
      - ptbtime1.ptb.de
      - ptbtime2.ptb.de
      - ptbtime3.ptb.de
    
    motd::template: 'profile/motd.epp'
    motd::content: 'Welcome to the world of OpenVox!'
```

* Test your code changes locally with `puppet apply`

```bash
    $ puppet apply manifests/site.pp
```

* Create a git branch `student<N>-time`, commit your changes and push the branch

```bash
    $ git switch -c student<N>-time
    Switched to a new branch 'sudent<N>-time'

    $ git add Puppetfile data/common.yaml site-modules/profile/manifests/base.pp

    $ git commit -m 'Add chrony to manage time servers to base profile'
    [student<N>-time 7436b8d] Add chrony to manage time servers to base profile
     3 files changed, 12 insertions(+), 1 deletion(-)

    $ git push origin student<N>-time
    Enumerating objects: 17, done.
    Counting objects: 100% (17/17), done.
    Delta compression using up to 10 threads
    Compressing objects: 100% (7/7), done.
    Writing objects: 100% (9/9), 1.11 KiB | 1.11 MiB/s, done.
    Total 9 (delta 2), reused 0 (delta 0), pack-reused 0
    To git@<git url>:control.git
     * [new branch]      student<N>-time -> student<N>-time
```

* Do a puppet agent run `puppet agent -t --environment student<N>-time`

```bash
    $ git agent -t --environment student<N>-time
```

* After success, merge the branch into `student<N>` and push it

```bash
    $ git checkout student<N>
    Switched to branch 'student<N>'

    $ git merge student<N>-time
    Updating 21f7047..7436b8d
    Fast-forward
    Puppetfile                             | 1 +
    data/common.yaml                       | 5 +++++
    site-modules/profile/manifests/base.pp | 7 ++++++-
    3 files changed, 12 insertions(+), 1 deletion(-)

    $ git push origin student<N>
    Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
    To git@<git url>:control.git
       21f7047..7436b8d  student<N> -> student<N>
```

* Do not forget to cleanup and remove the local an remote branch `student<N>-message`

```bash
    $ git branch -d student<N>-time
    Deleted branch student<N>-time (was 7436b8d).

    $ git push origin :student<N>-time
    To git@<git url>/control.git
     - [deleted]         student<N>-time
```
