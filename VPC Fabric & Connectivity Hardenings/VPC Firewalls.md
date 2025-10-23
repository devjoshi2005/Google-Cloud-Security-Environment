# Configuring VPC Firewalls

Link :- [VPC Firewalls between VM's](https://www.skills.google/focuses/19172?parent=catalog)


![image](image.png)

**Architecture Flowchart**
```mermaid
flowchart LR
  subgraph VPC "mynetwork"
    VM1[mynet-vm-1 Zone 1]:::vm
    VM2[mynet-vm-2 Zone 2]:::vm
  end

  subgraph CloudShell
    CS[Cloud Shell Client]:::client
  end

  CS -->|SSH / ICMP| VM1
  CS -->|SSH / ICMP| VM2
  VM1 -->|ICMP internal| VM2

  classDef vm fill:#ffd,stroke:#c80
  classDef client fill:#dff,stroke:#08c
```

**Project Summary**
This Guided Lab project demonstrates the provisioning of VPC networks and instances, exploring default firewall behavior, then configuring custom ingress and egress firewall rules with priorities to precisely control the connectivity between VMs.

**Business importance**
1) Security enforcement: Firewalls are the first line of defense in cloud networking, ensuring only authorized traffic reaches workloads.
2) Granular control: Rule priorities and directions (ingress/egress) allow fine‑tuned policies that balance security with operational needs.
3) Operational reliability: Proper firewall configuration prevents accidental exposure of services while enabling required internal communication.
4) Audit and compliance: Explicit firewall rules provide evidence of access control for regulated environments.

**Tools used and significance**
*VPC Networks (auto and custom)*: Provide isolated logical networks; auto networks include default firewall rules, custom networks start with deny‑all ingress.
*Firewall Rules*: Stateful rules that allow or deny traffic based on direction, protocol, port, source/destination, and priority.
*Firewall Rule Priorities*: Determine which rule is applied first; lower numbers = higher priority.
*Tags*: Apply rules selectively to instances, reducing risk of accidental over‑permissiveness.
*Cloud Shell / gcloud CLI*: Used to create networks, instances, and firewall rules reproducibly.
*ICMP / SSH traffic tests*: Validate connectivity and confirm firewall rule behavior.

**Technical value proposition**
1) Demonstrates the difference between default network rules and user‑created network rules.
2) Validates that user‑created networks start with deny‑all ingress and require explicit allow rules.
3) Shows how to apply firewall rules using tags and source ranges for targeted enforcement.
4) Explains priority evaluation and how deny/allow rules interact across ingress and egress.




**Step‑by‑step execution plan**

1) Create auto‑mode VPC and instances
```
gcloud compute networks create mynetwork --subnet-mode=auto
```
2) Create test VMs: default-vm-1 (default network), mynet-vm-1 (mynetwork, Zone 1), mynet-vm-2 (mynetwork, Zone 2).

3) Investigate default network

Observe default firewall rules: allow‑ssh, allow‑icmp, allow‑internal, allow‑rdp.

SSH into default-vm-1 to confirm default-allow-ssh works.

Delete default-vm-1 and default network for security hygiene.

Investigate user‑created network

Attempt SSH into mynet-vm-1 or mynet-vm-2 → fails (no ingress rules).

Confirms deny‑all ingress is enforced by default.

4) Create custom ingress firewall rules

Retrieve Cloud Shell external IP: ip=$(curl -s https://api.ipify.org)

Allow SSH from Cloud Shell IP: gcloud compute firewall-rules create mynetwork-ingress-allow-ssh-from-cs \ --network mynetwork --action ALLOW --direction INGRESS \ --rules tcp:22 --source-ranges $ip --target-tags=lab-ssh

Tag instances: gcloud compute instances add-tags mynet-vm-1 --zone Zone1 --tags lab-ssh gcloud compute instances add-tags mynet-vm-2 --zone Zone2 --tags lab-ssh

Verify SSH works from Cloud Shell.

Allow internal ICMP (ping) traffic
```
gcloud compute firewall-rules create mynetwork-ingress-allow-icmp-internal \ --network mynetwork --action ALLOW --direction INGRESS \ --rules icmp --source-ranges 10.128.0.0/9
```
Verify ping between mynet-vm-1 and mynet-vm-2 succeeds.

Confirm ping to external IP fails (blocked by NAT + firewall).

5) Set firewall rule priority (deny ICMP)

Create deny rule with higher priority: 
```
gcloud compute firewall-rules create mynetwork-ingress-deny-icmp-all \ --network mynetwork --action DENY --direction INGRESS \ --rules icmp --priority 500
```
Verify ping fails (deny rule overrides allow).

Update deny rule to priority 2000: gcloud compute firewall-rules update mynetwork-ingress-deny-icmp-all --priority 2000

Verify ping succeeds again (allow rule takes precedence).

6) Configure egress firewall rules

Create egress deny rule: 
```
gcloud compute firewall-rules create mynetwork-egress-deny-icmp-all \ --network mynetwork --action DENY --direction EGRESS \ --rules icmp --priority 10000
```

Verify ping fails again (both ingress and egress must allow traffic).

7) Cleanup