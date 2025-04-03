# Sample Solution

## Facter

* Take a look at the issue of facter at your leisure

```bash
    $ puppet facts |less
```

* Determine specific individual values

```bash
    $ puppet facts os.family
    {
      "os.family": "RedHat"
    }

    $ puppet facts os.release
    {
      full: "9.5",
      major: "9",
      minor: "5"
    }

    $ puppet facts networking.ip
    {
      "networking.ip": "192.168.56.142"
    }
```
