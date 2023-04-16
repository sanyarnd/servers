# Ansible's configuration for my servers

```sh
ansible-playbook handystuff.net.yml -i hosts.yml -K
ansible-playbook handystuff.net.yml -i hosts.yml --extra-vars "ansible_sudo_pass=yourPassword"
```

TODO:
* deploy without sudo pass