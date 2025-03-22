# Install K3s in an LXC Container

## Steps to Install K3s in a Container

- Configure Proxmox Host
- Configure LXC Container

# 1. Configure Proxmox Host

## Proxmox Shell:

### Configure `/etc/sysctl.conf`

```
vim /etc/sysctl.conf
```

Uncomment the following line:

```
net.ipv4.ip_forward=1
```

And add:

```
vm.swappiness=0
```

## Disable Swap:

```
swapoff -a
```

### Edit `/etc/fstab`

```
vim /etc/fstab
```

Comment out:

```
/dev/pve/swap none swap sw 0 0
```

# 2. Configure Proxmox LXC Container

## Create an Unprivileged LXC Container

Create an unprivileged LXC container with Ubuntu and disable swap (set swap (MiB) to 0).

## Modify the Container Configuration

Edit the container config file located at:

```
/etc/pve/lxc/$ID.conf
```

Replace `$ID` with your container's ID.

Add the following lines:

```
lxc.apparmor.profile: unconfined
lxc.cgroup2.devices.allow: a
lxc.cap.drop:
lxc.mount.auto: "proc:rw sys:rw"
```

## Configure Inside the LXC Container

Create the missing `/etc/rc.local` file:

```
#!/bin/sh -e
if [ ! -e /dev/kmsg ]; then
    ln -s /dev/console /dev/kmsg
fi
mount --make-rshared /
```

Set permissions and apply changes:

```
chmod +x /etc/rc.local
/etc/rc.local
```

# 3. Install K3s

## Install K3s

```
curl -sfL https://get.k3s.io | sh -s - server --disable servicelb --disable traefik --write-kubeconfig-mode 644
```

## Configure K3s Server

```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Verify Installation

```
kubectl get nodes
```

## Thank You

Sources:

- [Kubernetes inside Proxmox LXC](https://kevingoos.medium.com/kubernetes-inside-proxmox-lxc-cce5c9927942)
- [Installing K3s in an LXC Container](https://kevingoos.medium.com/installing-k3s-in-an-lxc-container-2fc24b655b93)
