# SOFABoot 动态模块实践

## 实验背景

[kc-sofastack-demo](https://github.com/sofastack-guides/kc-sofastack-demo) 分享中已经通过 SOFAStack 快速构建了一个电商微服务应用，
并且完成了对应用服务调用链路的跟踪及应用状态的监控。

在电商系统中，平台方往往不会满足商品的自然排序展示，必定会根据某种规则来将部分商品放置在列表最瞩目的地方，
当然也可能是平台方通过收集用户行为动态的为每个不同的用户推荐不同的商品展示列表。

本实验背景就是基于[kc-sofastack-demo](https://github.com/sofastack-guides/kc-sofastack-demo)的基础上，
根据现场同学对每个商品的购买总数（通过订单统计）来对商品列表进行动态排序。

## 实验内容

通过 SOFABoot 提供的动态模块能力及 SOFADashboard 的动态模块管控能力，实现商品列表排序策略的动态变更。通过在不重启宿主机，不更改应用配置的情况下实现
应用行为的改变。

* 项目工程架构图如下

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*ECEjR5hY0h0AAAAAAAAAAABkARQnAQ)

## 任务

### 1、任务准备

从 github 上将 demo 工程克隆到本地

```bash
git clone https://github.com/sofastack-guides/kc-sofastack-dynamic-demo.git
```

然后将工程导入到 IDEA 或者 eclipse。

### 2、将 SOFABoot 应用打包成 ark 包



#### step1 : 修改动态模块名称

> 在实际的应用场景中，不需要对其进行任何修改

如下图所示，对 dynamic-module/pom.xml 中的 artifactId 进行修改，将 {your-number} 修改为当前座位上的编号

![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*3aiqQpJL7VwAAAAAAAAAAABkARQnAQ)

#### step2 : 配置动态模块的打包插件

在 dynamic-module/pom.xml 中，增加 ark 打包插件，并进行配置：

![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*y2BvRKG14JUAAAAAAAAAAABkARQnAQ)


```xml
<plugin>
  <groupId>com.alipay.sofa</groupId>
  <artifactId>sofa-ark-maven-plugin</artifactId>
  <version>0.6.0</version>
  <executions>
    <execution>
      <!--goal executed to generate executable-ark-jar -->
      <goals>
        <goal>repackage</goal>
      </goals>
      <!--ark-biz 包的打包配置  -->
      <configuration>
        <!--是否打包、安装和发布 ark biz，详细参考 Ark Biz 文档，默认为false-->
        <attach>true</attach>
        <!--ark 包和 ark biz 的打包存放目录，默认为工程 build 目录-->
        <outputDirectory>target</outputDirectory>
        <!--default none-->
        <arkClassifier>executable-ark</arkClassifier>
        <!-- ark-biz 包的启动优先级，值越小，优先级越高-->
        <priority>200</priority>
        <!--设置应用的根目录，用于读取 ${base.dir}/conf/ark/bootstrap.application 配置文件，默认为 ${project.basedir}-->
        <baseDir>../</baseDir>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 3、构建宿主应用

在已下载下来的工程中，dynamic-stock-mng 作为实验的宿主应用工程模型。通过此任务，将 dynamic-stock-mng  构建成为动态模块的宿主应用。

#### step1 : 引入动态模块依赖

> 动态模块是通过 SOFAArk 组件来实现的，因此次数需要引入 SOFAArk 相关的依赖即可。关于 SOFAArk 可以参考[SOFABoot 类隔离](https://www.sofastack.tech/projects/sofa-boot/sofa-ark-readme/)
一节进行了解。

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*lM_1SoNIXIYAAAAAAAAAAABkARQnAQ)

* SOFAArk 相关依赖

    ```xml
    <dependency>
      <groupId>com.alipay.sofa</groupId>
      <artifactId>sofa-ark-springboot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>com.alipay.sofa</groupId>
      <artifactId>web-ark-plugin</artifactId>
    </dependency>
    <dependency>
      <groupId>com.alipay.sofa</groupId>
      <artifactId>config-ark-plugin</artifactId>
    </dependency>
      <dependency>
         <groupId>io.sofastack</groupId>
         <artifactId>dynamic-provider-{your-number}</artifactId>
         <version>1.0.0</version>
         <classifier>ark-biz</classifier>
     </dependency>
    ```
    将此配置文件中的 {your-number} 替换为当前座位编号
    
* 宿主应用打包插件

    ```xml
    <plugin>
        <groupId>com.alipay.sofa</groupId>
      <artifactId>sofa-ark-maven-plugin</artifactId>
      <version>0.6.0</version>
      <executions>
        <execution>
          <id>default-cli</id>
          <goals>
            <goal>repackage</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <priority>100</priority>
        <baseDir>../</baseDir>
        <bizName>stock-mng-{your-number}</bizName>
      </configuration>
    </plugin>
    ```
    
    将打包插件中的 {your-number} 替换为当前座位上的编号，同样在实际的场景中是不需要的。这里是希望通过应用名来进行隔离，已达到各位在实际操作中不会相互干扰。

#### step2 : 宿主应用配置

* 动态模块配置
 
    在当前项目的根目录 /conf/ark/bootstrap.properties 配置文件中添加配置如下：
    
    ```properties
    # 日志根目录
    logging.path=./logs
    # 配置服务器地址
    com.alipay.sofa.ark.config.address=zookeeper://116.62.20.143:2181,116.62.148.186:2181,121.43.174.16:2181
    # 宿主应用名
    com.alipay.sofa.ark.master.biz=stock-mng-{your-number}
    ```
    com.alipay.sofa.ark.master.biz 配置项为指定的动态模块宿主应用的名称，需与宿主应用打包插件中的 bizName 配置项保持一致。
    因此需要将 {your-number} 也替换为当前座位前的编号。

* SOFADashboard 客户端配置
 
    在 dynamic-stock-mng 的 resource/application.properties 配置文件中添加配置如下：
    
    ```properties
    management.endpoints.web.exposure.include=*
    com.alipay.sofa.dashboard.zookeeper.address=116.62.20.143:2181,116.62.148.186:2181,121.43.174.16:2181
    #skip jvm health check to startup host-app
    com.alipay.sofa.boot.skip-jvm-reference-health-check=true
    ```
    
    同时将此配置文件中的 {your-number} 替换为当前座位编号:
    
    ```properties
    # 替换 {your-number}  为当前座位编号
    spring.application.name=stock-mng-{your-number} 
    ```

### 4、打包 & 启动宿主应用

#### 执行 mvn clean package

配置完成之后，执行 mvn clean package 进行打包，此时 dynamic-provider 会被打包成动态模块包，如下图所示：

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*X1exTbM3r3cAAAAAAAAAAABkARQnAQ)

> 比如如果你填写的 {your-number} 为 29 ，则打包成功之后，会生成 dynamic-module/target 目录下 生成 dynamic-provider-29-1.0.0-ark-biz.jar 文件

#### 启动宿主应用

```bash
 java -jar dynamic-stock-mng/target/dynamic-stock-mng-1.0.0.jar
```

启动成功之后日志信息如下：

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*3N_nS6P223IAAAAAAAAAAABkARQnAQ)

### 5、SOFADashboard 管控端添加版本

在实际的操作中，一般需要手动录入动态模块信息，本次 workshop 中为了方便大家操作，已经事先将00-99 100 个插件录入到了数据库中。
因此打开插件面板，可以看下如下信息：

![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*F-RGTLZJYj8AAAAAAAAAAABkARQnAQ)

在查询框中输入你当前座位的编号，（例如你的编号为66）：

![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*x836RaqJ9QkAAAAAAAAAAABkARQnAQ)

点击查询之后将会索引到你的插件，此时可以基于此插件进行应用关联和版本添加。

* 关联应用

点击关联应用，将插件绑定到宿主应用。此处的宿主应用名为 dynamic-stock-mng application.properties 中的 spring.application.name 的值，
如 spring.application.name=stock-mng-66，则你当前操作的宿主应用名即为 stock-mng-66

![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*ZdXDS6YCQp4AAAAAAAAAAABkARQnAQ)

* 添加版本

    目前 SOFADashboard 支持两种协议的文件获取方式，一种是基于 http 协议的，一种是基于 file 协议的。基于 http 协议即你可以将自己的动态模块包放在一个http
    服务器上，例如：http://ip:port/filePth 类型路径；基于 file 协议则是直接从文件系统获取动态模块包，例如：file://filePath。这里因为都是基于本地打包，所以使用 file
    协议。
    
    ![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*ce6hR79Z-eQAAAAAAAAAAABkARQnAQ)
    
    例如我打包之后的文件位于 /Users/guolei.sgl/Downloads/kubecon/kc-sofastack-dynamic-demo/dynamic-provider/target 目录下，则需要在添加版本中填入的文件地址为：
    file:///Users/guolei.sgl/Downloads/kubecon/kc-sofastack-dynamic-demo/dynamic-provider/target/dynamic-provider-00-1.0.0-ark-biz.jar
    
    ![image.png](https://gw.alipayobjects.com/mdn/rms_ff360b/afts/img/A*b0wcQbCFOasAAAAAAAAAAABkARQnAQ)
    
    > dynamic-provider-00-1.0.0-ark-biz.jar 中 00 为你当前座位的编号

### 6、查看详情 & 推送安装命令

在完成上述操作之后，即可点击当前插件后面的详情，进入插件详情页

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*9gkxSoxPnqUAAAAAAAAAAABkARQnAQ)

在执行安装之前，可以 先访问下 http://localhost:8080 ，此处因为还没有模块提供 jvm 服务，因此展示的是默认的排序顺序，如下所示：

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*cKbZQIpM7GkAAAAAAAAAAABkARQnAQ)

然后点击安装，延迟1~2s之后，状态变更为 ACTIVATED ，为激活状态

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*Eft7SbV1xFEAAAAAAAAAAABkARQnAQ)

此时再次访问 http://localhost:8080 ，结果如下：

![image.png](https://gw.alipayobjects.com/mdn/rms_565baf/afts/img/A*rG8aTKl7g6MAAAAAAAAAAABkARQnAQ)


> 此结果仅供参考，排序结果随商品对应的订单量而动态改变

