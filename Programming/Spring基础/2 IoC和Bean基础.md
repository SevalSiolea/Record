## IoC容器和Bean简介

**依赖注入 (DI) 是 IoC 的一种特殊形式，其中对象仅通过构造函数参数、工厂方法参数或在对象实例构造或从工厂方法返回后设置在对象实例上的属性来定义其依赖项（即它们与之交互的其他对象）。然后，IoC 容器在创建 Bean 时注入这些依赖项。这个过程从根本上说是 Bean 本身控制其依赖项的实例化或位置的反面（因此得名控制反转），通过直接构造类或使用服务定位器模式等机制来实现。**

**`org.springframework.beans` 和 `org.springframework.context` 包是 Spring 框架 IoC 容器的基础。`BeanFactory` 提供配置框架和基本功能，而 `ApplicationContext` 添加更多企业级特定功能。**

`BeanFactory` 接口提供了一种高级配置机制，能够管理任何类型的对象。`ApplicationContext` 是`BeanFactory` 的子接口。它添加了

- 更轻松地与 Spring 的 AOP 功能集成

- 消息资源处理（用于国际化）

- 事件发布

- 应用程序层特定上下文，例如用于 Web 应用程序的 `WebApplicationContext`。

**`ApplicationContext` 是 `BeanFactory` 的完整超集，在本章中，我们专门使用它来描述 Spring 的 IoC 容器。**

**在 Spring 中，构成应用程序主干并由 Spring IoC 容器管理的对象称为 bean。bean 是由 Spring IoC 容器实例化、组装和管理的对象。bean 及其之间的依赖关系反映在容器使用的配置元数据中。**

## 容器概述

**`org.springframework.context.ApplicationContext` 接口代表 Spring IoC 容器，负责实例化、配置和组装 Bean。容器通过读取配置元数据来获取有关要实例化、配置和组装的组件的指令。配置元数据可以表示为带注解的组件类、带有工厂方法的配置类或外部 XML 文件或 Groovy 脚本。**

Spring 核心包含多个 `ApplicationContext` 接口的实现。在独立应用程序中，通常会创建 `AnnotationConfigApplicationContext` 或 `ClassPathXmlApplicationContext` 的实例。

**在大多数应用程序场景中，不需要显式用户代码来实例化一个或多个 Spring IoC 容器实例。**例如，在 Spring Boot 场景中，应用程序上下文会根据常见的设置约定隐式地为你引导。

下图显示了 Spring 工作原理的高级视图。您的应用程序类与配置元数据相结合，因此在创建和初始化 `ApplicationContext` 后，您将拥有一个完全配置且可执行的系统或应用程序。

![IoC工作原理](D:\Learn\Record\Programming\Spring基础\Image\2 IoC和Bean\IoC工作原理.png)

### 配置元数据

**Spring IoC 容器使用某种形式的配置元数据。此配置元数据表示您作为应用程序开发人员如何告诉 Spring 容器实例化、配置和组装应用程序中的组件。**

**Spring IoC 容器本身与实际写入此配置元数据的格式完全解耦。**

- **基于注解的配置：使用应用程序组件类上的基于注解的配置元数据定义 bean。**
- **基于 Java 的配置：使用基于 Java 的配置类，在应用程序类外部定义 bean。**

**Spring 配置至少包含一个，通常包含多个 bean 定义，容器必须管理这些定义。Java 配置通常在 `@Configuration` 类中使用 `@Bean` 注释的方法，每个方法对应一个 bean 定义。**

**这些 Bean 定义对应于构成应用程序的实际对象。**通常，人们不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是存储库和业务逻辑的责任。

### XML作为外部配置DSL

**基于 XML 的配置元数据将这些 Bean 配置为顶级 `<beans/>` 元素内的 `<bean/>` 元素。**
**`<bean id="..." class="...">......</bean>`**

- **`id` 属性是一个字符串，用于标识单个 Bean 定义。**
- **`class` 属性定义 Bean 的类型，并使用完全限定的类名。**

`id` 属性的值可用于引用协作对象。

**要实例化容器，需要将 XML 资源文件的路径提供给 `ClassPathXmlApplicationContext` 构造函数，该构造函数允许容器从各种外部资源（如本地文件系统、Java `CLASSPATH` 等）加载配置元数据。**

**`name`元素指的是JavaBean属性的名称，而`ref`元素指的是另一个bean定义的名称。`id`和`ref`元素之间的这种关联表达了协作对象之间的依赖关系。**

### 组合基于XML的配置元数据

将bean定义跨多个XML文件可能很有用。通常，每个单独的XML配置文件代表您架构中的逻辑层或模块。

**您可以使用`ClassPathXmlApplicationContext`构造函数从XML片段加载bean定义，此构造函数接受多个`Resource`位置。**
**或者，使用一个或多个`<import/>`元素来从另一个文件或多个文件中加载bean定义。**

