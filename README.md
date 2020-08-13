### NodeLocal DNSCache improves Cluster DNS performance by running a dns caching agent on cluster nodes as a DaemonSet

Overview: NodeLocal DNSCache is an optional GKE add-on that you can run in addition to kube-dns. NodeLocal DNSCache is implemented as a DaemonSet that runs a DNS cache on each node in your cluster. When a Pod makes a DNS request, the request goes to the DNS cache running on the same node as the Pod. If the cache can't resolve the DNS request, the cache forwards the request to:

- Cloud DNS for external hostname queries. These queries are forwarded to Cloud DNS by the local MetaData Server running on the same node as the Pod the query originated from.
- kube-dns for all other DNS queries. The kube-dns-upstream service is used by node-local-dns Pods to reach out to kube-dns Pods.

![alt text](https://cloud.google.com/kubernetes-engine/images/nodelocal-dns-cache-diagram.svg)

### Benefits of NodeLocal DNSCache

- Reduced average DNS lookup time
- Connections from Pods to their local cache don't create conntrack table entries. This prevents dropped and rejected connections caused by conntrack table exhaustion and race conditions.


### Details
- NodeLocal DNSCache requires GKE version 1.15 or higher
- Connections between the local DNS cache and kube-dns use TCP instead of UDP for improved reliability
- DNS queries for external URLs (URLs that don't refer to cluster resources) are forwarded directly to the local Cloud DNS metadata server, bypassing kube-dns
- NodeLocal DNSCache Pods listen on port 53, 9253 and 8080 on the nodes. Running any other hostNetwork Pod using the above ports or configuring hostPorts with
- the above ports causes NodeLocal DNSCache to fail and result in DNS errors.

### Enabling NodeLocal DNSCache on existing cluster GKE
cli gcloud
```
gcloud container clusters update cluster-name \
  --update-addons=NodeLocalDNS=ENABLED
```
GUI Console

![alt text](https://i.imgur.com/nF5v4RO.png)


### Verifying that NodeLocal DNSCache is enabled
```
kubectl get pods -n kube-system -o wide | grep node-local-dns
```

### Troubleshooting NodeLocal DNSCache
- Validating Pod configuration
```
kubectl exec -it pod-name -- cat /etc/resolv.conf | grep nameserver
```

- The nameserver IP should match the IP address output by:
```
kubectl get svc -n kube-system kube-dns -o jsonpath="{.spec.clusterIP}"
```
