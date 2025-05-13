# AdGuard Home Helm Chart

This Helm chart deploys AdGuard Home, a network-wide DNS ad blocker, on a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster
- Helm
- ArgoCD (for GitOps deployment)
- A Raspberry Pi or other server with IP 192.168.1.2

## Configuration

### Fixed IP Setup

This chart is configured to deploy AdGuard Home with a fixed IP address (192.168.1.2). The configuration includes:

- LoadBalancer service type to expose the service with a fixed IP
- DNS configuration pointing to reliable upstream DNS servers
- Initial configuration with admin/admin credentials

### Deployment

To deploy this chart:

1. Enable the deployment by setting `enabled: true` in values.yaml
2. Apply the changes using ArgoCD or direct Helm install:

```bash
# Using Helm directly
helm install adguard-home ./apps-infra/charts/networking/adguard -n adguard --create-namespace

# Using kubectl with ArgoCD
kubectl apply -f apps-infra/charts/networking/adguard/templates/main.yaml
```

### Accessing AdGuard Home

Once deployed, you can access AdGuard Home at:

- Web interface: http://192.168.1.2
- Default login: admin/admin (change immediately)

### Setting as DNS Server

To use AdGuard Home as your DNS server:

1. Configure your router to use 192.168.1.2 as the primary DNS server
2. Or configure individual devices to use 192.168.1.2 as their DNS server

## Configuration Options

The following table lists the configurable parameters and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `enabled` | Enable the deployment | `true` |
| `namespace` | Namespace for the deployment | `adguard` |
| `persistence.size` | Size of persistent volume | `1Gi` |
| `persistence.nodeAffinity` | Node to deploy the volume to | `raspberrypi` |
| `service.main.loadBalancerIP` | Fixed IP for the service | `192.168.1.2` |
| `timezone` | Timezone | `UTC` |

## Troubleshooting

If AdGuard Home is not accessible after deployment:

1. Check if the pod is running: `kubectl get pods -n adguard`
2. Check service status: `kubectl get svc -n adguard`
3. Check logs: `kubectl logs -n adguard -l app.kubernetes.io/name=adguard-home`

## Upgrading

To upgrade the chart:

1. Update the values in values.yaml
2. Apply the changes using ArgoCD or Helm upgrade 