```xml
<beans>
	<import resource="services.xml"/>
	<import resource="resources/messageSource.xml"/>
	<import resource="/resources/themeSource.xml"/>

	<bean id="bean1" class="..."/>
	<bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义从三个文件加载：`services.xml`、`messageSource.xml`和`themeSource.xml`。**所有位置路径都是相对于执行导入的定义文件的。**因此`services.xml`必须与执行导入的文件位于同一目录或类路径位置，而`messageSource.xml`和`themeSource.xml`必须位于导入文件位置下的`resources`位置。

如您所见，前导斜杠将被忽略。但是，鉴于这些路径是相对的，最好不要使用斜杠。
**可以使用相对路径“../”引用父目录中的文件，但这样做不建议。这样做会创建对当前应用程序外部文件的依赖关系。**特别是，不建议将此引用用于`classpath:` URL（例如，`classpath:../services.xml`），其中运行时解析过程会选择“最近”的类路径根，然后查看其父目录。类路径配置更改可能会导致选择不同的、不正确的目录。
您可以始终使用完全限定的资源位置而不是相对路径。但是，请注意，您正在将应用程序的配置与特定的绝对位置耦合。通常，最好为这些绝对位置保留一个间接层——例如，通过在运行时针对JVM系统属性解析的“${…}”占位符。

### 使用容器

**`ApplicationContext` 是一个高级工厂的接口，它能够维护不同 bean 及其依赖项的注册表。通过使用 `T getBean( String name, Class<T> requiredType )` 方法，您可以检索 bean 的实例。**

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext( "services.xml", "daos.xml" );

// retrieve configured instance
PetStoreService service = context.getBean( "petStore", PetStoreService.class );

// use configured instance
List<String> userList = service.getUsernameList();
```

`ApplicationContext` 接口还有一些其他方法用于检索 bean，但理想情况下，您的应用程序代码不应该使用它们。实际上，您的应用程序代码根本不应该调用 `getBean()` 方法，因此也不应该依赖于 Spring API。

## Bean概述

**Spring IoC 容器管理一个或多个 Bean。这些 Bean 是使用您提供给容器的配置元数据创建的。**

**在容器本身中，这些 Bean 定义表示为 `BeanDefinition` 对象，其中包含（除其他信息外）以下元数据：**

- **包限定的类名：通常是正在定义的 Bean 的实际实现类。**
- **Bean 行为配置元素：它说明 Bean 应该如何在容器中运行（范围、生命周期回调等）。**
- **对 Bean 完成其工作所需的其它 Bean 的引用：这些引用也称为协作者或依赖项。**
- **在新建对象中设置的其他配置设置：例如，管理连接池的 Bean 中池的大小限制或要使用的连接数。**

**这些元数据转化为一组属性，构成每个 Bean 定义。**

除了包含有关如何创建特定 Bean 的信息的 Bean 定义之外，`ApplicationContext` 实现还允许注册在容器外部（由用户）创建的现有对象。这是通过通过 `getBeanFactory()` 方法访问 `ApplicationContext` 的 `BeanFactory` 来完成的，该方法返回 `DefaultListableBeanFactory` 实现。`DefaultListableBeanFactory` 通过 `registerSingleton(..)` 和 `registerBeanDefinition(..)` 方法支持此注册。但是，典型的应用程序仅使用通过常规 Bean 定义元数据定义的 Bean。

**Bean 元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他内省步骤期间正确地推断它们。**
虽然在一定程度上支持覆盖现有元数据和现有单例实例，但运行时注册新 Bean（与对工厂的实时访问同时进行）不受官方支持，并且可能导致并发访问异常、Bean 容器中的状态不一致或两者兼而有之。

### 命名Bean

**每个 Bean 都有一个或多个标识符。这些标识符在托管 Bean 的容器中必须是唯一的。Bean 通常只有一个标识符。但是，如果它需要多个标识符，则额外的标识符可以被视为别名。**

**在基于 XML 的配置元数据中，您可以使用 `id` 属性、`name` 属性或两者来指定 Bean 标识符。`id` 属性允许您指定一个唯一的 `id`。如果您想为 Bean 引入其他别名，您也可以在 `name` 属性中指定它们，用逗号 (`,`)、分号 (`;`) 或空格分隔。**
如果您没有显式提供 `name` 或 `id`，容器将为该 Bean 生成一个唯一的名称。但是，如果您想通过 `ref` 元素或服务定位器样式查找来按名称引用该 Bean，则必须提供一个名称。

约定是使用标准 Java 约定为实例字段名称命名 Bean。也就是说，Bean 名称以小写字母开头，并从那里开始使用驼峰命名法。

使用类路径中的组件扫描，Spring 为未命名的组件生成 Bean 名称，遵循前面描述的规则：基本上，获取简单类名并将它的第一个字符转换为小写。但是，在（不常见）特殊情况下，如果有多个字符并且第一个和第二个字符都是大写，则会保留原始大小写。

### 在Bean定义之外为Bean设置别名

**在 Bean 定义的地方指定所有别名并不总是足够的。有时需要为在其他地方定义的 Bean 引入别名。**这在大型系统中很常见，在大型系统中，配置会分散在每个子系统中，每个子系统都有自己的对象定义集。

**在基于 XML 的配置元数据中，可以使用 `<alias/>` 元素来实现这一点。**
**`<alias name="fromName" alias="toName"/>`**
**在这种情况下，一个名为 `fromName` 的 Bean（在同一个容器中）也可以在使用此别名定义后被称为 `toName`。**

**如果您使用 Java 配置，则可以使用 `@Bean` 注解来提供别名。**

### 实例化Bean

**Bean 定义本质上是创建一个或多个对象的配方。容器在被要求时会查看命名 Bean 的配方，并使用该 Bean 定义封装的配置元数据来创建（或获取）一个实际的对象。**

