---
title: "Aruba Disable Bpdu Port"
date: 2022-01-31T16:41:40+01:00
draft: false
tags:
- Aruba
- Spanning Tree
---

When connecting our providers MPLS router (Cisco) to the Aruba switch, we did notice that the port on the Cisco router always came up and quickly went down..
The problem I think it was is that we have on the Aruba VLAN 100 for voice and the Cisco router has this also untagged on his configuration, this somehows fights
with eachother and the port became in the blocking state. We saw this when checking the STP status, when disabling the spaning tree protection for this port only,
the problem went away and the router did stay a live. 

From now on we'll not use VLAN 100 for voice anymore in the customer network, no more issues :-)

```bash
#check filtered ports
show spanning-tree 

#see port 47 config for spanning-tree
show spanning-tree 47 config

#disable spanning tree on this port and ignore everything
no spanning-tree 47 bpdu-protection
spanning-tree 47 bpdu-filter
```
