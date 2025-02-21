# Puppet DSL

TODO daily goal

## Resources

* Puppet includes in its installation some core resource types, for example:
    * user
    * group
    * file
    * package
    * service
* Additional ones can be provided by extenntions, with so-called modules, for example:
    * file_line
    * ini_setting
    * mysql_database
    * vcsrepo

A list can be generated with the **puppet describe** command!

### Syntax and Style

Puppet is as much about syntax as about style:

```puppet
    type { 'title':
      ensure          => present,
      attribute       => 'value',
      other_attribute => 'anaothe rvalue',
    }
    
    file { ['/tmp/test', '/root/file.txt']:
      ensure  => file,
      owner   => 'root',
      group   => 'root',
      mode    => '0644',
      content => 'some content!',
    }
```

### Resource Defaults and Blocks

* Puppet allows you to declare resource defaults

```puppet
    Type {
      attribute => 'value',
    }

    File {
      owner => 'root',
      group => 'root',
      mode  => '0644',
    }
```

For directories Puppet promotes a mode default of 0644 to 0755.

* With Puppet 4 a block based version was introduced

```puppet
    file {
      default:
        mode   => '0600',
        owner  => 'root',
        group  => 'root',
        ensure => file,
      ;
      '/etc/ssh_host_key':
      ;
      '/etc/ssh_host_dsa_key.pub':
        mode => '0644',
      ;
    }
```   

### References

Allows to reference other managed resources

```puppet
    Type['title']

    Service['sshd']
```

Can be used to override parameters:

```puppet
    file { ['/tmp/test', '/root/file.txt']:
      ensure  => file,
      owner   => 'root',
      group   => 'root',
      mode    => '0644',
      content => 'some content!',
    }

    File['/root/file.txt'] {
      mode    => '0640',
      content => 'Some different content.',
    }
```

**Note**: Using this sparingly to avoid confusion when everything is declared.

More commonly used to declare a sequence between different resources.

### Ordering

* Default order depends on order of declaration and implicit dependencies
* Ordering can be changed to:
    * Hash based ordering which was default in older versions
    * Random ordering to ensure all necessary dependencies are declared
* Dependencies can be set explicitly

#### Implicit

add some pics

#### Explecit

* 4 types of relationship
* Defined by metaparameter of all resources
* Simple Ordering:
  * require - referenced resource will be applied first
  * before - apply this resource before the reference
* Refresh Events:
  * subscribe - if reference is changed refresh this resource
  * notify - if this resource is changed refresh the reference

add some pics

### Metaparameter

## Functions

## Classes

Classes offering a way of grouping resources together and assigning data.

### Defining vs. Declaring

**Define**:
To specify the contents and behavior of a class. Defining a class doesn't automatically include it in a configuration; it simply makes it available to be declared.

```puppet
    class base {
      file { '/etc/motd':
        ensure => file,
        conatnet => 'Hello my friend!',
      }
      ...
    }
```

**Declare**:
To direct Puppet to include or instantiate a given class. To declare classes, use the include function. This tells Puppet to evaluate the class and manage all the resources declared within it.

* with include Function

```puppet
    include apache
```

* like any other resources

```puppet
    class { 'apache': }
```

#### Idempotency of include

* The function include is idempotent. That means you can use the include of the same class several times in your code.

```puppet
    include apache
    include apache
```

* The class is declared just once, the first time it was used.

**Notice**: A mix between the declaration with include and class doesn't work and passes to a duplicate declaration error.

### Namespaces

Namespaces are segments that identify the directory and file structure for classes:

| File path                        | Namespace             |
|----------------------------------|-----------------------|
| manifests/base.pp                | base                  |
| manifests/base/ssh.pp            | base::ssh             |
| manifests/linux/debian/apache.pp | linux::debian::apache |

Classes must correspond to the namespaces in the name.

```puppet
    class base::ssh {
      ...
    }

    include base::base
```

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

### Data Types

* Simple Data Types
  * Strings
  * Numbers
  * Booleans
  * Arrays
  * Hashes
  * Regular Expressions
  * Undef
  * Resource References
  * Default

* Abstract Data Types
  * Flexible Data Types
  * Parent Types

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

### Facts

also add trusted facts

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
