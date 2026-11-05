## Role of CoreDNS in Kubernetes

### Question  
What is the role of **CoreDNS** in a Kubernetes cluster? Why is it important?

### Short explanation of the question  
This question checks your understanding of **service discovery and DNS resolution** inside Kubernetes clusters — a key part of internal communication between services.

---

### Answer  
**CoreDNS** is the **default DNS server** used by Kubernetes to provide **service discovery**. It translates internal Kubernetes service names (like `my-service.default.svc.cluster.local`) into the corresponding Pod IPs or Cluster IPs, enabling communication between pods using DNS instead of hardcoded IP addresses.

---

### Detailed explanation of the answer for readers’ understanding

---

### 🌐 What is CoreDNS?

- A lightweight, extensible **DNS server** written in Go.
- Replaced **kube-dns** as the default DNS solution since Kubernetes v1.13.
- Deployed as a Kubernetes deployment in the `kube-system` namespace.

---

### 🧭 Why CoreDNS is critical?

In Kubernetes, services are accessed using DNS names like:

```
http://my-app.default.svc.cluster.local
```

Without CoreDNS:
- Pods wouldn’t be able to resolve service names.
- Inter-pod communication would break.
- Kubernetes' service discovery model would fail.

---

### 🔁 How CoreDNS Works

1. **Pod makes a DNS request** to resolve a service name.
2. The request is sent to the virtual IP `10.96.0.10` (default ClusterIP for CoreDNS).
3. CoreDNS uses Kubernetes API to resolve the DNS query.
4. It returns the appropriate Cluster IP or Pod IP (for headless services).

---

### 🔧 CoreDNS Configuration

The config lives in a **ConfigMap**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

---

### 💡 Common Use Cases

| Use Case                         | Example                                                                 |
|----------------------------------|-------------------------------------------------------------------------|
| **Service-to-service communication** | `curl http://orders.default.svc.cluster.local`                         |
| **StatefulSet communication**    | `mysql-0.my-db.default.svc.cluster.local`                              |
| **Pod discovery in custom DNS zones** | Extending CoreDNS with plugins for external name resolution            |

---

---

### Related
- [CoreDNS deep dive & troubleshooting](./38-coredns-deep-dive-and-troubleshooting.md) — the full resolution path (resolv.conf/kubelet vs kube-proxy), the `ndots:5` "3 errors behind a successful lookup", and crash/scale troubleshooting (OOM, conntrack, NodeLocal DNSCache).
