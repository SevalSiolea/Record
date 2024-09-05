**面向方面编程 (AOP) 通过提供另一种思考程序结构的方式来补充面向对象编程 (OOP)。**
**OOP 中模块化的关键单元是类，而在 AOP 中模块化的单元是方面。方面使跨多个类型和对象的关注点（例如事务管理）模块化成为可能。**此类关注点在 AOP 文献中通常称为“横切”关注点。

Spring 的关键组件之一是 AOP 框架。虽然 Spring IoC 容器不依赖于 AOP（这意味着如果你不想使用 AOP，则不必使用它），但 AOP 补充了 Spring IoC，以提供非常有能力的中间件解决方案。

AOP 在 Spring 框架中用于

- 提供声明式企业服务。最重要的此类服务是声明式事务管理。
- 允许用户实现自定义方面，用 AOP 补充他们对 OOP 的使用。

如果您仅对通用声明式服务或其他预先打包的声明式中间件服务（如池化）感兴趣，则无需直接使用 Spring AOP，并且可以跳过本章的大部分内容。

## AOP概念

### AOP的核心概念和术语

方面：**跨越多个类的关注点模块化。**事务管理是企业 Java 应用程序中跨领域关注点的一个很好的例子。**在 Spring AOP 中，方面是通过使用普通类（基于模式的方法）或使用 `@Aspect` 注解的普通类（AspectJ 风格）来实现的。**

连接点：**程序执行过程中的一个点，例如方法的执行或异常的处理。在 Spring AOP 中，连接点始终表示方法执行。**

通知：**方面在特定连接点采取的操作。**不同的通知类型包括“around”、“before”和“after”通知，将在后面讨论。许多 AOP 框架，包括 Spring，将通知建模为拦截器，并在连接点周围维护一个拦截器链。

切入点：**匹配连接点的谓词。通知与切入点表达式相关联，并在切入点匹配的任何连接点运行（例如，执行具有特定名称的方法）。连接点通过切入点表达式匹配的概念是 AOP 的核心，Spring 默认使用 AspectJ 切入点表达式语言。**

引入：**代表类型声明附加方法或字段。**Spring AOP 允许您向任何被通知的对象引入新的接口（以及相应的实现）。在 AspectJ 社区中，引入被称为类型间声明。

目标对象：**被一个或多个方面通知的对象。**也称为“被通知对象”。由于 Spring AOP 是通过使用运行时代理实现的，因此此对象始终是代理对象。

AOP 代理：**由 AOP 框架创建的对象，用于实现方面契约。**（通知方法执行等）在 Spring 框架中，AOP 代理是 JDK 动态代理或 CGLIB 代理。

织入：**将方面与其他应用程序类型或对象链接起来以创建被通知对象。**这可以在编译时（例如，使用 AspectJ 编译器）、加载时或运行时完成。**Spring AOP 与其他纯 Java AOP 框架一样，在运行时执行织入。**

### Spring AOP的通知类型

前置通知：**在连接点之前运行的通知，但没有能力阻止执行流程继续进行到连接点。**（除非它抛出异常）

后置返回通知：**在连接点正常完成之后运行的通知。**（例如，如果方法在不抛出异常的情况下返回）

后置异常通知：**在方法通过抛出异常时运行的通知。**

最终通知：**无论连接点以何种方式退出（正常或异常返回），都将运行的通知。**

环绕通知：**围绕连接点（例如方法调用）的通知。这是最强大的通知类型。环绕通知可以在方法调用之前和之后执行自定义行为。它还负责选择是否继续执行连接点，或者通过返回自己的返回值或抛出异常来跳过被通知方法的执行。**

环绕通知是最通用的通知类型。由于 Spring AOP 与 AspectJ 一样，提供了全面的通知类型，我们建议您使用能够实现所需行为的最低效的通知类型。使用最具体的通知类型提供了一个更简单的编程模型，并且错误的可能性更小。

所有通知参数都是静态类型的，因此您可以使用适当类型的通知参数（例如，方法执行返回的值的类型），而不是 `Object` 数组。

