# 1. springboot-dubbo架包依赖问题
- **问题描述**

springboot-dubbo使用时，parent 依赖为2.0以上时，server启动不了

- **解决方法**

将 parent 改为 springboot 1.5 版本，不要去依赖自己的父pom
