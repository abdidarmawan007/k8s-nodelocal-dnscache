### NodeLocal DNSCache improves Cluster DNS performance by running a dns caching agent on cluster nodes as a DaemonSet

Overview
NodeLocal DNSCache is an optional GKE add-on that you can run in addition to kube-dns. NodeLocal DNSCache is implemented as a DaemonSet that runs a DNS cache on each node in your cluster. When a Pod makes a DNS request, the request goes to the DNS cache running on the same node as the Pod. If the cache can't resolve the DNS request, the cache forwards the request to:

Cloud DNS for external hostname queries. These queries are forwarded to Cloud DNS by the local MetaData Server running on the same node as the Pod the query originated from.
kube-dns for all other DNS queries. The kube-dns-upstream service is used by node-local-dns Pods to reach out to kube-dns Pods.



