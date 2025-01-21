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



