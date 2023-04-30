# 核心技术

> Version 5.3.22

参考文档的这一部分涵盖了Spring框架中绝对不可或缺的所有技术。

其中最重要的是Spring框架的控制反转（IoC）容器。在对Spring框架的IoC容器进行全面处理之后，将全面介绍Spring的面向方面编程（AOP）技术。Spring框架有自己的AOP框架，概念上很容易理解，并且成功地解决了Java企业编程中80%的AOP需求。

还介绍了Spring与AspectJ的集成(就特性而言，目前AspectJ是最丰富的，当然也是Java企业领域中最成熟的AOP实现)。

##  1. IoC容器

本章介绍了Spring的控制反转（IoC）容器。

### 1.1. Spring IoC容器和bean简介

本章介绍了控制反转(IoC)原理的Spring框架实现。IoC也被称为依赖注入(DI)。在此过程中，对象仅通过构造函数参数、工厂方法的参数或在对象实例构造或从工厂方法返回后在对象实例上设置的属性来定义它们的依赖关系(即它们工作的其他对象)。然后容器在创建bean时注入这些依赖项。这个过程从根本上说是bean本身的逆过程(因此得名“控制反转”)，它通过使用类的直接构造或服务定位器模式等机制来控制依赖项的实例化或位置。

`org.springframework.beans`和`org.springframework.context `包是Spring框架的IoC容器的基础。`BeanFactory`接口提供了能够管理任何类型对象的高级配置机制。`ApplicationContext`是`BeanFactory`的子接口。它补充说：

- 更容易与Spring的AOP特性集成

- 消息资源处理(用于国际化)

- 事件发布

- 应用程序层特定的上下文，如`WebApplicationContext`用于web应用程序。

简而言之，`BeanFactory`提供了配置框架和基本功能，而`ApplicationContext`添加了更多特定于企业的功能。`ApplicationContext`是`BeanFactory`的一个完整超集，在本章Spring的IoC容器的描述中被专门使用。有关使用`BeanFactory`而不是`ApplicationContext`的更多信息，请参阅介绍`BeanFactory` API的部分。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中的许多对象之一。bean及其之间的依赖关系反映在容器使用的配置元数据中。

###  1.2. 容器概述

`org.springframework.context.ApplicationContext`接口表示Spring IoC容器，并负责实例化、配置和组装bean。容器通过读取配置元数据来获取关于实例化、配置和组装哪些对象的指令。配置元数据用XML、Java注释或Java代码表示。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖关系。

