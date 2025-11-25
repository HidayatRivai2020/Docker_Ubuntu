# Network Command Examples

## Creating Networks

### Basic Bridge Network
```bash
# Create simple bridge network
docker network create my-network

# Create with specific driver
docker network create -d bridge my-bridge

# Create with custom subnet
docker network create --subnet 192.168.1.0/24 my-network

# Create with subnet and gateway
docker network create \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  my-network
```

### Advanced Network Configuration
```bash
# Create with IP range
docker network create \
  --subnet 192.168.1.0/24 \
  --ip-range 192.168.1.128/25 \
  --gateway 192.168.1.1 \
  my-network

# Create internal network (no external access)
docker network create --internal private-network

# Create with IPv6 support
docker network create \
  --ipv6 \
  --subnet 2001:db8::/64 \
  my-ipv6-network

# Create with driver options
docker network create \
  -d bridge \
  -o "com.docker.network.bridge.name"="my-bridge" \
  -o "com.docker.network.bridge.enable_icc"="true" \
  my-network
```

### Overlay Networks (Swarm)
```bash
# Create overlay network
docker network create -d overlay my-overlay

# Create attachable overlay (for standalone containers)
docker network create -d overlay --attachable my-overlay

# Create encrypted overlay
docker network create \
  -d overlay \
  --opt encrypted \
  my-secure-overlay

# Create overlay with subnet
docker network create \
  -d overlay \
  --subnet 10.0.0.0/24 \
  --gateway 10.0.0.1 \
  my-overlay
```

### MACVLAN Networks
```bash
# Create macvlan network
docker network create \
  -d macvlan \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  -o parent=eth0 \
  my-macvlan

# Create with VLAN
docker network create \
  -d macvlan \
  --subnet 192.168.100.0/24 \
  -o parent=eth0.100 \
  my-vlan100
```

### IPVLAN Networks
```bash
# Create IPVLAN L2 network
docker network create \
  -d ipvlan \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  -o parent=eth0 \
  -o ipvlan_mode=l2 \
  my-ipvlan-l2

# Create IPVLAN L3 network
docker network create \
  -d ipvlan \
  --subnet 192.168.100.0/24 \
  -o parent=eth0 \
  -o ipvlan_mode=l3 \
  my-ipvlan-l3
```

### Networks with Labels
```bash
# Create with single label
docker network create \
  --label env=production \
  prod-network

# Create with multiple labels
docker network create \
  --label env=development \
  --label team=backend \
  --label app=myapp \
  dev-network
```

## Listing Networks

### Basic Listing
```bash
# List all networks
docker network ls

# List with quiet mode (IDs only)
docker network ls -q

# List without truncating IDs
docker network ls --no-trunc
```

### Filtered Listing
```bash
# Filter by driver
docker network ls --filter driver=bridge
docker network ls --filter driver=overlay

# Filter by name
docker network ls --filter name=my-network

# Filter by label
docker network ls --filter label=env=production

# Filter by type
docker network ls --filter type=custom
docker network ls --filter type=builtin

# Filter by scope
docker network ls --filter scope=local
docker network ls --filter scope=swarm

# Multiple filters
docker network ls \
  --filter driver=bridge \
  --filter label=env=production
```

### Custom Format
```bash
# Basic table format
docker network ls --format "table {{.ID}}\t{{.Name}}\t{{.Driver}}"

# Include scope and subnet
docker network ls --format "table {{.Name}}\t{{.Driver}}\t{{.Scope}}\t{{.Subnet}}"

# JSON format
docker network ls --format "{{json .}}"

# Custom output
docker network ls --format "Network: {{.Name}} ({{.Driver}})"
```

## Inspecting Networks

### Basic Inspection
```bash
# Inspect single network
docker network inspect my-network

# Inspect multiple networks
docker network inspect network1 network2

# Pretty print JSON
docker network inspect my-network | jq
```

### Formatted Inspection
```bash
# Get subnet
docker network inspect my-network \
  --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}'

# Get gateway
docker network inspect my-network \
  --format='{{range .IPAM.Config}}{{.Gateway}}{{end}}'

# Get driver
docker network inspect my-network --format='{{.Driver}}'

# List connected containers
docker network inspect my-network \
  --format='{{range .Containers}}{{.Name}} {{end}}'

# Get container IPs
docker network inspect my-network \
  --format='{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{"\n"}}{{end}}'

# Check if internal
docker network inspect my-network --format='{{.Internal}}'

# Get all labels
docker network inspect my-network --format='{{json .Labels}}' | jq
```

## Connecting Containers

