---
title: "Alarm shield"
linkTitle: "Alarm shield"
date: 2020-04-22
description: >
  In addition to shielding the machine, the alarm shield can also do more fine-grained shielding strategies. For example, you can specify monitoring indicators and a certain label.
---

Compared with Open-Falcon, which can only shield machines, Nightingale's alarm shielding has a more flexible granularity. It can configure a certain index of a certain machine, and even a certain tag of a certain index of a certain machine.
The configuration of the alarm masking is based on the service tree, which is purely for management convenience. For example, if a machine hangs on the sre.devops.n9e.judge.hna node of the service tree, we need to block the cpu.idle alarm of this machine. You can configure the blocking policy on the sre node or sre.devops , Or lower-level nodes, can achieve shielding effect.
Which node is appropriate for configuration?

- If you want to shield the service-related indicators, it is generally placed on the service node, for example, this machine is used for the n9e service, then configure the shielding strategy at the n9e node.
- If you want to shield the hardware-related indicators, you can shield it at the service node or the team node, that is, the devops node. Because the devops operation and maintenance students may also operate and maintain the services of other teams under the sre, such as the dfe team, as long as the operation and maintenance students can distinguish that this machine is devops
- If the machine is mounted on multiple nodes, one of the nodes is configured with a shielding strategy, and the related alarm strategy of the other node will also be affected. It is best to configure the shielding strategy on the nodes above the common parent node, which seems unambiguous.

The explanation is more complicated. In fact, there is no problem on which layer to put. The key is that the team must have a specification, which is convenient to manage later. Everyone knows which layer to check the blocking strategy.