## Spring框架概述

**Spring 使得创建 Java 企业应用程序变得容易。**Spring 支持 Groovy 和 Kotlin 作为 JVM 上的替代语言。从 Spring Framework 6.0 开始，Spring 需要 Java 17+。
**Spring 支持各种应用程序场景。**
**Spring 是开源的。它拥有一个庞大而活跃的社区。**

#### 我们所说的“Spring”

术语“Spring”在不同的语境下有不同的含义。**它可以用来指代 Spring Framework 项目本身，这是它的起源。大多数情况下，当人们说“Spring”时，指的是整个项目家族。**

<u>Spring Framework 被划分为多个模块。应用程序可以选择它们需要的模块。</u>**核心是核心容器的模块，包括配置模型和依赖注入机制。**除此之外，Spring Framework 为不同的应用程序架构提供了基础支持，包括消息传递、事务性数据和持久性以及 Web。**它还包括基于 Servlet 的 Spring MVC Web 框架。**

####  Spring 和 Spring Framework 的历史

Spring 于 2003 年诞生，旨在应对早期 J2EE 规范的复杂性。
除了 Spring 框架之外，还有其他项目，例如 Spring Boot、Spring Security、Spring Data、Spring Cloud、Spring Batch 等。重要的是要记住，每个项目都有自己的源代码仓库、问题跟踪器和发布节奏。

#### 设计理念

**在每个级别提供选择。Spring 允许您尽可能晚地推迟设计决策。**例如，您可以通过配置切换持久化提供程序，而无需更改代码。对于许多其他基础设施问题和与第三方 API 的集成，也是如此。
**适应不同的视角。**Spring 拥抱灵活性，并且对事情应该如何完成没有意见。它支持各种应用程序需求，并具有不同的视角。
**保持强大的向后兼容性。**Spring 的演变经过精心管理，以强制在版本之间进行很少的重大更改。Spring 支持精心选择的 JDK 版本和第三方库范围，以方便维护依赖于 Spring 的应用程序和库。
**关心 API 设计。**
**为代码质量设定高标准。**Spring 框架非常重视有意义、最新和准确的 javadoc。它是为数不多的几个项目之一，可以声称代码结构清晰，包之间没有循环依赖关系。

## Spring Boot概述

**Spring Boot 帮助您创建可运行的独立的、生产级的基于 Spring 的应用程序。**大多数 Spring Boot 应用程序只需要很少的 Spring 配置。
您可以使用 Spring Boot 创建 Java 应用程序，这些应用程序可以使用 `java -jar` 或更传统的 war 部署启动。

**我们的主要目标是**
为所有 Spring 开发提供一种彻底快速且易于访问的入门体验。
开箱即用，但当需求开始偏离默认值时，应迅速退出。
提供一系列适用于大型项目类别（例如嵌入式服务器、安全、指标、健康检查和外部化配置）的非功能特性。
绝对不生成代码（除非针对原生镜像），也不需要 XML 配置。

## 安装 Spring Boot

**Spring Boot 可以与“经典”Java 开发工具一起使用，也可以作为命令行工具安装。**
您可以像使用任何标准 Java 库一样使用 Spring Boot。为此，请在您的类路径中包含相应的 `spring-boot-*.jar` 文件。
Spring Boot 不需要任何特殊的工具集成，因此您可以使用任何 IDE 或文本编辑器。
此外，Spring Boot 应用程序没有任何特殊之处，因此您可以像运行和调试任何其他 Java 程序一样运行和调试 Spring Boot 应用程序。

**虽然您可以复制 Spring Boot jar 文件，但我们通常建议您使用支持依赖项管理的构建工具（例如 Maven 或 Gradle）。**
在许多操作系统上，可以使用包管理器安装 Maven。
Spring Boot 依赖项使用 `org.springframework.boot` 组 ID。
Spring Boot 依赖项使用 `org.springframework.boot` `group` 声明。

**Spring Boot CLI（命令行界面）是一个命令行工具，您可以使用它快速使用 Spring 进行原型设计。**您不需要使用 CLI 来使用 Spring Boot，但它是在没有 IDE 的情况下快速启动 Spring 应用程序的一种方法。

## 开发您的第一个 Spring Boot 应用程序

#### `@RestController` 和 `@RequestMapping` 注解

**我们 `MyApplication` 类上的第一个注解是 `@RestController`。这被称为*立体*注解。它为阅读代码的人和 Spring 提供了关于该类扮演特定角色的提示。在本例中，我们的类是一个 web `@Controller`，因此 Spring 在处理传入的 web 请求时会考虑它。**

**`@RequestMapping` 注解提供了“路由”信息。它告诉 Spring 任何具有`/`路径的 HTTP 请求都应该映射到`home`方法。**
**`@RestController` 注解告诉 Spring 将结果字符串直接渲染回调用者。**

`@RestController` 和 `@RequestMapping` 注解是 Spring MVC 注解（它们不是特定于 Spring Boot 的）。

#### `@SpringBootApplication` 注解

**第二个类级注解是 `@SpringBootApplication`。此注解被称为*元注解*，它组合了 `@SpringBootConfiguration`、`@EnableAutoConfiguration` 和 `@ComponentScan`。**

**其中，我们最感兴趣的注解是 `@EnableAutoConfiguration`。`@EnableAutoConfiguration` 告诉 Spring Boot 根据你添加的 jar 依赖项来“猜测”你希望如何配置 Spring。由于 `spring-boot-starter-web` 添加了 Tomcat 和 Spring MVC，因此自动配置假设你正在开发一个 web 应用程序，并相应地设置 Spring。**

自动配置旨在与“启动器”配合使用，但这两个概念并不直接相关。您可以自由选择启动器之外的 jar 依赖项。Spring Boot 仍然尽力自动配置您的应用程序。

#### `main` 方法

我们应用程序的最后部分是 `main` 方法。这是一个标准方法，遵循 Java 应用程序入口点的约定。**我们的 main 方法通过调用 `run` 来委托给 Spring Boot 的 `SpringApplication` 类。`SpringApplication` 启动我们的应用程序，启动 Spring，进而启动自动配置的 Tomcat Web 服务器。**我们需要将 `MyApplication.class` 作为参数传递给 `run` 方法，以告诉 `SpringApplication` 哪个是主要的 Spring 组件。`args` 数组也被传递进来，以公开任何命令行参数。