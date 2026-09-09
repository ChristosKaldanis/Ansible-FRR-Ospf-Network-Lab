# Ansible + FRRouting OSPF Network Lab

> **Volume 1 — Network Automation & Dynamic Routing**

A containerized network automation lab designed to demonstrate the deployment, configuration, and verification of a small dynamic routing environment using **Ansible, Containerlab, Docker, and FRRouting (FRR)**. The project consists of two FRRouting routers deployed as containers and connected through an IPv4 point-to-point link. Ansible provides the automation layer, configuring the Linux interfaces, enabling the FRR OSPF daemon, applying OSPF parameters, and verifying routing connectivity and neighbor relationships. The lab also focuses on practical network-engineering skills including inventory management, host-specific variables, Linux networking, OSPF troubleshooting, service recovery, and repeatable configuration workflows.

---

## Overview

The objective of this project is to build a reproducible network environment where routing infrastructure can be deployed and configured through automation rather than manual CLI configuration.

### Topology

```text
                         Kali Linux
                             │
                          Ansible
                             │
                           Docker
                    ┌────────┴────────┐
                    │                 │
                 Router 1          Router 2
                 FRRouting         FRRouting
              172.20.20.2        172.20.20.3
                    │                 │
                 eth1                 eth1
              10.0.0.1/30 ─────── 10.0.0.2/30
                         OSPF Area 0
```

The routers form an OSPF adjacency across the `10.0.0.0/30` point-to-point network.

---

## Technology Stack

| Technology       | Role                                 |
| ---------------- | ------------------------------------ |
| **Ansible**      | Network automation and configuration |
| **Containerlab** | Network topology deployment          |
| **Docker**       | Container runtime                    |
| **FRRouting**    | Routing platform                     |
| **OSPF**         | Dynamic routing protocol             |
| **Kali Linux**   | Automation host                      |

---

## Network Addressing

| Router   | Management IP | Interface | Link IP       | OSPF Router ID |
| -------- | ------------- | --------- | ------------- | -------------- |
| Router 1 | `172.20.20.2` | `eth1`    | `10.0.0.1/30` | `1.1.1.1`      |
| Router 2 | `172.20.20.3` | `eth1`    | `10.0.0.2/30` | `2.2.2.2`      |

**OSPF Area:** `0.0.0.0`

**Transit Network:** `10.0.0.0/30`

---

## Architecture

Ansible runs locally on the Kali Linux host and interacts with the FRRouting containers through Docker.

```text
┌──────────────────────┐
│      Kali Linux      │
│                      │
│       Ansible        │
└──────────┬───────────┘
           │
           │ docker exec
           │
     ┌─────┴─────┐
     │   Docker  │
     └─────┬─────┘
           │
    ┌──────┴──────┐
    │             │
┌───▼───┐     ┌───▼───┐
│ FRR R1│─────│ FRR R2│
└───────┘     └───────┘
    OSPF Area 0
```

The FRRouting image used in the lab does not provide SSH access. Therefore, Ansible uses a **local connection** and executes configuration commands inside the containers using `docker exec`.

---

## Repository Structure

```text
ansible-frr-ospf-network-lab/
│
├── README.md
├── LICENSE
├── .gitignore
├── ansible.cfg
│
├── inventory/
│   ├── hosts
│   ├── host_vars/
│   │   ├── 172.20.20.2.yml
│   │   └── 172.20.20.3.yml
│   └── group_vars/
│       └── routers.yml
│
├── network-lab/
│   ├── lab.yml
│   ├── bootstrap.yml
│   ├── configure-link.yml
│   ├── enable-ospf.yml
│   ├── configure-ospf.yml
│   └── test-frr.yml
│
└── docs/
    └── architecture.md
```

---

## Automation Workflow

The lab follows a simple deployment and configuration workflow:

```text
Containerlab Deployment
          │
          ▼
    FRR Containers
          │
          ▼
   Ansible Inventory
          │
          ▼
 Configure Network Link
          │
          ▼
     Enable OSPF
          │
          ▼
   Configure OSPF
          │
          ▼
 Verify Adjacency
```

This separates **topology deployment** from **network configuration**, making the environment easier to rebuild and troubleshoot.

---

## Deployment

Deploy the Containerlab topology:

```bash
cd /home/kali/ansible/network-lab
sudo containerlab deploy -t lab.yml
```

Return to the Ansible project directory:

```bash
cd /home/kali/ansible
```

Verify the inventory:

```bash
ansible-inventory --graph
```

Run the bootstrap workflow:

```bash
ansible-playbook network-lab/bootstrap.yml
```

Apply the OSPF configuration:

```bash
ansible-playbook network-lab/configure-ospf.yml
```

---

## Verification

Verify the router-to-router interface:

```bash
docker exec clab-ansible-lab-router1 ip addr show eth1
docker exec clab-ansible-lab-router2 ip addr show eth1
```

Test Layer 3 connectivity:

```bash
docker exec clab-ansible-lab-router1 ping -c 3 10.0.0.2
```

Verify the OSPF neighbor relationship:

```bash
docker exec clab-ansible-lab-router1 \
  vtysh -c "show ip ospf neighbor"
```

A successful adjacency should show:

```text
Neighbor ID     State
2.2.2.2         Full
```

The `Full` state confirms that the two routers have successfully established an OSPF adjacency.

---

## Key Engineering Concepts

This project demonstrates practical experience with:

* IPv4 addressing and `/30` point-to-point networks
* Linux network interfaces
* Docker networking
* Containerlab topology deployment
* FRRouting
* OSPF Area 0
* OSPF neighbor states
* Ansible inventories
* Host and group variables
* Configuration automation
* Network troubleshooting
* Service and topology recovery
* Repeatable network deployment

---

## Troubleshooting

One of the practical issues encountered during development was the distinction between **container lifecycle** and **Containerlab topology lifecycle**.

After a Linux VM restart, Docker restarted the FRRouting containers, but the Containerlab-created router-to-router interface was not automatically recreated. The topology therefore had to be redeployed before Ansible could configure `eth1`.

Recovery:

```bash
cd /home/kali/ansible/network-lab
sudo containerlab deploy -t lab.yml

cd /home/kali/ansible
ansible-playbook network-lab/bootstrap.yml
```

This highlighted an important operational concept: **container availability does not necessarily mean that the complete network topology has been restored.**

---

## Current Status

**Volume 1 — Completed**

* [x] Containerlab topology
* [x] FRRouting routers
* [x] Ansible inventory
* [x] Host and group variables
* [x] Automated interface configuration
* [x] OSPF configuration
* [x] OSPF adjacency verification
* [x] Troubleshooting and recovery testing
* [x] Project documentation

---

## Roadmap

### Volume 2

Planned expansion:

* Three or more routers
* Loopback interfaces
* Multiple routed networks
* OSPF-learned routes
* Link failure and convergence testing
* OSPF cost manipulation
* Multiple OSPF areas
* Ansible templates
* Improved idempotency
* Automated verification

### Volume 3

Future routing and automation work:

* BGP
* Route redistribution
* Prefix lists
* Route maps
* Routing policy
* Network testing
* CI/CD integration

---

## Project Purpose

This project is part of a hands-on network engineering portfolio focused on combining **routing fundamentals with infrastructure automation**.

The objective is not simply to configure OSPF, but to demonstrate the ability to **build, automate, verify, troubleshoot, and recover a network environment using modern tooling**.
