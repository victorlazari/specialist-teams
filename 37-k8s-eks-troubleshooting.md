# Kubernetes and EKS Deep Troubleshooting Guide: Production Operations & Tech Support

## 1. Introduction

In modern cloud-native architectures, Amazon Elastic Kubernetes Service (EKS) and Kubernetes serve as the backbone for mission-critical applications. However, the complexity of distributed systems introduces a myriad of failure modes that can disrupt production environments. This comprehensive guide is designed for tech support specialists, Site Reliability Engineers (SREs), and platform operators who are tasked with diagnosing and resolving the most severe and elusive issues in Kubernetes and EKS clusters. 

This document goes beyond basic `kubectl get pods` commands. It delves into the deep technical underpinnings of Kubernetes components, the AWS infrastructure that supports EKS, and the intricate interactions between them. We will explore worst-case scenarios, advanced diagnostic techniques, and battle-tested remediation strategies for CrashLoopBackOff, OOMKilled events, DNS resolution failures, Container Network Interface (CNI) problems, Node NotReady states, and IAM Roles for Service Accounts (IRSA) failures.

## 2. CrashLoopBackOff: Beyond the Basics

The `CrashLoopBackOff` state is one of the most common yet frustrating issues encountered in Kubernetes. It indicates that a pod is repeatedly crashing immediately after starting, and the kubelet is applying an exponential backoff delay before attempting to restart it. While the symptom is uniform, the root causes are highly diverse.

### 2.1. Diagnostic Workflow

When confronted with a `CrashLoopBackOff`, the initial step is to inspect the pod's logs and state. However, in production, logs might be missing if the container crashes too quickly.

1. **Inspect Previous Container Logs:**
   Use the `-p` (previous) flag to view the logs of the crashed container instance.
   ```bash
   kubectl logs <pod-name> -c <container-name> -p
   ```

2. **Analyze Pod Events:**
   The events associated with the pod often reveal issues related to scheduling, image pulling, or probe failures.
   ```bash
   kubectl describe pod <pod-name>
   ```

3. **Examine Exit Codes:**
   The exit code of the terminated container provides critical clues.
   - **Exit Code 1:** Application error (e.g., unhandled exception, missing configuration).
   - **Exit Code 137:** OOMKilled (Out of Memory) or forced termination via SIGKILL.
   - **Exit Code 143:** Graceful termination via SIGTERM.
   - **Exit Code 255:** Out of bounds exit code, often related to entrypoint script failures.

### 2.2. Common Root Causes and Deep Dives

#### 2.2.1. Misconfigured Liveness and Readiness Probes
Probes that are too aggressive or incorrectly configured can force the kubelet to repeatedly kill a perfectly healthy application.
- **Symptom:** The application starts, but the liveness probe fails before the application is fully initialized.
- **Deep Dive:** Analyze the `initialDelaySeconds`, `periodSeconds`, and `timeoutSeconds`. In slow-starting applications (e.g., legacy Java monoliths), the `initialDelaySeconds` must be sufficient. Alternatively, implement a `startupProbe` to defer liveness checks until the application has successfully started.

#### 2.2.2. Missing Dependencies and Configuration
Applications often crash if they cannot connect to a database, cache, or external API, or if required ConfigMaps/Secrets are missing.
- **Symptom:** Logs indicate connection timeouts or missing environment variables.
- **Deep Dive:** Verify the existence and contents of ConfigMaps and Secrets. Ensure that network policies or security groups are not blocking outbound traffic to required dependencies. Use ephemeral debug containers to test connectivity from within the pod's network namespace.
  ```bash
  kubectl debug -it <pod-name> --image=busybox:1.28 --target=<container-name>
  ```

#### 2.2.3. Entrypoint and Command Failures
Issues with the Dockerfile's `ENTRYPOINT` or `CMD`, or shell script syntax errors.
- **Symptom:** The container exits immediately with code 1 or 127 (command not found).
- **Deep Dive:** Override the command to keep the container alive for debugging.
  ```yaml
  command: ["sleep", "3600"]
  ```
  Once running, `exec` into the container and manually run the entrypoint script to observe the failure in real-time.