**连接点由切点匹配的概念是 AOP 的关键，它将 AOP 与仅提供拦截的旧技术区分开来。切点使通知能够独立于面向对象层次结构进行定位。**

### Spring AOP的功能和目标

**Spring AOP 是用纯 Java 实现的。**不需要特殊的编译过程。Spring AOP 不需要控制类加载器层次结构，因此适合在 servlet 容器或应用程序服务器中使用。

**Spring AOP 目前仅支持方法执行连接点（对 Spring bean 上方法的执行进行通知）。**
字段拦截没有实现，尽管可以在不破坏核心 Spring AOP API 的情况下添加对字段拦截的支持。如果您需要建议字段访问和更新连接点，请考虑使用 AspectJ 等语言。

**Spring AOP 与大多数其他 AOP 框架不同，其目标不是提供最完整的 AOP 实现。**
**相反，其目标是提供 AOP 实现与 Spring IoC 之间的紧密集成，以帮助解决企业应用程序中的常见问题。Spring 框架的 AOP 功能通常与 Spring IoC 容器一起使用。方面是使用正常的 bean 定义语法配置的，这是与其他 AOP 实现的关键区别。**

**Spring AOP 从未努力与 AspectJ 竞争以提供全面的 AOP 解决方案。Spring AOP 等基于代理的框架和 AspectJ 等完整框架都很有价值，它们是互补的，而不是竞争的。**Spring 无缝地将 Spring AOP 和 IoC 与 AspectJ 集成，以在一致的基于 Spring 的应用程序体系结构中启用 AOP 的所有用途。这种集成不会影响 Spring AOP API 或 AOP Alliance API。Spring AOP 保持向后兼容。

## 基于模式的AOP支持

**如果您更喜欢基于 XML 的格式，Spring 还提供对使用 `aop` 命名空间标记定义切面的支持。与使用 AspectJ 时完全相同的切点表达式和通知类型受支持。**
**要使用本节中描述的 aop 命名空间标记，您需要导入 `spring-aop` 架构。**

**在 Spring 配置中，所有切面和通知者元素都必须放在 `<aop:config>` 元素中。您可以在应用程序上下文配置中有多个 `<aop:config>` 元素。`<aop:config>` 元素可以包含切点、通知者和切面元素，这些元素必须按该顺序声明。**

**`<aop:config>` 配置风格大量使用了 Spring 的自动代理机制。**
如果您已经通过使用 `BeanNameAutoProxyCreator` 或类似内容显式地使用了自动代理，这可能会导致问题（例如，未编织建议）。建议的使用模式是仅使用 `<aop:config>` 风格或仅使用 `AutoProxyCreator` 风格，并且永远不要将它们混合使用。

### 声明切面

**切面是作为 Spring 应用程序上下文中 bean 定义的常规 Java 对象。状态和行为被捕获在对象的字段和方法中，切入点和通知信息被捕获在 XML 中。**

**您可以使用 `<aop:aspect>` 元素声明一个切面，并使用 `ref` 属性引用支持 bean。**支持切面的 bean 当然可以像任何其他 Spring bean 一样进行配置和依赖注入。

### 声明切入点

**您可以在 `<aop:config>` 元素中声明一个命名切入点，让切入点定义在多个切面和通知者中共享。**
**使用 `<aop:pointcut>` 元素声明切入点，用 `expression` 属性声明切入点表达式。**

**请注意，切入点表达式本身使用与 AspectJ 支持 中描述的 AspectJ 切入点表达式相同的语言。如果您使用基于模式的AOP，您还可以在切入点表达式中引用 @Aspect 类型中定义的命名切入点。**

**在切面内部声明切入点与声明顶级切入点非常相似，只要把 `<aop:pointcut>` 元素放在 `<aop:aspect>`元素中即可。**

### 声明通知

**基于模式的 AOP 支持使用与 AspectJ 风格相同的五种通知，并且它们具有完全相同的语义。**

