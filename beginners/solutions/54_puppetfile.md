# Sample Solution

## Puppetfile

* Create a `Puppetfile` in your working directory

```bash
    $ cd ~/puppetcode
    $ vim Puppetfile
```

* Add module `puppetlabs/motd` and its dependencies to your `Puppetfile`

```yaml
    mod 'puppetlabs/motd'
    mod 'puppetlabs/stdlib'
    mod 'puppetlabs/registry'
```

* Remove all directores from directory `./modules/` via `rm -rf modules/*`

```bash
    $ rm -rf ./modules/*
```
 
* Now execute `/opt/puppetlabs/bolt/bin/r10k puppetfile install -v` to reinstall the desired modules

```bash
    $ /opt/puppetlabs/bolt/bin/r10k puppetfile install -v
    INFO	 -> Using Puppetfile '/root/puppetcode/Puppetfile'
    INFO	 -> Deploying module to /root/puppetcode/modules/motd
    INFO	 -> Deploying module to /root/puppetcode/modules/stdlib
    INFO	 -> Deploying module to /root/puppetcode/modules/registry
```

* Check the content of the `./modules/` directory

```bash
    $ ls modules/
    motd  registry  stdlib
```
