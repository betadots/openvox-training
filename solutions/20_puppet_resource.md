# Sample Solution

## Puppet Resource Command

* check current state of resource `chronyd` of type service

```bash
    $ puppet resource service chronyd
    service { 'chronyd':
      ensure   => 'running',
      enable   => 'true',
      provider => 'systemd',
    }
```

* Install package `vim-enhanced` if it is not already installed

```bash
    $ puppet resource package vim-enhanced
    package { 'vim-enhanced':
      ensure   => 'purged',
      provider => 'dnf',
    }
    
    $ puppet resource package vim-enhanced ensure=installed
    Notice: /Package[vim-enhanced]/ensure: created
    package { 'vim-enhanced':
      ensure   => '2:8.2.2637-21.el9',
      provider => 'dnf',
    }
```

* Add user `kermit` and remove him again

Without Option `-s` you get the long and complete help.

```bash
    puppet describe user -s
    
    user
    ====
    Manage users.  This type is mostly built to manage system
    users, so it is lacking some features useful for managing normal
    users.
    This resource type uses the prescribed native tools for creating
    groups and generally uses POSIX APIs for retrieving information
    about them.  It does not directly modify `/etc/passwd` or anything.
    **Autorequires:** If Puppet is managing the user's primary group (as
    provided in the `gid` attribute) or any group listed in the `groups`
    attribute then the user resource will autorequire that group. If Puppet
    is managing any role accounts corresponding to the user's roles, the
    user resource will autorequire those role accounts.
    
    
    Parameters
    ----------
        allowdupe, attribute_membership, attributes, auth_membership, auths,
        comment, ensure, expiry, forcelocal, gid, groups, home, ia_load_module,
        iterations, key_membership, keys, loginclass, managehome, membership,
        name, password, password_max_age, password_min_age, password_warn_days,
        profile_membership, profiles, project, purge_ssh_keys, role_membership,
        roles, salt, shell, system, uid
    
    Providers
    ---------
        aix, directoryservice, hpuxuseradd, ldap, openbsd, pw, user_role_add,
        useradd, windows_adsi
```

```bash
    $ puppet resource user kermit ensure=present managehome=true
    Notice: /User[kermit]/ensure: created
    user { 'kermit':
      ensure   => 'present',
      provider => 'useradd',
    }
    
    $ getent passwd kermit
    kermit:x:1001:1001::/home/kermit:/bin/bash
    
    $ ls -al ~kermit/
    total 12
    drwx------. 2 kermit kermit  62 Mar  7 12:27 .
    drwxr-xr-x. 4 root   root    33 Mar  7 12:27 ..
    -rw-r--r--. 1 kermit kermit  18 Apr 30  2024 .bash_logout
    -rw-r--r--. 1 kermit kermit 141 Apr 30  2024 .bash_profile
    -rw-r--r--. 1 kermit kermit 492 Apr 30  2024 .bashrc
```

```bash
    $ puppet resource user kermit ensure=absent
    Notice: /User[kermit]/ensure: removed
    user { 'kermit':
      ensure   => 'absent',
      provider => 'useradd',
    }
    
    $ ls -al /home/
    total 0
    drwxr-xr-x.  4 root  root   33 Mar  7 12:27 .
    dr-xr-xr-x. 19 root  root  252 Mar  6 11:27 ..
    drwx------.  2  1001  1001  62 Mar  7 12:27 kermit
```

Doesn't help in retrospect:

```bash
    $ puppet resource user kermit ensure=absent managehome=true
    user { 'kermit':
      ensure   => 'absent',
      provider => 'useradd',
    }
    
    $ ls -al /home/
    total 0
    drwxr-xr-x.  4 root  root   33 Mar  7 12:27 .
    dr-xr-xr-x. 19 root  root  252 Mar  6 11:27 ..
    drwx------.  2  1001  1001  62 Mar  7 12:27 kermit
```

If you specify it at the same time, the directory including all user files is removed.
