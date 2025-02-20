# Puppet Environment

## Trainings Setup

TODO

## Concept

* Puppet is about abstraction
* Domain Specific Language allows for an abstract description of resources
  * Infrastructure as Code
  * Executable Documentation
* Define the desired state and not how to get there
  * Idempotence

## Workflow

pics communication cycle

## Facter

TODO

## Puppet Agent

* Agent runs as service on all managed nodes
* Requests periodically the desired configuration state from the Puppet server
* Sends information about itself to determine the configuration state (facts)
* Enforces the retrieved configuration state (catalog)
* Report enforcement back to server
* Supported platforms: 
  * most Linux distributions
  * Windows
  * many Unix distributions
  * Mac OS X
  * some Network devices

### Example Managing User

* Managing a user without Puppet means managing:
  * Existence of user and attributes
  * Group membership
  * Home directory
* Requires knowledge of:
  * Installed tools (useradd or adduser)
  * Options of those tools (-g or -G)
  * Creating, changing, deleting requires different tools (useradd, usermod, userdel)
* Challenges for automation:
  * Scripting
  * Error handling and logging
  * Multiple platforms

#### Example Managing User

* The same task with Puppet
  * Describe the user
  * Enforce with Puppet

```puppet
user { 'icinga':
  ensure     => present,
  gid        => 'icinga',
  groups     => ['icingacmd'],
  home       => '/var/spool/icinga',
  shell      => '/sbin/nologin',
  managehome => true,
}
```
### Resource Abstraction Layer

* Puppet uses a Resource Abstraction Layer
  * Groups similar resources into resource types
  * Resource types have multiple providers
* Resources can be declared always in the same fashion
* A resource will be managed by the best matching provider on the platform

TODO: pics of RAL

#### Providers

* Availability of a provider is determinated by testing for availability of required binaries
* Can be marked as default for specific facts (typically operatingsystem)
* Can support all or only a subset of features a resource type requires
* Example for resource package
  * **Providers**: *aix appdmg apple apt aptitude aptrpm blastwave dnf dpkg fink freebsd gem hpux macports nim openbsd opkg pacman pip3 pip pkg pkgdmg pkgin pkgng pkgutil portage ports portupgrade puppet_gem rpm rug sun sunfreeware up2date urpmi windows yum zypper*
  * **Features**: *holdable install_options installable package_settings purgeable reinstallable uninstall_options uninstallable upgradeable versionable virtual_packages*

#### Puppet Resource Command

* Puppet provides a command to directly interact with the Resource Abstraction Layer
* Querying all or one resource of a type returns Puppet code representation of current state
* Setting attributes will change state using available provider

```puppet
$ puppet resource package vim-enhanced
package { 'vim-enhanced':
  ensure => 'purged',
}
```

```puppet
$ puppet resource package vim-enhanced ensure=present
Notice: /Package[vim-enhanced]/ensure: created
package { 'vim-enhanced':
  ensure => '7.4.160-1.el7',
}
```

## Puppet Server

* The server runs on a central machine
* Includes a Certificate authority
* Authenticates agent connections
* Serves a compiled catalog with the desired configuration state on agent request
* Acts as file server
* Forwards reports to report handlers
* Puppet server runs jRuby (executed in a JVM)
* Supported on most Linux distributions

### Classification

TODO

### Environments

TODO

### Hiera

TODO

## PuppetDB

* Data Store:
  * Facts(last run)
  * Catalog (last run)
  * Reports (optional, timefram configurable)
* Enables:
  * Features like exported resources
  * API to query the data
* Uses:
  * Clojure (dialect of Lisp runs in JVM)
  * PostgreSQL

## Puppet Apply

* The Puppet apply command combines server and agent functionality
  * Takes a file containing Puppet code
  * Gathers information using facter
  * Compiles a catalog
  * Enforces the configuration
  * Can also run in simulation mode
* Useful for development and local testing or server-less setups
* Requires root privileges