**如果您使用基于 XML 的配置元数据，则可以在 `<bean/>` 元素的 `class` 属性中指定要实例化的对象的类型（或类）。**此 `class` 属性（在内部，是 `BeanDefinition` 实例上的 `Class` 属性）通常是必需的。
您可以通过以下两种方式之一使用 `Class` 属性。通常，为了指定在容器自身通过反射调用构造函数直接创建 Bean 时要构造的 Bean 类，这与使用`new`运算符的 Java 代码类似。其次，为了指定包含用于创建对象的`static`工厂方法的实际类，在容器调用类上的`static`工厂方法来创建 Bean 的不太常见的情况下。从`static`工厂方法调用返回的对象类型可能是同一个类或完全不同的类。

**如果您想为嵌套类配置 Bean 定义，可以使用嵌套类的二进制名称或源名称。**
例如，如果您在`com.example`包中有一个名为`SomeThing`的类，并且这个`SomeThing`类有一个名为`OtherThing`的`static`嵌套类，它们可以用美元符号 (`$`) 或点 (`.`) 分隔。因此，Bean 定义中`class`属性的值将是`com.example.SomeThing$OtherThing`或`com.example.SomeThing.OtherThing`。

### 使用构造函数实例化

**当您通过构造函数方法创建 Bean 时，所有普通类都可以被 Spring 使用并与 Spring 兼容。也就是说，正在开发的类不需要实现任何特定接口或以特定方式编码。只需指定 Bean 类就足够了。**但是，根据您为该特定 Bean 使用的 IoC 类型，您可能需要一个默认（空）构造函数。

**Spring IoC 容器可以管理您想要管理的几乎任何类。它不限于管理真正的 JavaBean。**
**大多数 Spring 用户更喜欢实际的 JavaBean，它们只包含一个默认（无参数）构造函数以及根据容器中的属性建模的适当的 setter 和 getter。**

在构造函数参数的情况下，容器可以在多个重载构造函数中选择一个相应的构造函数。

### 使用静态工厂方法实例化

**在定义使用静态工厂方法创建的 bean 时，使用 `class` 属性指定包含 `static` 工厂方法的类，并使用名为 `factory-method` 的属性指定工厂方法本身的名称。**
**您应该能够调用此方法（带可选参数，如下所述）并返回一个活动对象，该对象随后将被视为通过构造函数创建。**

在工厂方法参数的情况下，容器可以在同名多个重载方法中选择相应的方法。

### 使用实例工厂方法实例化

**与通过静态工厂方法实例化类似，使用实例工厂方法实例化会调用容器中现有 bean 的非静态方法来创建新的 bean。**
**要使用此机制，请将 `class` 属性留空，并在 `factory-bean` 属性中指定当前（或父级或祖先）容器中包含要调用的实例方法的 bean 的名称以创建对象。使用 `factory-method` 属性设置工厂方法本身的名称。**

在 Spring 文档中，“工厂 Bean” 指的是在 Spring 容器中配置的 Bean，它通过实例或静态工厂方法创建对象。相比之下，`FactoryBean`（注意大小写）指的是 Spring 特定的 `FactoryBean` 实现类。

### 确定Bean的运行时类型

确定特定 Bean 的运行时类型并非易事。
Bean 元数据定义中指定的类只是一个初始类引用，可能与声明的工厂方法相结合，或者是一个 `FactoryBean` 类，这可能导致 Bean 的运行时类型不同，或者在实例级工厂方法的情况下根本没有设置（通过指定的 `factory-bean` 名称解析）。
此外，AOP 代理可能会用基于接口的代理包装 Bean 实例，该代理对目标 Bean 的实际类型（仅其实现的接口）的暴露有限。

**查找特定 Bean 的实际运行时类型的推荐方法是针对指定 Bean 名称调用 `BeanFactory.getType`。**这将考虑上述所有情况，并返回 `BeanFactory.getBean` 调用针对相同 Bean 名称将返回的对象类型。

## 依赖项注入

**依赖注入 (DI) 是一个过程，其中对象仅通过构造函数参数、工厂方法的参数或在对象实例构建或从工厂方法返回后设置在其上的属性来定义其依赖项（即它们一起工作的其他对象）。然后，容器在创建 Bean 时注入这些依赖项。此过程从根本上是 Bean 本身通过直接构建类或服务定位器模式控制其依赖项的实例化或位置的逆过程（因此得名，控制反转）。**

遵循 DI 原则，代码会更简洁，当对象提供其依赖项时，解耦会更有效。对象不会查找其依赖项，也不会知道依赖项的位置或类。因此，你的类会变得更容易测试。

### 基于构造函数的依赖项注入

**基于构造函数的 DI 是通过容器调用具有多个参数的构造函数来实现的，每个参数都表示一个依赖项。使用特定参数调用`static` 工厂方法来构造 bean 几乎是等效的。**

**构造函数参数解析匹配是通过使用参数的类型来进行的。如果 bean 定义的构造函数参数中不存在潜在的歧义，则 bean 定义中构造函数参数的定义顺序就是 bean 实例化时将这些参数提供给适当构造函数的顺序。**

**当使用Bean时，使用 `<constructor-arg ref="..."/>` 提供。你无需在 `<constructor-arg/>` 元素中显式指定构造函数参数索引或类型。当引用另一个 Bean 时，类型是已知的，并且可以进行匹配。**
**当使用基本类型或预定义类时，使用 `<constructor-arg type="..." value="..."/>` 提供，其中`type`属性显式指定参数类型。**
**可以使用 `index` 属性显式指定构造函数参数的索引。除了解决多个简单值的不确定性之外，指定索引还可以解决构造函数具有两个相同类型参数的不确定性。索引从 0 开始。**

