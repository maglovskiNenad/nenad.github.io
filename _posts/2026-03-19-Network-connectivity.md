---
layout: post
title: "Network Connectivity Incident Report"
date: 2026-03-19 09:00:00 -0500
categories: linux
tages: network ubuntu connecgtivity report 
image:
    path: /assets/img/headers/Network_Connectivity_Incident_Report.png
---

# Network Connectivity Incident Report

The initial troubleshooting step was to inspect the current IP configuration:

```bash
ip a
```

This command was used to confirm that the interface was still up and that the system retained its local network configuration. The next step was to inspect the routing table:

```bash
ip route
```

At that point, it became clear that the system was missing its **default route**. In a normal configuration, the routing table includes an entry similar to:

```bash
default via 192.168.1.1 dev enp0s3
```

This route tells the system where to send traffic destined for networks outside the local subnet. Without it, the host may still communicate locally, but it cannot reach external networks such as the internet.

## Cause of the Issue

The problem was caused by the removal of the default gateway route. This can happen with a command such as:

```bash
sudo ip route del default
```

Once that route is deleted, the system no longer knows which gateway should be used for outbound traffic.

## Symptoms Observed

The system showed several clear signs of routing failure. Internet access was no longer available, communication with external addresses failed, and the `ip route` output no longer displayed a `default via ...` entry.

This confirmed that the interface itself was not necessarily the issue. The real problem was in the routing table.

## Resolution

To restore connectivity, the default route was manually added back:

```bash
sudo ip route add default via 192.168.1.1
```

If the interface needs to be specified explicitly, the command can also be written as:

```bash
sudo ip route add default via 192.168.1.1 dev enp0s3
```

After applying the fix, the routing table was checked again:

```bash
ip route
```

The expected result was the return of a valid default route, for example:

```bash
default via 192.168.1.1 dev enp0s3
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.50
```

---

Once the route was restored, normal network connectivity returned.






