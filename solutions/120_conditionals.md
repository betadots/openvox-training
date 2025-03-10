# Sample Solution

## Conditionals

* Use a condional to manage `chrony` only on RedHat platforms and `ntp` on Debian.

```bash
    $ cd ~/puppetcode
    $ git switch -c student<N>-ntp
    $ vim Puppetfile
```

```yaml
    mod 'puppetlabs/motd'
    mod 'puppetlabs/stdlib'
    mod 'puppetlabs/registry'
    mod 'puppet/chrony'
    mod 'puppetlabs/ntp'
```

```bash
    $ /opt/puppetlabs/bolt/bin/r10k puppetfile install -v
```

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content  = 'Have a nice day!',
    ) {
      include motd

      case $facts['os']['family'] {
        'redhat': {
          class { 'chrony':
            servers => $time_servers,
          }
        }
        'debian': {
          class { 'ntp':
            servers => $time_servers,
          }
        }
        default: {
          fail("Your platform ${facts['os']['name']} is not supported!")
        }
      }
      
    }
```

* Do also the complete git/puppet workflow

```bash
    $ puppet apply manifests/site.pp

    $ git add Puppetfile site-modules/profile/manifests/base.pp
    $ git commit -m 'Manage ntp on Debian platforms'
    $ git push origin student<N>-ntp

    $ puppet agent -t --environment student<N>-ntp

    $ git checkout student<N>
    $ git merge student<N>-ntp
    $ git push origin student<N>

    $ git branch -d student<N>-ntp
    $ git push origin :student<N>-ntp
```