**前置通知在匹配的方法执行之前运行。它在 `<aop:aspect>` 中使用 `<aop:before>` 元素声明。**
**正如我们在对 AspectJ 风格的讨论中指出的那样，使用命名切点可以显著提高代码的可读性。**
**`method` 属性标识一个方法，该方法提供通知的主体。必须为包含通知的 aspect 元素引用的 bean 定义此方法。在执行数据访问操作（与切点表达式匹配的方法执行连接点）之前，将调用 aspect bean 上的该方法。**

**后置返回通知在匹配的方法执行正常完成后运行。它在 `<aop:aspect>` 中的声明方式与前置通知相同，使用 `<aop:after-returning>` 元素即可。**
**与 AspectJ 风格一样，你可以在通知主体中获取返回值。为此，请使用 `returning` 属性指定应将返回值传递给其的参数的名称。`method` 方法必须声明一个同名参数，此参数的类型以与 `@AfterReturning` 中描述相同的方式约束匹配。**

**后置异常通知在匹配的方法执行通过抛出异常退出时运行。它在 `<aop:aspect>` 中使用 `<aop:after-throwing>` 元素声明。**
**与 AspectJ 风格一样，你可以在通知主体中获取抛出的异常。为此，请使用 `throwing` 属性指定应将异常传递给其的参数的名称。`method` 方法必须声明一个同名参数。此参数的类型以与 `@AfterThrowing` 所述相同的方式约束匹配。**

**无论匹配的方法执行如何退出，最终通知都会运行。您可以使用 `<aop:after>` 元素声明它。**

**最后一种通知是环绕通知。环绕通知在匹配的方法执行“周围”运行。它可以在方法运行之前和之后执行工作，并确定方法何时、如何，甚至是否实际运行。**
如果您需要以线程安全的方式在方法执行前后共享状态，则使用环绕通知，例如，启动和停止计时器。

**您可以使用 `<aop:around>` 元素声明环绕通知。**
**通知方法应将 `Object` 声明为其返回类型，并且该方法的第一个参数必须为 `ProceedingJoinPoint` 类型。**
在通知方法的主体中，您必须在 `ProceedingJoinPoint` 上调用 `proceed()`，以便底层方法运行。调用没有参数的 `proceed()` 将导致调用者原始参数在调用底层方法时提供给它。对于高级用例，有一个接受参数数组 (`Object[]`) 的 `proceed()` 方法的重载变体。当调用底层方法时，数组中的值将用作底层方法的参数。

### 通知参数

**基于模式的声明样式以与 AspectJ 支持中所述相同的方式支持完全类型的通知，方法是按名称匹配切入点参数和通知方法参数。**
**如果您希望明确指定通知方法的参数名称（不依赖于前面描述的检测策略），则可以使用通知元素的 `arg-names` 属性，该属性的处理方式与通知注解中的 `argNames` 属性相同。`arg-names` 属性接受逗号分隔的参数名称列表。**

### 引入

**引入（在 AspectJ 中称为类型间声明）允许切面声明通知对象实现给定的接口，并代表这些对象提供该接口的实现。**

**您可以在 `<aop:aspect>` 中使用 `<aop:declare-parents>` 元素进行引入，声明匹配类型具有新的父级。**
例如，给定一个名为 `UsageTracked` 的接口和一个名为 `DefaultUsageTracked` 的该接口的实现，以下方面声明服务接口的所有实现者也实现 `UsageTracked` 接口。

```xml
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

	<aop:declare-parents
		types-matching="com.xyz.service.*+"
		implement-interface="com.xyz.service.tracking.UsageTracked"
		default-impl="com.xyz.service.tracking.DefaultUsageTracked"/>

	<aop:before
		pointcut="execution(* com.xyz..service.*.*(..))
			and this(usageTracked)"
			method="recordUsage"/>

</aop:aspect>
```

**要实现的接口由 `implement-interface` 属性确定。`types-matching` 属性的值是 AspectJ 类型模式。匹配类型的任何 bean 都实现 `UsageTracked` 接口。**请注意，在前一个示例的通知之前，服务 bean 可以直接用作 `UsageTracked` 接口的实现。

### 通知者

