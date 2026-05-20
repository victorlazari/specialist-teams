# Kubernetes and EKS CLI Reference: Advanced Operations and Tech Support Guide

## 1. Introduction

In the high-stakes environment of production Kubernetes and Amazon Elastic Kubernetes Service (EKS), technical support engineers and site reliability engineers (SREs) require rapid, precise, and powerful tools to diagnose and remediate complex issues. This comprehensive CLI reference guide is designed specifically for tech support operations, focusing on advanced usage, worst-case scenarios, and practical one-liners. It covers the essential toolchain: `kubectl`, `eksctl`, `aws eks`, `helm`, and `k9s`.

This document serves as a definitive operational playbook. It moves beyond basic commands to provide deep diagnostic capabilities, state recovery procedures, and performance profiling techniques. Whether you are dealing with a widespread `NodeNotReady` incident, a subtle DNS resolution failure, or a complex Helm release rollback, this guide provides the exact commands needed to restore service stability.

---

## 2. `kubectl` Advanced Operations & One-Liners

The `kubectl` command-line tool is the primary interface for interacting with the Kubernetes API. For tech support, mastering `kubectl` means understanding how to extract precise state information, manipulate resources directly, and bypass standard abstractions when necessary.

### 2.1 Cluster & Node Diagnostics

When cluster stability is compromised, the first step is to assess the health of the underlying compute resources.

**Identify Nodes with High Resource Pressure:**
```bash
kubectl get nodes -o custom-columns="NAME:.metadata.name,CPU_ALLOCATABLE:.status.allocatable.cpu,MEMORY_ALLOCATABLE:.status.allocatable.memory,STATUS:.status.conditions[?(@.type=='Ready')].status"
```

**Find Nodes that are NotReady and Extract the Reason:**
```bash
kubectl get nodes --field-selector=status.phase!=Running -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].message}{"\n"}{end}'
```

**Drain a Node Aggressively (Ignoring DaemonSets and Local Data):**
In emergency situations where a node is failing and pods must be evicted immediately:
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data --force --grace-period=0
```

**Check Kubelet Logs via Node Shell (Requires Privileged Access):**
When SSH access is unavailable, you can spawn a privileged pod on the node to inspect system logs.
```bash
kubectl debug node/<node-name> -it --image=ubuntu -- chroot /host journalctl -u kubelet -f
```

### 2.2 Pod & Container Troubleshooting

Pod failures are the most common support tickets. Rapidly identifying the root cause—whether it's an OOMKill, a readiness probe failure, or a crash loop—is critical.

**List All Pods in CrashLoopBackOff or Error State Across All Namespaces:**
```bash
kubectl get pods -A --field-selector=status.phase!=Running | grep -v 'Completed'
```

**Extract the Exit Code and Reason for the Last Terminated Container:**
This is essential for diagnosing OOMKilled (Exit Code 137) or segmentation faults.
```bash
kubectl get pod <pod-name> -n <namespace> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason} - Exit Code: {.status.containerStatuses[0].lastState.terminated.exitCode}{"\n"}'
```

**Stream Logs from All Containers in a Deployment:**
```bash
kubectl logs -f deployment/<deployment-name> --all-containers=true --max-log-requests=10
```

**Execute a Network Troubleshooting Container in an Existing Pod's Network Namespace:**
If a pod lacks debugging tools (e.g., `curl`, `dig`, `ping`), attach an ephemeral debug container.
```bash
kubectl debug -it <pod-name> --image=nicolaka/netshoot --target=<container-name>
```

### 2.3 Networking & DNS Debugging

Networking issues in Kubernetes often manifest as intermittent timeouts or DNS resolution failures.

**Test DNS Resolution from within the Cluster:**
```bash
kubectl run -i --tty --rm debug-dns --image=busybox:1.28 --restart=Never -- nslookup kubernetes.default.svc.cluster.local
```

**Check CoreDNS Logs for Errors:**
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns -c coredns --tail=100 | grep -i error
```

**List All Services Without Endpoints (Orphaned Services):**
A service without endpoints will drop traffic. This command identifies them.
```bash
kubectl get endpoints -A | awk '$3 == "<none>" {print $1, $2}'
```

**Capture Packet Trace on a Specific Pod (Requires Netshoot):**
```bash
kubectl exec -it <pod-name> -- tcpdump -i eth0 -nn -s0 -w /tmp/capture.pcap
```

