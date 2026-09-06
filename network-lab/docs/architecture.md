# Network Lab Architecture

## Overview

This lab uses Containerlab to deploy two FRRouting routers as Docker containers. Ansible runs locally on the Kali Linux host and automates the configuration of the routers.

```text
                    Kali Linux
                        |
                     Ansible
                        |
                      Docker
                 _______|_______
                |               |
          Router 1           Router 2
       FRRouting 10.7.0   FRRouting 10.7.0
       172.20.20.2        172.20.20.3
                |               |
                +---------------+
                  10.0.0.0/30
```

## Components

| Component    | Purpose                      |
| ------------ | ---------------------------- |
| Kali Linux   | Automation host              |
| Ansible      | Configuration and automation |
| Containerlab | Network topology deployment  |
| Docker       | Container runtime            |
| FRRouting    | Routing software             |
| OSPF         | Dynamic routing protocol     |

## Router Addressing

| Router   | Management IP | Interface | Link IP     | OSPF Router ID |
| -------- | ------------- | --------- | ----------- | -------------- |
| Router 1 | 172.20.20.2   | eth1      | 10.0.0.1/30 | 1.1.1.1        |
| Router 2 | 172.20.20.3   | eth1      | 10.0.0.2/30 | 2.2.2.2        |

The router-to-router network is:

```text
10.0.0.0/30
```

OSPF operates in:

```text
Area 0
```

## Automation Flow

The configuration workflow is:

```text
Containerlab
     ↓
Deploy FRR containers
     ↓
Ansible inventory
     ↓
Configure eth1
     ↓
Enable ospfd
     ↓
Configure OSPF
     ↓
Verify neighbor adjacency
```

Because the FRRouting containers do not provide SSH access, Ansible uses a local connection and executes commands through Docker:

```text
Ansible
   ↓
Kali Linux
   ↓
docker exec
   ↓
FRRouting container
```

## OSPF Verification

A successful OSPF adjacency should show the neighbor in the `Full` state:

```text
Neighbor ID     State
2.2.2.2         Full
```

The `Full` state confirms that Router 1 and Router 2 have successfully established their OSPF adjacency.

## Project Scope

This version focuses on:

* Two-router topology
* IPv4 point-to-point connectivity
* FRRouting
* OSPF Area 0
* Ansible automation
* Configuration verification
* Basic troubleshooting and recovery

Future volumes will expand the topology with additional routers, loopbacks, multiple networks, OSPF-learned routes, link failures, and more advanced routing configuration.
