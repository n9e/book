---
title: "Index"
linkTitle: "Index"
date: 2020-04-12
description: >
  This section explains the operations related to monitoring indexes
---

## Clean up the monitoring index
Some monitoring indicators will remain on the page for a period of time after the user no longer reports (the default is 1 day). Some users may feel that it affects the operation, so we provide two interfaces for deleting indexes, which can delete indicators that are no longer reported.

#### 1. Delete the index according to endpoint + metric
```json
//body
{
	"endpoints":["127.0.0.1"],
	"metrics":["cpu.idle"]
}
```
```bash 
curl -d body http://index.addr/api/index/metrics
```
#### 2. Delete the index according to endpoint + metric + tagkv
```json 
//body
{
	"endpoints":["127.0.0.1"],
	"metric":"cpu.idle",
	"tagkv":[
		{
			"tagk":"",
			"tagv":[""]
		}
	]
}
```
```bash 
curl -d body http://index.addr/api/index/counter
```

## Rebuild the monitoring index
When the index restarts, the index data of the memory will be lost. The tsdb module caches the index sent to the index, so it will not be sent repeatedly in the short term. Although index has data persistence, when there is only one instance of index, when the process terminates, some new index data will be lost if it does not have time to be placed, resulting in the index not being found. Therefore, we provide an interface to rebuild the full index. By calling the interface, you can notify the tsdb module to update the full index of the monitoring indicators in its memory.

```bash 
curl http://tsdb.addr/api/tsdb/update-index
```