### 基于Setter的依赖项注入

**基于 Setter 的 DI 是通过容器在调用无参数构造函数或无参数 `static` 工厂方法以实例化 Bean 之后，在 Bean 上调用 Setter 方法来完成的。如果配置元数据在XML中，用`property`元素进行依赖注入。**

**`ApplicationContext` 支持对其管理的 Bean 进行基于构造函数和基于 Setter 的 DI。它还支持在通过构造函数方法注入一些依赖项之后进行基于 Setter 的 DI。**
您可以使用 `BeanDefinition` 的形式配置依赖项，并将其与 `PropertyEditor` 实例结合使用，以将属性从一种格式转换为另一种格式。
但是，大多数 Spring 用户不会直接（即以编程方式）使用这些类，而是使用 XML `bean` 定义、带注释的组件（即用 `@Component`、`@Controller` 等注释的类）或基于 Java 的 `@Configuration` 类中的 `@Bean` 方法。然后，这些源在内部转换为 `BeanDefinition` 实例，并用于加载整个 Spring IoC 容器实例。

**由于您可以混合基于构造函数和基于Setter的 DI，因此一个好的经验法则是对强制依赖项使用构造函数，对可选依赖项使用Setter方法。**
**Spring 团队通常提倡构造函数注入，因为它允许您将应用程序组件实现为不可变对象，并确保必需的依赖项不为 `null`。此外，构造函数注入的组件始终以完全初始化的状态返回给客户端（调用）代码。**
**Setter注入主要应仅用于可选依赖项，这些依赖项可以在类中分配合理的默认值。否则，必须在代码使用依赖项的任何位置执行非空检查。Setter注入的一个好处是，设置程序方法使该类的对象易于在以后重新配置或重新注入。**

### 依赖项解析过程

容器按如下方式执行 Bean 依赖项解析：

- 创建 `ApplicationContext` 并使用描述所有 Bean 的配置元数据对其进行初始化。配置元数据可以通过 XML、Java 代码或注释指定。
- 对于每个 Bean，其依赖项以属性、构造函数参数或静态工厂方法的参数的形式表示。在实际创建 Bean 时，会向 Bean 提供这些依赖项。
- 每个属性或构造函数参数都是要设置的值的实际定义，或对容器中另一个 Bean 的引用。
- 每个作为值的属性或构造函数参数都从其指定格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring 可以将以字符串格式提供的数值转换为所有内置类型，例如 `int`、`long`、`String`、`boolean` 等。

**Spring 容器在创建容器时验证每个 Bean 的配置。但是，Bean 属性本身直到实际创建 Bean 时才设置。**
**当创建容器时，会创建单例作用域的 Bean 并将其设置为预实例化（默认）。否则，仅在请求 Bean 时才创建 Bean。**
创建 Bean 可能会导致创建 Bean 循环依赖，因为会创建并分配 Bean 的依赖项及其依赖项的依赖项（依此类推）。请注意，这些依赖项之间的解析不匹配可能会在后期显示，即在首次创建受影响的 Bean 时。

如果您主要使用构造函数注入，则可能会创建无法解析的循环依赖项场景。
例如：类 A 通过构造函数注入需要类 B 的实例，而类 B 通过构造函数注入需要类 A 的实例。如果您将类 A 和 B 的 Bean 配置为相互注入，则 Spring IoC 容器会在运行时检测到此循环引用，并抛出 `BeanCurrentlyInCreationException`。
一种可能的解决方案是编辑某些类的源代码，以便通过 setter 而不是构造函数进行配置。
与典型情况（没有循环依赖项）不同，Bean A 和 Bean B 之间的循环依赖项迫使其中一个 Bean 在自身完全初始化之前注入到另一个 Bean 中。

您通常可以相信 Spring 会做正确的事情。它会在容器加载时检测配置问题，例如对不存在的 Bean 的引用和循环依赖项。
**Spring 尽可能晚地设置属性并解析依赖项，即在实际创建 Bean 时。这意味着，如果在创建对象或其依赖项时出现问题，则正确加载的 Spring 容器稍后在您请求对象时可能会生成异常。**
某些配置问题的这种潜在延迟可见性是 `ApplicationContext` 实现默认预实例化单例 Bean 的原因。在实际需要这些 Bean 之前创建这些 Bean 会花费一些前期时间和内存，但您可以在创建 `ApplicationContext` 时发现配置问题，而不是在以后。您仍然可以覆盖此默认行为，以便单例 Bean 延迟初始化，而不是急切地预实例化。
**如果没有循环依赖，当一个或多个协作 bean 注入到一个依赖 bean 中时，每个协作 bean 在注入到依赖 bean 之前都已完全配置。**这意味着，如果 bean A 依赖于 bean B，Spring IoC 容器在调用 bean A 上的 setter 方法之前将完全配置 bean B。换句话说，bean 已实例化（如果它不是预先实例化的单例），其依赖项已设置，并且相关的生命周期方法已调用。

## Bean作用域