### 2.4 Resource & Performance Profiling

Resource exhaustion leads to unpredictable behavior. Monitoring CPU and memory requests versus actual usage is a core support task.

**Sort Pods by Memory Usage (Requires Metrics Server):**
```bash
kubectl top pods -A --sort-by=memory
```

**Identify Pods Without Resource Limits Configured:**
```bash
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.containers[*].resources.limits}{"\n"}{end}' | grep -v "cpu"
```

### 2.5 RBAC & Security Auditing

Permissions issues often block deployments or operational tasks.

**Check if You (or a ServiceAccount) Can Perform an Action:**
```bash
kubectl auth can-i create deployments --namespace dev --as system:serviceaccount:dev:ci-cd
```

**List All ClusterRoleBindings Granting Cluster-Admin:**
```bash
kubectl get clusterrolebindings -o jsonpath='{range .items[?(@.roleRef.name=="cluster-admin")]}{.metadata.name}{"\t"}{.subjects[*].name}{"\n"}{end}'
```

---

## 3. `eksctl` Production Management

`eksctl` is the official CLI for Amazon EKS. In a support context, it is primarily used for cluster lifecycle management, node group scaling, and IAM integration.

### 3.1 Cluster Upgrades & Maintenance

Upgrading an EKS cluster requires careful orchestration of the control plane and data plane.

**Check Cluster Upgrade Readiness:**
```bash
eksctl utils update-cluster-logging --cluster <cluster-name> --region <region> --enable-types all
```

**Upgrade the Control Plane to a Specific Version:**
```bash
eksctl upgrade cluster --name <cluster-name> --version 1.28 --approve
```

### 3.2 Node Group Management

When nodes fail or require patching, node group management is necessary.

**Scale a Managed Node Group:**
```bash
eksctl scale nodegroup --cluster <cluster-name> --name <nodegroup-name> --nodes 5 --nodes-min 3 --nodes-max 10
```

**Drain and Delete a Node Group (Graceful Decommissioning):**
```bash
eksctl delete nodegroup --cluster <cluster-name> --name <old-nodegroup> --drain=true
```

### 3.3 IAM OIDC & Service Accounts

IRSA (IAM Roles for Service Accounts) is the standard for granting AWS permissions to pods. Misconfigurations here are a frequent source of support tickets.

**Create an IAM Role and Bind it to a Kubernetes ServiceAccount:**
```bash
eksctl create iamserviceaccount   --cluster=<cluster-name>   --namespace=<namespace>   --name=<service-account-name>   --attach-policy-arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess   --approve   --override-existing-serviceaccounts
```

**Verify the OIDC Provider is Associated with the Cluster:**
```bash
eksctl utils associate-iam-oidc-provider --cluster <cluster-name> --approve
```

---

## 4. `aws eks` CLI for Control Plane

The AWS CLI (`aws eks`) is used for interacting with the AWS API regarding the EKS control plane infrastructure.

### 4.1 Cluster Authentication & Kubeconfig

**Generate or Update Kubeconfig for an EKS Cluster:**
```bash
aws eks update-kubeconfig --region <region> --name <cluster-name> --alias <custom-alias>
```

**Assume a Role Before Updating Kubeconfig (Cross-Account Access):**
```bash
aws eks update-kubeconfig --region <region> --name <cluster-name> --role-arn arn:aws:iam::<account-id>:role/<role-name>
```

### 4.2 Add-on Management

EKS Add-ons (VPC CNI, CoreDNS, kube-proxy) must be kept up to date.

**List Installed Add-ons and Their Status:**
```bash
aws eks list-addons --cluster-name <cluster-name> --region <region>
```

**Describe a Specific Add-on to Check for Degradation:**
```bash
aws eks describe-addon --cluster-name <cluster-name> --addon-name vpc-cni --region <region> --query 'addon.health.issues'
```

### 4.3 Control Plane Logging & Diagnostics

When the API server is unresponsive, control plane logs in CloudWatch are the only source of truth.

**Enable All Control Plane Logs (API, Audit, Authenticator, ControllerManager, Scheduler):**
```bash
aws eks update-cluster-config     --region <region>     --name <cluster-name>     --logging '{"clusterLogging":[{"types":["api","audit","authenticator","controllerManager","scheduler"],"enable":true}]}'
```