## 3. OOMKilled: Memory Management and Deep Diagnostics

An `OOMKilled` (Out of Memory) event occurs when a container exceeds its allocated memory limit, prompting the Linux kernel's OOM killer to terminate the process to protect the node's stability.

### 3.1. Understanding Kubernetes Memory Limits vs. Requests

- **Requests:** The guaranteed amount of memory allocated to the container. Used by the scheduler to place the pod on a suitable node.
- **Limits:** The absolute maximum amount of memory the container is allowed to use. Exceeding this triggers the OOM killer.

### 3.2. Diagnostic Workflow

1. **Identify OOMKilled Pods:**
   ```bash
   kubectl get pods -A | grep OOMKilled
   ```

2. **Verify the Exit Code:**
   Check the pod description for `Reason: OOMKilled` and `Exit Code: 137`.

3. **Analyze Node-Level OOM Events:**
   Sometimes, the entire node experiences memory pressure, leading to the eviction of pods. Check the node's kernel logs (dmesg) for OOM killer invocations.
   ```bash
   dmesg -T | grep -i oom
   ```

### 3.3. Deep Dive: Java and Memory Limits

Java applications are notorious for OOM issues in Kubernetes due to the JVM's default heap sizing behavior.
- **The Problem:** Older JVMs do not respect cgroup memory limits. They calculate the default heap size based on the host node's total memory, not the container's limit. This inevitably leads to the JVM attempting to allocate more memory than the container is allowed, resulting in an OOMKilled event.
- **The Solution:** Ensure the use of Java 10+ (or Java 8u191+) which includes `UseContainerSupport` (enabled by default). Explicitly set the heap size using `-XX:MaxRAMPercentage` (e.g., 75.0) rather than hardcoding `-Xmx`, allowing the JVM to scale dynamically with the container's memory limit.

### 3.4. Memory Leaks and Profiling

If an application's memory usage grows unbounded over time, it likely has a memory leak.
- **Diagnostic Action:** Capture a heap dump before the container is killed. This can be challenging if the container crashes unpredictably.
- **Advanced Technique:** Configure the application to automatically generate a heap dump on OOM and write it to a persistent volume.
  ```bash
  -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/mnt/pv/heapdump.hprof
  ```
  Analyze the heap dump using tools like Eclipse MAT or VisualVM to identify the objects consuming the most memory.

## 4. DNS Resolution Issues: CoreDNS and VPC DNS

DNS resolution failures in Kubernetes manifest as applications being unable to communicate with other services or external endpoints. In EKS, DNS relies on CoreDNS running within the cluster and the AWS VPC DNS resolver.

### 4.1. Symptoms of DNS Failures

- Applications log `UnknownHostException` or `NXDOMAIN` errors.
- Intermittent connection timeouts to internal services (e.g., `my-svc.default.svc.cluster.local`).
- High latency in API calls due to DNS lookup delays.

### 4.2. Diagnostic Workflow

1. **Test DNS Resolution from a Pod:**
   Deploy a diagnostic pod (e.g., `dnsutils`) and test resolution using `nslookup` or `dig`.
   ```bash
   kubectl run -i --tty --rm debug --image=infoblox/dnstools --restart=Never -- sh
   # Inside the pod:
   nslookup kubernetes.default
   nslookup google.com
   ```