**您不仅可以控制要插入从特定 bean 定义创建的对象中的各种依赖关系和配置值，还可以控制从特定 bean 定义创建的对象的范围。这种方法功能强大且灵活，因为您可以通过配置选择要创建的对象的范围，而不必在 Java 类级别中设置对象的范围。**
**可以定义 bean 以在多个范围之一中部署。Spring Framework 支持六个范围，其中四个仅在使用支持 Web 的 `ApplicationContext` 时可用。**

|        范围        |                             描述                             |
| :----------------: | :----------------------------------------------------------: |
|     singleton      | （默认）将单个 bean 定义作用域限定为每个 Spring IoC 容器的单个对象实例。 |
|     prototype      |       将单个 bean 定义作用域限定为任意数量的对象实例。       |
|   request（Web）   | 将单个 bean 定义的作用域限定为单个 HTTP 请求的生命周期。也就是说，每个 HTTP 请求都有一个自己的 bean 实例，该实例根据单个 bean 定义创建。 |
|   session（Web）   |  将单个 bean 定义的作用域限定为 HTTP `Session` 的生命周期。  |
| application（Web） | 将单个 bean 定义的作用域限定为 `ServletContext` 的生命周期。 |
|  websocket（Web）  |   将单个 bean 定义的作用域限定为 `WebSocket` 的生命周期。    |

### 单例作用域

**仅管理一个单例 bean 的共享实例，并且对具有与该 bean 定义匹配的 ID 或 ID 的 bean 的所有请求都会导致 Spring 容器返回该特定 bean 实例。**
换句话说，当你定义一个 bean 定义并且它的作用域为单例时，Spring IoC 容器会创建由该 bean 定义定义的对象的一个实例。这个单一实例存储在这样的单例 bean 的缓存中，并且对该命名 bean 的所有后续请求和引用都会返回缓存对象。

Spring 的单例 bean 的概念不同于四人帮 (GoF) 模式书中定义的单例模式。GoF 单例硬编码对象的范围，以便每个 ClassLoader 仅创建一个特定类的实例。Spring 单例的作用域最好描述为按容器和按 bean。这意味着，如果你为单个 Spring 容器中的特定类定义一个 bean，Spring 容器会创建由该 bean 定义定义的类的唯一实例。

**单例作用域是 Spring 中的默认作用域。要在 XML 中将 bean 定义为单例，可以：**
**`<bean id="..." class="..." scope="singleton"/>`**

### 原型作用域

**bean 部署的非单例原型作用域会导致在每次对特定 bean 发出请求时创建一个新的 bean 实例。**也就是说，bean 注入到另一个 bean 中，或者你通过容器上的 `getBean()` 方法调用请求它。
**通常，你应该对所有有状态 bean 使用原型作用域，对无状态 bean 使用单例作用域。**

**可以如下在XML中将Bean定义为原型：**
**`<bean id="..." class="..." scope="prototype">`**

**与其他作用域相反，Spring 不会管理原型 bean 的完整生命周期。**
容器实例化、配置并以其他方式组装原型对象，并将其交给客户端，而不会进一步记录该原型实例。因此，虽然初始化生命周期回调方法在所有对象上调用，无论作用域如何，但在原型的情况下，不会调用配置的销毁生命周期回调。客户端代码必须清理原型作用域对象并释放原型 bean 持有的昂贵资源。
**在某些方面，Spring 容器在原型作用域 Bean 中的作用是 Java `new` 运算符的替代。该点之后的所有生命周期管理都必须由客户端处理。**

当您将单例作用域 Bean 与原型 Bean 的依赖项结合使用时，请注意在实例化时解析依赖项。因此，如果您将原型作用域 Bean 依赖注入到单例作用域 Bean 中，则会实例化一个新的原型 Bean，然后将其依赖注入到单例 Bean 中。原型实例是唯一提供给单例作用域 Bean 的实例。
但是，假设您希望单例作用域 Bean 在运行时重复获取原型作用域 Bean 的新实例。您无法将原型作用域 Bean 依赖注入到单例 Bean 中，因为该注入仅发生一次，即当 Spring 容器实例化单例 Bean 并解析和注入其依赖项时。如果您需要在运行时多次获取原型 Bean 的新实例，请参阅“方法注入”。

### 自定义范围

**bean 范围机制是可扩展的。你可以定义自己的范围，甚至重新定义现有范围，尽管后者被认为是不良做法，并且不能覆盖内置的 `singleton` 和 `prototype` 范围。**

**要将自定义范围集成到 Spring 容器中，需要实现 `org.springframework.beans.factory.config.Scope` 接口。**有关如何实现自己的范围的想法，请参阅 Spring Framework 本身提供的 `Scope` 实现和 `Scope` javadoc，它更详细地解释了需要实现的方法。

`Scope` 接口有四个方法从作用域中获取对象、从作用域中移除对象并让它们被销毁。

以下方法从基础作用域返回对象：
`Object get( String name, ObjectFactory<?> objectFactory )`

以下方法从基础作用域移除对象：
`Object remove( String name )`

以下方法注册一个回调，当作用域被销毁或作用域中的指定对象被销毁时，作用域应调用该回调：
`void registerDestructionCallback( String name, Runnable destructionCallback )`

以下方法获取基础作用域的会话标识符：
`String getConversationId()`
此标识符对于每个作用域都是不同的。对于会话作用域实现，此标识符可以是会话标识符。

**编写并测试一个或多个自定义 `Scope` 实现后，你需要让 Spring 容器了解你的新作用域。以下方法是向 Spring 容器注册新 `Scope` 的核心方法：**
`**void registerScope( String scopeName, Scope scope );**`

