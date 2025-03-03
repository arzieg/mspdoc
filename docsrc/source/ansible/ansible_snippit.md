# Snippit

## python virtual env

```
vars:
  venv_home: "{{ ansible_env.HOME }}/.virtualenvs"
  venv_path: "{{ venv_home }}/{{ proj_name }}"

tasks: 
 - name: Create python3 virtualenv
      pip:
        name:
          - pip
          - wheel
          - setuptools
        state: latest
        virtualenv: "{{ venv_path }}" 
        virtualenv_command: /usr/bin/python3 -m venv
```