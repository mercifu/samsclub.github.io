---
layout:     post
title:      parray 转换成 lpwstr
subtitle:   
description: parray 转换成 lpwstr 
date:       2019-07-03
author:     Sam
header-img: 
catalog: true
tags:
    - C++
    - ISSUE
---

```C++
LONG lUBound;
LONG lLBound;
SafeArrayGetLBound(  parray, 1, &lLBound );
SafeArrayGetUBound(  parray, 1, &lUBound );

VARIANT* pvarTemp;
SafeArrayAccessData( parray, (void**) &pvarTemp );

std::wstring description;

for (LONG lCount = lLBound; lCount <= lUBound; ++lCount, ++pvarTemp)
{
    std::wstring val = std::wstring(pvarTemp->bstrVal) + L" ";
    description = val + description ;
}

SafeArrayUnaccessData( parray );

LPWSTR descriptions = (LPWSTR)(description.c_str());
```