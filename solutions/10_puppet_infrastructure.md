# Sample Solution

## Facter

* Take a look at the issue of facter at your leisure

```bash
    $ facter |less
```

* Determine specific individual values

```bash
    $ facter os.family
    RedHat

    $ facter os.release
    {
      full => "9.5",
      major => "9",
      minor => "5"
    }

    $ facter networking.ip
    192.168.56.153
```