2. **Check CoreDNS Pods:**
   Ensure the CoreDNS pods are running and not in a CrashLoopBackOff state.
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   ```

3. **Inspect CoreDNS Logs:**
   Look for errors, warnings, or high query rates.
   ```bash
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```

### 4.3. Deep Dive: ndots and DNS Search Domains

A common cause of DNS performance issues and intermittent failures in Kubernetes is the `ndots` configuration in the pod's `/etc/resolv.conf`.
- **The Problem:** By default, Kubernetes sets `ndots:5`. This means that for any domain name with fewer than 5 dots (e.g., `google.com`), the DNS resolver will append all the search domains (e.g., `default.svc.cluster.local`, `svc.cluster.local`, `cluster.local`, `ec2.internal`) and query them sequentially before finally querying the absolute name. This results in multiple unnecessary DNS queries, increasing latency and load on CoreDNS.
- **The Solution:** If an application frequently queries external domains, reduce the `ndots` value in the pod's `dnsConfig`.
  ```yaml
  dnsConfig:
    options:
      - name: ndots
        value: "2"
  ```

### 4.4. EKS Specifics: VPC DNS Throttling

AWS limits DNS queries to the VPC DNS resolver (the `.2` address) to 1024 packets per second per elastic network interface (ENI).
- **The Problem:** If CoreDNS pods are concentrated on a few nodes, they can easily exceed this limit, resulting in dropped DNS queries and intermittent resolution failures.
- **The Solution:** Implement NodeLocal DNSCache. This deploys a DNS caching agent as a DaemonSet on every node, significantly reducing the load on CoreDNS and avoiding VPC DNS throttling by caching responses locally on the node.

## 5. CNI Problems: VPC CNI, IP Exhaustion, and Routing

The Container Network Interface (CNI) is responsible for assigning IP addresses to pods and configuring network routing. In EKS, the default is the Amazon VPC CNI, which assigns native VPC IP addresses to pods.

### 5.1. Symptoms of CNI Failures

- Pods remain in the `ContainerCreating` state indefinitely.
- Events show `FailedCreatePodSandBox` with errors related to network plugin initialization or IP allocation.
- Pods cannot communicate with each other or external networks.

### 5.2. Deep Dive: VPC IP Exhaustion

The most common issue with the Amazon VPC CNI is IP address exhaustion.
- **The Mechanism:** The VPC CNI attaches secondary ENIs to the EC2 worker nodes and assigns secondary IP addresses from the VPC subnets to these ENIs. Each EC2 instance type has a hard limit on the number of ENIs and IPs per ENI it can support.
- **The Problem:** If the subnets are too small, or if the node is running many small pods, the CNI may run out of available IP addresses to assign to new pods.
- **Diagnostic Action:**
  Check the subnet's available IP addresses in the AWS VPC Console.
  Check the `aws-node` daemonset logs for IP allocation errors.
  ```bash
  kubectl logs -n kube-system -l k8s-app=aws-node
  ```
- **The Solution:**
  1. **Custom Networking:** Configure the VPC CNI to use a secondary CIDR block (e.g., 100.64.0.0/10 - CGNAT space) specifically for pods, freeing up the primary VPC CIDR for nodes and other AWS resources.
  2. **Prefix Delegation:** Enable Prefix Delegation in the VPC CNI. Instead of assigning individual IPs to ENIs, it assigns /28 prefixes, significantly increasing the number of pods that can run on a single node and reducing the frequency of AWS API calls.

### 5.3. SNAT and Outbound Connectivity

Pods need to communicate with the internet (e.g., to pull images or access external APIs).
- **The Problem:** By default, the VPC CNI translates the pod's IP address to the node's primary IP address (Source Network Address Translation - SNAT) for traffic destined outside the VPC. If `AWS_VPC_K8S_CNI_EXTERNALSNAT` is misconfigured, or if the node lacks a NAT Gateway/Public IP, outbound traffic will fail.
- **Diagnostic Action:** Verify the `aws-node` DaemonSet environment variables. Ensure the subnets have appropriate route tables pointing to a NAT Gateway (for private subnets) or an Internet Gateway (for public subnets).

## 6. Node NotReady: Kubelet, Container Runtime, and EC2 Health

A node entering the `NotReady` state means the Kubernetes control plane can no longer communicate with the kubelet running on that node, or the kubelet has reported that the node is unhealthy.

### 6.1. Diagnostic Workflow

1. **Check Node Status and Conditions:**
   ```bash
   kubectl describe node <node-name>
   ```
   Look at the `Conditions` section. Is `MemoryPressure`, `DiskPressure`, or `PIDPressure` set to True?

2. **Access the Node:**
   If possible, SSH into the node or use AWS Systems Manager (SSM) Session Manager.

3. **Inspect Kubelet Logs:**
   The kubelet is the primary agent on the node. Its logs are crucial.
   ```bash
   journalctl -u kubelet -f
   ```

### 6.2. Common Root Causes

#### 6.2.1. Resource Exhaustion (CPU/Memory/Disk)
- **Symptom:** The node becomes unresponsive, and the kubelet fails to send heartbeats to the API server.
- **Deep Dive:** If a node runs out of memory and the OOM killer terminates the kubelet or container runtime (containerd/Docker), the node will become `NotReady`. Similarly, if the root filesystem (`/`) or the container runtime filesystem (`/var/lib/containerd`) reaches 100% capacity, the kubelet will mark the node with `DiskPressure` and eventually transition to `NotReady`.
- **Solution:** Implement proper resource requests and limits for all pods. Configure kubelet eviction thresholds (`evictionHard`) to proactively evict pods before the node reaches a critical state.

#### 6.2.2. Container Runtime Failures
- **Symptom:** The kubelet logs show errors communicating with the container runtime (e.g., containerd socket timeouts).
- **Deep Dive:** The container runtime can hang due to deadlocks, storage driver issues, or kernel bugs. Restarting the runtime service (`systemctl restart containerd`) often resolves the immediate issue, but root cause analysis requires inspecting the runtime logs (`journalctl -u containerd`).

#### 6.2.3. AWS Infrastructure Issues
- **Symptom:** The node is `NotReady`, and SSH/SSM access fails.
- **Deep Dive:** The underlying EC2 instance may have failed a status check (System Status Check or Instance Status Check). Check the AWS EC2 Console. If the instance is impaired, terminate it and let the Auto Scaling Group (ASG) or Karpenter provision a replacement.

## 7. IAM Roles for Service Accounts (IRSA) Failures

IRSA allows you to assign AWS IAM roles directly to Kubernetes Service Accounts. This provides fine-grained, pod-level access control to AWS resources (e.g., S3, DynamoDB) without relying on node-level IAM roles or hardcoded credentials.

### 7.1. How IRSA Works

IRSA relies on an OpenID Connect (OIDC) identity provider configured in the EKS cluster. When a pod uses a Service Account annotated with an IAM role, the EKS pod identity webhook injects AWS credentials (a web identity token file and environment variables) into the pod. The AWS SDKs within the pod use this token to assume the IAM role.

### 7.2. Symptoms of IRSA Failures

- Applications log `AccessDenied` errors when attempting to access AWS resources.
- The AWS SDK reports that it cannot find credentials.

### 7.3. Diagnostic Workflow and Deep Dive

1. **Verify the Service Account Annotation:**
   Ensure the Service Account has the correct annotation pointing to the IAM role ARN.
   ```bash
   kubectl describe sa <service-account-name> -n <namespace>
   # Look for: eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-role
   ```

2. **Verify Pod Configuration:**
   Check if the pod is actually using the Service Account and if the webhook successfully injected the environment variables and volume mounts.
   ```bash
   kubectl describe pod <pod-name>
   # Look for environment variables: AWS_ROLE_ARN and AWS_WEB_IDENTITY_TOKEN_FILE
   # Look for volume mounts: aws-iam-token
   ```
   *Failure Point:* If these are missing, the pod identity webhook might be failing, or the pod was created before the Service Account was annotated. Delete the pod to force a recreation.

3. **Inspect the IAM Role Trust Policy:**
   This is the most common point of failure. The IAM role must have a trust policy that allows the OIDC provider to assume the role, specifically for the designated Service Account and namespace.
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.region.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
         },
         "Action": "sts:AssumeRoleWithWebIdentity",
         "Condition": {
           "StringEquals": {
             "oidc.eks.region.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:<namespace>:<service-account-name>",
             "oidc.eks.region.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com"
           }
         }
       }
     ]
   }
   ```
   *Failure Point:* Typos in the namespace or service account name in the `Condition` block will result in silent failures (AccessDenied).