---

## 5. `helm` Advanced Release Management

Helm is the package manager for Kubernetes. Support operations often involve untangling failed deployments, recovering from stuck states, and verifying chart integrity.

### 5.1 Release Debugging & Rollbacks

**List All Failed or Pending Helm Releases Across All Namespaces:**
```bash
helm ls --all-namespaces -a | grep -E 'FAILED|pending'
```

**Rollback a Release to the Previous Successful Revision:**
```bash
helm rollback <release-name> 0 -n <namespace> --wait --timeout 10m
```

**Force a Rollback (Bypassing Hooks):**
If a pre-rollback hook is failing and blocking recovery:
```bash
helm rollback <release-name> <revision-number> -n <namespace> --no-hooks --force
```

### 5.2 Chart Verification & Templating

Before applying a chart, it is crucial to verify the exact manifests that will be generated.

**Render Chart Templates Locally for Inspection:**
```bash
helm template <release-name> <chart-path> -n <namespace> -f values-production.yaml > rendered-manifests.yaml
```

**Diff a Helm Upgrade Before Applying (Requires helm-diff plugin):**
```bash
helm diff upgrade <release-name> <chart-path> -n <namespace> -f values-production.yaml
```

### 5.3 State Recovery

Sometimes the Helm release secret becomes corrupted or out of sync with the actual cluster state.

**Get the Decoded Helm Release Secret (Advanced Debugging):**
```bash
kubectl get secret -n <namespace> sh.helm.release.v1.<release-name>.v1 -o jsonpath="{.data.release}" | base64 -d | base64 -d | gzip -d
```

---

## 6. `k9s` Power User Guide

`k9s` is a terminal-based UI to interact with your Kubernetes clusters. For tech support, it drastically reduces the time required to navigate resources, view logs, and execute commands.

### 6.1 Keyboard Shortcuts for Rapid Triage

- `:` : Enter command mode (e.g., `:pods`, `:deploy`, `:svc`).
- `/` : Filter the current view by regex.
- `l` : View logs for the selected pod or container.
- `w` : Toggle line wrap in log view.
- `s` : Open a shell inside the selected pod.
- `d` : Describe the selected resource.
- `e` : Edit the resource YAML directly.
- `ctrl-d` : Delete the selected resource.
- `shift-f` : Setup port-forwarding for the selected pod or service.

### 6.2 Custom Resource Views

`k9s` excels at managing Custom Resource Definitions (CRDs).

- Type `:crd` to view all Custom Resource Definitions.
- Select a CRD and press `Enter` to view all instances of that custom resource.
- This is invaluable for debugging operators (e.g., Prometheus Operator, Cert-Manager).

### 6.3 Plugins & Extensibility

`k9s` can be extended with custom plugins defined in `~/.config/k9s/plugins.yaml`.

**Example Plugin: Decode Secret**
This plugin allows you to press `x` on a Secret to view its decoded contents.
```yaml
plugins:
  decode-secret:
    shortCut: x
    confirm: false
    description: "Decode Secret"
    scopes:
      - secrets
    command: sh
    background: false
    args:
      - -c
      - "kubectl get secret $NAME -n $NAMESPACE -o json | jq '.data | map_values(@base64d)' | less"
```

---

## 7. Worst-Case Scenarios & Incident Response

Tech support operations are defined by how they handle worst-case scenarios. Below are playbooks for critical incidents.

### 7.1 API Server Unresponsive

**Symptoms:** `kubectl` commands time out; nodes transition to `NotReady`; controllers stop functioning.
**EKS Context:** In EKS, the control plane is managed by AWS.
**Response:**
1. Verify network connectivity to the EKS endpoint.
2. Check AWS Service Health Dashboard for regional EKS outages.
3. Review CloudWatch metrics for the EKS control plane (if enabled).
4. Ensure the VPC and subnets associated with the cluster have available IP addresses.
5. If the issue persists, escalate to AWS Support immediately, as control plane recovery is their responsibility.

### 7.2 Widespread Node NotReady