`registerScope(..)` 方法的第一个参数是与作用域关联的唯一名称。Spring 容器本身中此类名称的示例是 `singleton` 和 `prototype`。`registerScope(..)` 方法的第二个参数是你希望注册和使用的自定义 `Scope` 实现的实际实例。
然后，您可以创建遵守自定义 `Scope` 的作用域规则的 bean 定义。

## Bean的生命周期

**为了与容器管理的 Bean 生命周期进行交互，您可以实现 Spring 的 `InitializingBean` 和 `DisposableBean` 接口。容器会分别调用前者的 `afterPropertiesSet()` 和后者的 `destroy()` 方法，让 Bean 在初始化和销毁时执行某些操作。**

JSR-250 的 `@PostConstruct` 和 `@PreDestroy` 注解通常被认为是在现代 Spring 应用程序中接收生命周期回调的最佳实践。使用这些注解意味着您的 Bean 不与 Spring 特定的接口耦合。

**在内部，Spring 框架使用 `BeanPostProcessor` 实现来处理它可以找到的任何回调接口，并调用相应的方法。如果您需要自定义功能或 Spring 默认不提供的其他生命周期行为，您可以自己实现 `BeanPostProcessor`。**

**除了初始化和销毁回调之外，Spring 管理的对象还可以实现 `Lifecycle` 接口，以便这些对象可以参与由容器自身生命周期驱动的启动和关闭过程。**

### 初始化回调

**`org.springframework.beans.factory.InitializingBean` 接口允许 Bean 在容器设置 Bean 上所有必要的属性后执行初始化工作。**
**`InitializingBean` 接口指定了一个方法：**
`void afterPropertiesSet() throws Exception;`

**我们建议您不要使用`InitializingBean`接口，因为它会不必要地将代码耦合到 Spring。**
**作为替代方案，我们建议使用`@PostConstruct`注解或指定一个 POJO 初始化方法。在基于 XML 的配置元数据的情况下，您可以使用`init-method`属性来指定具有 void 无参数签名的该方法的名称。使用 Java 配置，您可以使用`@Bean`的`initMethod`属性。**

### 销毁回调

**实现`org.springframework.beans.factory.DisposableBean`接口可以让 bean 在包含它的容器被销毁时获得回调。**
**`DisposableBean`接口指定了一个方法：**
`void destroy() throws Exception;`

**我们建议您不要使用`DisposableBean`回调接口，因为它会不必要地将代码耦合到 Spring。**
**作为替代方案，我们建议使用`@PreDestroy`注解或指定 bean 定义支持的通用方法。使用基于 XML 的配置元数据，您可以在`<bean/>`上的`destroy-method`属性中使用它。使用 Java 配置，您可以使用`@Bean`的`destroyMethod`属性。**

Spring 还支持推断销毁方法，检测公共的`close`或`shutdown`方法。
这是 Java 配置类中`@Bean`方法的默认行为，并自动匹配`java.lang.AutoCloseable`或`java.io.Closeable`实现，也不会将销毁逻辑耦合到 Spring。
对于使用 XML 的销毁方法推断，您可以将`<bean>`元素的`destroy-method`属性分配一个特殊的`(inferred)`值，这将指示 Spring 自动检测 bean 类上的公共`close`或`shutdown`方法，以用于特定 bean 定义。您也可以在`<beans>`元素的`default-destroy-method`属性上设置此特殊的`(inferred)`值，以将此行为应用于一组完整的 bean 定义。

### 默认初始化和销毁方法

**作为应用程序开发人员，您可以编写应用程序类并使用名为 `init()` 的初始化回调，而无需为每个 bean 定义配置 `init-method="init"` 属性。Spring IoC 容器在创建 bean 时（并根据之前描述的标准生命周期回调契约）调用该方法。**

当您编写不使用 Spring 特定的 `InitializingBean` 和 `DisposableBean` 回调接口的初始化和销毁方法回调时，通常会编写名称为 `init()`、`initialize()`、`dispose()` 等等的方法。理想情况下，此类生命周期回调方法的名称在整个项目中应标准化，以便所有开发人员使用相同的名称并确保一致性。

**顶层 `<beans/>` 元素属性上 `default-init-method` 属性的存在会导致 Spring IoC 容器将 bean 类中名为 `init` 的方法识别为初始化方法回调。您可以通过使用顶层 `<beans/>` 元素上的 `default-destroy-method` 属性来类似地配置销毁方法回调。**

在现有 bean 类已经具有与约定不一致的回调方法的情况下，您可以通过使用 `<bean/>` 本身的 `init-method` 和 `destroy-method` 属性来指定方法名称以覆盖默认值。

### 组合生命周期机制

**您有三种选择来控制 bean 生命周期行为**

- **`InitializingBean` 和 `DisposableBean` 回调接口**
- **自定义的 `init()` 和 `destroy()` 方法**
- **`@PostConstruct` 和 `@PreDestroy` 注解**

**您可以将这些机制结合起来以控制给定的 Bean。**

为同一个 Bean 配置的多个生命周期机制，使用不同的初始化方法，调用顺序如下：

1. 使用 `@PostConstruct` 注解的方法

2. `InitializingBean` 回调接口定义的 `afterPropertiesSet()`

3. 自定义配置的 `init()` 方法

销毁方法按相同顺序调用：