4. **Verify AWS SDK Version:**
   Ensure the application is using an AWS SDK version that supports `AssumeRoleWithWebIdentity`. Older SDKs will ignore the injected environment variables and fall back to the node's IAM role (which should ideally have no permissions).

## 8. Worst-Case Scenarios and Disaster Recovery

In extreme situations, multiple failures can cascade, leading to a complete cluster outage.

### 8.1. Control Plane Exhaustion (API Server Overload)
- **Scenario:** A misconfigured controller or a massive surge in deployments floods the Kubernetes API server with requests, causing it to become unresponsive. `kubectl` commands time out.
- **Mitigation:** In EKS, the control plane is managed by AWS. However, you can still impact it. Identify the source of the API spam. If it's a specific deployment, scale it down. Utilize API Priority and Fairness (APF) to throttle abusive clients and protect critical system components.

### 8.2. Complete Network Partition
- **Scenario:** A misconfigured Network Policy or a failure in the CNI daemonset isolates all pods, preventing any internal or external communication.
- **Mitigation:** Temporarily disable enforcing Network Policies (if possible) to restore baseline connectivity. Roll back recent changes to the CNI configuration. If the CNI is completely broken, you may need to manually delete the CNI daemonset and reapply the manifest.

### 8.3. Etcd Data Corruption (Self-Managed Kubernetes)
- **Scenario:** The etcd database, which stores the entire cluster state, becomes corrupted or loses quorum.
- **Mitigation:** In EKS, AWS manages etcd backups and recovery. In self-managed clusters, this is a catastrophic event requiring a restore from the latest etcd snapshot.
  ```bash
  ETCDCTL_API=3 etcdctl snapshot restore snapshot.db     --name m1     --initial-cluster m1=http://host1:2380,m2=http://host2:2380,m3=http://host3:2380     --initial-cluster-token etcd-cluster-1     --initial-advertise-peer-urls http://host1:2380
  ```

