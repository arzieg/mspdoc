# Ansible

## Arbeitsumgebung (to be tested)

* pre-commit: `pip install pre-commit`
* ansible-lint: `pip3 install ansible-lint`
* ansible-doctor: `pip install ansible-doctor[ansible-core] --user`
    * https://ansible-doctor.geekdocs.de/setup/pip/
* shellcheck: `apt-get shellcheck`
    * https://www.shellcheck.net/
* ansible-builder:`pip3 install ansible-builder` - Automatisierung von Ansible Ausführungsumgebungen. 
    * https://github.com/ansible/ansible-
* ansible-navigator: `pip3 install 'ansible-navigator[ansible-core]'`- TUI für ansible
    * https://github.com/ansible/ansible-navigator
* molecule: `python3 -m pip install molecule ansible-core` - Testumgebung für Ansible
    * https://github.com/ansible/molecule
* antsiball-changelog: `pip install antsibull-changelog` - Create Changelog
    * https://github.com/ansible-community/antsibull-changelog


## Show facts
`ansible <hostname> -m ansible.builtin.setup`


## Debug

export ANSIBLE_KEEP_REMOTE_FILES=1 -> damit wird sichergestellt, dass auf dem target-host die tmp-files nicht gelöscht werden. Hilfreich beim debuggen. Die Tempfiles sind zu finden unter ~/.ansible/tmp/

ansible-playbook:   --force-handlers  -> Option verhindert den Abbruch bei Fehlern? 

### debug module
```
- debug: var=myvariable
- debug: msg="The value of myvariable is {{ var }}"
```

### Playbook debugger

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_debugger.html

```
- name: deploy 
  hosts: web 
  debugger: always  <- 
```

Debugger commands: 

| Command                | Shortcut    | Action                                            |
| :--------------------- | ----------- | ------------------------------------------------- |
| print                  | p           | Print information about the task                  |
| task.args[key] = value | no shortcut | Update module arguments                           |
| task_vars[key] = value | no shortcut | Update task variables (you must update_task next) |
| update_task            | u           | Re-create a task with updated task variables      |
| redo                   | r           | Run the task again                                |
| continue               | c           | Continue executing, starting with the next task   |
| quit                   | q           | Quit the debugger                                 |

Variables supportet by the debugger:

| Variable    | Description                            |
| :---------- | -------------------------------------- |
| p task      | The name of the task that failed       |
| p task.args | The module arguments                   |
| p result    | The result returned by the failed task |
| p vars      | Value of all known variables           |
| p vars[key] | Value of one variable                  |

### assert module

asset use jinja2! 

```
- name: Stat /boot/grub
  stat:
    path: /boot/grub
  register: st

- name: Assert that /boot/grub is a directory
  assert:
    that: st.stat.isdir
``` 

### Checking Your Playbook Before Execution

```
ansible-playbook --syntax-check playbook.yml   - syntax
ansible-playbook --list-hosts playbook.yml     - list hosts to execute on
ansible-playbook --list-tasks playbook.yml     - list tasks
ansible-playbook --check playbook.yml          - check mode / dry run
ansible-playbook --diff --check playbook.yml   - diff what will be changed / check mode
ansible-playbook --tags=xinx,database playbook.yml    - run only these tags
ansible-playbook --skip-tags=database playbook.yml    - skip these tags
ansible-playbook -vv --limit db playbook.yml   - limit (-l <hostname>)
      