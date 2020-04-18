---
title: "API"
linkTitle: "API"
weight: 9
date: 2020-04-17
description: >
  This section will introduce the APIs provided by Nightingale. You can use these APIs for secondary development and integration with internal systems. All operations on the page have corresponding APIs.
---

## Preface

1、The return of all interfaces is in the following format, the error message will be written into the `err` field, and the status code is all 200, unless the server fails.

```json
{
    "dat": {},
    "err": "error msg"
}
```

2、The relevant interfaces of `/api /portal` will be authenticated, using the basic auth method.

```bash
# For example, using curl can be tested like this:
curl -u root:1234 localhost/api/portal/self/profile
```

In the above example, it is assumed that the root account (password 1234) is used for testing. In the actual production environment, we can create several virtual accounts with permissions set to root, and use these virtual accounts as the authentication account of the caller.

3、There will be an nginx in front of each module of Nightingale, so the address of all interfaces is the nginx address by default. If you want to directly call the back-end module, it is also possible to pay attention to load balancing and failure retry.

4、For monapi related interfaces, you can refer to the file: `src /modules /monapi /http /routes /routes.go`，This file defines the path of all interfaces and the corresponding implementation method. If you feel that the interface is not enough or want to add some interfaces, you can submit a PR:-) The interface for querying index data is provided by the index module, the api path starts with `/api /index`, the interface for querying historical monitoring data is provided by the transfer module, and the api path starts with` /api /transfer`.

Below we will introduce some important interfaces. For other interfaces, please refer to the source code directly. After all, the code is not deceptive.