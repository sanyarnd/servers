# Ansible scripts for homelab and VPS servers

## Dependencies

Install the required Ansible collection before running playbooks:

```shell
ansible-galaxy collection install -r requirements.yml
```

## Run playbooks

All playbooks are located in the project root directory.

For example, to launch `homelab` playbook, use the following command:
```shell
ansible-playbook homelab.yml --ask-become-pass
```

### Running only some roles (tags)

You can limit the run to specific roles with `--tags`:

```shell
# Only base system + Docker
ansible-playbook homelab.yml --ask-become-pass --tags base,docker

# Only Home Assistant
ansible-playbook homelab.yml --ask-become-pass --tags homeassistant
```

## Prerequisites (devcontainers)

Devcontainers use `ssh-agent` to forward host keys inside a container.

`ssh-agent` is key-centric, instead of host-centric. It means it doesn't know user mapping and it will try all keys until one of them matches.

You will need 2 things:
1. Start `ssh-agent` service
    ```powershell
    # for Windows
    Set-Service ssh-agent -StartupType Automatic
    Start-Service ssh-agent
    Get-Service ssh-agent

    # for Linux
    eval "$(ssh-agent -s)"
    ```
2. Make sure all necessary keys are added via `ssh-add` command:
    ```powershell
    ssh-add ~/.ssh/<your-private-key>
    ```

You can troubleshoot added keys with `ssh-add -l`, which will list all available `ssh-agent` keys.

After that you can use private keys from your host system inside a devcontainer.
Make sure to specify the name of the user explicitly or setup username for container itself.


## Prerequisites (server)

All servers are running at least `Ubuntu 24.04`.

Make sure to run [unminimize](https://documentation.ubuntu.com/public-cloud/all-clouds-explanation/ubuntu-base-and-minimal-images/#what-is-unminimize-what-does-it-do), if the server was installed from a minimal image.

1. Create an `ansible` user with sudo rights:
    ```shell
    sudo adduser ansible
    sudo usermod -aG sudo ansible
    ```
2. Generate SSH key:
    ```shell
    ssh-keygen -t ed25519 -C "ansible@<server-name>"
    ```
3. Export SSH key to the server:
    ```shell
    mkdir -p ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    # add the key to the list
    nano ~/.ssh/authorized_keys
    ```
4. Modify your `~/.ssh/config`:
    ```shell
    Host <your-server-ip or DNS name>
      HostName <your-server-ip or DNS name>
      PreferredAuthentications publickey
      User ansible
      IdentityFile ~/.ssh/homelab_ansible
    ```
5. Disable password authentication:
    ```shell
    # Modify /etc/ssh/sshd_config
    sudo sh -c 'echo "PasswordAuthentication no\nPubkeyAuthentication yes\nPermitRootLogin no\n" > /etc/ssh/sshd_config.d/10-secure-ssh.conf'
    # Validate the configuration
    cat /etc/ssh/sshd_config.d/10-secure-ssh.conf
    # restart SSH
    sudo systemctl restart ssh
    # validate settings
    sudo sshd -T | grep passwordauthentication
    ```
6. Validate the connection:
    ```shell
    ansible servers -m ping
    ```