**Symptoms:** Multiple nodes simultaneously report `NotReady` status. Pods are evicted or stuck in `Terminating`.
**Response:**
1. **Check Kubelet Status:** Use SSM Session Manager or SSH to access a failing node. Run `systemctl status kubelet` and `journalctl -u kubelet`.
2. **Verify VPC CNI:** If the VPC CNI plugin fails, nodes cannot assign IPs to pods, leading to readiness failures. Check the `aws-node` daemonset logs in the `kube-system` namespace.
3. **Check Disk Pressure:** Nodes will become `NotReady` if they run out of disk space. Run `df -h` on the node.
4. **Review Auto Scaling Group (ASG):** Check the AWS console for ASG health checks or EC2 instance status checks failing.

### 7.3 CoreDNS CrashLoopBackOff

**Symptoms:** Internal service discovery fails. Pods cannot resolve `kubernetes.default.svc` or external domains.
**Response:**
1. **Check CoreDNS Logs:** `kubectl logs -n kube-system -l k8s-app=kube-dns`. Look for configuration errors or connection refused messages.
2. **Verify Node Connectivity:** CoreDNS pods must be able to reach the API server. Check security groups and network ACLs.
3. **Restart CoreDNS:** `kubectl rollout restart deployment coredns -n kube-system`.
4. **Check Resource Limits:** Ensure CoreDNS is not being OOMKilled. Increase memory limits if necessary.

### 7.4 Exhaustion of VPC IP Addresses

**Symptoms:** Pods remain in `ContainerCreating` state. VPC CNI logs show `Failed to allocate IP address`.
**Response:**
1. **Check Subnet IP Availability:** Use the AWS CLI to check available IPs in the cluster subnets.
   ```bash
   aws ec2 describe-subnets --subnet-ids <subnet-ids> --query 'Subnets[*].[SubnetId, AvailableIpAddressCount]' --output table
   ```
2. **Enable Custom Networking:** If the primary subnets are exhausted, configure the VPC CNI to use secondary subnets for pod IPs.
3. **Reduce WARM_IP_TARGET:** Adjust the VPC CNI environment variables to hold fewer unused IPs in reserve.

---

## 8. Advanced Troubleshooting Scenarios

### 8.1 Persistent Volume (PV) and Persistent Volume Claim (PVC) Issues

Storage issues can cause pods to remain in a `Pending` state indefinitely.

**Identify Unbound PVCs:**
```bash
kubectl get pvc -A | grep -v Bound
```

**Check Events for a Specific PVC:**
```bash
kubectl describe pvc <pvc-name> -n <namespace> | grep -A 10 Events
```

**Force Delete a Stuck PV:**
Sometimes a PV gets stuck in a `Terminating` state due to finalizers.
```bash
kubectl patch pv <pv-name> -p '{"metadata":{"finalizers":null}}'
```

### 8.2 Ingress and Load Balancer Debugging

When external traffic fails to reach your services, the issue often lies with the Ingress controller or the cloud provider's load balancer.

**Check AWS Load Balancer Controller Logs:**
```bash
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

**Verify Ingress Resource Configuration:**
```bash
kubectl describe ingress <ingress-name> -n <namespace>
```

**Test Internal Service Reachability from the Ingress Controller Pod:**
```bash
kubectl exec -it <ingress-controller-pod> -n <ingress-namespace> -- curl -v http://<service-name>.<service-namespace>.svc.cluster.local:<port>
```

### 8.3 Certificate and Secret Management

Expired certificates or misconfigured secrets can cause widespread outages.

**Check Certificate Expiration (Requires cert-manager):**
```bash
kubectl get certificates -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,READY:.status.conditions[?(@.type=='Ready')].status,EXPIRATION:.status.notAfter"
```

**Decode All Secrets in a Namespace (Use with Caution):**
```bash
kubectl get secrets -n <namespace> -o json | jq '.items[] | {name: .metadata.name, data: .data | map_values(@base64d)}'
```

---

## 9. Conclusion

Effective Kubernetes and EKS tech support requires a deep understanding of the underlying architecture and the ability to wield CLI tools with precision. This reference guide provides the foundational commands and advanced techniques necessary to diagnose, mitigate, and resolve complex production incidents. By mastering `kubectl`, `eksctl`, `aws eks`, `helm`, and `k9s`, support engineers can ensure the reliability, security, and performance of mission-critical containerized workloads.

Regularly practice these commands in a non-production environment to build muscle memory. In the heat of an incident, the ability to rapidly execute the correct diagnostic one-liner is the difference between a minor blip and a major outage.