1. 使用 `@PreDestroy` 注解的方法
2. `DisposableBean` 回调接口定义的 `destroy()`
3. 自定义配置的 `destroy()` 方法

如果为 Bean 配置了多个生命周期机制，并且每个机制都配置了不同的方法名称，则每个配置的方法将按照列出的顺序运行。但是，如果配置了相同的方法名称，对于这些生命周期机制中的多个，该方法仅运行一次。

### 启动和关闭回调

**`Lifecycle` 接口定义了任何具有自身生命周期要求（例如启动和停止某些后台进程）的对象的基本方法：**

```java
public interface Lifecycle {
	void start();
	void stop();
	boolean isRunning();
}
```

**任何 Spring 管理的对象都可以实现 `Lifecycle` 接口。然后，当 `ApplicationContext` 本身接收到启动和停止信号（例如，在运行时停止/重启场景）时，它会将这些调用级联到该上下文中定义的所有 `Lifecycle` 实现。**
它通过委托给 `LifecycleProcessor` 来实现。`LifecycleProcessor` 本身是 `Lifecycle` 接口的扩展。它还添加了另外两种方法来响应上下文刷新和关闭。

## Bean定义继承

**子 Bean 定义从父定义继承配置数据。子定义可以覆盖某些值或根据需要添加其他值。**使用父 Bean 定义和子 Bean 定义可以节省很多输入。实际上，这是一种模板形式。

如果您以编程方式使用 `ApplicationContext` 接口，子 Bean 定义由 `ChildBeanDefinition` 类表示。大多数用户不会在此级别使用它们。相反，他们会在诸如 `ClassPathXmlApplicationContext` 之类的类中声明式地配置 Bean 定义。
**当您使用基于 XML 的配置元数据时，您可以使用 `parent` 属性来指示子 Bean 定义，并将父 Bean 指定为此属性的值。**
**`<bean id="..." class="..." parent="..."></bean>`**

**子 Bean 定义如果未指定，则使用父定义中的 Bean 类，但也可以覆盖它。在后一种情况下，子 Bean 类必须与父类兼容（即，它必须接受父类的属性值）。**
**子 Bean 定义从父类继承作用域、构造函数参数值、属性值和方法覆盖，可以选择添加新值。**
您指定的任何作用域、初始化方法、销毁方法或 `static` 工厂方法设置都会覆盖相应的父设置。其余设置始终从子定义中获取：依赖项、自动装配模式、依赖项检查、单例和延迟初始化。

使用 `abstract` 属性显式地将父 Bean 定义标记为抽象。如果父定义未指定类，则需要显式地将父 Bean 定义标记为 `abstract`。
`<bean id="..." abstract="true"></bean>`
父 Bean 无法自行实例化，因为它不完整，并且也显式地标记为 `abstract`。当定义为 `abstract` 时，它只能用作纯模板 Bean 定义，作为子定义的父定义。尝试单独使用此类 `abstract` 父 Bean，通过将其引用为另一个 Bean 的 ref 属性或对父 Bean ID 执行显式 `getBean()` 调用，会导致错误。

## 基于注解的容器配置

**Spring 为基于注解的配置提供了全面的支持，通过在相关类、方法或字段声明上使用注解，对组件类本身的元数据进行操作。**

**注解注入在外部属性注入之前执行。因此，外部配置（例如 XML 指定的 bean 属性）在通过混合方法进行连接时，实际上会覆盖属性的注解。**

### 使用`@Autowired`

**您可以将 `@Autowired` 注解应用于构造函数。**
如果目标 Bean 仅定义一个构造函数，则不再需要在该构造函数上使用 `@Autowired` 注解。但是，如果存在多个构造函数并且没有主/默认构造函数，则至少必须在其中一个构造函数上使用 `@Autowired` 注解，以指示容器使用哪个构造函数。

**您还可以将 `@Autowired` 注解应用于传统setter 方法。**
**您还可以将 `@Autowired` 注解应用于具有任意名称和多个参数的方法。**
**您也可以将 `@Autowired` 注解应用于字段，甚至可以将其与构造函数混合使用。**

**您也可以通过将`@Autowired`注解添加到期望该类型数组的字段或方法中，来指示 Spring 从`ApplicationContext`提供特定类型的全部 Bean。对于类型化集合同样适用。只要预期的键类型为`String`，即使是类型化的`Map`实例也可以自动装配。值包含所有预期类型的 Bean，键包含相应的 Bean 名称。**
默认情况下，当给定注入点没有匹配的候选 Bean 时，自动装配会失败。对于声明的数组、集合或映射，至少需要一个匹配的元素。

**默认行为是将带注释的方法和字段视为指示必需的依赖项。**
您可以更改此行为，如以下示例所示，通过将注入点标记为非必需（即，通过在 `@Autowired` 中将 `required` 属性设置为 `false`）来使框架跳过无法满足的注入点。
如果其依赖项（或多个参数的情况下，其依赖项之一）不可用，则不会调用非必需方法。在这种情况下，非必需字段将不会被填充，而是保留其默认值。
换句话说，将 `required` 属性设置为 `false` 表示相应的属性对于自动装配是可选的，如果无法自动装配，则该属性将被忽略。这允许属性分配默认值，这些值可以通过依赖项注入选择性地覆盖。

