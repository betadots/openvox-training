# Practices with Profile Classes

## Grafana

Grafana is an open source application for the graphical representation of data from various data sources such as InfluxDB, MySQL, PostgreSQL, Prometheus and Graphite.

### Setup a Grafana with PostgreSQL as Database Backend

* Install module `puppetlabs-postgresql` and all its dependencies, pin your modules to the last version
* Start writing a class `profile::grafana` and manage a PostgreSQL database `grafana`
  * with owner and password
  * implement password as parameter `db_password`
* Write a smoke test
  * with an include of `profile::base`
  * and an explicit declaration of `profile::grafana` with set `db_password`
  * run the test locally

### Connect an upstream Nginx as a Reverse Proxy

* Check if module `puppet-nginx` and its dependencies are already installed
* Change the listener from Grafana to `localhost`
  * config option is `http_addr`
* Add class `nginx` to proxy to grafana on `localhost:3000`

### Secure your hosts with `nftables` local firewalls

* Install module `puppet-nftables` and all its dependencies
* Setup a firewall on all hosts
  * allow all outgoing traffic and incoming ssh and icmp
  * additinal only for Grafana hosts allow incomming http

A sample solution can be found [here](./solutions/150_profiles.md).
