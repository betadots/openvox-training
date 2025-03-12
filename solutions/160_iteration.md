# Sample Solution

## Iteration

* Extend your class `profile::base` with a parameter to manage many users
  * Parameter should be of type hash.

```bash
    $ cd ~/puppetcode
    $ vim site-modules/profile/manifests/base.pp
```

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content  = 'Have a nice day!',
      Array[String]       $packages = [],
      Hash[String, Hash]  $users    = {},
    ) {
      include motd
    
      stdlib::ensure_packages($packages)
    
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
    
      $users.each |String $usr, Hash $config| {
        profile::user { $usr:
          * => $config,
        }
      }
    }
```

* Write a defined resource `profile::user` for that case
  * Add a optional parameter for public_keys, use resource type `ssh_authorized_key`

```bash
    $ vim site-modules/profile/manifests/profile::user.pp
```

```puppet
    define profile::user (
      String                 $username    = $title,
      Array[String]          $groups      = [],
      Hash[String, Hash]     $public_keys = {},
    
    ) {
      user { $username:
        gid        => $username,
        managehome => true,
        groups     => $groups,
      }
    
      $public_keys.each |String $comment, Hash $authorized_key| {
        ssh_authorized_key { $comment:
          * => $authorized_key + { user => $username },
        }
      }
    }
```

### Bonus

* Add the option to remove users and their keys again

```puppet
    define profile::user (
      Enum['present', 'absent'] $ensure = 'present',
      String                    $username    = $title,
      Array[String]             $groups      = [],
      Hash[String, Hash]        $public_keys = {},
    
    ) {
      user { $username:
        ensure     => $ensure,
        gid        => $username,
        managehome => true,
        groups     => $groups,
      }
    
      $public_keys.each |String $comment, Hash $authorized_key| {
        $key_ensure = if $ensure == 'absent' or $authorized_key['ensure'] == 'absent' {
          'absent'
        } else {
          'present'
        }
    
        ssh_authorized_key { $comment:
          * => $authorized_key + { ensure => $key_ensure, user => $username },
        }
      }
    }
```