注入的构造函数和工厂方法参数是一个特殊情况，因为 `@Autowired` 中的 `required` 属性具有稍微不同的含义，这是由于 Spring 的构造函数解析算法可能会处理多个构造函数。
构造函数和工厂方法参数默认情况下实际上是必需的，但在单构造函数场景中有一些特殊规则，例如多元素注入点（数组、集合、映射）在没有匹配的 bean 时解析为空实例。这允许使用一种常见的实现模式，其中所有依赖项都可以在一个唯一的多个参数构造函数中声明。例如，声明为一个没有 `@Autowired` 注释的单个公共构造函数。

或者，您可以通过 Java 8 的 `java.util.Optional` 来表达特定依赖项的非必需性质。
从 Spring Framework 5.0 开始，您还可以使用 `@Nullable` 注释。

**您还可以将 `@Autowired` 用于众所周知的可解析依赖项的接口，例如`BeanFactory`和`ApplicationContext`。这些接口及其扩展接口会自动解析，无需任何特殊设置。**

`@Autowired`、`@Inject`、`@Value` 和 `@Resource` 注释由 Spring `BeanPostProcessor` 实现处理。这意味着您不能在您自己的 `BeanPostProcessor` 或 `BeanFactoryPostProcessor` 类型（如果有）中应用这些注释。这些类型必须通过使用 XML 或 Spring `@Bean` 方法显式“连接”。

## 基于Java的容器配置

### `@Bean`和@`Configuration`

**Spring 的 Java 配置支持中的核心构件是使用 `@Configuration` 注解的类和使用 `@Bean` 注解的方法。**

**`@Bean` 注解用于指示方法实例化、配置和初始化一个新的对象，该对象将由 Spring IoC 容器管理。对于熟悉 Spring 的XML 配置的人来说，`@Bean` 注解的作用与 `<bean/>` 元素相同。**
您可以将 `@Bean` 注解的方法与任何 Spring `@Component` 一起使用。但是，它们最常与 `@Configuration` bean 一起使用。

**使用 `@Configuration` 注解类表示其主要目的是作为 bean 定义的来源。此外，`@Configuration` 类允许通过在同一类中调用其他 `@Bean` 方法来定义 bean 之间的依赖关系。**

### 使用`@Bean`注解

**`@Bean` 是一个方法级注解，是 XML `<bean/>` 元素的直接对应项。您可以在 `@Configuration` 注解或 `@Component` 注解的类中使用 `@Bean` 注解。**

**您可以使用此方法在 `ApplicationContext` 中注册 Bean 定义，类型指定为方法的返回值。默认情况下，Bean 名称与方法名称相同。**
**以下`@Bean`配置与XML配置等效：**

```java
@Configuration
public class AppConfig {

	@Bean
	public TransferServiceImpl transferService() {
		return new TransferServiceImpl();
	}
}
```

```xml
<beans>
	<bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

您还可以使用默认方法定义 Bean。这允许通过在默认方法上实现带有 Bean 定义的接口来组合 Bean 配置。

```java
public interface BaseConfig {

	@Bean
	default TransferServiceImpl transferService() {
		return new TransferServiceImpl();
	}
}

@Configuration
public class AppConfig implements BaseConfig {

}
```

**带 `@Bean` 注解的方法可以具有任意数量的参数，这些参数描述了构建该 Bean 所需的依赖项。**解析机制与基于构造函数的依赖项注入非常相似。

**使用 `@Bean` 注解定义的任何类都支持常规生命周期回调，并且可以使用 JSR-250 中的 `@PostConstruct` 和 `@PreDestroy` 注解。**
**如果某个 bean 实现 `InitializingBean`、`DisposableBean` 或 `Lifecycle`，则容器会调用它们各自的方法。**
在构建期间直接调用 `init()` 方法也是同样有效的。当直接在 Java 中工作时，您可以对对象执行任何操作，而不必总是依赖容器生命周期。

`@Bean` 注解支持指定任意初始化和销毁回调方法，非常类似于 Spring XML 中 `bean` 元素上的 `init-method` 和 `destroy-method` 属性。
`@Bean( initMethod = "init" )`
`@Bean( destroyMethod = "destory" )`

**Spring 包含 `@Scope` 注解，以便您可以指定 Bean 的作用域。默认作用域为 `singleton`，但您可以使用 `@Scope` 注解覆盖此作用域。**

**默认情况下，配置类使用 `@Bean` 方法的名称作为结果 Bean 的名称。但是，可以使用 `name` 属性覆盖此功能。**
**有时需要为单个 Bean 提供多个名称，也称为 Bean 别名。`@Bean` 注解的 `name` 属性接受一个 String 数组用于此目的。**
**`@Bean( "..." )`**
**`@Bean( { "...", "...", "..." } )`**

有时，提供 Bean 的更详细文本描述很有帮助。要向 `@Bean` 添加描述，可以使用 `@Description` 注解。

### 使用`@Configuration`注解

**`@Configuration` 是一个类级别注解，表示一个对象是 bean 定义的来源。`@Configuration` 类通过 `@Bean` 注解的方法声明 bean。**
**对 `@Configuration` 类上的 `@Bean` 方法的调用也可以用来定义 bean 之间的依赖关系。**

**当 bean 之间存在依赖关系时，表达这种依赖关系只需让一个 bean 方法调用另一个 bean 方法。**
这种声明 bean 之间依赖关系的方法仅在 `@Bean` 方法在 `@Configuration` 类中声明时才有效。您不能通过使用普通的 `@Component` 类来声明 bean 之间的依赖关系。