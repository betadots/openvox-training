# Sample Solution

## Hiera

* Rewrite your node declarition to use only an `include motd` instead of a declaration with `class`

```puppet
    node default {
      include motd
    }
```

* Add key `motd::content` with any value to `data/common.yaml`

```yaml
    ---
    motd::content: 'Welcome to the world of OpenVox!'
```

* Run `puppet aplly` and check the result

```bash
    $ puppet apply manifests/site.pp
```

* Add a diffrent value to the same key in `data/nodes/<your certname>.yaml`

```bash
    $ mkdir data/nodes
```

```yaml
    ---
    motd::content: 'Welcome to the world of Puppet!'
```

* Tip: To get the correct certname use `puppet config print certname`

```bash
    $ puppet config print certname
```

* Rerun `puppet apply`

```bash
    $ puppet apply manifests/site.pp
```

### Bonus

* Add a new layer between *nodes* and *common.yaml* named `operatingsystem data (yaml version)` based on `facts.os.family`

```bash
    $ cd ~/puppetfile
    $ vim hiera.yaml
```

```yaml
---
    version: 5
    defaults:
      # The default value for "datadir" is "data" under the same directory as the hiera.yaml
      # file (this file)
      # When specifying a datadir, make sure the directory exists.
      # See https://puppet.com/docs/puppet/latest/environments_about.html for further details on environments.
      # datadir: data
      # data_hash: yaml_data
    hierarchy:
      - name: "Per-node data (yaml version)"
        path: "nodes/%{::trusted.certname}.yaml"
      - name: "operatingsystem data (yaml version)"
        path: "os/%{facts.os.family}.yaml"
      - name: "Other YAML hierarchy levels"
        paths:
          - "common.yaml"
``` 

* Add yaml file for the osfamily of your machine

```bash
    $ facter os.family
    RedHat

    $ mkdir data/os
    $ vim data/os/RedHat.yaml
    $ rm data/nodes/*
```

```puppet
    ---
    motd::content: 'Welcome to my world''
```

**Notice**: Keep in mind the Linux filesystems are case sensitive.

Do not forget to remove the dedicated node file if you wanna see some changes!
