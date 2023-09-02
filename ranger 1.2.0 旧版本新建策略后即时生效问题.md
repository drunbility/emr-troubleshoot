
### 现象

用户反馈ranger 在创建一条策略后立刻访问ranger 获取策略接口会返回404，这时新建一条策略会提示与已存在策略冲突

因为用户场景是通过 rest api 去更新 ranger 的策略的:

```
目前更新策略A的调用流程
1 获取A策略，查看策略是否存在 GET http://ip:port/service/public/v2/api/service/hive/policy/策略名A
2 删除A策略,   DELETE  http://ip:port/service/public/v2/api/policy/策略id
3 创建A策略， POST http://ip:port/service/plugins/policies  

当按照这个流程 连续更新A策略时，会出现耗时和缓存问题
作为参考，我们线上ranger 的策略量为 2万左右
```

### 结论


问题触发原因:
1. ranger-1.2.0中schema设计不合理,policy的多个属性使用了多张表来存储，导致每次生成policy会经过多次查询才能完成
2. ranger中更新策略后，ranger server会在内存中rebuild policy缓存
3. 在缓存rebuild完成前，如果有读取请求到达，会等待ranger.admin.policy.download.cache.max.waittime.for.update指定的时间，默认10s
4. 目前有2W+的策略，预计需要分钟级别重建完成

建议方法:
1. 不建议调整ranger.admin.policy.download.cache.max.waittime.for.update参数，该参数调整后，会导致ranger所有读取请求失败
2. 获取policy时，是否只需要判断 具有该policyName的策略存在，如果只需要判断是否存在而不关心策略具体内容，可以提供一个只判断是否policy存在的API，直接查库，绕开缓存
3. 或者将查询、增加的循环流程间隔调大

其他思路:
1. 考虑过将ranger-1.2.0的元数据升级到ranger-2.0.0，减少rebuild过程对数据库的方案，提升效率，相关ISSUE https://issues.apache.org/jira/browse/RANGER-2203,但schema变更过大，社区暂时无迁移方案

---
后续提供了,整体的 ranger 性能优化升级
见文档  **ranger性能问题改进方案**

