# OpenVox Training

* Puppet Environment
  * Trainings Setup
    * Puppet Server
    * Student workstation
    * GitLab
    * Elastic
    * Graphite/InfluxDB
  * Server/Agent
    * Workflow
    * Puppet Enterprise, Puppet FOSS vs. OpenVox
  * Facter
  * Puppet Agent
    * Example managing user
      * LAB puppet describe command
    * RAL
    * Provider
    * LAB puppet resource command
  * Puppet Server
    * Classification site.pp
    * ENC
    * Environments
  * PuppetDB
  * Puppet Apply
    * /root/puppetcode -> /etc/puppetlabs/code/envrionments/productions
    * DEMO site.pp
  * Hiera
* Puppet DSL Basics
  * Demo daily goal
    * motd including hiera, facts and templates (file, replace the module)
  * Resources (core and module types)
    * LAB file for motd
    * ? Ordering
      * implicit incl. eaxample user group etc.
      * explicit incl. example package, config, service
  * Classes
    * Syntax
    * Defining vs. Declaring
    * Idempotency (of include)   
  * Variables
    * Facts and trusted variables
    * LAB motd file resource
    * As class parameter
  * Modules
    * Why (auto loading, file serving, custom extensions, easy to share)
    * Forge and Voxpupuli
    * puppet module vs. Puppetfile and r10k (dependencies)
    * LAB motd
  * Templates
    * ERB vs. EPP
    * LAB motd
  * Defined resources
* Git for beginners (best in the morning of the second day)
  * Standard commands and Workflow
  * Branches and r10k
  * Transfer environment directory to git
  * Control repository
 
