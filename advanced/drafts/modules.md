# Module Design

## Good Module Design

* Only manage their own resources
  * Don't manage logrotating in your apache module
  * What happens if your Drupal module manages Apache and PHP?
* Be granular, portable and reusable
* Avoid exposing implementation details

## Architecting Modules

Motivation:

* Huge applications to manage, such as Apachei, Icinga or Tomcat
* A lot of code lines
* Complex dependency structures
* Some examples:
  * puppetlabs/apache
  * puppet/icinga2

### Main Class

* Split off your class in subclasses
* Define dependencies between these classes
* Dependencies between resources will be defined only between resources contain to the same subclass

```puppet
# @summary
#   Short description of what application the module manages
#
# @param package_name
#   description
#
# @param ...
#
class custom (
  String[1]                 $package_name,
  String[1]                 $service_name,
  Enum['running','stopped'] $ensure = running,
  Boolean                   $enable = true,
  String                    $param1 = $custom::params::param1,
) {
  contain custom::install, custom::config, custom::service

  Class['custom::install']
    -> Class['custom::config']
    ~> Class['custom::service']
}
```

### Platform Dependent Parameters

* Store values in module data (Hiera)

```yaml
---
version: 5

defaults:
  datadir: data
  data_hash: yaml_data

hierarchy:
  - name: "osfamily/major release"
    paths:
      - "os/%{facts.os.name}/%{facts.os.release.major}.yaml"
      - "os/%{facts.os.family}/%{facts.os.release.major}.yaml"
  - name: "osfamily"
    paths:
      - "os/%{facts.os.name}.yaml"
      - "os/%{facts.os.family}.yaml"
  - name: 'common'
    path: 'common.yaml'
```

### Sub Class 'install'

* Class to do all stuff of the installation
* Contains all resources that don't enforce a service restart if they changed

```puppet
# @summary
#   Short description
#
# @api private
#
class custom::install (
) {
  assert_private()

  $package = $custom::package_name

  package { $package:
    ensure => installed,
  }
}
```

### Sub Class 'config'

* Contains all resources that do enforce a service restart if they changed

```puppet
# @summary
#   Short description
#
# @api private
#
class custom::config (
) {
  assert_private()

  $param1 = $custom::param1

  file { $custom_config:
    ensure => file,
    content => epp('custom/custom.conf.epp'),
  }
}
```

### Sub Class 'service'

* Finally the service subclass handles all resources to manage the service respectively services

```puppet
# @summary
#   Short description
#
# @api private
#
class custom::service {
  assert_private()

  $name    = $custom::service_name
  $ensure  = $custom::ensure
  $enable  = $custom::enable

  service { $service:
    ensure => $ensure,
    enable => $enable,
  }
}
```

## Puppet development Kit

PDK or puppet development kit is the recommended tool for designing a module.

* Create new module (your inactive input is stored in metadata.json)

  ```console
  pdk new module <MODULE_NAME>`
  ```

  ```text
  |-- appveyor.yml
  |-- CHANGELOG.md
  |-- data
  |   `-- common.yaml
  |-- examples
  |-- files
  |   `-- httpd.conf
  |-- Gemfile
  |-- Gemfile.lock
  |-- hiera.yaml
  |-- manifests
  |-- metadata.json
  |-- Rakefile
  |-- README.md
  |-- spec
  |   |-- default_facts.yml
  |   `-- spec_helper.rb
  |-- tasks
  `-- templates
      `-- vhost.conf.erb
  ```

* Add new class

  ```console
  cd <MODULE_NAME>
  pdk new class <CLASS_NAME>
  ```

* Convert an existing module

  ```console
  cd <MODULE_NAME>
  pdk convert [options]
  ```
* Convert an existing module

  ```console
  cd <MODULE_NAME>
  pdk convert [options]
  ```

## Module Purpose

* How might others wish to use your module?
* Rethink assumptions
* Be on the side of customizability, e.g. for unsupported platforms and versions
* But don't forget sensible defaults
* Well thought out parameters reduce likelihood of module forks
* Hosting modules on GitHub encourages community contributions

## Generalizable Modules

* Avoid hardcoding data as much as possible
* Provide reasonable platform default parameters
* Calculate these in your params class
* Expose parameters for customization and overriding defaults
* Parameters form your public API
* Name well and don't change unless necessary
* Provide many parameters to cover all customization needs
* Provide defaults to everything possible
* Think about future maintainability

## Metadata File

* Names and dependencies must be namespaced with author name
* Dependencies can be specified on:
  * Specific versions (1.2.3)
  * Logical greater/less than (>1.2.3, <=1.2.3, etc)
  * Ranges of versions (>=1.0.0 <2.0.0)
  * Semantic major versions (1.x) or minor versions (1.2.x)

```yaml
{
  "name": "puppet-php",
  "version": "11.0.0",
  "author": "Vox Pupuli",
  "summary": "Generic PHP module that supports many platforms",
  "license": "MIT",
  "source": "https://github.com/voxpupuli/puppet-php",
  "project_page": "https://github.com/voxpupuli/puppet-php",
  "issues_url": "https://github.com/voxpupuli/puppet-php/issues",
  "description": "Puppet module that aims to manage PHP and extensions in a generic way on many platforms with sane defaults and easy configuration",
  "dependencies": [
    {
      "name": "puppetlabs/stdlib",
      "version_requirement": ">= 9.0.0 < 10.0.0"
    },
    ...
  ],
  "requirements": [
    {
      "name": "openvox",
      "version_requirement": ">= 7.0.0 < 9.0.0"
    }
  ],
  "operatingsystem_support": [
    {
      "operatingsystem": "Debian",
      "operatingsystemrelease": [
        "10",
        "11",
        "12"
      ]
    },
    {
      "operatingsystem": "OpenSUSE"
    }
  ]
}
```

## Versioning your Module

**MAJOR.MINOR.PATCH**

* Major version:
  * Increment on API incompatible changes
* Minor version:
  * Increment on backwards compatible updates
* Patch version:
  * Increment on bug fixes that don't change behaviour
* Provides trust in module updates
* Proper versioning allows users to pick and choose as needed
* Specify dependencies on specific module versions