**“通知者”的概念来自 Spring 中定义的 AOP 支持，在 AspectJ 中没有直接等效项。通知者就像一个小的独立方面，只有一条通知。通知本身由 bean 表示，并且必须实现 Spring 中的通知类型中描述的通知接口之一。通知者可以使用 AspectJ 切入点表达式。**

**Spring 使用 `<aop:advisor>` 元素支持通知者概念。**您最常看到它与事务建议结合使用，它在 Spring 中也有自己的命名空间支持。

```xml
<aop:config>

	<aop:pointcut id="businessService"
		expression="execution(* com.xyz.service.*.*(..))"/>

	<aop:advisor
		pointcut-ref="businessService"
		advice-ref="tx-advice" />

</aop:config>

<tx:advice id="tx-advice">
	<tx:attributes>
		<tx:method name="*" propagation="REQUIRED"/>
	</tx:attributes>
</tx:advice>
```

**除了前一个示例中使用的 `pointcut-ref` 属性之外，您还可以使用 `pointcut` 属性内联定义切入点表达式。**

## AspectJ 的AOP支持

**AspectJ 指的是一种将方面声明为使用注解的普通 Java 类的方式。**

AspectJ 样式是由 AspectJ 项目在 AspectJ 5 版本中引入的。
Spring 解释与 AspectJ 5 相同的注解，使用 AspectJ 提供的库进行切入点解析和匹配。不过，AOP 运行时仍然是纯 Spring AOP，并且不依赖于 AspectJ 编译器或织入器。

### 启用 AspectJ 支持

**要在 Spring 配置中使用 AspectJ 切面，您需要启用 Spring 对基于 AspectJ 切面的 Spring AOP 配置的支持，以及根据这些切面是否对 bean 进行通知来自动代理 bean。**自动代理是指，如果 Spring 确定一个 bean 被一个或多个切面通知，它会自动为该 bean 生成一个代理，以拦截方法调用并确保在需要时运行通知。

**要使用 Java `@Configuration` 启用 AspectJ 支持，请添加 `@EnableAspectJAutoProxy` 注解。**
**要使用基于 XML 的配置启用 AspectJ 支持，请使用 `<aop:aspectj-autoproxy>` 元素。**

### 声明切面

**启用 AspectJ 支持后，应用程序上下文中定义的任何 bean，只要其类是具有 `@Aspect` 注解的 AspectJ 方面，都会由 Spring 自动检测，并用于配置 Spring AOP。**
方面（用 `@Aspect` 注解的类）可以有方法和字段，与任何其他类相同。它们还可以包含切入点、建议和引入（类型间声明）。

可以通过 `@Configuration` 类中的 `@Bean` 方法在 Spring XML 配置中将方面类注册为常规 bean，或让 Spring 通过类路径扫描自动检测它们，与任何其他 Spring 管理的 bean 相同。但是，请注意，`@Aspect` 注解不足以在类路径中进行自动检测。为此，您需要添加一个单独的 `@Component` 注解。

### 声明切入点

**切入点确定感兴趣的连接点，从而使我们能够控制通知的运行时间。**
**Spring AOP 仅支持 Spring Bean 的方法执行连接点，因此可以将切入点视为匹配 Spring Bean 上方法的执行。**

**切入点声明包含两部分：包含名称和任何参数的签名，以及确定我们感兴趣的具体方法执行的切入点表达式。**
**在 AOP 的 AspectJ 注释样式中，切入点签名由常规方法定义提供，切入点表达式通过使用 `@Pointcut` 注释指示。用作切入点签名的该方法必须具有 `void` 返回类型。**
**构成 `@Pointcut` 注释值的切入点表达式是一个常规 AspectJ 切入点表达式。**

#### 支持的切入点指示符

Spring AOP 支持在切入点表达式中使用以下 AspectJ 切入点指示符 ：

