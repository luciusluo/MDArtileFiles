#前言

有时候根据系统的版本进行条件编译，但是经常会忘记这个长长的系统所提供的宏，记在博客吧！


```
__IPHONE_OS_VERSION_MAX_ALLOWED
```

#如何用

```

#if __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_7_0
- (BOOL)isConcurrent {
  return YES;
}

#else

- (BOOL)isAsynchronous {
  return YES;
}
#endif
```

#最后

忘记就过来搜索吧！