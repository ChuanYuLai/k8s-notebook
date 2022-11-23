## Firewall rules
GKE creates firewall rules automatically when creating the following resources:
- GKE clusters
- GKE Services
- GKE Ingresses

The priority for all automatically created firewall rules is 1000, which is the default value for firewall rules. If you would like more control over firewall behavior, you can create firewall rules with a higher priority. Firewall rules with a higher priority are applied before automatically created firewall rules.

## Source
- https://cloud.google.com/kubernetes-engine/docs/concepts/firewall-rules