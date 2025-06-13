---
title: 好的设计（一）
date: 2024-09-22 10:45:15
tags: design
---

**业务：**

如下图所示，比如我们现在接到一个需求，目的是通过canal监听mysql服务中的数据变动，实时的将变动了的数据同步到其他地方，比如[elasticsearch](https://zhida.zhihu.com/search?content_id=238282288&content_type=Article&match_order=1&q=elasticsearch&zhida_source=entity), 比如发mq信息，比如触发一些业务...，等。我要展示的是给所有的这些业务，用合理的设计模式写出来一个开闭性很好基底。

![img](https://pica.zhimg.com/v2-074a1b61dd58c8b97c9256a675f6739c_1440w.jpg)

**设计：**

　　该项目中[SpringBoot](https://zhida.zhihu.com/search?content_id=238282288&content_type=Article&match_order=1&q=SpringBoot&zhida_source=entity)一定会有一个Listenner主入口，只要canal监听mysql稍微有点打草惊蛇，我们都能够在这个Listenner中感知到，从而根据相应的事件做出对应的动作。请看伪代码:

```java
/**
     * 数据变动监听业务
     * @param beforeColumns 更新前的数据
     * @param afterColumns 更新后的数据
     * @param tableName 表名
     * @param eventType 操作类型 [insert,update,delete]
     * @param timestamp 数据发生变化时的时间戳
     */
public class CanalListener {
      public void listen(List<CanalEntry.Column> beforeColumns, List<CanalEntry.Column> afterColumns, String tableName, CanalEntry.EventType eventType, long timestamp) {
          if("user_info".equals(tableName))  {
              if (CanalEntry.EventType.INSERT.equals(eventType)) {
                //todo for insert
              } else if (CanalEntry.EventType.UPDATE.equals(eventType)) {
                //todo for update
              } else if (CanalEntry.EventType.DELETE.equals(eventType)) {
               //todo for delete
              }
          }
      }
}
```

看完后会发现一个设计上的问题: 目前是只维护了user_info表，那如果后面的迭代中还需要添加order表的监听逻辑，那还要往下堆下去吗? 假如过三年后这里维护的表已经又多了十几二十张表呢? 就会出现这样的一种情况: 很多if，if中if, if中有业务，那个时候的代码一定很"博大精神"，此乃"if"生一，一生二，三生万物，"万物"犹如火葬场。

**所以，与其**把全部东西堆进来到同一个位置，不如封装一个工厂，工厂可以根据表的名称给我们一个对应的表的处理器，处理器中update,insert,delete事件的处理方法都有，于是Listenner就可以这样写(优化后的伪代码):

```java
/**
   * 数据变动监听业务
   * @param beforeColumns 更新前的数据
   * @param afterColumns 更新后的数据
   * @param tableName 表名
   * @param eventType 操作类型 [insert,update,delete]
   * @param timestamp 数据发生变化时的时间戳
   */
public class CanalListener {
    public void listen(List<CanalEntry.Column> beforeColumns, List<CanalEntry.Column> afterColumns, String tableName, CanalEntry.EventType eventType, long timestamp) {
      //获取处理器
      TableHandler handler = TableHandlerFactory.getHandler(tableName);
      if (ObjectUtil.isEmpty(handler)) return;

      //执行各自的业务
      ListenEntity listenEntity = new ListenEntity(afterColumns, beforeColumns, timestamp); //监听到的数据对象
      if (CanalEntry.EventType.INSERT.equals(eventType)) {
          handler.insert(listenEntity);
      } else if (CanalEntry.EventType.UPDATE.equals(eventType)) {
          handler.update(listenEntity);
      } else if (CanalEntry.EventType.DELETE.equals(eventType)) {
          handler.delete(listenEntity);
      }
    }
}
```

核心监听器代码就这么点，少，精。如果想介入一个新的表要监听，则只需要以TableHandler为模板实现其insert,update,delete模板方法，将自己注册到工厂就好。

以下是工厂和处理器的实现思路，首先是工厂，工厂维护了一个HashMap, 从工厂中根据表名获取处理对象的时候，实际上是从hashMap中取的。注册处理器也是对hashMap的put操作。

```java
// 工厂
public class TableHandlerFactory {

    public static Map<String, TableHandler> tableNameHandlerMap = new HashMap();

    // 把表handler注册到TableHandlerFactory中
    public static void register(String tableName, TableHandler tableHandler) {

        if (StrUtil.isEmpty(tableName)) {
            throw new IllegalArgumentException("Registered tableName is empty");
        } else if (ObjectUtil.isEmpty(tableHandler)) {
            throw new IllegalArgumentException("Registered tableHandler is empty");
        }

        tableNameHandlerMap.put(tableName,tableHandler);
    }

    // 根据表名获取处理器
    public static TableHandler getHandler(String tableName) {
        if (StrUtil.isEmpty(tableName)) {
            throw new IllegalArgumentException("illegal input for get tableHandler,tableName is empty");
        }
        return tableNameHandlerMap.get(tableName);
    }
}
```

为什么这里TableHandler我用了抽象类而不是接口，好处是抽象类有默认的实现，当一个处理器没有实现抽象类的某个方法比如delete, 却在业务中调用了delete那就会默认抛出UnsupportedOperationException，或者还可以写成抛出RunTimeException("该方法没有被实现");

```java
public abstract class TableHandler implements InitializingBean {

    public void insert(ListenEntity listenEntity) {
        throw new UnsupportedOperationException();
    }

    public void update(ListenEntity listenEntity) {
        throw new UnsupportedOperationException();
    }

    public void delete(ListenEntity listenEntity) {
        throw new UnsupportedOperationException();
    }

}
```

表的业务逻辑都在handler处理器中

```java
@Component
public class OrderTableHandler extends TableHandler {

    private String tableName = "t_order";

    @Override
    public void insert(ListenEntity listenEntity) {
        //todo
    }

    @Override
    public void update(ListenEntity listenEntity) {
        //todo
    }

    @Override
    public void delete(ListenEntity listenEntity) {
        //todo
    }

    // Spring启动时注册自己到Factory工厂
    @Override
    public void afterPropertiesSet() throws Exception {
        TableHandlerFactory.register(tableName, this);
    }
}
```

Surprise! that's it.

其实我自己都不能很清楚的说出来这里用了哪些设计模式，我理解的设计模式是用好当下编程语言的特性，将接口，抽象类，类...以及任何某种特点or组件，把它们很好的结合起来，把业务设计底架变魔术一样恰到好处的实现，这是我个人对设计的一个认识。同时，设计是灵活的，对同一样东西的设计因人百花齐放，所以上述实现只是多种设计中的一种。

总之，不蛮干，祝您生活愉快。
