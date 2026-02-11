# Basic Course refresh

## OpenFact/Facter

```shell
sudo facter -p
sudo puppet facts
```

### Using facts in Puppet Code

```puppet
# 1. Modern: $facts Hash
$facts['networking']['fqdn']

# 2. Legacy top level (obsolete)
$::networking['fqdn']

# 3. Deprecated (legacy facts) - no more valid as of OpenVox/Puppet 8
$::fqdn
$::osfamily
$::operatingsystemmajrelease
```

### Creating Facts

```text
```

### External Facts

## Node classification

### Explizit NC

#### Rolen

```puppet
node <certname|fqdn> {
  # Puppet DSL
  # Welche Klasse soll vom Compiler in den Kataliog aufgenommen werden?
  include role::app::geo::backend
}
```

#### Profiles

```puppet
node <certname|fqdn> {
  # Grundlegende Config (Monitoring, Backup, Security, User, ...)
  include profile::base
  # Grundlegende Config für Geo (Repos, Keys, Certs, ...)
  include profile::app::geo
  # Spezifische Applikation
  include profile::app::geo::backend
}
```

### Hiera basierte Node Klassifizierung

## Hiera

Trennung von Code und Daten

Hiera ist ein Key-Value Store

## hiera.yaml Config

- datadir - wo liegen die Daten Strukturen
- hierarchies
  - path(s) - Pfade um identifzieren von Unterschieden

Was sind Unterschiede?

- Server im RZ1 bekommen andere DNS Server als Server im RZ2
- Server in der Netzzone secure bekommen andere User als Server in Netzzone basic
- Server mit App 1 in Prod bekommen andere DB Server als Server mit App 1 in Test
- Server mit App 1 werden anders konfigururiert als Server mit App 2
- Der eine Server mit App 1 in Prod ist anders als die anderen

Die Hierarchien müssen von SPezifisch (Node) zu Allgemein (common) aufgebaut werden:

- Node
- App - Stage
- App
- Netzzone
- Location
- Allgemeine

Hierarchien werden über Facts abstrahiert:

- nodes/%{facts.networking.fqdn}.yaml
- fvf/%{facts.app}-%{facts.app_stage}.yaml
- fvf/%{facts.app}.yaml
- zone/%{facts.net_zone}.yaml
- datacenter/%{facts.networking.domain}.yaml
- common

### Daten Lookup mit Hiera

Hiera analysiert die Ebenen von oben nach unten.
Erstes Vorfinden des gesuchten Keys wird zurückgeliefert.

Beispiel:

Node: lx31geo1p.muc.domain.tld

Application: geodata (geo)
Stage: Production (p)
Location: Munich (muc)

- nodes/lx31geo1p.muc.domain.tld.yaml
- app/geo-production.yaml
- app/geo.yaml
- datacenter/munich.yaml
- common.yaml

In app/geo-production.yaml

```yaml
---
geo::database_server: 'ux22db2p.muc.domain.tld'
```

## Puppet Module

Ein Modul ist ein Verzeichnis mir einer definierten Verzeichnisstruktur.
Das Verzeichnis muss in einem $modulepath hinterlegt sein.

Modulepath (aus environment.conf):

- site (site-modules)
- modules
- $basemodulepath

### Modul Struktur

```text
<modulepath>/<module>/
  |- manifests/       # Puppet Klasse und Defined Types
  |    |- init.pp     # class <module> { ... }
  |    |- install.pp  # class <module>::install { .. }
  |    |- config.pp   # class <module>::config { ... }
  |    \- service.pp  # class <module>::service { ... }
  |- files/           # statische Dateien
  |- templates/       # dynamische Dateien (Variable Ersetzung)
  |- lib/             # Puppet Erweiterungen (Ruby!! Custom Facts, Functions, Types, Providers)
  |- spec/            # Puppet Code Testing (unit und acceptance)
  |- hiera.yaml       # Data in Modules
  |- data/            # Hira Daten (NUR für das Modul)
  \- metadata.json    # Beschreibungsdatei und Test matrix
```

## Module Entwicklung

1. I.b.m. = Immer besser manuell
1. PDK (Puppet Development Kit)

```shell
cd site
pdk new module phoenix
```
