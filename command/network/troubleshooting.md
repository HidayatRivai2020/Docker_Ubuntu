# Troubleshooting

## Common Issues

### Network Not Found
**Problem**: `Error: No such network: my-network`

**Causes**:
- Network doesn't exist
- Typo in network name
- Network was removed
- Wrong Docker context

**Solutions**:
```bash
# List all networks
docker network ls

# Search for network by name
docker network ls --filter name=my-network

# Create the network
docker network create my-network

# Check current context
docker context ls
```

### Container Cannot Connect to Network
**Problem**: Container fails to connect to network

**Causes**:
- Container already connected to network
- IP address conflict
- Network driver issues
- Container not running

**Solutions**:
```bash
# Check container status
docker ps -a --filter name=mycontainer

# Check current networks
docker inspect mycontainer --format='{{json .NetworkSettings.Networks}}' | jq

# Try connecting with specific IP
docker network connect --ip 172.18.0.100 my-network mycontainer

# Disconnect first, then reconnect
docker network disconnect my-network mycontainer
docker network connect my-network mycontainer
```

### DNS Resolution Failure
**Problem**: Containers cannot resolve each other by name

**Causes**:
- Using default bridge network (no automatic DNS)
- Incorrect network configuration
- DNS server issues
- Container name not set

**Solutions**:
```bash
# Use user-defined bridge network
docker network create my-network
docker run -d --network my-network --name web nginx
docker run --network my-network alpine ping web

# Check DNS configuration
docker exec mycontainer cat /etc/resolv.conf

# Test DNS resolution
docker exec mycontainer nslookup web
docker exec mycontainer dig web

# Manually add DNS
docker run --dns 8.8.8.8 --network my-network alpine

# Add custom hostname entry
docker run --add-host db:192.168.1.10 myapp
```

### Port Already in Use
**Problem**: `Bind for 0.0.0.0:8080 failed: port is already allocated`

**Causes**:
- Another container using the port
- Host service using the port
- Previous container not properly removed

**Solutions**:
```bash
# Find what's using the port
netstat -tulpn | grep 8080
lsof -i :8080

# Find Docker containers using the port
docker ps --format "table {{.Names}}\t{{.Ports}}" | grep 8080

# Stop conflicting container
docker stop <container>

# Use different host port
docker run -p 8081:80 nginx

# Use host network (no port mapping)
docker run --network host nginx
```

### Cannot Remove Network
**Problem**: `Error: network has active endpoints`

**Causes**:
- Containers still connected
- Network in use by services

**Solutions**:
```bash
# Check connected containers
docker network inspect my-network --format='{{range .Containers}}{{.Name}} {{end}}'

# Disconnect all containers
docker network inspect my-network \
  --format='{{range .Containers}}{{.Name}}{{"\n"}}{{end}}' | \
  xargs -I {} docker network disconnect -f my-network {}

# Force remove
docker network rm -f my-network

# Remove with all containers
docker ps -a --filter network=my-network -q | xargs docker rm -f
docker network rm my-network
```

### Containers Cannot Communicate
**Problem**: Containers on same network cannot reach each other

**Causes**:
- Firewall rules blocking traffic
- Wrong network
- ICC (Inter-Container Communication) disabled
- Network driver issues

**Solutions**:
```bash
# Verify both on same network
docker network inspect my-network

# Test connectivity
docker exec container1 ping container2

# Check firewall rules
sudo iptables -L -n | grep DOCKER

# Check ICC setting
docker network inspect my-network --format='{{.Options}}'

# Create network with ICC enabled
docker network create -o "com.docker.network.bridge.enable_icc"="true" my-network

# Check routes
docker exec container1 ip route
```

### Overlay Network Not Working
**Problem**: Overlay network cannot be created or used

**Causes**:
- Swarm not initialized
- Wrong scope
- Encryption issues
- Network driver unavailable

**Solutions**:
```bash
# Initialize swarm
docker swarm init

# Create attachable overlay
docker network create -d overlay --attachable my-overlay

# Check swarm status
docker info | grep Swarm

# List overlay networks
docker network ls --filter driver=overlay

# Inspect overlay network
docker network inspect my-overlay
```

### IP Address Conflicts
**Problem**: Container cannot get IP or IP conflicts with existing

**Causes**:
- IP already assigned
- Subnet conflicts
- IP range exhausted

**Solutions**:
```bash
# Check network IPAM
docker network inspect my-network --format='{{json .IPAM}}' | jq

# Create network with larger subnet
docker network create --subnet 172.28.0.0/16 my-network

# Use specific IP range
docker network create \
  --subnet 172.28.0.0/24 \
  --ip-range 172.28.0.128/25 \
  my-network

# List all container IPs on network
docker network inspect my-network \
  --format='{{range .Containers}}{{.IPv4Address}}{{"\n"}}{{end}}'
```

### Network Isolation Not Working
**Problem**: Containers can access networks they shouldn't

**Causes**:
- Connected to multiple networks
- Using host network
- Firewall rules not set