- `execution`：**用于匹配方法执行连接点。在使用 Spring AOP 时，这是要使用的主要切入点指示符。**
- `within`：将匹配限制在特定类型内的连接点。
- `this`：将匹配限制在连接点，其中 Bean 引用（Spring AOP 代理）是给定类型的实例。
- `target`：将匹配限制在连接点，其中目标对象（被代理的应用程序对象）是给定类型的实例。
- `args`：将匹配限制在连接点，其中参数是给定类型的实例。
- `@target`：将匹配限制在连接点，其中执行对象的类具有给定类型的注释。
- `@args`：将匹配限制在连接点，其中传递的实际参数的运行时类型具有给定类型的注释。
- `@within`：将匹配限制在具有给定注释的类型内的连接点。
- `@annotation`：将匹配限制在连接点，其中连接点的主题具有给定的注释。

完整的 AspectJ 切入点语言支持 Spring 中不支持的其他切入点指示符。在 Spring AOP 解释的切入点表达式中使用这些切入点指示符会导致抛出 `IllegalArgumentException`。

**由于 Spring AOP 仅将匹配限制为方法执行连接点，因此前面对切入点设计器的讨论给出的定义比你在 AspectJ 编程指南中找到的定义更窄。**
此外，AspectJ 本身具有基于类型的语义，并且在执行连接点中，`this` 和 `target` 都引用同一对象：执行该方法的对象。Spring AOP 是一个基于代理的系统，它区分代理对象本身（绑定到 `this`）和代理后面的目标对象（绑定到 `target`）。

Spring AOP 还支持一个名为 `bean` 的附加 PCD。此 PCD 允许你将连接点的匹配限制为特定的命名 Spring bean 或一组命名 Spring bean（使用通配符时）。
`bean( idOrNameOfBean )`
`idOrNameOfBean` 令牌可以是任何 Spring bean 的名称。提供了使用 `*` 字符的有限通配符支持，因此，如果你为 Spring bean 建立了一些命名约定，则可以编写一个 `bean` PCD 表达式来选择它们。

#### 组合切入点表达式

**你可以使用 `&&,` `||` 和 `!` 组合切入点表达式。**
最佳实践是根据较小的命名切入点构建更复杂的切入点表达式。在按名称引用切入点时，应用正常的 Java 可见性规则，可见性不影响切入点匹配。

**在使用企业应用程序时，开发人员通常需要从多个方面引用应用程序的模块和特定操作集。我们建议定义一个专门的类，用于捕获为此目的常用的命名切入点表达式。**你可以通过引用类的完全限定名称与 `@Pointcut` 方法的名称相结合，在需要切入点表达式的任何地方引用在此类中定义的切入点。

#### `execution` 表达式

`execution` 表达式的格式如下：

```
execution
	(modifiers-pattern?
	ret-type-pattern
	declaring-type-pattern?name-pattern(param-pattern)
	throws-pattern?)
```

**除了返回类型模式、名称模式和参数模式之外，所有部分都是可选的。**
**返回类型模式确定连接点匹配时方法的返回类型必须是什么。**`*`最常作为返回类型模式使用。它匹配任何返回类型。完全限定的类型名称仅在方法返回给定类型时匹配。
**名称模式匹配方法名称。**你可以将`*`通配符用作名称模式的全部或部分。如果你指定声明类型模式，请包含一个尾随`.`将其连接到名称模式组件。
**参数模式稍微复杂一些：`()`匹配不带任何参数的方法，而`(...)`匹配任意数量（零个或更多）的参数。`(*)`模式匹配采用任何类型的一个参数的方法。**

以下示例显示了一些常见的切入点表达式：

- 执行任何公共方法

  ```
  execution(public * *(..))
  ```

- 执行任何名称以`set`开头的

  ```
  execution(* set*(..))
  ```

- 执行由`AccountService`接口定义的任何方法

  ```
  execution(* com.xyz.service.AccountService.*(..))
  ```

- 执行在`service`包中定义的任何方法

  ```
  execution(* com.xyz.service.*.*(..))
  ```

- 执行在 service 包或其子包之一中定义的任何方法

  ```
  execution(* com.xyz.service..*.*(..))
  ```

- service 包中的任何连接点（在 Spring AOP 中仅执行方法）

  ```
  within(com.xyz.service.*)
  ```

- service 包或其子包之一中的任何连接点（在 Spring AOP 中仅执行方法）

  ```
  within(com.xyz.service..*)
  ```

