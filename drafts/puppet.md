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

## Impementations

### Puppet Enterprise

* Packaging for additional platforms including additional components
* Puppet Enterprise Console as Webinterface
* Additional Components
  * Puppet Node Manager - group nodes on facts
  * Puppet Code Manager - combines r10k, Jenkins and beaker for testing and deploying code
  * Puppet Configuration Manager - helps troubleshoot dependencies
  * Task Management - combines bolt with Puppet Enterprise Console and Orchestration API
  * File Sync - keep your environment synchronized over all your Puppet server
* Supported Modules
* Automated Provisioning for some platforms
* Vendor support and service

### Puppet FOSS

TODO

### OpenVox

TODO

## Factor

* Facter returns key value pairs named facts
* It is used by Puppet to gather information about the node
* You can also run it on command line to list facts

```bash
   aio_agent_version => 7.18.0
   augeas => {
     version => "1.12.0"
   }
   disks => {
     vda => {
       size => "10.00 GiB",
       size_bytes => 10737418240,
       vendor => "0x1af4"
     }
   }
   ...
   system_uptime => {
     days => 0,
     hours => 2,
     seconds => 8378,
     uptime => "2:19 hours"
   }
   timezone => UTC
   virtual => kvm
```
   
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

### Agent as Service

* Agent runs as a service
  * Interval is configureable
  * Splay can be added for spread agent runs
  * Default interval is 30 minutes
* Cronjob can be used as alternative
* On demand is also possible

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

### Environments

* Environments allow to serve different code to different stages
* Directory enviroments
  * Path set on the server to serve an environment per directory
  * Directory contains a main manifest and modules
* Assignment
  * Node configuration requests an environment
  * Server can override it
* Default environment: production

```bash
$ ls -l /etc/puppetlabs/code/environments/production
data  environment.conf  hiera.yaml  manifests  modules
```
### Classification

#### Manifest site.pp

* Nodes are another resource type
* Only objects not declared in modules
* Matching
  * Exact match
  * Regex
  * Fuzzy name matching via Wildcards
  * Default

```puppet
# environments/production/manifests/site.pp

node 'www.example.com' {
  include apache
}

node /.example.com$/ {
  include base
}

node 'www.example.*' {
  include nginx
}

node default {
  notify { 'Node not configured': }
}
```

#### External Node Classifier

* External Node Classifier (ENC) is an alternative to site.pp
* Script providing a node declaration in yaml format
  * Simple script logic
  * Query configuration management database
  * Communication with a web frontend
   
#### Deploying Code to Puppet Server

* Ensure code is valid
  * Syntax validation
  * Style guide conformity
* Use environments for staging
  * No direct deployment in production
* Use version control system
  * History of changes
  * Enables Collaboration
  * Hooks enable automatic validation and deployment

### Hiera

* Hierarchy of lookups is configurable
  * Hierarchy level can be fix or use variables
  * Hiera 4 was never released as stable, replaced in Puppet 4.9
  * Environment and module configuration uses Hiera 5
* Different backends are avaiable
  * YAML/JSON - default
  * EYAML - YAML with encrypted fields
  * MySQL/PostgreSQL - Database lookup
  * LDAP and more

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
  - name: "Other YAML hierarchy levels"
    paths:
      - "common.yaml"
```

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
