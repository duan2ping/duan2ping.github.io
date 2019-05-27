---
title: OSGI基础
date: 2019-05-21 09:25:00
author: duan2ping
img: medias/featureimages/15.jpg
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: true
top: true
mathjax: false
summary: 该文章是OSGI的基础概述，分析OSGI核心规范及相关知识点。
categories: OSGI
tags:
  - OSGI
  - JAVA模块化
---







SGI技术的demo
    
    愿各位猿友在技术中觅得归处，源于生活而归于生活。

## 项目架构

###框架环境
````
  JDK:1.8
  OSGI框架：Felix
  OSGI容器：Apache Karaf 3
  WEB容器：Spring MVC 3
  OSGI服务容器：Blueprint,SpringDM
  ORM：Hibernate
  DB：mysql 
````
###项目结构
````
  base
  
    osgi-api：公共接口层
    osgi-base-features：base模块打包构建
    osgi-service：业务逻辑层
   
  web
  
    osgi-web：控制层
    osgi-web-feature：web模块打包构建
````
    
# OSGI基础概述

**前言**：该教程为OSGI基础概述，深入详细的讲解请查看OSGI官方文档。以下具体实现均为Apache下的OSGI实现，Eclipse实现可类比学习。以下很多概念理解来自
官方描述和自己的总结看法，如有不对欢迎指正。

**概述**：OSGI是Open Services Gateway initiative的缩写，叫做开放服务网关协议，通常可能指OSGi联盟、OSGi标准或者OSGi框架。

**OSGI**：OSGI联盟现在将OSGI定义为一种技术，该技术是指一系列用于定义Java动态化组件系统的标准。这些标准通过为大型分布式系统以及嵌入式系统提供一种模块化架构减少了软件的复杂度。

**个人理解**：OSGI是OSGI联盟提出基于JVM的一系列用于定义Java动态化组件系统的规范标准，通过这些标准提供一种“模块化”降低大型软件架构的复杂度及耦合性，实现动态可插拔的动态组件服务。简而言之就是一堆协议栈,规范标准。

**目的**：实现软件架构模块化，将一个大的系统拆分成多个独立的模块(Bundle)，通过OSGI规范使得各个模块间更好的达到高内聚、松耦合、可复用、热插拔。

更多详细内容请看OSGI官方规范文档

