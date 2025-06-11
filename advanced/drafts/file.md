# File Management

## file_line

* Ensures that a given line is contained within a file
* Custom resource type
* Included in puppetlabs/stdlib module since 2.1

Example usage:

```puppet
file_line { 'bashrc_proxy':
  ensure => present,
  path   => '/etc/bashrc',
  line   => 'export HTTP_PROXY=http://proxy:8080',
  match  => '^export HTTP_PROXY=',
}
```

For help, remeber:

```console
$ puppet describe file_line
```

## ini_setting

* Manages settings in an ini file (or similar)
* Custom resource types and function
* Included in puppetlabs/inifile modules

```puppet
ini_setting {'Java Home':
  ensure            => present,
  section           => 'Date',
  key_val_separator => '=',
  path              => '/etc/php/8.2/apache2/php.ini',
  setting           => 'date.timezone',
  value             => 'US/Pacific',
}
```

Results in:

```ini
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = US/Pacific
```

## augeas

* Treats files as trees of values
* Modify the tree as you like and write them back
* Uses augtool
* File parsing based on lenses
* Core module, extend in several augeas* modules

```console
$ cat /etc/yum.conf
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
...

$ augtool
augtool> ls /files/etc/yum.conf/main
cachedir = /var/cache/yum/$basearch/$releasever
keepcache = 0
...
```

Example usage:

```puppet
augeas { 'yum_config':
  context => '/files/etc/yum.conf/main',
  changes => [
    'set keepcache 1',
  ],
}
```

Help:

```puppet
$ puppet describe augeas
```

## concat

* Builds a complete file from fragments
* Defined resource type
* Included in puppetlabs/concat module

```puppet
$zone_file = '/var/named/training.zone'

concat{ $zone_file:
  owner => 'root',
  group => 'root',
  mode  => '0644',
}

concat::fragment{ 'managed by puppet zone file':
  target => $zone_file,
  source => 'puppet:///modules/named/managed.txt',
  order  => '05',
}

concat::fragment{ 'training_zone_header':
  target  => $zone_file,
  content => epp('dns/zone_header.epp'),
  order   => '20',
}
```