Spring提供了`ApplicationContext`接口的几个实现。在独立应用程序中，通常会创建`ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例。虽然XML是定义配置元数据的传统格式，但是可以通过提供少量的XML配置以声明的方式支持这些额外的元数据格式，从而指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序场景中，不需要显式的用户代码来实例化Spring IoC容器的一个或多个实例。例如，在一个web应用场景中，在应用程序的`web.xml`文件中，一个简单的8行(大约)web描述符XML通常就足够了(参见web应用程序的方便应用上下文实例化)。如果您使用用于Eclipse的Spring Tools(一个由Eclipse支持的开发环境)，那么只需单击几下鼠标或敲击几下键盘，就可以轻松创建这个样板配置。

下图显示了Spring如何工作的高级视图。您的应用程序类与配置元数据相结合，这样，在`ApplicationContext`创建和初始化之后，您就拥有了一个完全配置和可执行的系统或应用程序。

![container-magic](spring文档.assets/container-magic.png)

#### 1.2.1. 元数据配置

如上图所示，Spring IoC容器使用一种配置元数据的形式。这个配置元数据表示作为应用程序开发人员，您如何告诉Spring容器实例化、配置和组装应用程序中的对象。

配置元数据传统上以简单直观的XML格式提供，本章的大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

> 基于xml的元数据不是唯一允许的配置元数据形式。Spring IoC容器本身与实际编写此配置元数据的格式完全解耦。现在，许多开发人员为他们的Spring应用程序选择基于java的配置。

有关在Spring容器中使用其他形式的元数据的信息，请参见:

- 基于注释的配置：Spring 2.5引入了对基于注释的配置元数据的支持。

- 基于java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多特性成为核心Spring框架的一部分。因此，您可以使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新特性，请参阅`@Configuration`、`@Bean`、`@Import`和`@DependsOn`注释。

Spring配置由容器必须管理的至少一个(通常不止一个)bean定义组成。基于xml的配置元数据将这些bean配置为顶级`<beans/>`元素中的`<bean/>`元素。Java配置通常在`@Configuration`类中使用`@bean`注解的方法。

这些bean定义对应于构成应用程序的实际对象。通常，您需要定义服务层对象、数据访问对象(dao)、表示对象(如Struts Action实例)、基础设施对象(如Hibernate `SessionFactories`)、JMS队列(JMS `Queues`)等等。通常，不需要在容器中配置细粒度的域对象，因为创建和加载域对象通常是dao和业务逻辑的职责。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。参见使用AspectJ来用Spring注入域对象的依赖。

下面的例子展示了基于xml的配置元数据的基本结构:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

> id属性是标识各个bean定义的字符串。
>
> class属性定义bean的类型并使用全类名。
>

`id`属性的值指的是协作对象。本例中没有显示用于引用协作对象的XML。有关更多信息，请参阅依赖关系。

#### 1.2.2. 实例化容器

提供给`ApplicationContext`构造器的位置路径或路径是资源字符串，它们允许容器从各种外部资源加载配置元数据，例如本地文件系统、Java `CLASSPATH`等。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> 在了解了Spring的IoC容器之后，您可能想更多地了解Spring的`Resource`抽象(如参考资料中所述)，它提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。具体来说，`Resource`路径用于构建应用程序上下文，如应用程序上下文和资源路径中所述。

服务层对象`(services.xml)`配置文件示例如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

下面的例子展示了数据访问对象`daos.xml`文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由`PetStoreServiceImpl`类和两个类型为`JpaAccountDao`和`JpaItemDao`(基于JPA对象关系映射标准)的数据访问对象组成。 `property name`元素引用JavaBean属性的名称，`ref`元素引用另一个bean定义的名称。`id`和`ref`元素之间的链接表示协作对象之间的依赖关系。关于配置对象依赖关系的详细信息，请参见依赖关系。

##### 编写基于xml的配置元数据

让bean定义跨多个XML文件可能很有用。通常，每个单独的XML配置文件代表体系结构中的一个逻辑层或模块。

可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。这个构造函数接受多个`Resource`位置，如前一节所示。或者，使用一次或多次`<import/>`元素从另一个或多个文件加载bean定义。下面的例子展示了如何这样做:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义是从三个文件加载的：`services.xml`、`messageSource.xml`和`themeSource.xml`。所有位置路径都相对于执行导入的定义文件，因此`services.xml`必须与执行导入的文件位于相同的目录或类路径位置，而`messageSource.xml`和`themeSource.xml`必须位于导入文件位置下方的资源位置。如您所见，斜杠会被忽略。然而，考虑到这些路径是相对的，最好完全不使用斜杠。根据Spring Schema，被导入的文件的内容，包括顶级`<beans/>`元素，必须是有效的XML bean定义。

> 使用相对的"../"引用父目录中的文件是可能的，但不建议这样做。/”路径。这样做会在当前应用程序之外的文件上创建一个依赖项。特别地，不建议将此引用用于`classpath:` URLs(例如，`classpath:../services.xml`)，其中运行时解析过程选择“最近的”类路径根，然后查看其父目录。类路径配置更改可能导致选择不同的、不正确的目录。
>
> 你总是可以使用完全限定的资源位置而不是相对路径:例如，`file:C:/config/services.xml`或`classpath:/config/services.xml`。但是，请注意，您正在将应用程序的配置耦合到特定的绝对位置。通常更可取的做法是对这种绝对位置保持间接——例如，通过在运行时根据JVM系统属性解析的“${…}”占位符。

命名空间本身提供了导入指令特性。除了普通bean定义之外，Spring提供的一系列XML名称空间还提供了更多的配置特性—例如，`context`和`util`名称空间。

#### 1.2.3. 使用容器

`ApplicationContext`是高级工厂的接口，该工厂能够维护不同bean及其依赖项的注册表。通过使用方法`T getBean(String name，Class<T> requiredType)`，您可以检索bean的实。

`ApplicationContext`允许您读取bean定义并访问它们，如下面的示例所示:

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，引导看起来非常类似。它有一个不同的上下文实现类，它支持groovy(但也理解XML bean定义)。下面的例子展示了Groovy的配置:

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是`GenericApplicationContext`与阅读器委托的组合——例如，XML文件的`XmlBeanDefinitionReader`，如下所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

你也可以为Groovy文件使用`GroovyBeanDefinitionReader`，如下面的例子所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以在同一个`ApplicationContext`上混合和匹配这样的阅读器委托，从不同的配置源读取bean定义。

然后可以使用`getBean`检索bean的实例。ApplicationContext接口还有一些用于检索bean的其他方法，但理想情况下，应用程序代码永远不应该使用它们。实际上，您的应用程序代码根本不应该调用`getBean()`方法，因此根本不依赖Spring api。例如，Spring与web框架的集成为各种web框架组件(如控制器和jsf管理的bean)提供了依赖注入，允许您通过元数据(如自动装配注释)声明对特定bean的依赖。

###  1.3. Bean解读

Spring IoC容器管理一个或多个bean。这些bean是用您提供给容器的配置元数据创建的(例如，以XML `<bean/>`定义的形式)。

在容器本身内部，这些bean定义被表示为`BeanDefinition`对象，它包含(包括其他信息)以下元数据：

- 包限定的类名：通常是正在定义的bean的实际实现类。
- Bean行为配置元素，它规定Bean在容器中应该如何行为(范围、生命周期回调，等等)。

- 对bean执行其工作所需的其他bean的引用。这些引用也称为协作者或依赖项。

- 要在新创建的对象中设置的其他配置设置—例如，池的大小限制或在管理连接池的bean中使用的连接数。


此元数据转换为组成每个bean定义的一组属性。下表描述了这些属性:

|                          | 解释       |
| ------------------------ | ---------- |
| Class                    | 实例化bean |
| Name                     | 命名bean   |
| Scope                    | bean范围   |
| Constructor arguments    | 依赖注入   |
| Properties               | 依赖注入   |
| Autowiring mode          |            |
| Lazy initialization mode |            |
| Initialization method    |            |
| Destruction method       |            |

