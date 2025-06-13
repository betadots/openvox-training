# Facts

Accessing via top scope varibale `$facts`:

```puppet
notify { "Your operatingsystem is: ${facts['os']['name'}": }
```

## Custom Facts

Developing facts in modules:

* Custom facts have to be executable scripts
* By convention, the file name has to match the name of the fact
* Multiple facts can be distributed with a single module

```console
$ tree {MODULE PATH}/{MODULE NAME}
|-- lib
|   |-- facter
|   |   `-- {MODULE NAME}_{FACT NAME}.rb
```

Test facts locally by setting FACTERLIB or RUBYLIB environment variables:

```console
$ export RUBYLIB={MODULE PATH}/{MODULE NAME}/lib
$ facter {MODULE NAME}_{FACT NAME}
```

* The fact is set to the string value returned from setcode block
* Since facter 2.0 array and hash also permitted as return value

```ruby
Facter.add('role') do
  setcode do
    role = Facter::Core::Execution.exec('cat /etc/role')
    role.gsub(/<.*?>/m, "")
  end
end
```

If on a string no other manipulation is needed you can pass the string to setcode:

```ruby
Facter.add('role') do
  setcode 'cat /etc/role'
end
```

## External Facts

* Could be YAML, JSON, text files or executable scripts
* Also distributed via pluginsync from {MODULE NAME}/facts.d

```console
$ cat {MODULE NAME}/facts.d/datacenter.yaml
---
location: berlin
cluster: webserver

$ cat {MODULE NAME}/facts.d/datacenter.txt
location=berlin
cluster=webserver

$ cat {MODULE NAME}/facts.d/datacenter.py
#!/usr/bin/env python
data = {"location" : "berlin", "cluster" : "webserver" }

for k in data:
print "%s=%s" % (k,data[k])

$ facter location
berlin
```