### Basic Connection
```bash
# Connect running container to network
docker network connect my-network web

# Connect with specific IP
docker network connect --ip 192.168.1.100 my-network web

# Connect with IPv6
docker network connect --ip6 2001:db8::100 my-network web

# Connect with both IPv4 and IPv6
docker network connect \
  --ip 192.168.1.100 \
  --ip6 2001:db8::100 \
  my-network web
```

### Connection with Aliases
```bash
# Connect with single alias
docker network connect --alias web-server my-network web

# Connect with multiple aliases
docker network connect \
  --alias web \
  --alias www \
  --alias frontend \
  my-network web

# Connect with IP and alias
docker network connect \
  --ip 192.168.1.100 \
  --alias web-server \
  my-network web
```

### Advanced Connection
```bash
# Connect with link (deprecated but still works)
docker network connect --link db:database my-network web

# Connect to multiple networks at container creation
docker run -d \
  --network frontend \
  --network backend \
  --name app \
  nginx
```

## Disconnecting Containers

### Basic Disconnection
```bash
# Disconnect container from network
docker network disconnect my-network web

# Force disconnect (even if container is running)
docker network disconnect -f my-network web

# Disconnect from multiple networks
docker network disconnect net1 web
docker network disconnect net2 web
```

## Removing Networks

### Basic Removal
```bash
# Remove single network
docker network rm my-network

# Remove multiple networks
docker network rm network1 network2 network3

# Remove all unused networks
docker network prune

# Remove with force (disconnect containers first)
docker network rm -f my-network
```

### Filtered Removal
```bash
# Prune with filter
docker network prune --filter "label=temporary"

# Prune without confirmation
docker network prune -f

# Remove all custom networks
docker network rm $(docker network ls --filter type=custom -q)

# Remove networks by label
docker network rm $(docker network ls --filter label=env=dev -q)
```

## Multi-Network Scenarios

### Frontend-Backend Separation
```bash
# Create networks
docker network create frontend
docker network create backend

# Run web server on frontend
docker run -d --network frontend --name web nginx

# Run app server on both networks
docker run -d \
  --network frontend \
  --name app \
  myapp
docker network connect backend app

# Run database on backend only
docker run -d --network backend --name db postgres
```

### Service Mesh Pattern
```bash
# Create networks for different tiers
docker network create dmz
docker network create app-tier
docker network create data-tier

# Load balancer in DMZ
docker run -d --network dmz --name lb nginx

# App servers in app-tier and DMZ
docker run -d --network app-tier --name app1 myapp
docker network connect dmz app1

# Database in data-tier only
docker run -d --network data-tier --name db postgres

# Connect app to database network
docker network connect data-tier app1
```

### Gateway Priority
```bash
# Set default gateway priority at creation
docker run -d \
  --network name=net1,gw-priority=1 \
  --network net2 \
  --name app \
  myapp

# Connect with gateway priority
docker network connect \
  --gateway-priority=100 \
  net3 \
  app
```

## Network Utilities

### Network Information
```bash
# Get network ID by name
docker network ls --filter name=^my-network$ -q

# Count networks
docker network ls -q | wc -l

# List network drivers
docker network ls --format '{{.Driver}}' | sort -u

# Find networks without containers
docker network ls --filter "dangling=true"
```

### Container Network Info
```bash
# Get container's networks
docker inspect web --format='{{json .NetworkSettings.Networks}}' | jq

# Get container's IP addresses
docker inspect web \
  --format='{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}'

# Get container's network names
docker inspect web \
  --format='{{range $net, $conf := .NetworkSettings.Networks}}{{$net}} {{end}}'

# Check if container is on network
docker network inspect my-network \
  --format='{{range .Containers}}{{if eq .Name "web"}}Connected{{end}}{{end}}'
```

### Troubleshooting Commands
```bash
# Test connectivity between containers
docker exec web ping -c3 db

# Check DNS resolution
docker exec web nslookup db
docker exec web dig db

# View routing table
docker exec web ip route

# View network interfaces
docker exec web ip addr
docker exec web ifconfig

# Test port connectivity
docker exec web nc -zv db 5432
docker exec web telnet db 5432

# Trace network path
docker exec web traceroute db

# Check network statistics
docker stats --format "table {{.Name}}\t{{.NetIO}}"
```

## Docker Compose Network Examples

### Basic Compose Network
```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
  
  app:
    image: myapp
    networks:
      - frontend
      - backend
  
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

### Advanced Compose Network
```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      frontend:
        ipv4_address: 172.28.0.10
        aliases:
          - web-server
  
  app:
    image: myapp
    networks:
      frontend:
      backend:
        priority: 1000

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
  
  backend:
    driver: bridge
    internal: true
    labels:
      env: production
      tier: backend
```

### External Network in Compose
```yaml
version: '3.8'

services:
  app:
    image: myapp
    networks:
      - existing-network

networks:
  existing-network:
    external: true
    name: my-existing-network
```