#### 编写良好的切入点

AspectJ 只能处理它被告知的内容。**为了实现匹配的最佳性能，您应该考虑您试图实现的目标，并在定义中尽可能缩小匹配的搜索空间。**现有的指示符自然地分为三组：类型、范围和上下文：

- 类型指示符选择特定类型的连接点：`execution`
- 范围指示符选择一组感兴趣的连接点（可能是多种类型）：`within`
- 上下文指示符基于上下文进行匹配（并可选绑定）：`this`，`target`和`args`

**一个写得很好的切入点至少应包括前两种类型（类型和范围）。**

### 声明通知

**通知与切入点表达式相关联，并在切入点匹配的方法执行之前、之后或周围运行。切入点表达式可以是内联切入点，也可以是对命名切入点的引用。**

**您可以使用`@Before`注解在切面中声明前置通知。**

**后置返回通知在匹配的方法执行正常返回时运行。您可以使用`@AfterReturning`注解声明它。**
**有时，您需要在通知主体中访问实际返回的值。您可以使用`@AfterReturning`的形式，该形式将返回值绑定以获取该访问权限。在`returning`属性中使用的名称必须与通知方法中参数的名称相对应。当方法执行返回时，返回值将作为相应的参数值传递给通知方法。`returning`子句还将匹配限制为仅那些返回指定类型值的执行方法，无法返回完全不同的引用。**

**当匹配的方法执行通过抛出异常退出时，后置异常通知将运行。您可以使用`@AfterThrowing`注解来声明它。**
**通常，您希望通知仅在抛出给定类型的异常时运行，并且您也经常需要在通知主体中访问抛出的异常。您可以使用`throwing`属性来限制匹配，使用`Throwable`作为异常类型，并将抛出的异常绑定到通知参数。在`throwing`属性中使用的名称必须与通知方法中参数的名称相对应。当方法执行通过抛出异常退出时，异常将作为相应的参数值传递给通知方法。`throwing`子句还将匹配限制为仅那些抛出指定类型异常的方法执行。**

请注意，`@AfterThrowing`并不表示一般的异常处理回调。具体来说，`@AfterThrowing`通知方法只应该接收来自连接点（用户声明的目标方法）本身的异常，而不是来自伴随的`@After`/`@AfterReturning`方法的异常。

**最终通知在匹配的方法执行退出时运行。它通过使用`@After`注解来声明。After通知必须准备好处理正常和异常返回条件。它通常用于释放资源和类似目的。**
AspectJ 中的`@After`通知类似于 try-catch 语句中的 finally 块。它将针对任何结果（正常返回或从连接点抛出的异常）被调用。

**环绕通知在匹配方法的执行“周围”运行。它有机会在方法运行之前和之后执行工作，并确定方法何时、如何甚至是否真正运行。**
如果您需要以线程安全的方式在方法执行之前和之后共享状态，通常使用around通知 - 例如，启动和停止计时器。

**环绕通知通过在方法上使用 `@Around` 注解来声明。该方法应该声明 `Object` 作为其返回类型，并且该方法的第一个参数必须是 `ProceedingJoinPoint` 类型。**在通知方法的主体中，您必须在 `ProceedingJoinPoint` 上调用 `proceed()` 以便底层方法运行。调用没有参数的 `proceed()` 将导致在调用底层方法时将调用者的原始参数提供给底层方法。对于高级用例，`proceed()` 方法有一个重载变体，它接受一个参数数组 (`Object[]`)。数组中的值将在调用底层方法时用作底层方法的参数。

**环绕通知返回的值是调用者看到的返回值。**
例如，一个简单的缓存方面可以在有缓存的情况下从缓存中返回一个值，或者在没有缓存的情况下调用 `proceed()` 并返回该值。请注意，`proceed` 可以在环绕通知的主体中调用一次、多次或根本不调用。所有这些都是合法的。
如果您将环绕通知方法的返回类型声明为`void`，则始终将`null`返回给调用方，从而有效地忽略了对`proceed()`的任何调用的结果。因此，建议环绕通知方法声明`Object`的返回类型。
通知方法通常应返回从调用`proceed()`返回的值，即使底层方法具有`void`返回类型。但是，通知可以选择返回缓存的值、包装的值或其他值，具体取决于用例。

