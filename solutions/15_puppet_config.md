# Sample Solution

## Puppet Agent Configuration

* Check out the current state of `report` that controls the sending of reports

```bash
    $ puppet config print report
    true
```

* Change the behavoir

```bash
    $ puppet config set report false
```

* After that take a look in the confiuguration file `puppet.conf`

```bash
    $ cat /etc/puppetlabs/puppet/puppet.conf
    [main]
    report = false
    # This file can be used to override the default puppet settings.
    # See the following links for more details on what settings are available:
    # - https://puppet.com/docs/puppet/latest/config_important_settings.html
    # - https://puppet.com/docs/puppet/latest/config_about_settings.html
    # - https://puppet.com/docs/puppet/latest/config_file_main.html
    # - https://puppet.com/docs/puppet/latest/configuration.html
```
