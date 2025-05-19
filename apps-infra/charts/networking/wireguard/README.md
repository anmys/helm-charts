# WireGuard Helm Chart

This Helm chart deploys a WireGuard VPN server on Kubernetes, using the linuxserver/wireguard container.

## Prerequisites

- Kubernetes cluster
- Helm 3
- A persistent volume for storing WireGuard configuration

## Installation

```bash
# From the helm-charts directory
helm upgrade --install wireguard ./apps-infra/charts/networking/wireguard \
  --namespace wireguard \
  --create-namespace \
  --set enabled=true
```

## Configuration

The main configuration is handled through the `values.yaml` file. Key configurations:

- `enabled`: Set to `true` to deploy the WireGuard server
- `replicaCount`: Number of replicas (should be 1 for WireGuard)
- `persistence.enabled`: Enable persistent storage
- `persistence.size`: Size of the persistent volume claim
- `service.type`: Service type (LoadBalancer, NodePort, etc.)
- `service.port`: Port for the WireGuard service (default: 51820)

## Initial Setup

The chart includes initialization jobs that:

1. Set up the required directories and permissions
2. Create initial empty configurations
3. Fix permissions for the WireGuard executables

After installation, WireGuard will generate configuration files for the server and any defined peers.

## Helper Script

A helper script is included to assist with common WireGuard operations:

```bash
# Make the script executable if needed
chmod +x ./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh

# Set the namespace if not using default
export NAMESPACE=wireguard

# Show status
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh status

# Get logs
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh logs

# Fix permissions (if needed)
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh fix-perms

# Restart WireGuard
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh restart

# Get QR code for a peer
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh qr peer1
```

## Troubleshooting

### Permission Denied Errors

If you see permission denied errors in the logs, you can run:

```bash
./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh fix-perms
```

This will fix permissions on:
- `/usr/bin/readlink`
- `/usr/bin/wg`
- `/usr/bin/wg-quick`
- `/etc/wireguard` directory
- `/config` directory

### Network/DNS Issues

The chart configures the WireGuard pod with:
- `hostNetwork: true` for direct network access
- `dnsPolicy: "ClusterFirstWithHostNet"` for enhanced DNS resolution
- A default CoreDNS configuration that forwards to Google DNS (8.8.8.8, 8.8.4.4)

### WireGuard Not Starting

If WireGuard fails to start, check:

1. Logs: `./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh logs`
2. Permissions: Run the fix-perms command
3. Restart the service: `./apps-infra/charts/networking/wireguard/scripts/wireguard-helper.sh restart`

## Architecture

This chart includes:

1. **Main Deployment**: The WireGuard server container
2. **Init Job**: Creates necessary directories and configurations
3. **Post-Install Hook**: Fixes permissions after deployment
4. **Service**: Exposes the WireGuard port
5. **PVC**: Persistent volume for storing configurations

## Notes

- The WireGuard container runs as root (user 0) for necessary permissions
- Additional capabilities (NET_ADMIN, SYS_MODULE, SYS_ADMIN) are granted for network management
- The container has `hostNetwork: true` for direct network access 