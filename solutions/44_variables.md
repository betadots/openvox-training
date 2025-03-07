# Sample Solution

## Variable Namespace

* Replace again the title of your `notify` and use `$::motd_file`

```puppet
    $var = 'Puppet'

    node default {
      $var = "OpenVox"

      notify { "Hello welcome to the world of ${::var}!": }
    }
```

* Rerun `puppet apply`

```bash
    $ puppet apply manifests/site.pp
    Notice: Compiled catalog for <certname of your workstation> in environment production in 0.08 seconds
    Notice: Hello welcome to the world of Puppet!
    Notice: /Stage[main]/Main/Node[default]/Notify[Hello welcome to the world of Puppet!]/message: defined 'message' as 'Hello welcome to the world of Puppet!'
    Notice: Applied catalog in 0.03 seconds
```
