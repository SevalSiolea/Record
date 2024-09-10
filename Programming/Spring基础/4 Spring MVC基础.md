## DispatcherServlet

**Spring MVC 与许多其他 Web 框架一样，围绕前端控制器模式设计，其中一个中央 `Servlet`（`DispatcherServlet`）提供了一个用于请求处理的共享算法，而实际工作由可配置的委托组件执行。**

**`DispatcherServlet`（与任何 `Servlet` 一样）都需要根据 Servlet 规范使用 Java 配置或在 `web.xml` 中进行声明和映射。反过来，`DispatcherServlet` 使用 Spring 配置来发现它用于请求映射、视图解析、异常处理、以及更多内容的委托组件。**

以下 Java 配置示例注册并初始化 `DispatcherServlet`，它由 Servlet 容器自动检测：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

	@Override
	public void onStartup( ServletContext servletContext ) {

		// Load Spring web application configuration
		AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
		context.register( AppConfig.class );

		// Create and register the DispatcherServlet
		DispatcherServlet servlet = new DispatcherServlet( context );
		ServletRegistration.Dynamic registration = servletContext.addServlet( "app", servlet );
		registration.setLoadOnStartup( 1 );
		registration.addMapping( "/app/*" );
	}
}
```

 除了直接使用 ServletContext API 之外，你还可以扩展 `AbstractAnnotationConfigDispatcherServletInitializer` 并覆盖特定方法。
可以使用 `GenericWebApplicationContext` 作为 `AnnotationConfigWebApplicationContext` 的替代方案。

以下 `web.xml` 配置示例注册并初始化 `DispatcherServlet`：

```xml
<web-app>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/app-context.xml</param-value>
	</context-param>

	<servlet>
		<servlet-name>app</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value></param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>app</servlet-name>
		<url-pattern>/app/*</url-pattern>
	</servlet-mapping>

</web-app>
```

Spring Boot 遵循不同的初始化顺序。Spring Boot 不会挂接到 Servlet 容器的生命周期，而是使用 Spring 配置来引导自身和嵌入式 Servlet 容器。在 Spring 配置中检测到 `Filter` 和 `Servlet` 声明，并将其注册到 Servlet 容器中。

### 上下文层次结构

**`DispatcherServlet` 期望一个 `WebApplicationContext`（一个普通 `ApplicationContext` 的扩展）用于自身的配置。`WebApplicationContext` 与 `ServletContext` 和与其关联的 `Servlet` 存在链接。**它也绑定到 `ServletContext`，以便应用程序可以使用 `RequestContextUtils` 上的静态方法查找 `WebApplicationContext`，如果它们需要访问它。

对于许多应用程序，拥有单个 `WebApplicationContext` 很简单且足够。**也可以拥有一个上下文层次结构，其中一个根 `WebApplicationContext` 在多个 `DispatcherServlet`（或其他 `Servlet`）实例之间共享，每个实例都有自己的子 `WebApplicationContext` 配置。**

根 `WebApplicationContext` 通常包含基础设施 Bean，例如数据存储库和需要在多个 `Servlet` 实例之间共享的业务服务。这些 Bean 实际上是继承的，可以在特定于 Servlet 的子 `WebApplicationContext` 中被覆盖（即重新声明），该子 `WebApplicationContext` 通常包含特定于给定 `Servlet` 的 Bean。

以下示例配置了 `WebApplicationContext` 层次结构：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[] { RootConfig.class };
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[] { App1Config.class };
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/app1/*" };
	}
}
```

如果不需要应用程序上下文层次结构，应用程序可以通过 `getRootConfigClasses()` 返回所有配置，并从 `getServletConfigClasses()` 返回 `null`。

以下示例显示了 `web.xml` 等效项：

```xml
<web-app>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/root-context.xml</param-value>
	</context-param>

	<servlet>
		<servlet-name>app1</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/app1-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>app1</servlet-name>
		<url-pattern>/app1/*</url-pattern>
	</servlet-mapping>

</web-app>
```

如果不需要应用程序上下文层次结构，应用程序可以仅配置一个“根”上下文，并将 contextConfigLocation Servlet 参数留空。

### 特殊Bean类型

**`DispatcherServlet` 将请求委托给特殊的 Bean 来处理请求并呈现相应的响应。 所谓“特殊 Bean”是指由 Spring 管理的 `Object` 实例，它们实现了框架契约。 这些 Bean 通常带有内置契约，但您可以自定义它们的属性，并扩展或替换它们。**

- `HandlerMapping`：**将请求映射到处理程序，以及用于预处理和后处理的拦截器列表。**两个主要的 `HandlerMapping` 实现是 `RequestMappingHandlerMapping`（支持 `@RequestMapping` 注解方法）和 `SimpleUrlHandlerMapping`（维护 URI 路径模式到处理程序的显式注册）。
- `HandlerAdapter`：**帮助 `DispatcherServlet` 调用映射到请求的处理程序，无论处理程序实际上是如何调用的。`HandlerAdapter` 的主要目的是屏蔽 `DispatcherServlet` 免受此类细节的影响。**
- `HandlerExceptionResolver`：**用于解析异常的策略，可能将异常映射到处理程序、HTML 错误视图或其他目标。**
- `ViewResolver`：**将处理程序返回的基于逻辑 `String` 的视图名称解析为实际的 `View`，以便将其渲染到响应中。 **

### Web MVC配置

**应用程序可以声明在特殊Bean类型中列出的处理请求所需的 基础设施 Bean。`DispatcherServlet` 检查每个特殊 Bean 的 `WebApplicationContext`。如果没有匹配的 Bean 类型，它将回退到 `DispatcherServlet.properties`中列出的默认类型。**

在大多数情况下，MVC 配置 是最佳起点。它在 Java 或 XML 中声明所需的 Bean，并提供更高级别的配置回调 API 来对其进行自定义。

Spring Boot 依赖于 MVC Java 配置来配置 Spring MVC，并提供许多额外的便捷选项。

### Servlet配置

**在 Servlet 环境中，您可以选择以编程方式配置 Servlet 容器，作为 `web.xml` 文件的替代方案或与之结合使用。**

以下示例注册了 `DispatcherServlet`：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

	@Override
	public void onStartup( ServletContext container ) {
		XmlWebApplicationContext appContext = new XmlWebApplicationContext();
		appContext.setConfigLocation( "/WEB-INF/spring/dispatcher-config.xml" );

		ServletRegistration.Dynamic registration = container.addServlet( "dispatcher", new DispatcherServlet( appContext ) );
		registration.setLoadOnStartup( 1 );
		registration.addMapping( "/" );
	}
}
```

**`WebApplicationInitializer` 是 Spring MVC 提供的一个接口，它确保您的实现被检测到并自动用于初始化任何 Servlet 3 容器。**
`WebApplicationInitializer` 的一个抽象基类实现名为 `AbstractDispatcherServletInitializer`，通过覆盖方法来指定 servlet 映射和 `DispatcherServlet` 配置的位置，使注册 `DispatcherServlet` 变得更加容易。

对于使用基于 Java 的 Spring 配置的应用程序：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return null;
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[] { MyWebConfig.class };
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}
}
```

如果您使用基于 XML 的 Spring 配置：

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

	@Override
	protected WebApplicationContext createRootApplicationContext() {
		return null;
	}

	@Override
	protected WebApplicationContext createServletApplicationContext() {
		XmlWebApplicationContext cxt = new XmlWebApplicationContext();
		cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
		return cxt;
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}
}
```

最后，如果您需要进一步自定义 `DispatcherServlet` 本身，可以覆盖 `createDispatcherServlet` 方法。

### 处理请求

**`DispatcherServlet` 处理请求的步骤如下：**

- **搜索 `WebApplicationContext` 并将其绑定到请求中，作为控制器和其他处理元素可使用的属性。**默认情况下，它绑定到 `DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE` 键下。
- **将区域解析器绑定到请求，以便处理过程中的元素可以解析区域。**如果您不需要区域解析，则不需要区域解析器。
- **将主题解析器绑定到请求，以便确定要使用的主题。**如果您不使用主题，可以忽略它。
- **如果您指定了多部分文件解析器，则会检查请求中是否存在多部分内容。**如果找到多部分内容，则会将请求包装在 `MultipartHttpServletRequest` 中，以便处理过程中的其他元素进行进一步处理。
- **搜索合适的处理器。如果找到处理器，则会运行与处理器关联的执行链（预处理器、后处理器和控制器）以准备用于渲染的模型。或者，对于带注解的控制器，可以在 `HandlerAdapter` 中渲染响应，而不是返回视图。**
- **如果返回了模型，则渲染视图。如果没有返回模型（可能是由于预处理器或后处理器拦截了请求，可能是出于安全原因），则不会渲染视图，因为请求可能已经完成。**

**在 `WebApplicationContext` 中声明的 `HandlerExceptionResolver` bean 用于解析请求处理期间抛出的异常。这些异常解析器允许自定义逻辑来处理异常。**

为了支持 HTTP 缓存，处理程序可以使用 `WebRequest` 的 `checkNotModified` 方法。

**您可以通过在 `web.xml` 文件中的 Servlet 声明中添加 Servlet 初始化参数（`init-param` 元素）来自定义单个 `DispatcherServlet` 实例。**

- `contextClass`：实现 `ConfigurableWebApplicationContext` 的类，由此 Servlet 实例化和本地配置。默认情况下，使用 `XmlWebApplicationContext`。
- `contextConfigLocation`：传递给上下文实例（由 `contextClass` 指定）的字符串，以指示可以在何处找到上下文。该字符串可能包含多个字符串（使用逗号作为分隔符）以支持多个上下文。
- `namespace`：`WebApplicationContext` 的命名空间。默认为 `[servlet-name]-servlet`。

### 路径匹配

**Servlet API 将完整请求路径显示为 `requestURI`，并进一步将其细分为 `contextPath`、`servletPath` 和 `pathInfo`，其值因 Servlet 的映射方式而异。**
**Spring MVC 需要根据这些输入确定用于映射处理程序的查找路径。**该路径应排除 `contextPath` 和任何 `servletMapping` 前缀（如果适用）。

对 `servletPath` 和 `pathInfo` 进行解码，这使得它们无法直接与完整的 `requestURI` 进行比较，以便派生出 `lookupPath`，并且必须对 `requestURI` 进行解码。但是，这会引入自身的问题，因为路径可能包含编码的保留字符，例如 `"/"` 或 `";"`，这些字符在解码后又会改变路径的结构，这也可能导致安全问题。此外，Servlet 容器可能会在不同程度上对 `servletPath` 进行规范化，这使得进一步无法对 `requestURI` 执行 `startsWith` 比较。

这就是最好避免依赖于带有基于前缀的 `servletPath` 映射类型的 `servletPath` 的原因。如果 `DispatcherServlet` 被映射为带有 `"/"` 的默认 Servlet，或者在没有前缀的情况下以 `"/*"` 映射，并且 Servlet 容器为 4.0+，那么 Spring MVC 能够检测到 Servlet 映射类型，并避免使用 `servletPath` 和 `pathInfo`。在 3.1 Servlet 容器上，假设使用相同的 Servlet 映射类型，可以通过在 MVC 配置中的路径匹配中提供带有 `alwaysUseFullPath=true` 的 `UrlPathHelper` 来实现等效功能。

幸运的是，默认 Servlet 映射 `"/"` 是一个不错的选择。但是，仍然存在一个问题，即需要对 `requestURI` 进行解码，以便能够与控制器映射进行比较。由于可能会解码改变路径结构的保留字符，因此这再次变得不可取。如果不需要此类字符，则可以拒绝它们（例如 Spring Security HTTP 防火墙），或者可以使用 `urlDecode=false` 配置 `UrlPathHelper`，但控制器映射需要与编码路径匹配，这可能并不总是有效。此外，有时 `DispatcherServlet` 需要与另一个 Servlet 共享 URL 空间，并且可能需要按前缀进行映射。

**当使用 `PathPatternParser` 和已解析模式作为使用 `AntPathMatcher` 进行字符串路径匹配的替代方案时，上述问题得到解决。`PathPatternParser` 从 Spring MVC 5.3 版本开始可以使用，并且从 6.0 版本开始默认启用。**与需要对查找路径解码或对控制器映射进行编码的 `AntPathMatcher` 不同，已解析的 `PathPattern` 与路径的已解析表示（称为 `RequestPath`）匹配，一次一个路径段。这允许单独解码和清理路径段值，而无需改变路径结构的风险。已解析的 `PathPattern` 还支持使用 `servletPath` 前缀映射，只要使用 Servlet 路径映射并且前缀保持简单（即没有编码字符）。

### 拦截

**所有 `HandlerMapping` 实现都支持处理程序拦截，这在您希望跨请求应用功能时很有用。`HandlerInterceptor` 可以实现以下内容：**

- `preHandle(..)`：**在实际处理程序运行之前调用的回调函数，返回布尔值。如果方法返回 `true`，则继续执行；如果返回 `false`，则绕过执行链的其余部分，并且不会调用处理程序。**
- `postHandle(..)`：**在处理程序运行后调用的回调函数。**
- `afterCompletion(..)`：**在完成请求后调用的回调函数。**

拦截器不适合作为安全层，因为它们可能与带注解的控制器路径匹配不匹配。通常，我们建议使用 Spring Security，或者使用类似的方法与 Servlet 过滤器链集成，并在尽可能早的时间应用。

### 异常

**如果在请求映射期间发生异常，或者从请求处理程序抛出异常，`DispatcherServlet` 会委托给一连串的 `HandlerExceptionResolver` bean 来解析异常并提供替代处理，通常是错误响应。**

**下表列出了可用的 `HandlerExceptionResolver` 实现：**

- `SimpleMappingExceptionResolver`：**异常类名和错误视图名之间的映射。对于在浏览器应用程序中渲染错误页面很有用。**
- `DefaultHandlerExceptionResolver`：**解析由 Spring MVC 抛出的异常，并将它们映射到 HTTP 状态代码。**
- `ResponseStatusExceptionResolver`：**解析带有`@ResponseStatus`注解的异常，并根据注解中的值将其映射到 HTTP 状态码。**
- `ExceptionHandlerExceptionResolver`：**通过调用`@Controller`或`@ControllerAdvice`类中的`@ExceptionHandler`方法来解析异常。**

**可以通过在 Spring 配置中声明多个`HandlerExceptionResolver` Bean 并根据需要设置它们的`order`属性来形成异常解析器链。`order`属性值越高，异常解析器的位置越靠后。**

**`HandlerExceptionResolver`的契约规定它可以返回：**

- **指向错误视图的`ModelAndView`。**
- **如果异常在解析器中得到处理，则返回一个空的`ModelAndView`。**
- **如果异常仍然未解决，则返回`null`，以便后续解析器尝试解决，如果异常在最后仍然存在，则允许其冒泡到 Servlet 容器。**

MVC 配置会自动声明用于默认 Spring MVC 异常、带有`@ResponseStatus`注解的异常以及支持`@ExceptionHandler`方法的内置解析器。可以自定义该列表或替换它。

**如果任何`HandlerExceptionResolver`都无法解决异常，因此该异常被传播，或者响应状态被设置为错误状态，则 Servlet 容器可以使用 HTML 渲染默认错误页面。**
**要自定义容器的默认错误页面，可以在`web.xml`中声明错误页面映射。**

```xml
<error-page>
	<location>/error</location>
</error-page>
```

对于上述示例，当异常冒泡或响应具有错误状态时，Servlet 容器将在容器内执行 ERROR 分派到配置的 URL（例如，`/error`）。然后，`DispatcherServlet`会处理它。

Servlet API 不提供在 Java 中创建错误页面映射的方法。但是，可以使用 `WebApplicationInitializer` 和最小的`web.xml`。

### 视图解析

**Spring MVC 定义了 `ViewResolver` 和 `View` 接口，它们允许你在浏览器中渲染模型，而无需绑定到特定的视图技术。 `ViewResolver` 提供视图名称和实际视图之间的映射。 `View` 在移交给特定视图技术之前处理数据准备。**

**下表提供了有关 `ViewResolver` 层次结构的更多详细信息：**

- `AbstractCachingViewResolver`：**`AbstractCachingViewResolver` 的子类缓存他们解析的视图实例。缓存提高了某些视图技术的性能。**
- `UrlBaseViewResolver`：**`ViewResolver` 接口的简单实现，它将逻辑视图名称直接解析为 URL，而无需显式映射定义。**如果你的逻辑名称以简单的方式匹配你的视图资源的名称，而不需要任意映射，那么这是合适的。
- `InternalResourceViewResolver`：**`UrlBasedViewResolver` 的便捷子类。**它支持 `InternalResourceView`（实际上是 Servlet 和 JSP）以及 `JstlView` 等子类。你可以使用 `setViewClass(..)` 为此解析器生成的所有视图指定视图类。
- `BeanNameViewResolver`：**`ViewResolver` 接口的实现，它将视图名称解释为当前应用程序上下文中 bean 的名称。**这是一个非常灵活的变体，它允许基于不同的视图名称混合和匹配不同的视图类型。每个这样的 `View` 都可以定义为一个 bean，例如在 XML 或配置类中。

**你可以通过声明多个解析器 bean 来链接视图解析器，并在必要时通过设置 `order` 属性来指定顺序。请记住，order 属性越高，视图解析器在链中的位置就越靠后。**

**`ViewResolver` 的契约规定，它可以返回 null 来指示找不到视图。**但是，对于 JSP 和 `InternalResourceViewResolver`，找出 JSP 是否存在的方法是通过 `RequestDispatcher` 执行分派。**因此，你必须始终将 `InternalResourceViewResolver` 配置为视图解析器的总体顺序中的最后一个。**

## 带注解的控制器

**Spring MVC 提供了一个基于注解的编程模型，其中 `@Controller` 和 `@RestController` 组件使用注解来表示请求映射、请求输入、异常处理等。带注解的控制器具有灵活的方法签名，不必扩展基类或实现特定接口。**

### 声明

**您可以使用 Servlet 的 `WebApplicationContext` 中的标准 Spring bean 定义来定义控制器 bean。`@Controller` 构造型允许自动检测，与 Spring 在类路径中检测 `@Component` 类并为其自动注册 bean 定义的一般支持保持一致。它也充当带注解类的构造型，指示其作为 Web 组件的角色。**

要启用对这些 `@Controller` bean 的自动检测，您可以将组件扫描添加到您的 Java 配置中：

```java
@Configuration
@ComponentScan( "org.example.web" )
public class WebConfig {
	// ...
}
```

以下示例显示了前面示例的 XML 配置等效项：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		https://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="org.example.web"/>

	<!-- ... -->

</beans>
```

**`@RestController` 是一个组合注解，它本身用 `@Controller` 和 `@ResponseBody` 进行元注解，以指示一个控制器，其每个方法都继承了类型级别的 `@ResponseBody` 注解，因此直接写入响应主体，而不是使用 HTML 模板进行视图解析和渲染。**

在某些情况下，您可能需要在运行时用 AOP 代理装饰控制器。在这种情况下，特别是对于控制器，我们建议使用基于类的代理。对于控制器上直接使用此类注解的情况，这将自动生效。如果控制器实现了接口，并且需要 AOP 代理，您可能需要显式配置基于类的代理。

请注意，从 6.0 版本开始，使用接口代理，Spring MVC 不再仅根据接口上的类型级 `@RequestMapping` 注解来检测控制器。请启用基于类的代理，否则接口也必须具有 `@Controller` 注解。

### 映射请求

**您可以使用 `@RequestMapping` 注解将请求映射到控制器方法。它具有各种属性，可以按 URL、HTTP 方法、请求参数、标头和媒体类型进行匹配。您可以在类级别使用它来表达共享映射，或者在方法级别使用它来缩小到特定的端点映射。**

**还有 `@RequestMapping` 的 HTTP 方法特定快捷方式变体：`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping`，`@PatchMapping`。**这些快捷方式是自定义注解，它们被提供是因为，大多数控制器方法应该映射到一个特定的 HTTP 方法，而不是使用 `@RequestMapping`，它默认情况下匹配所有 HTTP 方法。`@RequestMapping` 仍然需要在类级别表达共享映射。

**`@RequestMapping` 不能与在同一元素（类、接口或方法）上声明的其他 `@RequestMapping` 注解一起使用。**如果在同一元素上检测到多个 `@RequestMapping` 注解，将记录警告，并且只使用第一个映射。这也适用于组合的 `@RequestMapping` 注解，例如 `@GetMapping`、`@PostMapping` 等。

### URI模式

**`@RequestMapping` 方法可以使用 URL 模式进行映射。**
**`PathPattern` 是一个预解析的模式，与作为 `PathContainer` 预解析的 URL 路径匹配。此解决方案专为 Web 使用而设计，可以有效地处理编码和路径参数，并高效地匹配。**
`AntPathMatcher` 将字符串模式与字符串路径匹配。这是原始解决方案，也用于 Spring 配置中选择类路径、文件系统和其他位置的资源。它效率较低，字符串路径输入对于有效处理编码和其他 URL 问题是一个挑战。
**`PathPattern` 是 Web 应用程序的推荐解决方案。**它从版本 5.3 开始启用用于 Spring MVC，并且从版本 6.0 开始默认启用。

**`PathPattern` 支持与 `AntPathMatcher` 相同的模式语法。**
**此外，它还支持捕获模式，例如 `{*spring}`，用于匹配路径末尾的 0 个或多个路径段。`PathPattern` 还限制了 `**` 用于匹配多个路径段的使用，使其仅允许在模式末尾使用。这消除了在为给定请求选择最佳匹配模式时出现歧义的许多情况。**

**语法 `{varName:regex}` 声明一个 URI 变量，该变量具有正则表达式。**
**您可以在类和方法级别声明 URI 变量，捕获的 URI 变量可以使用 `@PathVariable` 访问。**
URI 变量会自动转换为适当的类型，或者引发 `TypeMismatchException`。默认情况下支持简单类型，您可以注册对任何其他数据类型的支持。

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

	@GetMapping("/pets/{petId}")
	public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
		// ...
	}
}
```

**您可以显式命名 URI 变量。例如，`@PathVariable("customId")`**。但如果名称相同并且您的代码使用 `-parameters` 编译器标志编译，则可以省略此细节。

URI 路径模式还可以包含嵌入的 `${…}` 占位符，这些占位符在启动时使用 `PropertySourcesPlaceholderConfigurer` 相对于本地、系统、环境和其他属性源解析。例如，您可以使用它根据某些外部配置参数化基本 URL。

**当多个模式匹配 URL 时，必须选择最佳匹配。这取决于是否启用解析的 `PathPattern` 用于使用。**使用 `PathPattern.SPECIFICITY_COMPARATOR` 或 `AntPathMatcher.getPatternComparator( String path )` 完成。两者都有助于对模式进行排序，使更具体的模式位于顶部。
如果模式具有较少的 URI 变量（计为 1）、单个通配符（计为 1）和双通配符（计为 2），则该模式更具体。如果得分相同，则选择更长的模式。如果得分和长度相同，则选择具有比通配符更多的 URI 变量的模式。
默认映射模式 (`/**`) 不计入评分，并且始终排在最后。此外，前缀模式（例如 `/public/**`）被认为比没有双通配符的其他模式更不具体。

### 模型

**您可以使用 `@ModelAttribute` 注解：**

- **在 `@RequestMapping` 方法中的方法参数上，用于从模型中创建或访问 `Object`，并通过 `WebDataBinder` 将其绑定到请求。**
- **作为 `@Controller` 或 `@ControllerAdvice` 类中的方法级注解，用于在任何 `@RequestMapping` 方法调用之前帮助初始化模型。**
- **在 `@RequestMapping` 方法上，用于标记其返回值是模型属性。**

控制器可以包含任意数量的`@ModelAttribute` 方法。所有此类方法都在同一个控制器中的`@RequestMapping` 方法之前调用。`@ModelAttribute` 方法也可以通过`@ControllerAdvice` 在控制器之间共享。

`@ModelAttribute` 方法具有灵活的方法签名。它们支持与`@RequestMapping` 方法相同的许多参数，除了`@ModelAttribute` 本身或与请求主体相关的任何内容。

当未显式指定名称时，将根据`Object` 类型选择默认名称。您可以始终使用重载的`addAttribute` 方法或通过`@ModelAttribute` 上的`name` 属性（用于返回值）来分配显式名称。

**您也可以在`@RequestMapping` 方法上使用`@ModelAttribute` 作为方法级注释，在这种情况下，`@RequestMapping` 方法的返回值将被解释为模型属性。**这通常不需要，因为它是 HTML 控制器中的默认行为，除非返回值是`String`，否则将被解释为视图名称。`@ModelAttribute` 也可以自定义模型属性名称。

### 异常

`@Controller` 和 `@ControllerAdvice` 类可以具有 `@ExceptionHandler` 方法来处理控制器方法中的异常。

异常可能与正在传播的顶级异常匹配（例如，直接抛出的 `IOException`），或者与包装异常内的嵌套原因匹配（例如，包装在 `IllegalStateException` 中的 `IOException`）。从 5.3 开始，这可以在任意原因级别匹配，而以前只考虑直接原因。

**对于匹配的异常类型，最好将目标异常声明为方法参数。**

**当多个异常方法匹配时，通常优先选择根异常匹配而不是原因异常匹配。更具体地说，`ExceptionDepthComparator` 用于根据异常类型从抛出异常类型的深度对异常进行排序。**
**注释声明可以缩小要匹配的异常类型。您甚至可以使用特定异常类型的列表以及非常通用的参数签名。**

在 `IOException` 变体中，该方法通常使用实际的 `FileSystemException` 或 `RemoteException` 实例作为参数调用，因为它们都扩展自 `IOException`。但是，如果任何此类匹配的异常在本身为 `IOException` 的包装异常中传播，则传入的异常实例就是该包装异常。

**我们通常建议您在参数签名中尽可能具体，以减少根异常类型和原因异常类型之间不匹配的可能性。考虑将多匹配方法分解为单独的 `@ExceptionHandler` 方法，每个方法通过其签名匹配单个特定异常类型。**

在多 `@ControllerAdvice` 中，我们建议在优先级较高的 `@ControllerAdvice` 上声明您的主要根异常映射，并使用相应的顺序。虽然根异常匹配优先于原因匹配，但这是在给定控制器或 `@ControllerAdvice` 类的方法中定义的。这意味着优先级较高的 `@ControllerAdvice` bean 上的原因匹配优先于优先级较低的 `@ControllerAdvice` bean 上的任何匹配，包括根匹配。

**最后但并非最不重要的一点，`@ExceptionHandler` 方法实现可以选择通过以原始形式重新抛出异常来退出处理给定异常实例。这在您只对根级别匹配或特定上下文中无法静态确定的匹配感兴趣的情况下很有用。重新抛出的异常将通过剩余的解析链传播，就好像给定的 `@ExceptionHandler` 方法最初没有匹配一样。**

**`@ExceptionHandler` 方法支持以下参数：**

- `HadlerMethod`： 用于访问引发异常的控制器方法。
- `WebRequest`：无需直接使用 Servlet API 即可访问请求参数以及请求和会话属性。
- `HttpMethod`：请求的 HTTP 方法。
- `@SessionAttribute`：要访问任何会话属性。
- `@RequestAttribute`：要访问任何请求属性。

`@ExceptionHandler`方法支持以下返回值：

- `ResponseBody`：返回值通过`HttpMessageConverter`实例进行转换，并写入响应。
- `HttpEntity<B>`，`ResponseEntity<B>`：返回值指定完整响应（包括 HTTP 标头和正文）通过`HttpMessageConverter`实例进行转换，并写入响应。
- `ErrorResponse`：呈现带有正文中详细信息的 RFC 9457 错误响应。
- `String`：要与`ViewResolver`实现一起解析并与隐式模型一起使用的视图名称——通过命令对象和`@ModelAttribute`方法确定。处理程序方法还可以通过声明`Model`参数（如前所述）以编程方式丰富模型。
- `View`：要与隐式模型一起用于渲染的`View`实例——通过命令对象和`@ModelAttribute`方法确定。处理程序方法还可以通过声明`Model`参数（如前所述）以编程方式丰富模型。
- `@ModelAttribute`：要添加到模型中的属性，视图名称通过`RequestToViewNameTranslator`隐式确定。
- `ModelAndView`对象：要使用的视图和模型属性，以及可选的响应状态。
- `void`：如果方法具有`void`返回类型（或`null`返回值），并且还具有`ServletResponse`和`OutputStream`参数，或者具有`@ResponseStatus`注释，则该方法被认为已完全处理响应。
- 其它任何返回值：如果返回值与上述任何返回值都不匹配，并且不是简单类型（由 `BeanUtils#isSimpleProperty` 确定，默认情况下，它将被视为要添加到模型中的模型属性。如果它是简单类型，则它将保持未解析状态。

## 处理程序方法

### `@RequestParam`

**可以使用 `@RequestParam` 注解将 Servlet 请求参数（即查询参数或表单数据）绑定到控制器中的方法参数。**

**默认情况下，使用此注解的方法参数是必需的。**但您可以通过将 `@RequestParam` 注解的 `required` 标志设置为 `false` 或通过使用 `java.util.Optional` 包装器声明参数来指定方法参数是可选的。

**如果目标方法参数类型不是 `String`，则会自动应用类型转换。**

**将参数类型声明为数组或列表允许为同一个参数名称解析多个参数值。**
当 `@RequestParam` 注解被声明为 `Map<String, String>` 或 `MultiValueMap<String, String>` 时，如果没有在注解中指定参数名称，则该映射将使用每个给定参数名称的请求参数值填充。

`@RequestParam` 是可选的。默认情况下，任何简单值类型（由 `BeanUtils#isSimpleProperty` 确定）且未被任何其他参数解析器解析的参数，都被视为已使用 `@RequestParam` 注解。

### `@RequestHeader`

**可以使用 `@RequestHeader` 注解将请求头绑定到控制器中的方法参数。**

考虑以下请求，其中包含标头：

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

以下示例获取 `Accept-Encoding` 和 `Keep-Alive` 标头值：

```java
@GetMapping("/demo")
public void handle(
		@RequestHeader("Accept-Encoding") String encoding,
		@RequestHeader("Keep-Alive") long keepAlive) {
	//...
}
```

**如果目标方法参数类型不是 `String`，则会自动应用类型转换。**

当 `@RequestHeader` 注解用于 `Map<String, String>`、`MultiValueMap<String, String>` 或 `HttpHeaders` 参数时，该映射将填充所有标头值。

内置支持可用于将逗号分隔字符串转换为数组或字符串集合或类型转换系统已知的其他类型。例如，使用 `@RequestHeader("Accept")` 注释的方法参数可以是 `String` 类型，也可以是 `String[]` 或 `List<String>` 类型。

### `@CookieValue`

**您可以使用 `@CookieValue` 注解将 HTTP cookie 的值绑定到控制器中的方法参数。**

**如果目标方法参数类型不是 `String`，则会自动应用类型转换。**

### `@ModelAttribute`

**`@ModelAttribute` 方法参数注解将请求参数绑定到模型对象上。**

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) {
	// method logic...
}
```

`Pet` 实例可以是：

- 从模型中访问，它可能由 `@ModelAttribute` 方法添加。
- 从 HTTP 会话中访问，如果模型属性在类级别的 `@SessionAttributes` 注解中列出。
- 通过默认构造函数实例化。
- 通过“主构造函数”实例化，该构造函数的参数与 Servlet 请求参数匹配。参数名称通过字节码中运行时保留的参数名称确定。

### `@RequestBody`

**您可以使用 `@RequestBody` 注解将请求主体读取并反序列化为一个 `Object`，通过一个 `HttpMessageConverter`。**

您可以将 `@RequestBody` 与 `jakarta.validation.Valid` 或 Spring 的 `@Validated` 注解结合使用，这两个注解都会导致应用标准 Bean 验证。

**`HttpEntity` 与使用 `@RequestBody`几乎相同，但基于一个暴露请求头和主体容器对象。**

### `@ResponseBody`

**您可以在方法上使用 `@ResponseBody` 注解，使返回值通过 `HttpMessageConverter` 序列化到响应主体。**

**`@ResponseBody` 也支持在类级别使用，在这种情况下，它会被所有控制器方法继承。**
**这就是 `@RestController` 的效果，它只不过是一个用 `@Controller` 和 `@ResponseBody` 标记的元注解。**

**`ResponseEntity` 类似于 `@ResponseBody`，但具有状态和标头。**主体通常将作为值对象提供，由其中一个已注册的 `HttpMessageConverters` 呈现为对应的响应表示。