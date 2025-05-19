# WireGuard Helm Chart

This Helm chart deploys WireGuard VPN, a fast, modern, and secure VPN tunnel, on a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster
- Helm
- ArgoCD (for GitOps deployment)
- A Raspberry Pi or other server with IP 192.168.1.2
- MetalLB load balancer configured for the IP range

## Configuration

### Fixed IP Setup

This chart is configured to deploy WireGuard with a fixed IP address (192.168.1.2). The configuration includes:

- LoadBalancer service type to expose the service with a fixed IP
- UDP port 51820 exposed for WireGuard connections
- Persistent volume for configuration and connection data

### Deployment

To deploy this chart:

1. Enable the deployment by setting `enabled: true` in values.yaml
2. Apply the changes using ArgoCD or direct Helm install:

```bash
# Using Helm directly
helm install wireguard ./apps-infra/charts/networking/wireguard -n wireguard --create-namespace

# Using kubectl with ArgoCD
kubectl apply -f apps-infra/charts/networking/wireguard/templates/main.yaml
```

### Accessing WireGuard

Once deployed, you can access WireGuard using client applications:

- Server URL: 192.168.1.2
- Port: 51820 (UDP)
- Configuration files will be generated automatically, accessible from the persistent volume

### Client Setup

WireGuard clients are available for multiple platforms:

1. Desktop: Windows, macOS, Linux
2. Mobile: iOS, Android

For each client, import the configuration file from the server to establish a connection.

## Configuration Options

The following table lists the configurable parameters and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `enabled` | Enable the deployment | `false` |
| `namespace` | Namespace for the deployment | `wireguard` |
| `persistence.size` | Size of persistent volume | `1Gi` |
| `persistence.nodeAffinity` | Node to deploy the volume to | `raspberrypi` |
| `service.main.loadBalancerIP` | Fixed IP for the service | `192.168.1.2` |
| `env.INTERNAL_SUBNET` | Internal subnet for VPN clients | `10.13.13.0/24` |
| `env.ALLOWEDIPS` | Networks routed through VPN | `0.0.0.0/0` |

## Security Considerations

This deployment requires privileged mode due to the nature of VPN services. It specifically needs:

- NET_ADMIN capability
- SYS_MODULE capability
- Privileged container execution

## Troubleshooting

If WireGuard is not accessible after deployment:

1. Check if the pod is running: `kubectl get pods -n wireguard`
2. Check service status: `kubectl get svc -n wireguard`
3. Check logs: `kubectl logs -n wireguard -l app.kubernetes.io/name=wireguard`

## Upgrading

To upgrade the chart:

1. Update the values in values.yaml
2. Apply the changes using ArgoCD or Helm upgrade 