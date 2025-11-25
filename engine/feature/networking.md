# Networking

Docker networking enables containers to connect and communicate with each other, external networks, and non-Docker services.

## Network Types

### Bridge Network
- **Default Network**: Containers use bridge network by default
- **Isolated Environment**: Containers on same bridge can communicate
- **NAT-based**: Network Address Translation for external access
- **Private Subnet**: Each bridge has its own subnet (e.g., 172.17.0.0/16)
- **Port Publishing**: Expose container ports to host
- **DNS Resolution**: Built-in DNS for container name resolution
- **Best For**: Single-host container communication

### Host Network
- **No Isolation**: Container shares host's network stack
- **Direct Access**: Container uses host's IP and ports
- **Performance**: Best network performance (no NAT overhead)
- **Port Conflicts**: Container ports must not conflict with host
- **Limited Isolation**: No network security isolation
- **Linux Only**: Not available on Docker Desktop for Mac/Windows
- **Best For**: High-performance network applications

### Overlay Network
- **Multi-Host**: Connects containers across multiple Docker hosts
- **Swarm Mode**: Requires Docker Swarm initialization
- **Encryption**: Optional IPsec encryption between nodes
- **Service Discovery**: Built-in DNS-based service discovery
- **Load Balancing**: Ingress load balancing for services
- **VXLAN-based**: Uses VXLAN tunneling technology
- **Best For**: Multi-host container orchestration

### None Network
- **Complete Isolation**: No network access at all
- **Loopback Only**: Only localhost (127.0.0.1) available
- **No External Access**: Cannot reach internet or other containers
- **Custom Networking**: Used when implementing custom network stack
- **Best For**: Maximum isolation, custom network configuration

### MACVLAN Network
- **Physical Network Integration**: Container appears as physical device on network
- **MAC Address**: Each container gets unique MAC address
- **Direct Routing**: No port mapping required
- **VLAN Support**: Can connect to VLANs
- **Performance**: Near-native network performance
- **Promiscuous Mode**: Requires promiscuous mode on host interface
- **Best For**: Legacy applications expecting physical network presence

### IPVLAN Network
- **IP-based**: Shares parent interface's MAC address
- **Modes**: L2 (layer 2) and L3 (layer 3)
- **No Promiscuous Mode**: Doesn't require promiscuous mode
- **VLAN Support**: Can connect to VLANs like MACVLAN
- **Better Performance**: More efficient than MACVLAN
- **Best For**: Cloud environments, high-density deployments

## Network Drivers

### Built-in Drivers
- **bridge**: Default driver for single-host networking
- **host**: Remove network isolation, use host network stack
- **overlay**: Multi-host networking for Swarm
- **macvlan**: Assign MAC addresses to containers
- **ipvlan**: Assign IP addresses without unique MAC
- **none**: Disable all networking

### Third-Party Drivers
- **Weave**: Mesh networking with automatic discovery
- **Calico**: Policy-based networking and security
- **Flannel**: Simple overlay network for Kubernetes
- **Cilium**: eBPF-based networking and security

## Port Publishing
- **Publishing Ports**: Make container services accessible from outside
- **Host Port Binding**: Map container port to host port
- **Random Ports**: Docker can assign random available ports
- **Multiple Ports**: Publish multiple ports simultaneously
- **Protocol Support**: TCP and UDP protocols

## IP Address Management
- **DHCP-like**: Docker assigns IPs from network subnet
- **Subnet Pool**: Each network has its own subnet
- **Gateway**: Automatic gateway assignment
- **DNS**: Built-in DNS server at 127.0.0.11
- **IP Reuse**: IPs can be reused after container removal

## DNS and Service Discovery
- **Embedded DNS Server**: Docker provides DNS at 127.0.0.11
- **Container Name Resolution**: Resolve containers by name
- **Service Discovery**: Automatic discovery within same network
- **External DNS**: Forward external queries to host DNS
- **DNS Round-Robin**: Load balancing via DNS for replicas

## Multi-Network Containers
- **Multiple Interfaces**: Container can have multiple network interfaces
- **Default Gateway**: First network is default gateway
- **Gateway Priority**: Set priority for default gateway selection
- **Network Segmentation**: Separate frontend and backend networks

## Network Security

### Network Isolation
- **Default Bridge**: Containers can access each other by IP
- **User-Defined Bridge**: Better isolation with name-based access
- **Internal Networks**: No external access
- **Network Segmentation**: Separate sensitive services

### Network Policies
- **Firewall Rules**: Use iptables/nftables for filtering
- **Network Plugins**: Calico, Cilium for advanced policies
- **Encryption**: Enable overlay network encryption
- **Access Control**: Limit inter-container communication

## Network Performance

### Performance Considerations
- **Host Network**: Best performance (no NAT)
- **Bridge Network**: Slight overhead from NAT
- **Overlay Network**: Higher latency (VXLAN encapsulation)
- **MACVLAN/IPVLAN**: Near-native performance

### Optimization Tips
- **Disable Userland Proxy**: Use `"userland-proxy": false` in daemon.json
- **Use Host Network**: When maximum performance needed
- **MTU Settings**: Adjust MTU for network environment
- **Network Driver**: Choose appropriate driver for workload

### Network Metrics
- **Throughput**: Bytes sent/received
- **Packet Loss**: Dropped packets
- **Latency**: Round-trip time
- **Connections**: Active connections count
- **Errors**: Network errors and failures

## Best Practices

### Network Design
- **Use User-Defined Networks**: Better than default bridge
- **Network Segmentation**: Separate frontend, backend, data layers
- **Name-Based Access**: Use container names instead of IPs
- **Internal Networks**: For services not needing external access
- **Overlay for Multi-Host**: Use overlay networks in Swarm mode

### Security
- **Principle of Least Privilege**: Only expose necessary ports
- **Internal Networks**: Use for sensitive services
- **Network Encryption**: Enable for overlay networks
- **Firewall Rules**: Configure host firewall appropriately
- **Regular Audits**: Review network configurations

### Performance
- **Choose Right Driver**: Match driver to use case
- **Disable Userland Proxy**: For better performance
- **Monitor Network Usage**: Track bandwidth consumption
- **Optimize MTU**: Adjust for network environment
- **Use Host Network**: When maximum performance critical

### Management
- **Consistent Naming**: Use clear network names
- **Documentation**: Document network architecture
- **Regular Cleanup**: Remove unused networks
- **Label Networks**: Use labels for organization
- **Version Control**: Track network configurations in code