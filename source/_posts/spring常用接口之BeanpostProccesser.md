---
title: spring常用接口之BeanpostProccesser
date: 2019-06-18 16:54:06
categories: spring框架
tags: spring
---

## springBean生命周期

作用：BeanPostProcessor在bean初始化前后做自定义处理

场景：需要自定义swagger,使所有请求显示固定的请求头参数(自定义Docket类)，但是swagger被架构组封装到sdk中，所以在BeanPostProcessor中对Docket的bean做处理


```
/**
 * 自定义swagger
 *
 * @author xiantao.wu
 * @create 2019/3/269:51
 **/
@Component
public class GiveAppBeanPostProcessor implements BeanPostProcessor {
    //bean初始化之前的操作
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    //bean初始化之后的操作
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        //swagger统一增加请求头
        if ("docket".equals(beanName)) {
            return builderHeader((Docket) bean);
        }
        return bean;
    }

    private Docket builderHeader(Docket docket) {
        List<Parameter> pars = new TreeList();
        pars.add(new ParameterBuilder().name("x-os-name").description("系统类型").defaultValue("iOS")
                .modelRef(new ModelRef("string")).parameterType("header").required(true).build());

        pars.add(new ParameterBuilder().name("x-os-version").description("系统版本").defaultValue("10.3.3")
                .modelRef(new ModelRef("string")).parameterType("header").required(true).build());

        return docket.globalOperationParameters(pars);
    }
}

```


