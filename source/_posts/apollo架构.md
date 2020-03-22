---
title: apollo架构
date: 2020-03-22 16:58:07
tags: 
    - apollo
    - 动态配置中心
---

# apollo简介
    前段时间公司项目需要，由传统的静态打包配置切换到了动态配置中心，在此总结一些使用心得。
    
github项目地址： [apollo](https://github.com/ctripcorp/apollo) 

理论还是要了解的，有助于快速理解整体架构。这里主要从架构和客户端使用原理入手



# 架构原理

引用作者的架构图：

<!-- more -->

![img](https://res.infoq.com/articles/ctrip-apollo-configuration-center-architecture/zh/resources/1apollo_by_songshun-1528302390687.png)



### 四个核心模块及其主要功能

1. ConfigService

- 提供配置获取接口
- 提供配置推送接口
- 服务于Apollo客户端

2. AdminService

- 提供配置管理接口
- 提供配置修改发布接口
- 服务于管理界面Portal

3. Client

- 为应用获取配置，支持实时更新
- 通过MetaServer获取ConfigService的服务列表
- 使用客户端软负载SLB方式调用ConfigService

4. Portal

- 配置管理界面
- 通过MetaServer获取AdminService的服务列表
- 使用客户端软负载SLB方式调用AdminService

### 三个辅助服务发现模块

1. Eureka

- 用于服务发现和注册
- Config/AdminService注册实例并定期报心跳
- 和ConfigService住在一起部署

2. MetaServer

- Portal通过域名访问MetaServer获取AdminService的地址列表
- Client通过域名访问MetaServer获取ConfigService的地址列表
- 相当于一个Eureka Proxy
- 逻辑角色，和ConfigService住在一起部署

3. NginxLB

- 和域名系统配合，协助Portal访问MetaServer获取AdminService地址列表
- 和域名系统配合，协助Client访问MetaServer获取ConfigService地址列表
- 和域名系统配合，协助用户访问Portal进行配置管理



简易架构图：

![img](https://res.infoq.com/articles/ctrip-apollo-configuration-center-architecture/zh/resources/1apollo_arch_v1-1528302391929.png)

要点：

1. ConfigService是一个独立的微服务，服务于Client进行配置获取。
2. Client和ConfigService保持长连接，通过一种拖拉结合(push & pull)的模式，实现配置实时更新的同时，保证配置更新不丢失。
3. AdminService是一个独立的微服务，服务于Portal进行配置管理。Portal通过调用AdminService进行配置管理和发布。
4. ConfigService和AdminService共享ConfigDB，ConfigDB中存放项目在某个环境的配置信息。ConfigService/AdminService/ConfigDB三者在每个环境(DEV/FAT/UAT/PRO)中都要部署一份。
5. Protal有一个独立的PortalDB，存放用户权限、项目和配置的元数据信息。Protal只需部署一份，它可以管理多套环境。



**V5 版本架构图示：**

![img](https://res.infoq.com/articles/ctrip-apollo-configuration-center-architecture/zh/resources/1apollo_arch_v5-1528302391563.png)

# 源码解析

客户端

spring接入：
$$
http\://www.ctrip.com/schema/apollo=com.ctrip.framework.apollo.spring.config.NamespaceHandler
$$
spring boot接入：
$$
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.ctrip.framework.apollo.spring.boot.ApolloAutoConfiguration
org.springframework.context.ApplicationContextInitializer=\
com.ctrip.framework.apollo.spring.boot.ApolloApplicationContextInitializer
org.springframework.boot.env.EnvironmentPostProcessor=\
com.ctrip.framework.apollo.spring.boot.ApolloApplicationContextInitializer
$$


以spring使用为例：

```java
public class NamespaceHandler extends NamespaceHandlerSupport {
    private static final Splitter NAMESPACE_SPLITTER = Splitter.on(",").omitEmptyStrings().trimResults();

    @Override
    public void init() {
    	//解析器注册，支持apollo的xml标签
        registerBeanDefinitionParser("config", new BeanParser());
    }

	//xml解析器
    static class BeanParser extends AbstractSingleBeanDefinitionParser {
        @Override
        protected Class<?> getBeanClass(Element element) {
            //属性
            return ConfigPropertySourcesProcessor.class;
        }

        @Override
        protected boolean shouldGenerateId() {
            return true;
        }

        @Override
        protected void doParse(Element element, BeanDefinitionBuilder builder) {
            String namespaces = element.getAttribute("namespaces");
            //default to application
            if (Strings.isNullOrEmpty(namespaces)) {
            //所以默认会处理application配置
                namespaces = ConfigConsts.NAMESPACE_APPLICATION;
            }
			//优先级
            int order = Ordered.LOWEST_PRECEDENCE;
            String orderAttribute = element.getAttribute("order");

            if (!Strings.isNullOrEmpty(orderAttribute)) {
                try {
                    order = Integer.parseInt(orderAttribute);
                } catch (Throwable ex) {
                    throw new IllegalArgumentException(
                            String.format("Invalid order: %s for namespaces: %s", orderAttribute, namespaces));
                }
            }
            PropertySourcesProcessor.addNamespaces(NAMESPACE_SPLITTER.splitToList(namespaces), order);
        }
    }
}

```



NamespaceHandlerSupport 是spring xml配置的解析抽象类，解析结果为BeanDefinition，这是spring bean实例的描述类，描述信息包含类的构造器，成员变量等



ConfigPropertySourcesProcessor ：将加载的BeanDefinition 处理，apollo拦截了spring bean的注册过程，所以在获取到apollo配置的属性值后，最终完成向容器注册的过程。

以下是初始化启动时的的配置过程

```java
// BeanRegistrationUtil
   }

        BeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(beanClass).getBeanDefinition();

        if (extraPropertyValues != null) {
            for (Map.Entry<String, Object> entry : extraPropertyValues.entrySet()) {
                beanDefinition.getPropertyValues().add(entry.getKey(), entry.getValue());
            }
        }

        registry.registerBeanDefinition(beanName, beanDefinition);
```

动态配置中心最重要是就是支持配置信息的实时变化，apollo在启动时通过spi，注入以下实例

```java
com.ctrip.framework.apollo.internals.DefaultInjector
```

```java
public class DefaultInjector implements Injector {
    private com.google.inject.Injector m_injector;

    public DefaultInjector() {
        try {
            m_injector = Guice.createInjector(new ApolloModule());
        } catch (Throwable ex) {
            ApolloConfigException exception = new ApolloConfigException("Unable to initialize Guice Injector!", ex);
            Tracer.logError(exception);
            throw exception;
        }
    }

 // 省略。。。

    private static class ApolloModule extends AbstractModule {
        @Override
        protected void configure() {
            bind(ConfigManager.class).to(DefaultConfigManager.class).in(Singleton.class);
            bind(ConfigFactoryManager.class).to(DefaultConfigFactoryManager.class).in(Singleton.class);
            bind(ConfigRegistry.class).to(DefaultConfigRegistry.class).in(Singleton.class);
            bind(ConfigFactory.class).to(DefaultConfigFactory.class).in(Singleton.class);
            bind(ConfigUtil.class).in(Singleton.class);
            bind(HttpUtil.class).in(Singleton.class);
            bind(ConfigServiceLocator.class).in(Singleton.class);
            //这个长轮询服务就是监听远程仓库的配置信息变化
            bind(RemoteConfigLongPollService.class).in(Singleton.class);
        }
    }
}
```



长轮询服务源码实现如下

```java
private void doLongPollingRefresh(String appId, String cluster, String dataCenter) {
    final Random random = new Random();
    ServiceDTO lastServiceDto = null;
    while (!m_longPollingStopped.get() && !Thread.currentThread().isInterrupted()) {
        if (!m_longPollRateLimiter.tryAcquire(5, TimeUnit.SECONDS)) {
            //wait at most 5 seconds
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
            }
        }
      //省略。。。
            HttpRequest request = new HttpRequest(url);
            request.setReadTimeout(LONG_POLLING_READ_TIMEOUT);

            transaction.addData("Url", url);

            final HttpResponse<List<ApolloConfigNotification>> response =
                    m_httpUtil.doGet(request, m_responseType);

            logger.debug("Long polling response: {}, url: {}", response.getStatusCode(), url);
            if (response.getStatusCode() == 200 && response.getBody() != null) {
                //更新本地bean的spring value
                updateNotifications(response.getBody());
                updateRemoteNotifications(response.getBody());
                transaction.addData("Result", response.getBody().toString());
                notify(lastServiceDto, response.getBody());
            }

            //try to load balance
            if (response.getStatusCode() == 304 && random.nextBoolean()) {
                lastServiceDto = null;
            }

            m_longPollFailSchedulePolicyInSecond.success();
         //省略。。。。
    }
}
```

当长轮询收到配置变更之后，触发本地与远程的同步任务

com.ctrip.framework.apollo.internals.RemoteConfigRepository#sync

```java
protected synchronized void sync() {
    Transaction transaction = Tracer.newTransaction("Apollo.ConfigService", "syncRemoteConfig");

    try {
        ApolloConfig previous = m_configCache.get();
        ApolloConfig current = loadApolloConfig();
		//虽然304不会更新通知，但是每次更新后会缓存在本地，这里仅作安全验证
        //reference equals means HTTP 304
        if (previous != current) {
            logger.debug("Remote Config refreshed!");
            m_configCache.set(current);
            this.fireRepositoryChange(m_namespace, this.getConfig());
        }

        if (current != null) {
            Tracer.logEvent(String.format("Apollo.Client.Configs.%s", current.getNamespaceName()),
                    current.getReleaseKey());
        }

        transaction.setStatus(Transaction.SUCCESS);
    } catch (Throwable ex) {
        transaction.setStatus(ex);
        throw ex;
    } finally {
        transaction.complete();
    }
}
```



3种spring value 变更事件

```java
//1
public class ConfigFileChangeEvent {
    private final String namespace;
    private final String oldValue;
    private final String newValue;
    private final PropertyChangeType changeType;
}
//2
public class ConfigChangeEvent {
    private final String m_namespace;
    private final Map<String, ConfigChange> m_changes;
}
//3
public class ConfigChange {
    private final String namespace;
    private final String propertyName;
    private String oldValue;
    private String newValue;
    private PropertyChangeType changeType;
}
```



处理spring value值变更

com.ctrip.framework.apollo.spring.property.AutoUpdateConfigChangeListener

```java
public void onChange(ConfigChangeEvent changeEvent) {
    Set<String> keys = changeEvent.changedKeys();
    if (CollectionUtils.isEmpty(keys)) {
        return;
    }
    for (String key : keys) {
        //这里的对于spring容器的bean处理非常精髓，每个IOC容器作为map的一个value，同时每一个key对应的所有bean实例value引用
        // 1. check whether the changed key is relevant
        Collection<SpringValue> targetValues = springValueRegistry.get(beanFactory, key);
        if (targetValues == null || targetValues.isEmpty()) {
            continue;
        }

        // 2. check whether the value is really changed or not (since spring property sources have hierarchies)
        if (!shouldTriggerAutoUpdate(changeEvent, key)) {
            continue;
        }

        // 3. update the value
        for (SpringValue val : targetValues) {
            updateSpringValue(val);
        }
    }
}

//更新字段和方法
public void update(Object newVal) throws IllegalAccessException, InvocationTargetException {
        if (isField()) {
            injectField(newVal);
        } else {
            injectMethod(newVal);
        }
    }

    private void injectField(Object newVal) throws IllegalAccessException {
        Object bean = beanRef.get();
        if (bean == null) {
            return;
        }
        boolean accessible = field.isAccessible();
        field.setAccessible(true);
        field.set(bean, newVal);
        field.setAccessible(accessible);
    }
```





至此对于apollo客户端的实现基本结束了，上面列出的代码是配置变更的核心实现，可以看出确实逻辑相对简单，实现也很巧妙，最适合新手入门的中间件我是认可的。

# 使用指南：





apollo架构相比其他中间件而言，没有使用太多高深技巧，正如官网上所说的这可能是最适合初学者通过源码来学习的分布式中间件产品

## 客户端

参见github官网wiki

[客户端wiki](https://github.com/ctripcorp/apollo/wiki/Java客户端使用指南)