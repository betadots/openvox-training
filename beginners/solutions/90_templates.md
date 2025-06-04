# Sample Solution

## Templates

* Write an EPP template without parameterization for `motd` in your `profile` module

```bash
    $ cd ~/puppetcode
    $ mkdir site-modules/profile/templates
    $ vim site-modules/profile/templates/motd.epp
```

* Use some facts in your message

```puppet
    Welcome on <%= $facts['networking']['hostname'] %>!
    Your system is up since <%= $facts['system_uptime']['hours'] %> hours.
```

* Reconfigure your hiera data (e.g. `data/common.yaml`) to use the template

```yaml
    ---
    motd::template: 'profile/motd.epp'
    motd::content: 'Welcome to the world of OpenVox!'
```

* Run `puppet apply`

```bash
    $ puppet apply manifests/site.pp --show_diff
    Warning: Scope(Class[Motd]): Both $template and $content parameters passed to motd, ignoring content
    Notice: Compiled catalog for <certname of your student workstation> in environment production in 0.33 seconds
    Notice: /Stage[main]/Motd/File[/etc/motd]/content:
    --- /etc/motd	2025-03-09 11:26:16.119011277 +0000
    +++ /tmp/puppet-file20250309-13963-7shvt4	2025-03-09 13:02:09.501769356 +0000
    @@ -1 +1,2 @@
    -Welcome to my world!
    \ No newline at end of file
    +Welcome on <hostname of your student workstation>!
    +Your system is up since 53 hours.
    
    Notice: /Stage[main]/Motd/File[/etc/motd]/content: content changed '{sha256}c019565d606f06c47718174fc5cad74a959d8b2b0d8e15c4da52a6374fa8ecf6' to '{sha256}1beca554bb2aa6c20211f2cd7148d65fddb629d95b7356ec1c5a65885fcde5ed'
    Notice: Applied catalog in 0.06 seconds
```

### Bonus

* Extend then template to output a variable from `profile::base` scope

```puppet
    class profile::base (
      String $content = 'Have a nice day!',
    ) {
      include motd
    }
```

```puppet
    Welcome on <%= $facts['networking']['hostname'] %>!
    Your system is up since <%= $facts['system_uptime']['hours'] %> hours.
    <%= $profile::base::content %>
```
