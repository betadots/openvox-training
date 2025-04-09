# Solution: Grafana

## Setup a Grafana with PostgreSQL as Database Backend

* Install modules `puppetlabs-postgresql` and `puppet-grafana` and all their dependencies, pin your modules to the last version 

```bash
    $ cd ~/puppetcode
    $ cat > Puppetfile <<EOF
    # The following directive installs modules to the managed moduledir.
    moduledir 'modules'

    mod 'puppetlabs-stdlib',        '9.7.0'

    mod 'puppetlabs-motd',          '7.2.0'
    mod 'puppetlabs-registry',      '5.0.3'

    mod 'puppet-nginx',             '6.0.1'
    mod 'puppetlabs-concat',        '9.1.0'

    mod 'puppet-chrony',            '4.0.0'
    mod 'puppet-systemd',           '8.1.0'
    mod 'puppetlabs-inifile',       '6.2.0'

    mod 'puppet-grafana',           '14.1.0'
    mod 'puppet-archive',           '7.1.0'

    mod 'puppetlabs-postgresql',    '10.5.0'
    mod 'puppetlabs-apt',           '10.0.1'
    EOF

    $ /opt/puppetlabs/bolt/bin/r10k puppetfile install -v
```

* Start writing a class `profile::grafana` and manage a PostgreSQL database `grafana`
  * with owner and password
  * implement password as parameter `db_password`

```bash
    $ cd ~/puppetcode
    $ vim site-modules/profile/manifests/grafana.pp
```

```puppet
    class profile::grafana (
      String[1]   $db_password,
    ) {
      include postgresql::server

      postgresql::server::db { 'grafana':
        user     => 'grafana',
        password => $db_password,
        owner    => 'grafana',
      }
    }
```

* Write a smoke test
  * with an include of `profile::base`
  * and an explicit declaration of `profile::grafana` with set `db_password`
  * run the test locally

```bash
    $ cd ~/puppetcode
    $ mkdir site-modules/profile/examples
    $ vim site-modules/profile/examples/grafana.pp
```

```puppet
    include profiel::base

    class { 'profile::grafana':
      db_password => 'supersecret',
    }
```

```bash
    $ cd ~/puppetcode
    $ puppet apply site-modules/profile/examples/grafana.pp
```

* Expand your new class with a declarition of class `grafana`
  * db type is `postgres`
  * the postgresql dbms is listening on port `5432` on localhost

```puppet
    class profile::grafana (
      String[1]   $db_password,
    ) {
      include postgresql::server
    
      postgresql::server::db { 'grafana':
        user     => 'grafana',
        password => $db_password,
        owner    => 'grafana',
      }
    
      -> class { 'grafana':
        cfg => {
          database => {
            type     => 'postgres',
            host     => '127.0.0.1:5432',
            name     => 'grafana',
            user     => 'grafana',
            password => $db_password,
          },
        },
      }
    }
```

* Rerun your smoke test, if succeded grafana is reachable on port 3000

```bash
    $ puppet apply site-modules/profile/examples/grafana.pp
```


## Connect an upstream Nginx as a Reverse Proxy

* Check if module `puppet-nginx` and its dependencies are already installed
* Change the listener from Grafana to `localhost`
  * config option is `http_addr`

```puppet
    class profile::grafana (
      String[1]   $db_password,
    ) {
      include postgresql::server
    
      postgresql::server::db { 'grafana':
        user     => 'grafana',
        password => $db_password,
        owner    => 'grafana',
      }
    
      -> class { 'grafana':
        cfg => {
          server   => {
            http_port => '127.0.0.1:3000',
          },
          database => {
            type      => 'postgres',
            host      => '127.0.0.1:5432',
            name      => 'grafana',
            user      => 'grafana',
            password  => $db_password,
          },
        },
      }
    }
```
 
* Add class `nginx` to proxy to grafana on `localhost:3000`

```puppet
    class profile::grafana (
      String[1]   $db_password,
    ) {
      include postgresql::server
    
      postgresql::server::db { 'grafana':
        user     => 'grafana',
        password => $db_password,
        owner    => 'grafana',
      }
    
      -> class { 'grafana':
        cfg => {
          server   => {
            http_addr     => '127.0.0.1',
          },
          database => {
            type          => 'postgres',
            host          => '127.0.0.1:5432',
            name          => 'grafana',
            user          => 'grafana',
            password      => $db_password,
          },
        },
      }
    
      include nginx
      nginx::resource::server { 'grafana.example.com':
        listen_port => 80,
        proxy       => 'http://localhost:3000',
      }
    }
```


## Secure your hosts with `nftables` local firewalls

* Install module `puppet-nftables` and all its dependencies

```bash
    $ cd ~/puppetcode
    $ cat > Puppetfile <<EOF
    # The following directive installs modules to the managed moduledir.
    moduledir 'modules'

    mod 'puppetlabs-stdlib',        '9.7.0'

    mod 'puppetlabs-motd',          '7.2.0'
    mod 'puppetlabs-registry',      '5.0.3'

    mod 'puppet-nginx',             '6.0.1'
    mod 'puppetlabs-concat',        '9.1.0'

    mod 'puppet-chrony',            '4.0.0'
    mod 'puppet-systemd',           '8.1.0'
    mod 'puppetlabs-inifile',       '6.2.0'

    mod 'puppet-grafana',           '14.1.0'
    mod 'puppet-archive',           '7.1.0'

    mod 'puppetlabs-postgresql',    '10.5.0'
    mod 'puppetlabs-apt',           '10.0.1'

    mod 'puppet-nftables',          '4.2.0'
    EOF

    $ /opt/puppetlabs/bolt/bin/r10k puppetfile install -v
```

* Setup a firewall on all hosts
  * allow all outgoing traffic and incoming ssh and icmp
  * additinal only for Grafana hosts allow incomming http

```puppet
    class profile::base (
      Array[Stdlib::Host] $time_servers,
      String              $content  = 'Have a nice day!',
      Array[String]       $packages = [],
      Hash[String, Hash]  $users    = {},
    ) {
      include motd
      include nftables
    
      stdlib::ensure_packages($packages)
      ...
    }
```

```bash
    $ cd ~/puppetcode
    $ vim data/common.yaml
```

```yaml
    ---
    classes:
      - profile::base
    
    profile::base::time_servers:
      - ptbtime1.ptb.de
      - ptbtime2.ptb.de
      - ptbtime3.ptb.de
    
    nftables::out_all: true
    nftables::in_ssh: true
    nftables::in_icmp: true
    ...
```

```puppet
    class profile::grafana (
      String[1]   $db_password,
    ) {
      include nftables::rules::http
      include postgresql::server
    
      postgresql::server::db { 'grafana':
        user     => 'grafana',
        password => $db_password,
        owner    => 'grafana',
      }
    
      -> class { 'grafana':
        cfg => {
          server   => {
            http_addr     => '127.0.0.1',
          },
          database => {
            type          => 'postgres',
            host          => '127.0.0.1:5432',
            name          => 'grafana',
            user          => 'grafana',
            password      => $db_password,
          },
        },
      }
    
      include nginx
      nginx::resource::server { 'grafana.example.com':
        listen_port => 80,
        proxy       => 'http://localhost:3000',
      }
    }
``` 
