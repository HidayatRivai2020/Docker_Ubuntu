# network

## Overview
- Manages Docker networks for container communication
- Creates, inspects, lists, and removes networks
- Connects and disconnects containers to/from networks
- Provides isolation and connectivity between containers
- Supports multiple network drivers (bridge, host, overlay, macvlan, ipvlan)
- Enables service discovery and DNS resolution

## Basic Syntax
```bash
docker network [COMMAND] [OPTIONS]
```

## Subcommands

### Network Lifecycle
- `create` - Create a new network
- `ls, list` - List all networks
- `inspect` - Display detailed network information
- `rm, remove` - Remove one or more networks
- `prune` - Remove all unused networks

### Container Connection
- `connect` - Connect a container to a network
- `disconnect` - Disconnect a container from a network

## Common Options

### Create Options
- `-d, --driver` - Driver to manage the network (bridge, overlay, macvlan, etc.)
- `--subnet` - Subnet in CIDR format (e.g., 172.28.0.0/16)
- `--gateway` - IPv4 or IPv6 gateway for the network
- `--ip-range` - Allocate container IP from a sub-range
- `--ipv6` - Enable IPv6 networking
- `--internal` - Restrict external access to the network
- `--attachable` - Enable manual container attachment (overlay networks)
- `--label` - Set metadata on the network
- `-o, --opt` - Set driver specific options

### Connect Options
- `--ip` - IPv4 address for the container
- `--ip6` - IPv6 address for the container
- `--alias` - Add network-scoped alias for the container
- `--link` - Add link to another container (deprecated)

### List Options
- `-q, --quiet` - Only display network IDs
- `-f, --filter` - Filter output based on conditions
- `--format` - Format output using Go template
- `--no-trunc` - Don't truncate output

### Inspect Options
- `-f, --format` - Format output using Go template
- `-v, --verbose` - Verbose output

## How It Works

### Network Creation Process
1. **Driver Selection** - Choose appropriate network driver
2. **Subnet Allocation** - Assign IP range from address pool
3. **Gateway Setup** - Configure default gateway
4. **Network Namespace** - Create isolated network namespace
5. **DNS Configuration** - Set up embedded DNS server

### Network Connection Flow
```
Container Created → Connect to Network → Assign IP Address
                                               ↓
                                    Update DNS Records
                                               ↓
                                    Configure Routing
                                               ↓
                                    Enable Communication
```

### Network Drivers

#### Bridge (Default)
- **Purpose**: Single-host container networking
- **Isolation**: Network namespace isolation
- **DNS**: Automatic DNS resolution by container name
- **Use Case**: Most common for local development

#### Host
- **Purpose**: Remove network isolation
- **Performance**: Best network performance
- **Ports**: Container uses host's ports directly
- **Use Case**: High-performance applications

#### Overlay
- **Purpose**: Multi-host container networking
- **Requirements**: Docker Swarm mode
- **Encryption**: Optional IPsec encryption
- **Use Case**: Distributed applications across multiple hosts

#### MACVLAN
- **Purpose**: Assign MAC addresses to containers
- **Integration**: Containers appear as physical devices
- **Requirements**: Promiscuous mode on host interface
- **Use Case**: Legacy applications requiring layer 2 connectivity

#### IPVLAN
- **Purpose**: IP-based connectivity without unique MAC
- **Modes**: L2 (layer 2) and L3 (layer 3)
- **Advantage**: No promiscuous mode required
- **Use Case**: Cloud environments, high-density deployments

#### None
- **Purpose**: Complete network isolation
- **Access**: Only loopback interface
- **Use Case**: Maximum security isolation

## Best Practices

### Network Design
- **Use user-defined networks** instead of default bridge for better isolation
- **Create separate networks** for different application tiers (frontend, backend, database)
- **Use meaningful network names** that reflect their purpose
- **Document network architecture** for team understanding
- **Plan IP addressing** to avoid conflicts with existing infrastructure

### Security
- **Use internal networks** for services not needing external access
- **Enable encryption** for overlay networks in production
- **Implement network segmentation** to limit blast radius
- **Restrict port exposure** to minimum required
- **Regular audits** of network configurations

### Performance
- **Choose appropriate driver** for workload requirements
- **Disable userland proxy** in daemon.json for better performance
- **Use host network** when maximum performance is critical
- **Monitor network metrics** to identify bottlenecks
- **Optimize MTU settings** for network environment

### Management
- **Label networks** for better organization and filtering
- **Regular cleanup** of unused networks with prune
- **Use Docker Compose** for complex multi-network setups
- **Version control** network configurations
- **Implement naming conventions** across environments