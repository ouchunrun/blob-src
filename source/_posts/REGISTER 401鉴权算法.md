---
title: REGISTER 401鉴权算法
date: 2021-6-20
tags: [sip]
---

###  response 鉴权计算方法

```
let ha1 = md5(username:realm:password)
let ha2 = md5(method:uri)
let response = md5(ha1:nonce:nc:cnonce:qop:ha2)
```

- `REGISTER`中的 `cnone` 和 `nc` 是根据`401`中的`qop`计算出来的，如果没有`qop`，那`REGISTER`也不需要带`cnone` 和 `nc`