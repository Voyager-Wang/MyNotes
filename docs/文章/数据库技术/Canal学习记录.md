## 启动-deployer

### 1.CanalLauncher

主要作用：

- 启动CanalStarter

### 2.CanalStarter

主要作用：

- 启动CanalController
- 

待确认：配置项canal.serverMode，默认tcp，支持kafka，rocketmq

补充：MigrateMap.makeComputingMap作用，如果map中没有对应的key，则通过get(key)获取元素时返回apply返回的值

```java
managerClients = MigrateMap.makeComputingMap(new Function<String, PlainCanalConfigClient>(){
  public PlainCanalConfigClient apply(String managerAddress) {
    return getManagerClient(managerAddress);
  }
});
```

### 3.CanalController

主要作用：

- 维护CanalServer

待确认：配置InstanceMode：SPRING或者MANAGER