# How to Install Docker on Ubuntu

## Prerequisites

### OS Requirements
To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:
- Ubuntu Noble 24.04 (LTS)
- Ubuntu Jammy 22.04 (LTS)  
- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)

Docker Engine for Ubuntu is compatible with x86_64 (or amd64), armhf, arm64, s390x, and ppc64le (ppc64el) architectures.

### Uninstall Old Versions (if any)
Before installing Docker Engine, uninstall any conflicting packages:

```bash
sudo apt remove docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc
```

## Installation Methods

### Method 1: Install Using the APT Repository (Recommended)

#### Step 1: Set up Docker's APT Repository
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

#### Step 2: Install Docker Engine
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Step 3: Verify Installation
```bash
sudo docker run hello-world
```

### Method 2: Install from a Package
If you can't use Docker's APT repository, you can download the `.deb` file manually and install it.

#### Step 1: Download Docker Packages
1. Go to https://download.docker.com/linux/ubuntu/dists/
2. Select your Ubuntu version in the list
3. Go to `pool/stable/` and select the applicable architecture (`amd64`, `armhf`, `arm64`, or `s390x`)
4. Download the following `.deb` files for the Docker Engine, CLI, containerd, and Docker Compose packages:
   - `containerd.io_<version>_<arch>.deb`
   - `docker-ce_<version>_<arch>.deb`
   - `docker-ce-cli_<version>_<arch>.deb`
   - `docker-buildx-plugin_<version>_<arch>.deb`
   - `docker-compose-plugin_<version>_<arch>.deb`

#### Step 2: Install the .deb Packages
Update the paths in the following example to where you downloaded the Docker packages:

```bash
sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
  ./docker-ce_<version>_<arch>.deb \
  ./docker-ce-cli_<version>_<arch>.deb \
  ./docker-buildx-plugin_<version>_<arch>.deb \
  ./docker-compose-plugin_<version>_<arch>.deb
```

#### Step 3: Verify Installation
```bash
sudo docker run hello-world
```

### Method 3: Install Using Convenience Script
For testing and development environments only (not recommended for production):

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Post-Installation Steps

### Start Docker Service
The Docker service starts automatically after installation. To verify:
```bash
sudo systemctl status docker
```

If Docker is not running, start it manually:
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Add User to Docker Group (Optional)
To run Docker commands without sudo:
```bash
sudo usermod -aG docker $USER
```
**Note:** Log out and log back in for this change to take effect.

### Verify Docker Version
```bash
docker --version
docker compose version
```

## Troubleshooting

### Permission Denied Error
If you get permission denied errors when running Docker commands:
1. Make sure your user is in the docker group: `groups $USER`
2. If not, add yourself: `sudo usermod -aG docker $USER`  
3. Log out and log back in

### Docker Service Not Starting
If Docker service fails to start:
```bash
sudo systemctl status docker
sudo journalctl -u docker.service
```