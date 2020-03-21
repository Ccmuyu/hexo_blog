# dubbo源码精读（一）

dubbo作为java rpc框架中的一员，重要性就不多说了，每次出去跟人交流也总提到这个，虽然项目中一直在用，可却从来没完整的、系统的学习过dubbo。今天开始迈出dubbo第一步：

关于dubbo的额接本概念解释这里不做过多介绍，可以参考这篇博文,从架构层面理解dubbo:

dubbo的底层原理：https://blog.csdn.net/qq_33101675/article/details/78701305    

下图为我在项目中的dubbo2.5.3jar包目录结构

![dubbo目录](/img/dubbo/dir.png)

## hession 包

dubbo支持多种序列化方式，有dubbo、hessian、java、json、nativejava，（具体下文会介绍），hessian是dubbo默认的序列化方式。

dubbo这里是直接使用hessian的一个开源实现放到了com.alibaba包下，也即是上图中的con.caucho.hessian包。这个包中主要是io包，hessian对于数据的序列化和反序列化所有实现全部在这里。其余两个包是RSA加密相关和工具类。有兴趣可以参考下，个人认为没有必要深究

不出意外。序列化的顶级接口：

```
package com.alibaba.com.caucho.hessian.io;

import java.io.IOException;

/**
 * Serializing an object. 
 */
public interface Serializer {
  public void writeObject(Object obj, AbstractHessianOutput out)
    throws IOException;
}

```

hessian序列化的具体实现代码在com.alibaba.com.caucho.hessian.io.Hessian2Output中。这里不在展示源码和贴图，简单来说就是将（byte[]） 数组转换为二进制数据，并写入流中。反序列化则刚好相反，将二进制数据转化字符编码，进而转化为java的数据类型。

反序列化的顶级接口：

```
package com.alibaba.com.caucho.hessian.io;

import java.io.IOException;

/**
 * Deserializing an object. 
 */
public interface Deserializer {
  public Class getType();

  public Object readObject(AbstractHessianInput in)
    throws IOException;
  
  public Object readList(AbstractHessianInput in, int length)
    throws IOException;
  
  public Object readLengthList(AbstractHessianInput in, int length)
    throws IOException;
  
  public Object readMap(AbstractHessianInput in)
    throws IOException;
  
  public Object readObject(AbstractHessianInput in, String []fieldNames)
    throws IOException;
}

```

## dubbo包

到这里，才是真正dubbo的核心实现。从前面的图中可以看到，dubbo的实现主要包含这些内容：

1. cache：本地缓存，缓存服务列表

2. common：资源工具类和注解等，包括io、日志、拓展类、JSON、线程池、各种工具类

3. config：dubbo配置的解析器，包括元注解、xml配置解析器及解析对象的定义，以下代码为xml解析的入口。

   ```
   package com.alibaba.dubbo.config.spring.schema;
   
   import org.springframework.beans.factory.xml.NamespaceHandlerSupport;
   
   import com.alibaba.dubbo.common.Version;
   import com.alibaba.dubbo.config.ApplicationConfig;
   import com.alibaba.dubbo.config.ConsumerConfig;
   import com.alibaba.dubbo.config.ModuleConfig;
   import com.alibaba.dubbo.config.MonitorConfig;
   import com.alibaba.dubbo.config.ProtocolConfig;
   import com.alibaba.dubbo.config.ProviderConfig;
   import com.alibaba.dubbo.config.RegistryConfig;
   import com.alibaba.dubbo.config.spring.AnnotationBean;
   import com.alibaba.dubbo.config.spring.ReferenceBean;
   import com.alibaba.dubbo.config.spring.ServiceBean;
   
   /**
    * DubboNamespaceHandler
    * 
    * @author william.liangf
    * @export
    */
   public class DubboNamespaceHandler extends NamespaceHandlerSupport {
   
   	static {
   		Version.checkDuplicate(DubboNamespaceHandler.class);
   	}
   
   	public void init() {
   	    registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
           registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
           registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
           registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
           registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
           registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
           registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
           registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
           registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
           registerBeanDefinitionParser("annotation", new DubboBeanDefinitionParser(AnnotationBean.class, true));
       }
   
   }
   ```

   

4. container：对spring、log4j、logback、servlet等的支持

5. monitor：对provider、consumer的监控和数据统计，核心的实现在类DubboMonitor中

6. registry：服务的注册与发现，其中服务支持redis和zookeeper两种，如果有需要也可以自己实现注册中心，实现registryService和注册工厂即可。zk的注册实现在ZookeeperRegistry中。这也是项目中最常用的注册中心

7. remoting：主要提供了一些远程通信的实现，包括http、p2p、telnet。还集成了zookeeper的客户端

8. rpc：这是dubbo作为一个服务调用框架的核心功能。包含了对集群的处理、监听、通信协议、代理的实现、过滤器、rpc容器与调用等

今天就先到这里，后续对rpc继续写个专题。