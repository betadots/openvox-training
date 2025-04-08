# Sample Solution

## Hiera Lookup Behavior

* Add a new layer between nodes and common.yaml named `operatingsystem data (yaml version)` based on `facts.os.family`

```bash
    $ cd ~/puppetcode
    $ vim hiera.yaml`
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

* Add a new parameter `packages` to `profile::base` to install some packages (have a look at function stdlib::ensure_packages)

```bash
    $ vim site-modules/profile/manifests/base.pp
```

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content  = 'Have a nice day!',
      Array[String]       $packages = [],
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
    }
```
  
* Use a node specific, os specific list of packages and a default one in `common.yaml`

```bash
    $ vim data/os/RedHat.yaml
```

```yaml
    ---
    profile::base::packages:
      - postgresql
      - vim-enhanced
```

```bash
    $ vim data/os/Debian.yaml
```

```yaml
    ---
    profile::base::packages:
      - postgresql-client
      - vim
```

```bash
    $ vim data/common.yaml
```

```yaml
    ---
    profile::base::packages:
      - ca-certificates
      - htop
```

* Do a merge behavior change and explain your decision

```bash
    $ vim data/common.yaml
```

```yaml
    ---
    profile::base::packages:
      - ca-certificates
      - htop

    lookup_options:
      profile::base::packages:
        merge: unique
```
