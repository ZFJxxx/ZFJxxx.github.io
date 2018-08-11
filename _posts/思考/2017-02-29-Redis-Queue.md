---
layout: post
title:  Redis实现消息队列
date:   2017-02-29 19:15:10
categories:  思考
tags: 思考
keywords: interview
description: 
---

在完成问答系统时，发现在高并发的情况下系统还是会出现一些阻塞卡顿的情况，于是决定对整个系统进行异步化处理。当一个用户登录的时候，系统的登录模块就会对登录行为进行处理。在系统横向拓展，功能越来越丰富繁杂时，登录后就会引发连续性的其它模块对登录行为进行的链式处理。在系统架构功能越来越丰富复杂，横向功能拓展得越来越多的时候，登录功能可能会发生阻塞，但是用户主要的关注点还是更多的在我是否登录上，系统的阻塞可能会影响用户的体验，这时候就需要将整个系统异步化。
用户登录之后就向消息队列中发布一个事件，关注登录模块的模块并不马上对登录这个事件进行处理，而是不断地轮询消息队列中是否有自己关注的事件发生。当查询到自己关注的事件之后，才对该事件进行处理，这就是整个系统的异步化。这样的好处就是将整个系统进行解耦，模块之间并不会马上进行链式处理，而是通过队列，在系统资源允许的时候才对事件进行处理，将系统异步化之后能够极大的优化用户的体验，提高整个社交问答系统的并发性。
消息队列原理就是基于生产者和消费者的设计思想，生产者将事件发送到一个阻塞队列之中，然后不断轮询阻塞队列是否有自己关注的事件，如果有，就从队列中取出事件触发消费者进行处理。

![](http://p7lixluhf.bkt.clouddn.com/RedisQueue.PNG)

## Producer
发送者功能很简单，将消息发送到Redis的list中去就行了。
```
public class EventProducer {
    @Autowired
    JedisAdapter jedisAdapter;

    public boolean fireEvent(EventModel eventModel){
        try{
            String json = JSONObject.toJSONString(eventModel); //转化成JSON格式的字符串
            String key = RedisKeyUtil.getEventQueueKey();
            jedisAdapter.lpush(key,json);
            return true;
        }catch (Exception e){
            return false;
        }
    }
}
```

## 定义事件
```
public class EventModel {
    //发生的事件名称（比如点赞）
    private EventType type;
    //事件产生者（谁点赞）
    private int actorId;
    //事件发生的对象（对什么点赞）
    private int entityId;
    private int entityType;
    private int entityOwnerId;
}
```
EventType是枚举型的事件类型
```
public enum EventType {
    LIKE(0),
    COMMENT(1),
    LOGIN(2),
    MAIL(3),
    FOLLOW(4),
    UNFOLLOW(5);

    private int value;
    EventType(int value){
        this.value = value;
    }

    public int getValue(){
        return this.value;
    }
}
```

## Consumer
消费者就是开启一个线程，循环的从队列中取事件，取出来之后交给handler做相应的处理。

```
public class EventConsumer implements InitializingBean, ApplicationContextAware {
    private static final Logger logger = LoggerFactory.getLogger(EventConsumer.class);
    private Map<EventType, List<EventHandler>> config = new HashMap<EventType, List<EventHandler>>();
    private ApplicationContext applicationContext;

    @Autowired
    JedisAdapter jedisAdapter;

    @Override
    public void afterPropertiesSet() throws Exception {
        //通过Spring找到所有实现EventHandler的类，只要实现EventHandler就可以
        Map<String, EventHandler> beans = applicationContext.getBeansOfType(EventHandler.class);
        if (beans != null) {
            for (Map.Entry<String, EventHandler> entry : beans.entrySet()) {
                List<EventType> eventTypes = entry.getValue().getSupportEventTypes();

                for (EventType type : eventTypes) {
                    if (!config.containsKey(type)) {
                        config.put(type, new ArrayList<EventHandler>());
                    }
                    config.get(type).add(entry.getValue());
                }
            }
        }

        //不断的从队列中取出事件来处理，循环来查队列中是否有对应的Event
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while(true) {
                    String key = RedisKeyUtil.getEventQueueKey();
                    List<String> events = jedisAdapter.brpop(0, key);

                    for (String message : events) {
                        if (message.equals(key)) {
                            continue;
                        }

                        EventModel eventModel = JSON.parseObject(message, EventModel.class);
                        if (!config.containsKey(eventModel.getType())) {
                            logger.error("不能识别的事件");
                            continue;
                        }

                        for (EventHandler handler : config.get(eventModel.getType())) {
                            handler.doHandle(eventModel);
                        }
                    }
                }
            }
        });
        thread.start();
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
