# Sample Solution

## Classification

* Add new layer for role `trusted.extensions.pp_role` after node specific and before os layer

```bash
    $ cd ~/puppetcode
    $ vim hiera.yaml
```

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
      - name: "role data (yaml version)"
        path: "roles/%{trusted.extensions.pp_role}.yaml"
      - name: "operatingsystem data (yaml version)"
        path: "os/%{facts.os.family}.yaml"
      - name: "Other YAML hierarchy levels"
        paths:
          - "common.yaml"
```

* Create a `webserver.yaml` for the role with a key `classes` (array) and item `profile::base`

```bash
    $ mkdir data/roles
    $ vim data/roles/webserver.yaml
```

```yaml
    ---
    classes:
      - profile::base
      - profile::myapp
```

* Set the merge behavior to `first` 

```bash
    $ vim data/common.yaml
```

```yaml
    ---
    classes:
      - profile::base

    ...

    lookup_options:
      profile::base::packages:
        merge: unique
```

* Do the required changes in your `site.pp` lookup and declare classes

```bash
    $ vim manifests/site.pp
```

```puppet
node default {
  lookup('classes', Array[String], 'first').include
}
```

* Discuss the merge behavior `first` vs. `unique` for this use case
* Also discuss how to use `pp_environment` for a staging of your servers and roles
