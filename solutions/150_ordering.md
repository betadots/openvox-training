# Sample Solution

## Ordering

* Write a simple profile class for nginx consisting of
  * package `nginx`
  * config file `/etc/nginx/nginx.conf
  * and service `nginx`

```bash
    $ cd ~/puppetcode
    $ vim site-modules/profile/manifests/nginx.pp
```

```puppet
    class profile::nginx {
      package { 'nginx':
        ensure => installed,
        before => File['/etc/nginx/nginx.conf'],
      }

      file { '/etc/nginx/nginx.conf':
        ensure => file,
        owner  => 'root',
        group  => 'root',
        mode   => '0644',
        source => 'puppet:///modules/profile/nginx.conf',
        notify => Service['nginx']
      }

      service { 'nginx':
        ensure => running,
        enable => true,
      }
    }
```

**Tip**: copy the config file from a installation you did and remove the package, remeber the `puppet resource` command.

```bash
    $ puppet resource package nginx ensure=installed
    
    $ mkdir site-modules/profile/files
    $ cp /etc/nginx/nginx.conf site-modules/profile/files/

    $ puppet resource package nginx ensure=purged

    $ vim data/roes/webserver.yaml
```

```yaml
    ---
    classes:
      - profile::base
      - profile::myapp
      - profile::nginx
```

A sample solution can be found [here](./solutions/150_ordering.md).

### Bonus

* Use a template instead of a static config file with at least a class parameter, e.g. the port

```puppet
    class profile::nginx (
      Stdlib::Ensure::Service $ensure = 'running',
      Boolean                 $enable = true,
      Stdlib::Port            $port   = 80,
    ) {
      package { 'nginx':
        ensure => installed,
        before => File['/etc/nginx/nginx.conf'],
      }

      file { '/etc/nginx/nginx.conf':
        ensure  => file,
        owner   => 'root',
        group   => 'root',
        mode    => '0644',
        content => epp('modules/profile/nginx.conf.epp', { port => $port }),
        notify  => Service['nginx']
      }

      service { 'nginx':
        ensure => $ensure,
        enable => $enable,
      }
    }
```

```bash
    $ cp site-modules/profile/files/nginx.conf site-modules/profile/templates/nginx.conf.epp
    $ vim site-modules/profile/templates/nginx.conf.epp
```

```puppet
    <%- |
          Stdlib::Port $port,
        | -%>
    # For more information on configuration, see:
    #   * Official English Documentation: http://nginx.org/en/docs/
    #   * Official Russian Documentation: http://nginx.org/ru/docs/
    
    ...
    
        server {
            listen       <%= $port %>;
            listen       [::]:<%= $port %>;
            server_name  _;
            root         /usr/share/nginx/html;
    ...
```
