
---
title: "Introduction"
linkTitle: "Introduction"
weight: 2
date: 2020-04-10
description: >
  Make a simple and intuitive display of Nightingale, explain the similarities and differences with Open-Falcon, explain its architecture and future development.
---

## Nightingale introduction

This section gives a brief introduction to Nightingale, so that you can have a preliminary intuitive feeling for the system and prepare for a deeper understanding.

### Nightingale system interface

Looking directly at the system interface is the most intuitive way to understand the system. Here are a few pictures to give everyone a preliminary understanding.

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-tree.png)

The picture above is the object page under the node. The object refers to the target monitoring object, usually refers to the machine. This tree is called the service tree. From top to bottom, it is described as the organizational structure relationship, product service module relationship, engine room and machine mounting relationship. This is a grouping mechanism for machines. This tree supports flexible customization. When the actual monitoring strategy is configured, it is necessary to configure different alarm strategies for different machines with different purposes. Therefore, a flexible machine grouping mechanism is very important.

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-stra.png)

The above picture is the monitoring strategy configuration page. The strategies in the list are all applied to the didi.storage node. Therefore, all the machines mounted on all child nodes under this node will apply this strategy. Any machine that reaches the relevant threshold will trigger an alarm. It is super convenient. :-)

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-screen.png)

The customization of the monitoring dashboard has been greatly improved in terms of ease of use. It supports chart thresholds and chart classification. New charts and sorting management are all visible and what you get. Customizing the inspection dashboard is no longer difficult.


### Similarities and differences with Open-Falcon

Since Nightingale has a deep relationship with Open-Falcon, it is certain that everyone will be more interested in the similarities and differences between each other. Overall, the concept has a strong continuity, and the improvement in architecture and product experience is huge.

#### Differences from Open-Falcon

- Alarm engine was refactored to push-pull combination mode，The push mode ensures the efficiency of policy judgments, and the pull mode supports nodata alarms. Remove the original nodata components to simplify the difficulty of system deployment
- Introduced service tree, hierarchically manage the endpoint, remove the original flat HostGroup, and remove the alarm template at the same time. The alarm strategy is directly bound to the service tree node, greatly improving flexibility and ease of use.
- Remove the original database-based index library, change to the memory mode, and use an index module to process the index alone, avoiding the embarrassment of the original MySQL single expression to the billion level, and the efficiency of the index is greatly improved after the memory-based index.
- Storage module `Graph` introduced Facebook's Gorilla, which is the memory TSDB. The data of the last few hours is stored in memory by default, which greatly improves the efficiency of data query. The hard disk storage method still uses rrdtool.
- The `judge` module of the alarm engine achieves automatic fault removal through the heartbeat mechanism, and there is no need to worry about the failure of a single judge to cause some strategies to fail. The index module uses a similar method to ensure system availability.
- The client has built-in log matching index extraction capability. The web page supports the configuration of log matching rules, and also supports the way to read the configuration file in the specific directory of the target machine, making business index monitoring easier to use.
- The portal (api in falcon-plus), uic, dashboard, hbs, and alarm are combined into a module called monapi, which simplifies the overall system deployment difficulty, and the original inter-module call becomes an in-process method call, with better stability.

#### Similarities with Open-Falcon

- The data model remains consistent, and it is still the organization of metric, endpoint, and tags, so the agent can be reused. The agent in Nightingale is called collector, which combines the logic of the original Open-Falcon agent and falcon-log-agent, and various monitoring plugins can also be reused.
- The data flow and processing logic of Nightingale and Open-Falcon are similar, and still use a flexible push model, which is divided into two links for data storage and alarm judgment. In the next section, we will explain in detail based on the architecture diagram.

### Nightingale architecture overview

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-arch.png)

- The `collector` is an agent, which can collect common indicators of the machine, support log monitoring, plug-in mechanism, and business can directly report data through the API interface.
- `Transfer` provides an rpc interface to receive the data reported by the collector, and then pass the data through a consistent hash algorithm, and then forward it to multiple tsdb and multiple judges.
- Tsdb-The original graph component, used to store historical data, supports configuration in double-write mode, which improves the system's disaster tolerance. At the same time, tsdb will forward a copy of monitoring data to index.
- `Index` is an index module, replaces the original mysql program. The `index` builds an index in memory, facilitates data retrieval, and greatly improves performance.
- `Judge` is an alarm engine. Judge synchronizes monitoring strategies from monapi (portal), and then makes alarm judgments by the received data. If the threshold is reached, an alarm event is generated and pushed to redis.
- `Monapi` (alarm) reads the event generated by the judge from redis, then performing secondary processing, adding some meta information, generating an alarm message, and pushes back to redis finally.
- Multiple sending components, such as mail-sender, sms-sender, etc., read alarm messages from redis, send alarms, and extract various senders to facilitate customization.
- Monapi integrates the functions of the original multiple modules and provides APIs for front-end calls. The api prefix is ​​`/api /portal`. We use transfer to query data. The original query component is removed. The api prefix is` /api /transfer` The api prefix of the query is `/api /index`, so using nginx, you can forward the request to different backends through different locations.
- The database still is mysql, and the main storage contents include: user information, team information, tree node information, alarm strategy, monitoring dashboard, shieldding strategy, collection strategy, heartbeat information of some components, etc.

*For more information，the video may be helpful：[Nightingale architecture overview](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-arch-intro.mp4)*

### Nightingale Roadmap

1、Provide monitoring index aggregation components. The current architecture can solve the machine-level and module-level monitoring, but the cluster-level monitoring indicators need to aggregate the indicators of all modules and machines in the entire cluster and do some operations such as summing and averaging. We already have some related components, but it will take time to release.

2、Support container monitoring, combine with Prometheus, or directly collect cadvisor data. There is no specific plan now and we will continue our efforts in this aspect.

3、Improve and support more monitoring plugins. Many plugins of the Open-Falcon community can be used directly. We will try to add plugins that the community does not have. For plugins that have been out of repair in the community, we will reorganize them to make Nightingale's surroundings more complete.