## 8.4. Persistent Volume (PV) and Persistent Volume Claim (PVC) Failures
Storage issues can cause pods to hang in `ContainerCreating` or fail to start.
- **Scenario:** A pod requests a PVC, but the underlying EBS volume fails to attach to the EC2 node.
- **Deep Dive:** Check the PVC status (`kubectl get pvc`). If it's `Pending`, the storage class might be misconfigured or the AWS EBS CSI driver might be failing. Check the CSI driver logs.
- **EBS Limits:** EC2 instances have limits on the number of attached EBS volumes. If a node reaches this limit, new pods requiring storage will fail to schedule or start.

## 8.5. Ingress Controller and Load Balancer Bottlenecks
- **Scenario:** External traffic fails to reach services, or latency is extremely high.
- **Deep Dive:** In EKS, this often involves the AWS Load Balancer Controller. Check the controller logs for errors provisioning ALBs or NLBs. Verify that the subnets are correctly tagged (`kubernetes.io/role/elb` or `kubernetes.io/role/internal-elb`) so the controller can discover them.

## 8.6. Certificate Expiration and TLS Failures
- **Scenario:** Webhooks fail, or internal communication between components breaks down due to expired TLS certificates.
- **Deep Dive:** Kubernetes relies heavily on mutual TLS (mTLS). Check the expiration dates of certificates in the `kube-system` namespace. Use tools like `cert-manager` to automate certificate renewal and prevent these outages.

## 9. Advanced Debugging Tools and Techniques

