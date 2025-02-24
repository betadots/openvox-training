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

A list can be generated with the `puppet describe` command!

### Syntax and Style

Puppet is as much about syntax as about style

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

### Exec Resource

* Resource to execute commands
* Avoid if possible
* If required use an attribute for idempotence
  * creates
  * onlyif / unless
  * refreshonly
* Use full path or provide path as attribute

```puppet
   exec { 'command':
     path        => '/usr/sbin/:/sbin/',
     refreshonly => true,
     timeout     => 60,
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

For directories Puppet promotes a mode default of `0644` to `0755`.

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

Can be used to override parameters

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

![Implicit dependencies](implicit_dependencies.png)

#### Implicit dependencies of `user`

![Implicit dependencies of user](implicit_dependencies_user.png)

#### Implicit dependencies of `exec`

![Implicit dependencies of user](implicit_dependencies_exec.png)

#### Explicit

* 4 types of relationship
* Defined by metaparameter of all resources
* Simple Ordering:
  * require - referenced resource will be applied first
  * before - apply this resource before the reference

![](explicit_dependencies_ordering.png)

* Refresh Events:
  * subscribe - if reference is changed refresh this resource
  * notify - if this resource is changed refresh the reference

![](explicit_dependencies_refresh.png)

#### Dependency Chains

* Alternative syntax for explicit dependency
* Works great with references
* Simple Ordering:
  * before: Package['openssh-server'] -> File['sshd_config']
  * require: Package['openssh-server'] <- File['sshd_config']
* Refresh Events:
  * notify: File['sshd_config'] ~> Service['sshd']
  * subscribe: File['sshd_config'] <~ Service['sshd']
* For readability use before (->) and notify (~>)!

### Metaparameter

TODO

## Functions

TODO

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

* The function include is idempotent. That means you can use the `include` of the same class several times in your code.

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

### Refernces of Classes

The reference to a class consists of the keyword `Class` and its namespace:

```puppet
    Class['class namespace']

    Class['base::ssh']
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

```puppet
    $num  = 4711
    $bool = true
    $arr  = ['item1', 4711]
    $hsh  = { key1 => 'value1', key2 => 4711 } 
```

* Abstract Data Types
  * Flexible Data Types
  * Parent Types

### Scope

![Variable scope](scope-euler-diagram.png)

### Accessing Variables

Shortname accesses Local Scope
```puppet
    $httpd_confdir
```

Qualified name accesses scope defined by namespace

* Top Scope (also contains facts) and Node Scope:
```puppet
    $::kernel
```
* Out-of-Scope:
```puppet
    $apache:mod::status::extended_status
```

Interpolation

```puppet
    $httpd_confdir = "${conf_dir}/httpd}"
```

### Facts

* All determined facts of a node are available as top scope variables during the compilation of the catalog:

```puppet
    $facts['kernel']
```

* All trusted information such as certname or all other properties from the certifikate are also avaiable':

```puppet
    $trusted['certname']
```

**Notice**: $trusted is empty during a `puppet apply`.

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

## Defined Resources

TODO

## Modules

TODO

### Parameter Lookup

TODO

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

TODO

## Defined Resources

TODO
