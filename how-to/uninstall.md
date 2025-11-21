# How to Uninstall Docker on Ubuntu

## Uninstall Docker Engine

### Step 1: Uninstall Docker Packages
```bash
sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

### Step 2: Remove Images, Containers, and Volumes
```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

### Step 3: Remove Repository and Keyring
```bash
sudo rm /etc/apt/sources.list.d/docker.sources
sudo rm /etc/apt/keyrings/docker.asc
```

**Note:** You have to delete any edited configuration files manually.