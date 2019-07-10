---
title: JSON使用经验之java对象与json字符串的互转问题
date: 2019-07-09 16:26:49
categories: json
tags: json
---

## java对象转json字符串


### fastJson实现方法
```
SerializeConfig config = new SerializeConfig();
        config.propertyNamingStrategy = PropertyNamingStrategy.SnakeCase;
        String json = JSON.toJSONString(app, config);
        System.out.println("fastJson---" + json);

console result:
        fastJson---{"app_id":431431,"image_url":"http://www.image.com/xxx.png","link_url":"http://www.baidu.com","name":"美团外卖"}
```
### Jackson实现方法
```
ObjectMapper mapper = new ObjectMapper();
        mapper.setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
        String json = mapper.writeValueAsString(app);
        System.out.println("jackson--" + json);

console result:
      jackson--{"app_id":431431,"name":"美团外卖","image_url":"http://www.image.com/xxx.png","link_url":"http://www.baidu.com"}
```
### Gson实现方法
```
 GsonBuilder gsonBuilder = new GsonBuilder();
        gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
        Gson gson = gsonBuilder.create();

        String json = gson.toJson(app);
        System.out.println("Gson--" + json);

console result:
      Gson--{"app_id":431431,"name":"美团外卖","image_url":"http://www.image.com/xxx.png","link_url":"http://www.baidu.com"}
```

## json字符串转java对象


### fastJson实现方法
```
 App app1 = JSON.parseObject(jsonStr, App.class);
```
### Jackson实现方法
```
 ObjectMapper mapper = new ObjectMapper();
        mapper.setPropertyNamingStrategy(PropertyNamingStrategy.LOWER_CAMEL_CASE);
        App app = mapper.readValue(jsonStr, App.class);
```
### Gson实现方法
```
        Gson gson = new Gson();
        Person person = gson.fromJson(json, Person.class)
```

