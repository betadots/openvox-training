# Sample Soltution

## Classes

* Examine the `puppetlabs/motd` module, e.g. [here](https://github.com/puppetlabs/puppetlabs-motd/blob/main/REFERENCE.md) or

```bash
    $ less modules/motd/manifests/init.pp
```

* and use it to manage a message of the day

```bash
    $ cd ~/puppetcode
    $ vim manifests/site.pp
```

* add your code to yout node declaration in `manifests/site.pp`

```puppet
    node default {
      class { 'motd':
        content => 'Hello wellcome to the world of OpenVox!',
      }
    }
```
