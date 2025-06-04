# Sample Solution

## Variables

* Assign a variable with any value outside your node decrations in your `manifests/site.pp`

```puppet
    $var = 'Puppet'

    node default {
      notify { 'Hello welcome to the world of Puppet': }
    }
```

* Use this variable as title in your `notify` resource

```puppet
    $var = 'Puppet'

    node default {
      notify { "Hello welcome to the world of ${var}!": }
    }
```

* Run `puppet apply` on your `manifests/site.pp`

```bash
    $ puppet apply manifests/site.pp
    Notice: Compiled catalog for <certname of your workstation> in environment production in 0.08 seconds
    Notice: Hello welcome to the world of Puppet!
    Notice: /Stage[main]/Main/Node[default]/Notify[Hello welcome to the world of Puppet!]/message: defined 'message' as 'Hello welcome to the world of Puppet!'
    Notice: Applied catalog in 0.03 seconds
```

* Now add the same variable inside your node declaration and assign a differnt value

```puppet
    $var = 'Puppet'

    node default {
      $var = "OpenVox"

      notify { "Hello welcome to the world of ${var}!": }
    }
```

* Rerun `puppet apply` and compare the results

```bash
    $ puppet apply manifests/site.pp
    Notice: Compiled catalog for <certname of your workstation> in environment production in 0.07 seconds
    Notice: Hello welcome to the world of OpenVox!
    Notice: /Stage[main]/Main/Node[default]/Notify[Hello welcome to the world of OpenVox!]/message: defined 'message' as 'Hello welcome to the world of OpenVox!'
    Notice: Applied catalog in 0.03 seconds
```