### 通知参数

**Spring 提供完全类型的通知，这意味着您在通知签名中声明所需的参数，而不是始终使用`Object[]`数组。**

**任何通知方法都可以将其第一个参数声明为`org.aspectj.lang.JoinPoint`类型的参数。**
**请注意，环绕通知需要声明`ProceedingJoinPoint`类型的第一个参数，它是`JoinPoint`的子类。**

**`JoinPoint`接口提供了一些有用的方法：**

- `getArgs()`：返回方法参数。
- `getThis()`：返回代理对象。
- `getTarget()`：返回目标对象。
- `getSignature()`：返回正在通知的方法的描述。
- `toString()`：打印正在通知的方法的有用描述。

**要将参数值提供给通知主体，可以使用`args`的绑定形式。如果在`args`表达式中使用参数名称代替类型名称，则在调用通知时，将使用相应参数的值作为参数值。**

```java
@Before( "execution( * com.xyz.dao.*.*(...) ) && args( account, ... )" )
public void validateAccount( Account account ) {
	// ...
}
```

`args(account,..)` 切入点表达式的部分有两个目的。首先，它将匹配限制为仅那些方法执行，其中方法至少接受一个参数，并且传递给该参数的参数是`Account`的实例。其次，它通过`account`参数使实际的`Account`对象可用于通知。

另一种写法是声明一个“提供”`Account` 对象值的切入点，当它匹配连接点时，然后从通知中引用命名的切入点。

```java
@Pointcut( "execution( * com.xyz.dao.*.*(...) ) && args (account, ... )" )
private void accountDataAccessOperation( Account account ) {}

@Before( "accountDataAccessOperation( account )" )
public void validateAccount( Account account ) {
	// ...
}
```

**代理对象 (`this`)、目标对象 (`target`) 和注解 (`@within`、`@target` 和 `@args`) 都可以以类似的方式绑定。**

### 通知顺序

**当多个通知都想要在同一个连接点运行时会发生什么？Spring AOP 遵循与 AspectJ 相同的优先级规则来确定通知执行的顺序。**

**优先级最高的通知首先在“进入时”运行。因此，给定两个前置通知，优先级最高的通知首先运行。**

**从连接点“退出时”，优先级最高的通知最后运行。因此，给定两个后置通知，优先级最高的通知将第二个运行。**

**当在不同方面定义的两个通知都需要在同一个连接点运行时，除非你另行指定，否则执行顺序是未定义的。你可以通过指定优先级来控制执行顺序。**在方面类中实现 `org.springframework.core.Ordered` 接口，或者使用 `@Order` 注解对其进行注解。给定两个方面，从 `Ordered.getOrder()`（或注解值）返回较低值的方面具有较高的优先级。

### 引入

**引入（在 AspectJ 中称为类型间声明）使方面能够声明被通知对象实现给定接口，并代表这些对象提供该接口的实现。**

**可以使用 `@DeclareParents` 注解进行引入。此注解用于声明匹配类型具有新的父类。**
例如，给定一个名为 `UsageTracked` 的接口和该接口的实现名为 `DefaultUsageTracked`，以下方面声明所有服务接口的实现者也实现 `UsageTracked` 接口：

```java
@Aspect
public class UsageTracking {

	@DeclareParents(value="com.xyz.service.*+", defaultImpl=DefaultUsageTracked.class)
	public static UsageTracked mixin;

	@Before("execution(* com.xyz..service.*.*(..)) && this(usageTracked)")
	public void recordUsage(UsageTracked usageTracked) {
		usageTracked.incrementUseCount();
	}

}
```

**要实现的接口由带注解字段的类型确定。`@DeclareParents` 注解的 `value` 属性是 AspectJ 类型模式。**
任何匹配类型 Bean 都实现了 `UsageTracked` 接口。请注意，在前面示例的前置通知中，服务 Bean 可以直接用作 `UsageTracked` 接口的实现。