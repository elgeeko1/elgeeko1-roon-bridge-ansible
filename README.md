# Ansible role for roon-bridge-docker

This role deploys the Docker image [elgeeko/roon-bridge](https://hub.docker.com/repository/docker/elgeeko/roon-bridge) to a host and configures it to run at startup.

## Requirements

Provisioning host:

- ansible 2.9 or later

Target:

- Ubuntu 18.04 or later

## How to use this role

### Method 1: Install using ansible-galaxy

The most robust way to install this role is to use ansible-galaxy,
which is installed with ansible by default. ansible-galaxy is effectively a package manager for ansible. It installs roles
from the community, or from private git repos, into your local machine for use in your playbooks.

Start by adding this role to an ansible-galaxy dependency file. Typically this file lives alongside your playbook file and by convention is named `requirements.yml`.

Add the following section to `requirements.yml`:

```yaml
roles:
  - name: elgeeko1-roon-bridge-ansible
    src: https://github.com/elgeeko1/elgeeko1-roon-bridge-ansible
    version: main
```

If there is already a `roles` section, simply append this role to
add it as a dependency.

Then, install the role using ansible-galaxy:

`ansible-galaxy install -r requirements.yml -v`

### Method 2: Clone this repository into a `roles` directory

Use this method if you plan to modify this role, or if for some
reason the ansible-galaxy method fails.

Starting from your playbook directory, change into the `roles`
directory and clone:

```bash
cd roles/
git clone https://github.com/elgeeko1/elgeeko1-roon-bridge-ansible
```

## Configuring

The variable `roon_bridge_path` defines the filesystem path on the host to store
roon data, and defaults to `/opt/RoonBridge`. This should be a persistent
location in your filesystem for storing the Roon configuration and cache.

## Security

### Host network mode (default)

By default, Roon Bridge will run in a docker container with the `host` network
mode. This is less secure but is a more surefire configuration.

Host network mode is set with the role variable `roon_network_mode: host`

### Macvlan network mode

Alternately the Roon Bridge container can run with the `macvlan` network mode, which is both robust in that the container appears to be on your
local network, and secure in that it runs in an isolated docker network.

*Warning* macvlan network mode does not work over wireless connections.

macvlan network mode is set by the following role variables:

```yaml
roon_network_mode: macvlan
roon_network_macvlan:
  name:         "roon"
  subnet:       "192.168.1.0/24"
  gateway:      "192.168.1.1"
  interface:    "{{ ansible_default_ipv4.interface }}"
  ipv4_address: "192.168.1.50" # IP address of Roon Bridge
```

### Bridged network mode

This mode is unsupported and should only be used if you will access roon-bridge
from within the same Docker bridge network. roon-bridge will not be discoverable
over a network as broadcast and multicast messages are not supported in bridge
mode.

Bridged network mode is set with the role variable
`roon_network_mode: bridge`

### Privileged execution

This role starts Roon Bridge in an unprivileged docker container. USB and ALSA
devices are mapped from the host into the container to allow Roon  to use
USB DACs attached to the host. Devices may be hot-swapped without restarting the
container.

If you are unable to access your DAC, or if you need a bus other than USB (such
as I2C), you will either need to map the device into the container, or revert to
using the insecure docker privileged execution mode.

`roon_container_privileged: true`
