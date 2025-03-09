# Sample Solution

## Profile Pattern

* Create a dircetory `site-modules` in your environment

```bash
    $ cd ~/puppetcode
    $ mkdir site-modules
```

* Add this one to the `modulepath` in the `environment.conf`

```bash
    # Each environment can have an environment.conf file. Its settings will only
    # affect its own environment. See docs for more info:
    # https://puppet.com/docs/puppet/latest/config_file_environment.html
    
    # Any unspecified settings use default values; some of those defaults are based
    # on puppet.conf settings.
    
    # If these settings include relative file paths, they'll be resolved relative to
    # this environment's directory.
    
    # Allowed settings and default values:
    
    modulepath = ./site-modules:./modules:$basemodulepath
    # manifest = (default_manifest from puppet.conf, which defaults to ./manifests)
    # config_version = (no script; Puppet will use the time the catalog was compiled)
    # environment_timeout = (environment_timeout from puppet.conf, which defaults to 0)
        # Note: unless you have a specific reason, we recommend only setting
        # environment_timeout in puppet.conf.
```

* Create a new module `profile` to the new path

```bash
    $ mkdir -p site-modules/profile/manifests
```

* Add a class `base` to the new module

```bash
    $ vim site-modules/profile/manifests/base.pp
```

* Transfer the declarations from your `site.pp` to the new class

```puppet
    class profile::base {
      include motd
    }
```

* and replace the declaration in your `site.pp` with your new class

```puppet
    node default {
      include profile::base
    }
```

* Run `puppet apply`
