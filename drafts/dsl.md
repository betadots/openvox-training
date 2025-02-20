# Puppet DSL

TODO daily goal

## Resources

## Functions

## Classes

* Separation of configuration and data
* Automatic Lookup of parameters was introduced in Puppet 3
* Improvements in Puppet 4.9
* Default is
  * Hiera, a hierachical lookup
  * One global configuration

## Variables

**Syntax**:

```puppet
    $variable = 'value'

    $httpd_confdir = '/etc/httpd'
```

* Can be used in expressions, functions and resource attributes
* Some naming conventions enforced, some keywords reserved
* Depending on scope
* Different data types
* Actually are constants!

### Scope

add pics of sopce

### Accessing Variables

**Shortname accesses Local Scope**:
```puppet
    $httpd_confdir
```

**Qualified name accesses scope defined by namespace**:
Top Scope (also contains facts) and Node Scope:
```puppet
    $::kernel
```
Out-of-Scope:
```puppet
    $apache:mod::status::extended_status
```

**Interpolation**:

```puppet
    $httpd_confdir = "${conf_dir}/httpd}"
```

### Parameterized Classes

* Classes can take parameters to change their behaviour
* The parameters should have a defined data type
* Also with parameters a class is still a singleton

```puppet
    class apache (
      Enum['running','stopped'] $ensure        = 'running',
      Boolean                   $enable        = true,
      Boolean                   $default_vhost = false,
      Hash[String, String]      $vhosts        = {},
    ) {
      ...
    }
```

**Declaring parameterized class**:

* Include function takes a class with all its defaults
* You can declare a class like every resource with parameters

```puppet
    class { 'apache':
      ensure        => running,
      default_vhost => true,
    }
```

## Modules

### Parameter Lookup

### Puppet Forge

http://forge.puppet.com

* Community platform for modules
  * Thousands of modules by many different authors
  * Searchable
  * Supported, Partner supported and Approved Modules
  * Number of Downloads and Scoring system
* Command Line Interface puppet module
  * Search
  * Install
  * List installed modules

## Templates

## Defined Resources