**Solutions**:
```bash
# Check all container networks
docker inspect mycontainer --format='{{json .NetworkSettings.Networks}}' | jq

# Create internal network
docker network create --internal private-net

# Disconnect from unwanted networks
docker network disconnect bridge mycontainer

# Verify internal flag
docker network inspect private-net --format='{{.Internal}}'
```

## Debugging Commands

### Network Inspection
```bash
# Full network details
docker network inspect <network>

# Network driver
docker network inspect <network> --format='{{.Driver}}'

# Network subnet
docker network inspect <network> --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}'

# Connected containers
docker network inspect <network> --format='{{len .Containers}} containers'

# Network scope
docker network inspect <network> --format='{{.Scope}}'
```

### Container Network Info
```bash
# All networks for container
docker inspect <container> --format='{{json .NetworkSettings.Networks}}' | jq

# Container IP addresses
docker inspect <container> --format='{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}'

# Container gateway
docker inspect <container> --format='{{range .NetworkSettings.Networks}}{{.Gateway}} {{end}}'

# Network aliases
docker inspect <container> --format='{{range .NetworkSettings.Networks}}{{.Aliases}}{{end}}'
```

### Connectivity Testing
```bash
# Ping test
docker exec <container> ping -c3 <target>

# DNS resolution
docker exec <container> nslookup <hostname>
docker exec <container> dig <hostname>

# Port connectivity
docker exec <container> nc -zv <host> <port>
docker exec <container> telnet <host> <port>

# Trace route
docker exec <container> traceroute <target>
docker exec <container> mtr -n -c 5 <target>

# HTTP test
docker exec <container> curl -v http://<target>
docker exec <container> wget -O- http://<target>
```

### Network Interface Info
```bash
# List interfaces
docker exec <container> ip addr
docker exec <container> ifconfig

# Routing table
docker exec <container> ip route
docker exec <container> route -n

# ARP table
docker exec <container> ip neigh
docker exec <container> arp -a

# Network statistics
docker exec <container> netstat -s
docker exec <container> ss -s
```

### Packet Capture
```bash
# Capture traffic
docker exec <container> tcpdump -i eth0 -w /tmp/capture.pcap

# Capture specific port
docker exec <container> tcpdump -i eth0 port 80

# Capture and display
docker exec <container> tcpdump -i eth0 -n

# Copy capture file
docker cp <container>:/tmp/capture.pcap ./
```

### Host Network Inspection
```bash
# Docker network bridge
ip addr show docker0

# Bridge interfaces
brctl show

# IPtables rules
sudo iptables -L -n -v -t nat
sudo iptables -L -n -v -t filter

# Network namespaces
sudo ip netns list
```

## Exit Codes and Meanings

### Network Command Exit Codes
- **0**: Success
- **1**: General error
- **125**: Docker daemon error
- **126**: Container command cannot be invoked
- **127**: Container command not found

### Common Error Messages

**"could not find plugin"**
- Network driver not available
- Install required plugin or use built-in driver

**"address already in use"**
- Subnet conflicts with existing network
- Choose different subnet or remove conflicting network

**"no suitable address pool"**
- Default address pools exhausted
- Configure custom address pools in daemon.json

**"network not found"**
- Network doesn't exist or was removed
- Create network or check network name

**"endpoint not found"**
- Container not connected to network
- Connect container to network first

## Recovery Procedures

### Reset Docker Networking
```bash
# Stop Docker daemon
sudo systemctl stop docker

# Remove network state
sudo rm -rf /var/lib/docker/network/

# Start Docker daemon (recreates default networks)
sudo systemctl start docker

# Verify default networks
docker network ls
```

### Recreate Network
```bash
# Backup network configuration
docker network inspect my-network > network-backup.json

# Disconnect all containers
for container in $(docker network inspect my-network --format='{{range .Containers}}{{.Name}} {{end}}'); do
  docker network disconnect my-network $container
done

# Remove network
docker network rm my-network

# Recreate from backup
# Extract subnet, gateway, etc. from backup.json and recreate
docker network create \
  --subnet <subnet> \
  --gateway <gateway> \
  my-network

# Reconnect containers
for container in <container-list>; do
  docker network connect my-network $container
done
```

### Fix Network Connectivity
```bash
# Restart container networking
docker restart <container>

# Disconnect and reconnect
docker network disconnect <network> <container>
docker network connect <network> <container>

# Recreate container with same settings
docker commit <container> temp-image
docker stop <container>
docker rm <container>
docker run -d --name <container> --network <network> temp-image
docker rmi temp-image
```

### Clear Network Issues
```bash
# Remove all custom networks
docker network rm $(docker network ls --filter type=custom -q)

# Prune unused networks
docker network prune -f

# Reset iptables (Linux)
sudo iptables -t nat -F
sudo iptables -t filter -F
sudo systemctl restart docker

# Flush DNS cache (if using dnsmasq)
sudo systemctl restart dnsmasq
```

### Restore Default Network
```bash
# If default bridge is missing
sudo systemctl stop docker
sudo ip link delete docker0
sudo systemctl start docker

# Verify
docker network ls
docker network inspect bridge
```