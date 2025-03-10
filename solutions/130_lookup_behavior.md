# Sample Solution

## Hiera Lookup Behavior

* Add a new layer between nodes and common.yaml named `operatingsystem data (yaml version)` based on `facts.os.family`
* Add a new parameter `packages` to `profile::base` to install some packages
* Use a node specific, os specific list of packages and a default one in `common.yaml`
* Do a merge behavior change and explain your decision
