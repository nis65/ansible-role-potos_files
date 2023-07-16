
# Ansible Role - potos\_files

This role works for me, but does not satisfy the potos acceptance rules yet:

* the automated checks are currently failing
* meta is not up to date

This role is a generic way of distributing arbitrary files by
adding them (and some meta information) to your specs repository.
See below for variables (and defaults).

## Example Playbook

As this role is tested via Molecule one can use [that
playbook](./molecule/default/converge.yml) as a starting point:

```yaml
---

- name: Converge
  hosts: all
  gather_facts: yes
  tasks:
    - name: run role
      ansible.builtin.include_role:
        name: 'ansible-role-potos_template'
```

## Role Variables

Each file you want to distribute must be located in your specs repo below `files/potos_files/`.
From there, it should be located in the same path as it will be on the target machine.

For each file, an element in the `potos_files` list must be created. Take the following
example:

```yaml

potos_files:
  - src: "usr/local/share/applications/Local.desktop"
    dest: "/usr/local/share/applications/Local.desktop"
  - src: "usr/local/bin/mount_smb_sudo"
    dest: "/usr/local/bin/mount_smb_sudo"
    mode: "755"
  - src: "tmp/silly_example"
    dest: "/tmp/silly_example"
    owner: "nobody"
    group: "nobody"
    mode: "606"
```

`src` and `dst` are mandatory, `owner`, `group` and `mode` are optional
and fall back to the default values (see below). Of course, you
can override the defaults too.

The role creates directories on the target as needed, using
`potos_files_default_owner`, `potos_files_default_group`
and `potos_files_default_dir_mode`:

The default variables are defined in [defaults/main.yml](./defaults/main.yml):

```yaml
---

potos_files_default_owner: "root"
potos_files_default_group: "root"
potos_files_default_file_mode: "0644"
potos_files_default_dir_mode: "0755"
```

An experimental feature allows to edit a single line in a file that
is delivered by a debian package:

```yaml
potos_files_diverts:
  - path: "/usr/share/themes/Yaru/gtk-3.0/gtk.css"
    holder: "mlc"
    linematch: "/etc/mlc/yaru_override.css"
    linecontent: '@import "/etc/mlc/yaru_override.css";'
```

From the Debian Manual: *It is possible to have `dpkg` not overwrite a file 
when it reinstalls the package it belongs to, and to have it put the file 
from the package somewhere else instead.* 

So what the above setting currently does: 

* it *diverts* the `gtk.css` file to `gtk.css.diverted-mlc` in the same directory.
  Next time the package delivering that file is upgraded or reinstalled, `dpkg`
  will not touch `gtk.css` any more, but overwrite `gtk.css.diverted-mlc` with
  its newest version (`community.general.dpkg_divert`).
* the diverted file is copied to the original filename `gtk.css` if it does
  not exist yet (`ansible.builtin.copy`). This prevents unneeded ansible "changes" 
  but may miss a fresh version after an update of the underlying package. To be improved.
* `gtk.css` is searched for the last occurrence of the `linematch` and replaces that
  line with the `linecontent`. If no match is found, `linecontent` is appended at
  the end of the file as new last line (`ansible.builtin.lineinfile`).

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role -r . main ansible-role-potos_template
```

## Requirements

N/A

## License

See [LICENSE](./LICENSE)

## Author Information

[Project Potos](https://github.com/projectpotos)

