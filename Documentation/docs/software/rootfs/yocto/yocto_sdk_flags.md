修改 `build/conf/local.conf`，增加或者覆盖如下变量

```
DEBUG_BUILD = "0" 
DEBUG_FLAGS = ""                                                                
FULL_OPTIMIZATION = "-O3 -pipe"
```

这些变量原始定义于 `poky/meta/conf/bitbake.conf ` **609** 行附近