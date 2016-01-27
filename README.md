find-package
=========

A simple tool for finding the latest package at file system. Using this method to deploy without package server.

Requirements
------------

find command and bash shell

Role Variables
--------------

```yml

# package dir
package_dir: "/path/rpm/"

# package name prefix for find command
package_name_prefix: "package-name*"

# command for find last package built
package_find_command: "find {{ package_dir }} -name '{{ package_name_prefix }}' -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d ' ' "

# for other usage
package_install_dir: "/opt/package-version-release/"

# for multiple packages
package_list:
  - name: "my-package-name-1"
    find_command: "find {{ package_dir }} -name '{{ package_name_prefix }}' -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d ' ' "
  - name: "my-package-name-2"
    find_command: "find {{ package_dir }} -name '{{ package_name_prefix }}' -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d ' ' "


```

Dependencies
------------

None

Example Playbook
----------------

```yml
- hosts: localhost
  roles:
  - { role: changyy.find-package, package_dir: "/data/rpm/production/" , package_name_prefix: "package-*" }

- hosts: 127.0.0.1

  tasks:
  - name: copy package
    copy: src="{{hostvars['localhost']['package_path']}}" dest=/tmp

  # - name: show multiple package info
  #   debug: var="{{item}}"
  #   with_items: hostvars['localhost']['rpm_file_info']
  # - name: show multiple package path
  #   debug: var="{{item}}"
  #   with_items: hostvars['localhost']['rpm_file_path']
```

```bash
$ ansible-playbook test.yml
PLAY [localhost] ************************************************************** 

GATHERING FACTS *************************************************************** 
ok: [localhost]

TASK: [changyy.find-package | show package prefix name] *********************** 
ok: [localhost] => {
    "msg": "package-*"
}

TASK: [changyy.find-package | show find package command] ********************** 
ok: [localhost] => {
    "msg": "find /data/rpm/production/ -name 'package-*' -printf '%T@ %p\n' | sort -n | tail -1 | cut -f2- -d ' ' "
}

TASK: [changyy.find-package | find package path] ****************************** 
changed: [localhost -> 127.0.0.1]

TASK: [changyy.find-package | save package path] ****************************** 
ok: [localhost]

TASK: [changyy.find-package | save package dir] ******************************* 
ok: [localhost]

TASK: [changyy.find-package | show package path] ****************************** 
ok: [localhost] => {
    "msg": "/data/rpm/production/package-production-1.0.0-17.noarch.rpm"
}

PLAY [127.0.0.1] ************************************************************** 

GATHERING FACTS *************************************************************** 
ok: [localhost]

TASK: [copy package] ********************************************************** 
ok: [localhost]

PLAY RECAP ******************************************************************** 
localhost                  : ok=9    changed=1    unreachable=0    failed=0   
```

You may want to keep the directory for debug and install this package on this way:

```yml

  tasks:

  - name: install alien for rpm install
    apt: name={{ item }} update_cache=yes state=latest
    with_items:
      - alien

  - name: prepare package dir
    file: path={{hostvars['localhost']['package_dir']}} state=directory mode=0755

  - name: deploy package via copy
    copy: src={{hostvars['localhost']['package_dir']}} dest={{hostvars['localhost']['package_dir']}}

  - name: install package
    command: bash -c 'alien -i "{{hostvars['localhost']['package_path']}}"'

```

License
-------

BSD

Author Information
------------------

- Original : Yuan-Yi Chang