* [OSGI核心规范](https://osgi.org/specification/osgi.core/7.0.0/framework.module.html)
* [OSGI服务规范](https://osgi.org/specification/osgi.cmpn/7.0.0/index.html)

> 优点

* 开发
  * 复杂性的降低：基于OSGi的组件模型bundle能够隐藏内部实现，bundle基于服务进行交互。 
  * 复用：很多第三方的组件可以以bundle的形式进行复用。 
  * 简单：核心的API总过包括不超过30个类和接口。 
  * 小巧：OSGi R4框架的实现仅需要300KB的JAR file就足够。在系统中引入OSGI几乎没有什么开销。 
  * 非侵入式：服务可以以POJO的形式实现，不需要关注特定的接口。
  
* 部署管理  
  * 易于部署：OSGi定义了组件是如何安装和管理的，标准化的管理API使得OSGi能够和现有和将来的各种系统有机的集成。 
  * 动态更新：这是OSGi被最经常提起的一个特性，即所谓的“热插拔”特性，bundle能够动态的安装、启动、停止、更新和卸载，而整个系统无需重启。 
  * 适配性：这主要得益于OSGi提供的服务机制、组件可以动态的注册、获取和监听服务，使得系统能够在OSGi环境调整自己的功能。 
  * 透明：提供了管理API来访问内部状态，因此通常无需去查看日志，能够通过命令行进行调试。 
  * 版本化：bundle可以版本化，多版本能够共存而不会影响系统功能，解决了JAR hell的问题。（这在开发时也提供了很大的帮助） 
  * 快速：这得益于OSGi的类加载机制，和JAR包的线性加载不同，bundle委托式的类加载机制，使得类的加载无需进行搜索，这又能有效的加快系统的启动速度。 
  * 懒加载：OSGi技术采用了很多懒加载机制。比如服务可以被注册，但是直到被使用时才创建。

> OSGI分层模型

**引子**：活于人世，处处皆是分层。通过对大千世界的分门别类、归纳划分，将本身抽象而混乱的事物变得清晰具体而规整。

![osgi分层模型](https://github.com/duan2ping/world/blob/master/osgi-base/images/osgi-hierarchical-model.png?raw=true)

  * Services(服务层)：服务层为Bundle的java开发者提供了一个动态、简单的编程一致性模型，并使得Bundle服务开发和部署简化，而且Bundle服务接口与Bundle服务实现无耦合。这个模型允许开发者使用Bundle服务接口来绑定具体的服务，具体的服务实现将会在运行期获得。
  * Life Cycle(生命周期层)：生命周期层提供了管理Bundle生命周期的API，而且这个API为Bundle提供了运行期模型，定义了Bundle如何被启动、停止、安装、更新以及卸载。另外，还提供了一个完善的事件驱动API，用于在OSGi framework中管理和控制Bundle。Life Cycle层强依赖于Module层，但是不一定需要Security层依赖。
  * Modules(模块层)：模块层为Java定义了模块化模型，并且克服了Java部署模型中的一些缺点。在模块化模型中为Bundle间的Java package共享和隔离定义了严格的规则。这个Module层能够独立于Life Cycle层和Service层使用，在Life Cycle层中提供了管理模块化Bundle的API，另外Service层提供了Bundle间的通信模型。
  * Security(安全层)：安全层基于Java2 security，但是在这基础上添加了大量的限制，并补充了相应的Java标准。在这个层次中定义了安全包格式，以及运行时如何与java2 security层进行交互的方式。
  * Execution Environment(底层执行环境,目前主要是java运行环境)： 定义在特定平台中可用的方法和类。
  * Bundles：Bundles不属于OSGI系统底层框架，而是由开发人员制作的OSGI组件(最小模块化单元)。
   
 综上所诉，其实OSGI最核心的就是三个模块，模块层、生命周期层和服务层。

> OSGI层间的相互作用

**概述**：通过对OSGI的分层实现各个层的责任划分从而降低系统的复杂度和耦合度，各层各司其职(分而治之)。各层通过暴露的API实现层间的交互，实现系统的聚合。

![osgi层间的相互作用](https://github.com/duan2ping/world/blob/master/osgi-base/images/osgi-interactions-between-layers.png?raw=true)
  
   
> OSGI常见实现

  * Apache Felix：Apache下的OSGI实现(Apache还有另外一个项目Aries，这个项目主要基于Felix，对OSGi企业标准进行了实现[Blueprint,JNDI,JPA,JTA,CDI])
  * Eclipse Equinox：Exlipse下的OSGI实现(Eclipse便是使用该框架实现的)
  * Knopflerfish：Markwave公司下的OSGI实现 
 
> OSGI企业级解决方案

**引子**：大厦焉能凭空起，皆是块块砖头积。技术亦是如此，我们可能突发奇想得到灵感想设计一个怎样的程序，通过构思设计将程序细化整理便成了这个程序的实现的设计规范。
通过构思我们得到了程序的整体设计构思，然后根据构思再一点点的通过coding实现。终于我们通过无数的commit实现了程序。这便是这个程序的核心代码实现。这个程序
虽然实现了核心的功能设计，然而人的欲望总是贪婪不满(这才促就了人类的发展)，总觉得少点东西。然后又基于此核心，设计新的扩展功能又形成新的规范。再coding实现
这些扩展功能。最终使得这个程序越来越贴近我们想要的样子。同理，OSGI组织设计和规范了OSGI的构思。上面说的OSGI实现便是根据设计规范对OSGI的核心实现。然而它并
不能满足一些使用者的需求(作为一个核心基础，当然是越单纯越好。低耦合，从而才能增强扩展性衍生更多的实现)，从而产生了新的实现，例如下面说的karaf，便是基于
OSGI实现(Felix/Equinox)扩展或改进了很多设计。

**Karaf**:是Apache基于OSGI实现的应用运行环境，提供了一个轻量级的OSGI容器。提供热部署、动态配置、日志管理、安全管理、本地系统集成、可编程扩展控制台、ssh远程访问等等功能。

![karaf模型图](https://github.com/duan2ping/world/blob/master/osgi-base/images/karaf-model.png?raw=true)


**Virgo**:是Eclipse基于OSGi实现的OSGI应用服务器，旨在运行企业Java和Spring驱动的OSGi应用程序。它提供了一个简单而全面的平台，用于开发，部署和服务企业OSGi应用程序。

![virgo模型图](https://github.com/duan2ping/world/blob/master/osgi-base/images/virgo-model.png?raw=true)


# OSGI核心规范

**概述**：该部分主要介绍OSGI核心底层基础规范。

## 模块层

**概述**：无论何种设计的出现都是为了更好的提高代码或系统的扩展性、高效性、复杂性。从面向对象思想开始，它是对不同类型事物的分类，将每个事物的功能行为尽
可能的做到高内聚低耦合，它主要是强调对类的抽象划分其实也是一种模块化。而OSGI模块化则是通过对系统模块间的划分，将每个模块尽可能的做到高内聚低耦合。
两种思想目的是一致只是粒度不一样。

### Bundle

**概述**：Framework定义了模块化单元，这个模块化单元称为Bundle。一个Bundle由Java类和其他资源文件组成，并且可以为终端用户功能。Bundle通过良好的定义，Bundle之间可以通过导入（import）和导出（export）来共享Java package。在OSGi Framework中，只有Bundle是部署Java应用的实体对象。

**个人理解**：Bundle是一个包含了代码，资源和元数据并且符合OSGI规范的以jar形式存在的一种模块化单元。因此一个符合OSGI规范的Bundle一定首先是符合JAR规范的jar文件。

**区别**：Bundle的本质也是jar，只是比jar多了一些OSGI规范特定的元信息数据(META-INFO/MANIFEST.MF)。

![bundle](https://github.com/duan2ping/world/blob/master/osgi-base/images/osgi-bundle.png?raw=true)

#### Bundle元数据常用头信息

**Bundle-ActivationPolicy**: lazy
    Bundle-ActivationPolicy指定了framework第1次如何启动Bundle。

**Bundle-Activator**: com.acme.fw.Activator
    Bundle-Activator指定了如何启动和停止Bundle的类。

**Bundle-Category**: osgi, test, nursery
    Bundle-Category表示逗号分隔的分类名称。

**Bundle-ClassPath**: /jar/http.jar
    Bundle-ClassPath定义了逗号分隔的路径,内容为Jar文件路径和类路径(Bundle内部)。点号（’.’ \u002E）表示Jar的根目录，而且也是默认的。

**Bundle-ContactAddress**: 2400 Oswego Road, Austin, TX 74563
    Bundle-ContactAddress提供发行者的联系地址。

**Bundle-Copyright**: OSGi (c) 2002
    Bundle-Copyright指Bundle的版权信息。

**Bundle-Description**: Network Firewall
    Bundle-Description定义了Bundle的简短描述信息。

**Bundle-DocURL**: http://www.example.com/Firewall/doc
    Bundle-DocURL指Bundle的文档链接地址。

**Bundle-License**: http://www.opensource.org/licenses/jabberpl.php
    Bundle-License提供了一个机器可读的许可证信息(license)，这个是可选项。提供这个头信息的目的是多个组织自动化处理许可证，例如Bundle被使用前的许可证验证。而且这个头结构提供了唯一的license命名，license信息是被读者可以阅读的，但是这个头信息只是纯粹的信息管理代理，OSGi Framework并不会处理。

**Bundle-Localization**: OSGI-INF/l10n/bundle
    Bundle-Localization包含Bundle的本地化文件地址,默认值是OSGI-INF/l10n/bundle。其他翻译文件如OSGI-INF/l10n/bundle_de.properties, OSGI-INF/l10n/bundle_nl.properties。

**Bundle-ManifestVersion**: 2
    Bundle-ManifestVersion定义了Bundle需要遵守本规范的规则，默认值是1表示第3个版本的Bundle，2表示第4个版本的或者更后发布的版本，也可以为OSGi Framework定义更高的数字。

**Bundle-Name**: Firewall
    Bundle-Name定义了一个有可读性的名字来表示Bundle○ UNINSTALLED（未安装）：状态值为整型数1。此时Bundle中的资源是不可用的。

**Bundle-RequiredExecutionEnvironment**: CDC-1.0/Foundation-1.0
    Bundle-RequiredExecutionEnvironment指示在OSGI framekwork上必须支持的可执行环境，用逗号分隔。

**Bundle-SymbolicName**: com.acme.daffy
    Bundle-SymbolicName为Bundle提供了一个全局唯一的名字。在framework中多次安装Bundle时，Bundle SymbolicName和version必须唯一。SymbolicName是基于反域名解析的，这部分头是必须的。

**Bundle-Vendor**: OSGi Alliance
    Bundle-Vendor描述Bundle的发行者信息。

**Bundle-Version**: 1.1
    Bundle-Version表示Bundle的版本信息，默认值是0.0.0 。

**DynamicImport-Package**: com.acme.plugin.*
    DynamicImport-Package包含了一个逗号分隔的动态导入包清单，也可以写*动态导入所有依赖。（动态导入包）

**Export-Package**: org.osgi.util.tracker;version=1.3
    Export-Package包含导出package声明信息。
    
**Import-Package**: org.osgi.util.tracker,org.osgi.service.io;version=1.4
    Import-Package声明Bundle导入的包。
    
**Bundle-Blueprint**:OSGI-INF/blueprint/blueprint.xml,OSGI-INF/blueprint/blueprint-cli.xml    
    Bundle-Blueprint描述了Bundle加载的Blueprint配置文件路径。
    
**Fragment-Host**: org.eclipse.swt; bundle-version="[3.0.0,4.0.0)"
    Fragment-Host定义了本片段中的主Bundle。

**Provide-Capability**: com.acme.dict; from=nl; to=de; version:Version=1.2
    Provide-Capability描述了Bundle提供的一组Capability。

**Require-Bundle**: com.acme.chess
    Require-Bundle描述了该Bundle import了其他Bundle export的内容。

更多详细信息可查看[OSGI官方规范文档](https://osgi.org/specification/osgi.core/7.0.0/framework.module.html)

#### BundleActivator(Bundle激活器)

**概述**：用于对Bundle的start和stop事件的监听，当Bundle启动或停止时会回调start和stop方法。

**实现**

```java
package com.duan2ping;

/**
* 根据需求选择，该类可有可无
*/
public class Activator implements BundleActivator{
    /**
     * Bundle启动时执行的方法
     * 用于对资源的初始化
     * @param bundleContext
     */
    public void start(BundleContext bundleContext) {
        logger.info("Bundle start");
    }

    /**
     * Bundle停止时执行的方法
     * 用于释放资源
     * @param bundleContext
     */
    public void stop(BundleContext bundleContext) {
        logger.info("Bundle stop");
    }
}

```

#### BundleContext(Bundle上下文)

**概述**：Bundle在Framework中的执行上下文环境。当启动或者停止一个Bundle的时候，Framework将它发送到一个bundle的激活器(Bundle Activator)。
该接口提供了Bundle和底层框架的交互方法。主要分为两部分，一部分是和部署与生命周期管理相关，另一部分则是关于利用服务层进行Bundle间交互的方法。
 

#### Bundle间的隔离

**概念**：“模块化”的模块是一些相对独立的实体，OSGI中一个模块就是一个Bundle，它们都是独立，隔离的。OSGI对每一个Bundle都分别用一个Classloader来加载实现隔离,所以默认情况下Bundle间的类是相互隔离且不可见的。

#### Bundle间的依赖(包的可见性)

**引子**：在实际开发中各模块间必然会有依赖和交互，但Bundle间又是隔离的，如何能访问其他Bundle的类呢？

**概念**：传统java开发中对jar的引用很少去关注jar包的隔离性且jar之间的依赖并不清晰(各种jar冲突)，而OSGI规范中Bundle被有计划的依赖，显示的声明bundle间的依赖关系。
 
##### Import-Package/Export-Package

**概述**：OSGI在Bundle的元信息清单(META-INF/MANIFEST.MF)添加了两项标记Import-Package/Export-Package，用于声明Bundle导入和导出的包。

**实现**：我们的目标是在MANIFEST.MF中声明导入和导出的包信息，我们可以自己编写MANIFEST.MF，但实际开发中不可能手动编写，容易出错不易维护，一般都是通过插件生成(插件下面再详述)。

**Export package**：指定Bundle需要暴露的包——>Export package:com.test.api;version=”1.0” (当前包中的类，子包的需单独导出或使用*)

**Import package**：指定Bundle依赖的包——>Import package:com.test.api;version=” [1.0,2.0)”(引用这个包中的类且版本在指定version区间中,子包中类需要单独引用或使用*)。

* 依赖解析规则：当有多个包提供者时
  * 1.已解析的Bundle优先级高于未解析的
  * 2.优先级相同优先版本高的
  * 3.版本相同优先最先安装的Bundle

**注意**：因为bundle间是隔离的，所以Import package时，bundle类加载器会委托给Export package这个包的类加载器去加载这个包的类
     import的包找不到时Bundle无法正常启动。

#### Bundle的依赖

* java.开头的包：由JDK提供了，代码中直接import。
* org.osgi开头(包括core、compendium等)：由OSGI规范提供的，已经包含在OSGI框架(Felix)中，开发时需要导入(编译环境)但是发布程序中不需要
包含(运行环境)，由Felix提供，因此需要在maven中添加scope为provided。
* 第三方jar包(多Bundle共享，公共Bundle)：直接OSGI化放入到容器指定目录下或在容器中安装。
* 第三方jar包(单Bundle依赖)：使用maven的Bundle插件指定Embed-Dependency过滤OSGI化后嵌入Bundle的Bundle-ClassPath中。
* 使用自己开发Bundle的依赖：直接Export-Package后，Import-Package即可。
* 为非OSGi第三方依赖项创建包
  * wrap方式：bundles:install wrap:mvn:commons-lang/commons-lang/2.4
  * 包含依赖再导出：添加第三方依赖再将依赖的包导出


#### Bundle编译环境和运行时环境

* 编译环境：配置dependencies是为了解决编译环境的依赖(防止编译出错)【第三方jar使用Maven-Bundle-plugin插件的Embed-Dependency添加到Bundle的cClasspath】

* 运行时环境：配置Import-Package才是为了真正的OSGI容器中程序的依赖【查询系统提供的类使用命令package:exports,找到需要的直接Import-Package即可】

#### Bundle类加载机制

**引子**：我们知道在Java中类的加载机制默认并且也是Java开发推荐的模型是双亲委派模型。这样保证了类的唯一性防止系统类被篡改确保系统的安全性和可靠性。对于一个底层系统环境
来说我觉得是必要的。但是做为实际业务需求它是有缺陷的，优点总是与缺点并存。正是这种安全可靠束缚了类的多样性、动态性，而OSGi实现模块化的关键就在于多样性和动态可插拔。这种
类加载机制肯定不可能满足Bundle类的隔离以及多个同类(双亲委派便是一山不容二虎，而我们愿意看到的是Love And Peace)共存，热更新等等现代化的开放思想(底层需要封建(稳定)，上层需要开放(多样))。
封建的社会终究打破，自由的灵魂终得安所。势必这种机制会被新的需求打破，当然这也不是第一次了(第一次是ClassLoader本身的设计早于双亲委派因此为了兼容当然只能妥协。第二次是SPI设计，因双亲委派模型
自身的缺陷所导致不得不引入线程上下文类加载器。第三次便是OSGI了，为了满足模块化间的隔离和动态插拔)。那么打破后OSGI的类加载模型又是如何的呢？如下图所示。

**概述**：通过上面的OSGI类加载模型可以看到OSGI类加载模型不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构。因此搞懂了OSGI类加载模型，对于理解Java的类加载模型就算不能上天入地，
也能呼风唤雨了。

![OSGI类加载模型](https://github.com/duan2ping/world/blob/master/osgi-base/images/osgi-classloader-model.png?raw=true)

##### Bundle类加载器

* 父类加载器：由Java平台直接提供，最典型的场景包括启动类加载器(Bootstrap ClassLoader )扩展类加载器(Extension ClassLoader)和应用程序类
加载器(Application ClassLoader)。在一些特殊场景中(如将OSGi内嵌入一个Web中间件)还会有更多的加载器组成。它们用于加载以“java.*”开头的类以及
在父类委派清单中声明为要委派给父类加载器加载的类。

* Bundle类加载器：每个Bundle都有自己独立的类加载器，用于加载本Bundle中的类和资源。当一个Bundle去请求加载另一个Bundle导出的Package中的类时
，要把加载请求委派给导出类的那个Bundle的加载器处理,而无法自己去加载其他Bundle的类。

* 其他加载器：譬如线程上下文类加载器、框架类加载器等。它们并非OSGi规范中专门定义的，但是为了实现方便，在许多OSGi框架中都会使用。例如框架类加载器
，OSGi框架实现一般会将这个独立的框架类加载器用于加载框架实现的类和关键的服。
务接口类 。


##### Bundle类加载顺序

**概述**：如果是java.开头的包(jdk核心包)就委托给父类加载器加载，如果该包是导入包则委托给导出该包的类加载器加载，否则在自身bundle的类路径下查找。

* ①如果类或资源在以java.*开头的Package中,那么这个请求需要委派给父类加载器加载否则，继续下一个步骤搜索。如果将这个请求委派给父类加载器后发现类或资源不存在，
那么搜索终止并宣告这次类加载请求失败。

* ②如果类或资源在父类委派清单(org.osgi.framework. bootdelegation)所列明的Package中，那么这个请求也将委派给父类加载器。如果将这个请求委派
给父类加载器后，发现类或资源不存在，那么搜索将跳转到一个步骤。

* ③如果类或资源在Import-Package标记描述的Package中，那么请求将委派给导出这个包的Bundle的类加载器，否则搜索过程将跳转到下一个步骤。如果将这个请求委派给
Bundle类加载器后，发现类或资源不存在，那么搜索终止并宣告这次类加载请求失败。

* ④如果类或资源在Require-Bundle导入的一个或多个Bundle 的包中，这个请求将按照Require-Bundle指定的Bundle清单顺序逐一委派给对应Bundle的类加载器 , 由于被委派的
加载器也会按照这里描述的搜索过程查找类，因此整个搜索过程就构成了深度优先的搜索策略。如果所有被委派的Bundle类加载器都没有找到类或资源，那么搜索将转到下一个步骤。

* ⑤搜索Bundle内部的Classpath。如果类或资源没有找到，那么这个搜索将转到下一个步骤。

* ⑥搜索每个附加的Fragment Bundle的Classpath。搜索顺序将按这些Fragment Bundle的ID升序搜索。如果这个类或资源没有找到，那么搜索转到下一个步骤。

* ⑦如果类或资源在某个Bundle已声明导出的Package中，或者包含在已声明导入(Import-Package或Require-Bundle)的Package中，那么这次搜索过程将以
没有找到指定的类或资源而终止。

* ⑧如果类或资源在某个使用DynamicImport-Package声明导入的Package中，那么将尝试在运行时动态导入这个Package。如果在某个导出该Package的Bundle
中找到需要加载的类，那么后面的类加载过程将按照步骤③处理。

* ⑨如果可以确定找到一个合适的完成动态导入的Bundle，那么这个请求将委派给该Bundle的类加载器。如果无法找到任何合适的Bundle来完成动态导入，那么搜索终止并宣
告此次类加载请求失败。当将动态导入委派给另一个Bundle类加载器时，类加载请求将按照步骤③处理。

![Bundle类加载机制](https://github.com/duan2ping/world/blob/master/osgi-base/images/bundle-classloader.png?raw=true)
   
        
## 生命周期层

**概述**：生命周期层提供API来控制包的安全性和Bundle生命周期操作。该层基于模块和安全层。

**注释**：该层主要是实现OSGI规范的框架底层维护。

### Bundle对象

**概述**：OSGI框架中对每一个安装的Bundle都会关联一个Bundle对象，用于管理Bundle的生命周期。

### Bundle标识

* Bundle标识(Identifier)：一个长整型整数，在一个Bundle整个生命周期中由OSGI框架赋值的唯一标识。该ID为自增长，可通过Bundle对象的getBundleId()方法获取。
* Bundle位置(Location)：Bundle的位置信息，可通过Bundle对象的getLocation()获取。
* Bundle特征名称(Symbolic Name)：由开发人员指定。Bundle版本和特征名称是Bundle全局唯一标识。可通过Bundle对象的getSymbolicName()获取。

### Bundle的状态

* UNINSTALLED(未安装)：状态值为整型数1。此时Bundle中的资源是不可用的。
* INSTALLED(已安装)：状态值为整型数2。此时Bundle已经通过了OSGI框架的有效性校验并分配了Bundle ID，本地资源已加载，但尚未对其依赖关系进行解析处理。
* RESOLVED(已解析)：状态值为整数4。此时Bundle已经完成了依赖关系解析并已经找到所有依赖包，而且自身导出的Package已可以被其它Bundle导入使用。在此种状态的Bundle要么是已经准备好运行，要么就是被停止了。
* STARTING(启动中)：状态值为整型数8。此时Bundle的BundleActivator的start()方法已经被调用但是尚未返回。如果start()方法正常执行结束，Bundle将自动转换到ACTIVE状态； 否则如果start()方法抛出了异常，Bundle将退回到RESOLVED状态。
* STOPPING(停止中)：状态值为整型数16。此时Bundle的BundleActivator的stop()方法已经被调用但是尚未返回。无论stop()是正常结束还是抛出了异常，在这个方法退出之后，Bundle的状态都将转为RESOLVED。
* ACTIVE(已激活)：状态值为整型数32。Bundle处于激活状态，说明BundleActivator的start()方法已经执行完毕，如果没有其他动作，Bundle将继续维持ACTIVE状态。

### Bundle生命周期

**概述**：Bundle在OSIG框架中的状态流转。

![bundle生命周期](https://github.com/duan2ping/world/blob/master/osgi-base/images/bundle-lifecycle.png?raw=true)


## 服务层

**引子**：

**概述**：OSGi服务层定义了与生命周期层高度集成的动态协作模型。该服务模型是发布，查找和绑定模型。服务是在服务注册表的一个或多个Java接口下注册的普通
Java对象。Bundle可以注册服务，查找服务，或在注册状态发生变化时接收通知。

个人理解：该层主要是提提供了Bundle间服务实现的模型和对服务的管理维护。

* 优点
  * 松耦合：服务通过面向接口编程，将业务接口和实现分离。服务调用者不关心服务的实现细节，服务提供者不关心调用的方式。
  * 动态性：服务在程序运行中可注册和注销
  * 可重用：服务创建后能用于多个模块的使用。
  * 多样性：接口解决了java类的多继承问题。通过对接口的多种实现，使服务变得更加多样化。

### Bundle间更松散的耦合

**概述**：在实际开发中Bundle间必然是会有很多依赖和交互的。前面我们提到了一种Import-Package/Export-Package，这种对包的依赖是强耦合。在当下微服务
分布式横行的架构下，面向服务编程模型显得更加灵活动态，松耦合。在实现项目架构中我们通常是这样设计的。模块A会将自己对外提供的API封装成一个单独的
Bundle并将包export。模块A的其他Bundle负责实现这些接口(需要Import Bundle API暴露的package)并将具体实现服务注册到OSIG框架中。而模块B中的
Bundle如果想调用模块A中的服务，则可Import模块A的Bundle API导出的包，并从OSGI框架获取到对应接口的服务实现。我们通过面向接口编程实现的服务模型
降低了模块间的耦合。

![osgi模块交互模型](https://github.com/duan2ping/world/blob/master/osgi-base/images/module-interaction-model.png?raw=true)


### OSGI服务模型

**概述**：OSGi框架提供了一个中心化的注册表，这个注册表遵从publish-find-bind模型。

**注意**：因为服务注册表是在OSGI框架中的，因此这些服务都是本地服务。“本地”意味着它只是在osgi framework内有效，不可跨osgi framework调用更不可
跨JVM调用。

![osgi服务模型](https://github.com/duan2ping/world/blob/master/osgi-base/images/osgi-service-model.png?raw=true)

#### 服务组成

* 服务
* 服务引用
* 服务接口
* 注册服务
* 服务属性
* 查询服务
* 服务事件
* 服务监听
* 服务追踪器

#### 服务具体实现

* 传统注册式服务(不推荐使用)

  * 概述：传统方式下，我们注册服务都是在bundle的激活器(Activator)中使用BundleContext.registerService()方法完成的。而服务的获取需要通过
         BundleContext.getServiceReference()获取ServiceReference实例，进而使用BundleContext.getService()得到真正的服务实例。

  * 缺点

     * 产生较多的样板式代码：OSGI的Bundle是动态化的，伴随着Bundle的安装和卸载，它所发布的服务也会动态地处于可用或不可用的状态，因此每次使用服务的时候，我们都需要借助BundleContext对象去服务注册中心查找，而不能通过一次查找，一劳永逸地持有服务对象的引用。尽管有ServiceListener和ServiceTracker帮助我们监听和跟踪服务的状态，但是总体而言这种方式较为繁琐且容易出错
     * 影响启动时间：服务在激活器中注册时，需要实例化所有要发布的服务对象，因为激活器的start()方法是同步调用的，所以会影响到整个应用的启动时间。
     * 加大内存的占用：在激活器中注册服务时，我们需要实例化所有的服务对象，但是这些服务在应用运行期间，并不一定会用到，这在无形中加大了内存的占用。
     * API依赖引起的平台侵入性：使用传统方式注册和使用服务会用到大量的OSGI API从而产生与OSGi平台的耦合，如果要将代码复用到非OSGi场景之中，需要较多的重构工作。

```java
package com.duan2ping;

public class Activator implements BundleActivator{
     
    // 注册服务
    public void start(BundleContext bundleContext) {
        // 服务的属性，可用于后续服务查询的过滤
        Dictionary<String,String> dictionary = new Properties();
        dictionary.setProperty("name","duan2ping");
        dictionary.setProperty("type","test");
        // 注册服务 （服务名称，服务实现，服务属性）
        /* ServiceRegistration对象可用于对服务属性的更新以及服务的注销，它和发布服务的bundle的生命周期相互依存，也就是随Bundle生而生死而灭 */
        ServiceRegistration registration = bundleContext.registerService(UserService.class.getName(),new UserServiceImpl(),dictionary);
    }

    // 查找服务
    public void stop(BundleContext bundleContext) {
        // 获取服务引用，真实服务的间接引用(实现对真实服务和调用者的解耦，避免直接依赖)
        /* 第二个参数则是通过服务的属性对服务进行过滤，具体规则可参考OSGI官方文档:https://osgi.org/specification/osgi.core/7.0.0/framework.service.html#framework.service.servicereferences */
        ServiceReference ref = context.getServiceReference(UserService.class.getName(),"(&(name=test)(objectClass=com.duan2ping.UserServiceImpl))");
        // 根据服务引用获取到真实的服务
        UserService service = (UserService) context.getService(ref);
    }
}

```

* Blueprint

  * 概述:一个IOC容器，下面会再介绍。

  * 优点
    * 低耦合：通过IOC方式的实现降低了服务间的耦合。
    * 动态性：由容器管理服务动态注入，服务随时可用和不可用。

```xml

<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0">
        
    <!-- 引用服务 -->
    <reference id="bookService" interface="BookService"/>

    <!-- 暴露服务 -->
    <service interface="BookService" ref="bookServiceImpl"/>

</blueprint>

```


* Declarative Service声明式服务


* IPOJO


# OSGI组件规范（企业服务规范）

**概述**：该部分主要介绍基于OSGI核心规范而扩展衍生出的相关组件服务规范。

## 常用实现

**引子**：

**概述**：OSGI在核心规范基础上扩展了很多特性。核心规范主要是定义OSGI框架的基础机制，而远远满足不了当下流行的开发模式以及企业需求。

* 常见服务组件

  * (Log Service Specification)日志服务规范
  * (Http Service Specification)http服务规范
  * (Declarative Services Specification)声明式服务规范
  * (Configuration Admin Service Specification)配置管理服务规范
  * (JTA Transaction Services Specification)JTA事务规范
  * (Data Service Specification for JDBC™ Technology)JDBC数据库服务规范
  * (JNDI Services Specification)JNDI服务规范
  * (JPA Service Specification)JPA服务规范
  * (Web Applications Specification)Web应用程序规范
  * (Blueprint Container Specification)Blueprint容器规范

## Blueprint

[Blueprint规范文档](https://osgi.org/specification/osgi.cmpn/7.0.0/service.blueprint.html)

**引子**：在Java开发中，Spring想必一定是无人不知的。它的很多设计思想都让我们的开发设计有了大大的改变。在这里要提的一个设计思想是IOC(控制反转)，
它通过一个Bean容器来控制类属性的注入，而不是像传统那种通过直接new或者使用工厂创建的方式使得类间的直接耦合。简而言之就是通过第三方系统来创建
类中依赖属性，从而达到类和类的解耦。

**概述**：OSGI中的IOC容器规范。

**理解**：由上我们得知，既然Spring也有IOC，那Blueprint跟它除了功能一致还有什么联系呢？在互联网技术中从不需要造轮子，既然已经有了这么好还成熟的IOC
框架，由何必去另辟蹊径呢。因此，Blueprint的规范便是源于Spring更具体的就如OSGI规范中所述，源于Spring Dynamic Modules(SpringDM，后面会介绍)。
所以熟悉Spring必然对Blueprint似曾相识得心应手犹如他乡遇故知。

**CDI**：提到了JAVA和IOC那当然不得不提一提CDI了。CDI（Contexts and Dependency Injection 上下文依赖注入），是JAVA官方提供的依赖注入规范。
当然可想而知该规范也是源于Spring思想。在[javax-inject](http://javax-inject.github.io/javax-inject/api/javax/inject/package-summary.html)
扩展包定义了java CDI规范接口，其实里面很简单就五个注解一个接口(@Inject,@Named,@Qualifier,@Scope,@Singleton)。既然是官方规范，当然作为
第三方IOC容器必然是实现该规范的。所以Spring也实现了这个规范(规范本身就源于Spring所以应该算是Spring兼容Java CDI)。深入了解可查看[CDI规范详情](https://docs.oracle.com/javaee/6/tutorial/doc/giwhb.html)。

* 规范实现

  * [Apache Aries Blueprint实现](http://aries.apache.org/modules/blueprint.html)
  * [Eclipse Gemini Blueprints实现](https://www.eclipse.org/gemini/blueprint/documentation/reference/3.0.0.M01/html/index.html)

* 具体配置(Apache Aries的实现)

  * 手动配置
     * 描述：在resource目录下，需要先创建OSGI-INF/blueprint两级目录，然后在里面建立相关的xml文件，在此xml文件的名字不需要固定，也可以为多
       个，只要在此目录，均会被加载。
       
       ```xml
        <blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0">
               <!-- bundle内部使用，跟Spring一样bean带有很多属性
                    scope：创建方式
                       prototype：原型模式
                       singleton：单例模式
                    factory-method：指定创建的构造函数
                    init-method：创建实例的初始化方法，实例创建时执行
                    destroy-method：实例销毁方法，实例销毁时执行
               -->
              <bean id="bookServiceImpl" class="com.duan2ping.service.impl.BookServiceImpl" scope="singleton"/>
       
              <!-- 对外提供服务，暴露给其他bundle使用 -->
              <service id="bookServcie" interface="com.duan2ping.api.BookServcie" ref="bookServiceImpl">
                   <!-- 添加服务属性 -->
                   <service-properties>
                      <entry key="name" value="test"/>
                   </service-properties>
              </service>
       
              <!-- 引用服务，调用其他bundle暴露的服务 -->
              <reference id="systemService" interface="com.duan2ping.api.SystemService"/>
        </blueprint>
       ```
 
  * 注解
     * 概述：在当下注解开发如此便捷(简化配置)，当然是必不可缺的扩展。这里有部分使用Java CDI注解。
     * 描述
       * @Singleton：将该类加入Blueprint容器管理并以单例方式创建等同于<bean></bean>
       * @OsgiServiceProvider：将该类作为一个OSGI服务发布(必须添加@Singleton)，classes为实现的契约(接口)，@Properties配置服务的扩展属性。
         等同于<service></service>
       * @OsgiService：引用OSGI服务，必须配合Inject使用，查找方式取决于@Inject。等同于<reference></reference>
       * @Inject：从Blueprint容器中查找(按类型)依赖并注入。可通过@Named注解按id查询。
       
     ```java
     package com.duan2ping;
     
     @OsgiServiceProvider(classes={BookService.class})
     @Properties(@Property(name = "name",value = "test"))
     @Singleton
     public class BookServiceImpl implements BookService{
        
        @Inject
        @Named("bookDao")
        private BookDao bookDao;
    
        @Inject
        @OsgiService
        private SystemService systemService;
        // ...
     }
   
     ```
     * Blueprint注解插件 
        概述：我们都知道不管注解有多方便，只不过是程序识别注解给我们生成了相应的配置。表象可以千变万化，本质却始终如一。就像Spring一样也是通过配置
        Scan来扫描注解，它是运行时解析的。而OSIG中这个是Bundle元数据的一项描述(Bundle-Blueprint)，因此它应该是编译后就有的静态xml文件，所以
        这个操作是通过一个插件来做的。它会通过扫描指定的包来生成Blueprint配置文件。
        
        ```xml
           <!-- 扫描osgi相关注解生成blueprint -->
            <plugin>
                <groupId>org.apache.aries.blueprint</groupId>
                <artifactId>blueprint-maven-plugin</artifactId>
                <version>1.3.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>blueprint-generate</goal>
                        </goals>
                        <phase>process-classes</phase>
                    </execution>
                </executions>
                <configuration>
                    <!-- 指定blueprint扫描注解的包路径 -->
                    <scanPaths>
                        <scanPath>com.duan2ping</scanPath>
                    </scanPaths>
                </configuration>
            </plugin>
        ```

## Configuration Admin

[Configuration Admin规范文档](https://osgi.org/specification/osgi.cmpn/7.0.0/service.cm.html)

**引子**：在软件开发中必不可缺的便是配置文件，通过一堆配置来实现部分模块或逻辑的动态性。我们总是期望修改配置而不需要重启服务，而OSGI正是打着热加载的口号。
连Bundle都是热插拔又更何况区区一个配置文件。当然配置文件必须也是热更新实时加载的才能配得上如此嚣张的框架。

**概述**：OSGI的配置管理服务规范。

**理解**：

* 核心组件

  * Dictionary(配置字典)：用于保存配置的实体，包含配置的属性和值信息。
  * 
  (配置管理)：维护OSGI框架中的所有配置。
  * Configuration(配置对象)：配置信息实体，包含配置字典。
  * ManagedService：监听单个配置文件。
  * ManagedServiceFactory：监听多个配置文件。
  * ConfigurationListener：监听所有配置文件。

* 具体配置

  * Blueprint



## Spring DM
[Spring-dm官方文档](https://docs.spring.io/spring-osgi/docs/current/reference/html)


## Bundle的构建

**概述**：通过前面的一堆理论，我们应该对OSGI有了大概的理解。那么接下来就开始撸代码看看具体一个Bundle是如何构建的，揭开它神秘的面纱。

**提示**：我们前面说过，Bundle是一个符合OSIG规范的jar文件，它跟jar文件一样，只是在MANIFEST.MF文件中有OSGI相关的元信息。

### Bnd

**概述**：对于构建和分析OSGi Bundle来说，Bnd是一种极为强大的底层工具。它是由Peter Kriens(OSGi联盟的技术总监)开发的，OSGi联盟使用它来构建自己的
API套件、兼容性测试以及引用实现的Bundle。作为一种底层工具，它很容易继承，并且可以直接从命令行调用，可以被ANT任务所使用，也可以嵌入到 Maven和IDE中。

**理解**：Bnd是许多支持OSGi的流行软件开发工具的引擎。目的是为了降低OSGI开发的复杂度。

该方式暂未使用过，感兴趣的可以查看[Bnd官方文档](https://bnd.bndtools.org)

### Bndtools构建

**引子**：作为一个IDEA的使用者实在嫉妒Eclipse插件功能的强大和稳定。一个人的能力跟表象毫无关系，当然要是有能力再一点美貌那就更好不过了。Eclipse本身
就是OSGI框架实现的，那当然能很好的兼容OSGI。

**概述**：BndTools使用Bnd作为它的“引擎”。所有主要功能都是bnd本身提供的，而 BndTools只是描述什么时候应该调用bnd，并以更好的形式来展现结果。由于很
多其它工具也集成了Bnd，所以Bnd所使用的描述文件几乎已经成为一种事实上的标准，这意味着BndTools开发者很容易与使用其它工具的开发者协作，也可以在选择
其它工具的时候很容易地完成迁移。

**理解**：Bndtools是一个基于Bnd的Eclipse插件，它使用Bnd来创建OSGi包。Bndtools提供了一个IDE，用于以最少的工作量和最大的作用来开发Bundle，还提
供了编辑和分析Bundle描述符以及依赖关系的方式，设置和执行运行配置(run configurations)的方式等功能。

* 安装
  
  * 注意：Bndtools现在支持所有4.4 Luna以上版本的Eclipse，包括4.6 Neon。
   
  * 通过Eclipse Marketplace安装Bndtools(推荐)
  
    * 点击Eclipse上方的Help选择Eclipse Marketplace，在搜索框中输入bnd回车搜索到最新的Bndtools点击安装，按提示一步步安装即可。
  
    ![Eclipse Marketplace安装Bndtools](https://github.com/duan2ping/world/blob/master/osgi-base/images/install_Bndtools_1.gif?raw=true)
   
  * 通过Eclipse Install New Software
    
    * 点击Eclipse上方的Help选择Install New Software，在Work with输入框中输入对应Bndtools版本的url，Eclipse会识别到对应的软件，勾选后
      按提示一步步安装即可。
      
    * Bndtools历史版本可查看[Bndtools插件版本列表](https://dl.bintray.com/bndtools/bndtools)。
  
    ![Eclipse Install New Software安装Bndtools](https://github.com/duan2ping/world/blob/master/osgi-base/images/install_Bndtools_2.gif?raw=true)
    
* 使用

  * 独白：已经安装成功了，迫不及待的赶紧来实操一把，看看这玩意到底有什么绝世武功，竟敢如此猖狂。

> 很遗憾笔者使用的不是Eclipse，实在不敢乱下结论，不行万里路怎敢说知万卷书，待笔者先行愿后续相遇此处已有见地，接下来的路就让官网陪你走完吧，[Bndtools官方教程](https://bndtools.org/tutorial.html)。


### Maven插件构建(maven-bundle-plugin)(推荐)

**引子**：前者方式或许很强大，但是入门不易(概念多)且依托于Eclipse，对于Maven用户和入门者来说下面的这种方式更为简单。

**概述**：maven-bundle-plugin，它是另一种集成了Bnd的工具。该Maven插件是用于对Bundle的构建，以及Bundle元信息的生成(最终目的如此，当然底层还做了很多校验解析依赖分析处理等等)。

**注意**：默认情况下，插件会导入所有在java文件中导入或在Blueprint上下文中引用的包。它还会导出所有不包含字符串impl或internal的包。

```xml

 <plugin>
    <groupId>org.apache.felix</groupId>
    <artifactId>maven-bundle-plugin</artifactId>
    <version>3.0.1</version>
    <extensions>true</extensions>
    <configuration>
        <instructions>
            <!-- Bundle标识符 -->
            <Bundle-SymbolicName>${project.artifactId}</Bundle-SymbolicName>
            <!-- Bundle版本 -->
            <Bundle-Version>${project.version}</Bundle-Version>
            <!-- 暴露指定包,如不写默认导出所有 -->
            <Export-Package>
                com.duan2ping.api.*;version=${project.version},
            </Export-Package>
            <!-- 私有包 -->
            <Private-Package>com.duan2ping.api.private.*</Private-Package>
            <!-- 指定依赖包 -->
            <Import-Package>
                org.hibernate.proxy*,
                javax.persistence*,
                org.osgi.framework;version="[1.6,2)",
                org.slf4j;version="[1.7,2)"
            </Import-Package>
            <!-- 指定激活器 -->
            <Bundle-Activator>ApiActivetor</Bundle-Activator>
        </instructions>
    </configuration>
</plugin>

```

详细配置请看[官方文档](http://felix.apache.org/documentation/subprojects/apache-felix-maven-bundle-plugin-bnd.html)

* 使用：参考examples/simple-bundle

  * 创建一个标准的Java Maven项目(作为Java开发者这就不详述了)
  * 添加OSGI框架核心接口依赖org.osgi.core，并且配置Import-Package:org.osgi.framework
  * 在pom文件中添加maven-bundle-plugin插件，<configuration>里的配置都为可选，都有默认值。
  * 将packagin改为<packaging>bundle</packaging>
  * 编译，然后会看到编译结果有一个xxx.jar。对，你没看错，它就是我们絮絮叨叨所谓的Bundle；模块化的最小单元；三五句离不开的东西。为啥还是jar文件呢？
    前面我们说过，Bundle是一个包含了代码，资源和元数据并且符合OSGI规范的以jar形式存在的一种模块化单元。因此它还是jar包，Bundle和jar的区别就在于
    MANIFEST.MF中的元数据。可以参考前面的资料，[Bundle元数据](#bundle元数据常用头信息)。
  * 运行，现在我们有了Bundle
  
> 按以上操作，一个简单的Bundle就已经大功告成了。是不是很简单？越是简单的东西，越是深不可测。

#### maven原型创建osgi项目

##### Simple Bundle
```text
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.karaf.archetypes \
    -DarchetypeArtifactId=karaf-bundle-archetype \
    -DarchetypeVersion=4.0.0 \
    -DgroupId=com.duan2ping.ob \
    -DartifactId=osgi-base-bundle \
    -Dversion=1.0-SNAPSHOT \
    -Dpackage=com.duan2ping.ob.bundle
```

##### Blueprint Bundle
```text
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.karaf.archetypes \
    -DarchetypeArtifactId=karaf-blueprint-archetype \
    -DarchetypeVersion=4.0.0 \
    -DgroupId=com.duan2ping.ob \
    -DartifactId=osgi-base-blueprint \
    -Dversion=1.0-SNAPSHOT \
    -Dpackage=com.duan2ping.ob.blueprint
```
    
    
##### Command Bundle

```text
 mvn archetype:generate \
     -DarchetypeGroupId=org.apache.karaf.archetypes \  
     -DarchetypeArtifactId=karaf-command-archetype \  
     -DarchetypeVersion=3.0.6 \  
     -DgroupId=com.duan2ping.ob \   
     -DartifactId=osgi-base-command \   
     -Dversion=1.0-SNAPSHOT \   
     -Dpackage=com.duan2ping.ob.command
```
> 会提示输入 
command:命令名称,也是类名称
description:命令描述
scope:命令作用域
    
##### Web Bundle

```text
mvn archetype:generate \
   -DarchetypeGroupId=org.ops4j.pax.web.archetypes \
   -DarchetypeArtifactId=wab-gwt-archetype \
   -DarchetypeVersion=2.1.2 \
   -DgroupId=com.duan2ping.ob \
   -DartifactId=osgi-base-web \
   -Dversion=1.0
```

##### Feature project

```text
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.karaf.archetypes \
    -DarchetypeArtifactId=karaf-feature-archetype \
    -DarchetypeVersion=4.0.0 \
    -DgroupId=com.duan2ping.ob \
    -DartifactId=osgi-base-feature \
    -Dversion=1.0-SNAPSHOT \
    -Dpackage=com.duan2ping.ob.feature
```

##### Kar project

```text
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.karaf.archetypes \
    -DarchetypeArtifactId=karaf-kar-archetype \
    -DarchetypeVersion=4.0.0 \
    -DgroupId=com.duan2ping.ob \
    -DartifactId=osgi-base-kar \
    -Dversion=1.0-SNAPSHOT \
    -Dpackage=com.duan2ping.ob.kar
```


##### Assembly project

```text
 mvn archetype:generate \
     -DarchetypeGroupId=org.apache.karaf.archetypes \
     -DarchetypeArtifactId=karaf-assembly-archetype \
     -DarchetypeVersion=4.0.0 \
     -DgroupId=com.duan2ping.ob \
     -DartifactId=osgi-base-assembly \
     -Dversion=1.0 \
     -Dpackage=com.duan2ping.ob.assembly
```









