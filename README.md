# home-service

My home service stack running on a [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5//) with [Raspberry Pi OS](https://www.raspberrypi.com/software/). Applications are run as [docker](https://www.docker.com/) containers and managed by docker compose to support my home infrastructure.

## Core components

- [direnv](https://github.com/direnv/direnv): Update environment per working directory.
- [renovate](https://github.com/renovatebot/renovate): Universal dependency automation tool.
- [sops](https://github.com/getsops/sops): Manage secrets which are commited to Git using [Age](https://github.com/FiloSottile/age) for encryption.
- [task](https://github.com/go-task/task): A task runner / simpler Make alternative written in Go.

## Setup

### System configuration

> [!IMPORTANT]
> A non-root user must be created (if not already) and used.

2. Install required system deps and reboot

    ```sh
    sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
    sudo apt install git
    ```

3. Make a new [SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), add it to GitHub and clone your repo

    ```sh
    export GITHUB_USER="joryirving"
    curl https://github.com/$GITHUB_USER.keys > ~/.ssh/authorized_keys
    sudo install -d -o $(logname) -g $(logname) -m 755 ~/git/home-service
    git clone git@github.com:$GITHUB_USER/home-service.git ~/git/home-service
    ```

4. Install additional system deps and reboot

    ```sh
    cd ~/git/home-service
    task deps
    ```

5. Add user to docker group

    ```sh
    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```

7. Create an Age public/private key pair for use with sops

    ```sh
    age-keygen -o /var/opt/home-service/age.key
    ```

### Container configuration

> [!TIP]
> _To encrypt files with sops **replace** the **public key** in the `.sops.yaml` file with **your Age public key**. The format should look similar to the one already present._

View the [apps](./apps) directory for documentation on configuring an app container used here, or setup your own by reviewing the structure of this repository.

Using the included [Taskfile](./Taskfile.yaml) there are helper commands to start, stop, restart containers and more. Run the command below to view all available tasks.

```sh
task --list
```

### Optional configuration

#### Fish shell

> [!TIP]
> _🐟 [fish](https://fishshell.com/) is awesome, you should try fish!_
```sh
chsh -s /usr/bin/fish
# IMPORTANT: Log out and log back in
task dotfiles
```

## Network topology

| Name | Subnet | DHCP range |
|------|--------|------------|
| LAN | 192.168.1.0/24 | 6-254 |
| GUESTS | 192.168.6.0/24 | 6-254 |
| IOT | 192.168.10.0/24 | 6-254 |
| CAMERA | 192.168.20.0/24 | 6-254 |
| TRUSTED | 192.168.30.0/24 | 6-254 |
| SERVERS | 10.69.1.0/24 | 6-254 |

## Related Projects

- [onedr0p/home-service](https://github.com/onedr0p/home-service/): Original repo where most of the config was taken from. Fedora IoT/x86 based.
- [bjw-s/nix-config](https://github.com/bjw-s/nix-config/): NixOS driven configuration for running a home service machine, a nas or [nix-darwin](https://github.com/LnL7/nix-darwin) using [deploy-rs](https://github.com/serokell/deploy-rs) and [home-manager](https://github.com/nix-community/home-manager).
- [truxnell/nix-config](https://github.com/truxnell/nix-config): NixOS driven configuration for running your entire homelab.
