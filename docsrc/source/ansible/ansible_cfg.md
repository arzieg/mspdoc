# Ansible.cfg

**defaults**
``` 
[defaults] 
stdout_callback             = debug          # human readable errors
inventory                   = inventory      # inventory name
timeout                     = 300            # time out

```

**ssh**
```
[ssh_connection]
ssh_args                    = -F ./ssh.cfg
```