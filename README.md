# [Lidarr][o]
Lidarr installation from public release tarball.

## [Requirements][i]
Requires [r_pufky.arr][g] galaxy-ng collection. See
[reference documentation][h] for troubleshooting and config variables.

Install size: ~280MB

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
Existing databases are migrated automatically when new versions are installed.
Migration between database platforms is not supported and must be done
manually.

### NOTE
> New installs REQUIRE 'none' to create a login user; authentication can be
> toggled on after the user exists in the database. See
> [FAQ for initial authentication configuration][p].

### Database
PostgreSQL 14+ is highly recommended over default SQLite database.

Postgres requires pre-existing databases with root privileges to initialize
tables and manage migrations. Postgres databases are NOT backed up by Lidarr
and must be managed outside of this role. An example postgres [config.xml][m]
is provided within the role.

See [reference documentation][h] and [authoritative instructions][l].

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                   | Notes
 ------|------------------------|-------
  1    | lidarr_flg_backup      | Backup config data. Exits role.
  2    | lidarr_flg_restore     | Restore config data. Exits role.
  3    | lidarr_flg_maintenance | Preform role maintenance tasks.
  4    | lidarr_flg_install     | Install required packages, users, etc.
  5    | lidarr_flg_config      | Install user-defined config.

### Example Playbooks
Lidarr will automatically generate a configuration file if one is not provided.
Example configuration files are located in [files][n].

#### New Deployments
``` yaml
# Config data: /var/opt/lidarr.
- name: 'Lidarr default install.'
  ansible.builtin.include_role:
    name: 'r_pufky.arr.lidarr'
```

#### Static Deployments
Config data is deployed as templates, allowing for vault use of sensitive
information. **However** this will deploy static files and overwrite existing
DB's, etc. Existing installs likely want to use **backup** and **restore**.

``` yaml
- name: 'Statically deploy Lidarr with pre-existing DB and config.'
  ansible.builtin.include_role:
    name: 'r_pufky.arr.lidarr'
  vars:
    lidarr_flg_config: true
    lidarr_cfg_d: 'host_vars/lidarr.example.com/data'
```

#### Dynamic Deployments
Use **backup** and **restore** to carry over existing data to a new instance;
or leave lidarr_flg_config disabled to leave existing config untouched.

``` yaml
- name: 'Upgrade Lidarr version without touching configuration.'
  ansible.builtin.include_role:
    name: 'r_pufky.arr.lidarr'
  vars:
    lidarr_srv_version: 'v4.0.17.NEW_VERSION'
    lidarr_flg_config: false

- name: 'Create backup.'
  ansible.builtin.include_role:
    name: 'r_pufky.arr.lidarr'
  vars:
    lidarr_flg_backup: true
    lidarr_cfg_backup_d: '/tmp'

- name: 'Restore from backup.'
  ansible.builtin.include_role:
    name: 'r_pufky.arr.lidarr'
  vars:
    lidarr_flg_restore: true
    lidarr_cfg_backup_d: '/tmp'
```

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | Lidarr | Notes
 ---------|--------|---------|--------|-------
  6.x.x   | 13     | 2.20    | V3     | Ansible 2.20, feature flags, and semantic versioning.
  5.x.x   | 13     | 2.18    | V3     | Migrate to r_pufky.arr.
  4.x.x   | 13     | 2.18    | V3     | Migrate to Debian Trixie.
  3.x.x   | 12     | 2.18    | V2     | Implement data annotations.
  2.x.x   | 12     | 2.18    | V2     | Migrate to Lidarr V2.
  1.x.x   | 12     | 2.11    | V0     | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]

[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_lidarr/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_arr
[h]: https://r-pufky.github.io/docs/media/lidarr
[i]: https://github.com/r-pufky/ansible_lidarr/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_lidarr/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_lidarr/blob/main/defaults/main/ports.yml
[l]: https://wiki.servarr.com/lidarr/postgres-setup
[m]: https://github.com/r-pufky/ansible_lidarr/blob/main/templates/postgres/config.xml
[n]: https://github.com/r-pufky/ansible_lidarr/blob/main/templates/
[o]: https://lidarr.audio
[p]: https://wiki.servarr.com/lidarr/faq#forced-authentication