# Sample Solution

## Classification

* Write a site.pp for your working node

```bash
    $ cd ~/puppetcode
    $ vim manifests/site.pp
```

* Use resource notify to display any text you want

```puppet
    node default {
      notify { 'Hello welcome to the world of Puppet!': }
    } 
```

The notices are in the same format as a normal report.

```bash
    $ puppet apply manifests/site.pp
    Notice: Compiled catalog for <certname of your workstation> in environment production in 0.08 seconds
    Notice: Hello welcome to the world of Puppet!
    Notice: /Stage[main]/Main/Node[default]/Notify[Hello welcome to the world of Puppet!]/message: defined 'message' as 'Hello welcome to the world of Puppet!'
    Notice: Applied catalog in 0.03 seconds
```