### 9.1. Ephemeral Containers
Introduced as a stable feature in recent Kubernetes versions, ephemeral containers allow you to attach a debug container to a running pod without restarting it. This is invaluable for debugging distroless images or containers that lack basic shell utilities.
```bash
kubectl debug -it <pod-name> --image=nicolaka/netshoot --target=<container-name>
```

### 9.2. Network Packet Capture (tcpdump)
When diagnosing complex network issues (e.g., dropped packets, unexpected resets), capturing traffic at the node or pod level is necessary.
- **Node Level:** SSH into the node and run `tcpdump` on the `eni` or `cali` interfaces.
- **Pod Level:** Use an ephemeral container with `tcpdump` installed to capture traffic directly within the pod's network namespace.

### 9.3. Strace and Sysdig
For deep system-level debugging, tools like `strace` (to trace system calls) and `sysdig` (for comprehensive system exploration) can reveal exactly what a process is doing, which files it's trying to open, and where it's getting stuck.

## 9.4. EKS Control Plane Logging
EKS provides the ability to export control plane logs to Amazon CloudWatch. This is critical for auditing and troubleshooting API server, scheduler, and controller manager issues.
- **Enable Logging:** Ensure that API, audit, authenticator, controllerManager, and scheduler logs are enabled in the EKS cluster configuration.
- **Analysis:** Use CloudWatch Logs Insights to query the logs. For example, to find all failed API requests:
  ```
  fields @timestamp, @message
  | filter @logStream like /^kube-apiserver/
  | filter responseStatus.code >= 400
  | sort @timestamp desc
  ```

## 9.5. Karpenter vs. Cluster Autoscaler
Scaling issues often lead to pending pods. Understanding the autoscaling mechanism is crucial.
- **Cluster Autoscaler:** Relies on AWS Auto Scaling Groups (ASGs). It can be slow and requires careful configuration of node groups.
- **Karpenter:** A newer, high-performance autoscaler that bypasses ASGs and provisions EC2 instances directly based on pod requirements. Troubleshooting Karpenter involves checking its logs and the `Provisioner` custom resources.

## 10. Security and Compliance Troubleshooting

### 10.1. Pod Security Admission (PSA)
With the deprecation of Pod Security Policies (PSP), PSA is the standard for enforcing security standards.
- **Symptom:** Pods fail to create with errors related to security context (e.g., `violates PodSecurity "restricted"`).
- **Solution:** Review the namespace labels (`pod-security.kubernetes.io/enforce`) and adjust the pod's `securityContext` to comply with the required profile (Privileged, Baseline, or Restricted).

### 10.2. Network Policies
- **Symptom:** Pods can communicate with some services but not others.
- **Solution:** Network Policies act as firewalls for pods. Use tools like `cilium network-policy-editor` or carefully review the YAML definitions to ensure the `podSelector`, `ingress`, and `egress` rules are correctly configured. Remember that Network Policies are default-deny once applied to a pod.

## 11. Conclusion

Troubleshooting Kubernetes and EKS in production requires a deep understanding of both the Kubernetes architecture and the underlying AWS infrastructure. By mastering the diagnostic workflows for CrashLoopBackOff, OOMKilled, DNS, CNI, Node readiness, and IRSA, tech support specialists and SREs can rapidly identify root causes and implement robust solutions. 

The key to successful operations is not just fixing the immediate issue, but understanding *why* it happened and implementing preventative measures—such as proper resource limits, optimized DNS configurations, and robust IAM policies—to ensure the long-term stability and resilience of the cluster. Continuous monitoring, proactive alerting, and regular disaster recovery drills are essential components of a mature Kubernetes operational strategy.

---
**References:**
[1] Kubernetes Documentation: Debugging Pods - https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/
[2] AWS EKS Documentation: Troubleshooting IAM Roles for Service Accounts - https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting-iam-roles-for-service-accounts.html
[3] Amazon VPC CNI Plugin for Kubernetes - https://github.com/aws/amazon-vpc-cni-k8s
[4] CoreDNS Performance and Scaling - https://coredns.io/manual/scaling/
