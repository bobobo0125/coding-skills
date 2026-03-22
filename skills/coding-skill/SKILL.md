---
name: java-coding-standards
description: "Java编码规范：基于阿里Java开发规范，包含MVC和DDD两种架构的开发模板。适用于Spring Boot服务的命名、不可变性、Optional使用、Stream、异常、泛型和项目结构。"
---

# Java编码规范

适用于Spring Boot服务中可读、可维护的Java (17+)代码规范。

> **规范参考**：阿里巴巴Java开发规范（Alibaba Java Coding Guidelines）

## 架构选择

根据项目架构选择对应的实现模板：

| 架构 | 适用场景 | 模板文件 |
|------|---------|---------|
| **MVC架构** | 业务逻辑相对简单的小型项目、快速开发 | MVC开发模板.md |
| **DDD架构** | 业务复杂的中大型项目、需要领域建模 | DDD开发模板.md |

## 何时启用

- 在Spring Boot项目中编写或审查Java代码
- 强制执行命名、不可变性或异常处理规范
- 使用record、sealed类或模式匹配（Java 17+）
- 审查Optional、Stream或泛型的使用
- 构建包和项目结构
- 开发新功能时参考实现模板

---

## 阿里Java开发规范要点

### 命名规范

**【强制】类名使用UpperCamelCase风格，以下情形例外：DO/BO/DTO/VO/AO/PO等**
```java
// ✅ Good
public class UserDO {}
public class MarketDTO {}
public class OrderVO {}

// ❌ Bad
public class user {}
public class marketDto {}
```

**【强制】方法名、参数名、成员变量、局部变量使用lowerCamelCase风格**
```java
// ✅ Good
private MarketRepository marketRepository;
public Market findBySlug(String slug) {}

// ❌ Bad
private MarketRepository MarketRepository;
public Market FindBySlug(String slug) {}
```

**【强制】常量命名使用UPPER_SNAKE_CASE风格**
```java
// ✅ Good
private static final int MAX_PAGE_SIZE = 100;

// ❌ Bad
private static final int maxPageSize = 100;
```

### 代码格式

**【强制】缩进使用4个空格，禁止使用Tab字符**
```java
// ✅ Good
if (condition) {
    doSomething();
}

// ❌ Bad
if (condition) {
doSomething();
}
```

**【强制】左大括号前不换行，右大括号后换行**
```java
// ✅ Good
public class Test {
    public void method() {
        if (condition) {
            doSomething();
        }
    }
}
```

### 注释规范

**【强制】类、类成员、方法必须添加JavaDoc注释**
```java
/**
 * 市场服务类
 * 负责市场的CRUD操作
 *
 * @author developer
 * @since 1.0.0
 */
public class MarketService {}

/**
 * 根据slug查询市场
 *
 * @param slug 市场slug
 * @return 市场信息
 */
public Optional<Market> findBySlug(String slug) {}
```

**【强制】方法内部单行注释，在被注释语句上方另起一行**
```java
// ✅ Good
if (condition) {
    // 处理业务逻辑
    doSomething();
}

// ❌ Bad
if (condition) { // 处理业务逻辑
    doSomething();
}
```

### 控制语句

**【强制】在if/else/for/while/do等语句中必须使用大括号**
```java
// ✅ Good
if (condition) {
    return result;
}

// ❌ Bad
if (condition)
    return result;
```

**【强制】禁止在条件判断中执行复杂的方法或表达式**
```java
// ✅ Good
boolean isValid = validateInput(param);
if (isValid) {
    process();
}

// ❌ Bad
if (validateInput(param) && process() && otherMethod()) {
    // 复杂判断难以阅读
}
```

### 异常处理

**【强制】异常捕获后必须进行处理，不能生吞异常**
```java
// ✅ Good
try {
    doSomething();
} catch (Exception e) {
    log.error("操作失败", e);
    throw new BusinessException("操作失败");
}

// ❌ Bad
try {
    doSomething();
} catch (Exception e) {
    // 生吞异常，隐藏问题
}
```

**【强制】使用非受检异常（RuntimeException）处理业务异常**
```java
// ✅ Good
throw new BusinessException("业务错误");

// 避免使用受检异常，除非必须
```

### 日志规范

**【强制】日志必须使用占位符方式，禁止字符串拼接**
```java
// ✅ Good
log.info("查询市场 slug={}", slug);
log.error("查询失败 slug={}, error={}", slug, e.getMessage(), e);

// ❌ Bad
log.info("查询市场 " + slug);
log.error("查询失败 " + e.getMessage());
```

**【强制】日志级别使用规范**
- ERROR：影响功能的异常
- WARN：不影响功能但需要关注的问题
- INFO：关键业务流程
- DEBUG：调试信息

### 并发处理

**【强制】创建线程或线程池时必须指定有意义的名称**
```java
// ✅ Good
ThreadFactory threadFactory = new ThreadFactoryBuilder()
    .setNamePrefix("market-pool-")
    .build();

// ❌ Bad
ExecutorService executor = Executors.newFixedThreadPool(10);
```

**【强制】多线程环境下，共享资源必须进行同步控制**

---

## 核心原则

- 清晰优于技巧
- 默认不可变；最小化共享可变状态
- 快速失败并抛出有意义的异常
- 一致的命名和包结构

### 命名规范示例

```java
// ✅ 类/Record：PascalCase
public class MarketService {}
public record Money(BigDecimal amount, Currency currency) {}

// ✅ 方法/字段：camelCase
private final MarketRepository marketRepository;
public Market findBySlug(String slug) {}

// ✅ 常量：UPPER_SNAKE_CASE
private static final int MAX_PAGE_SIZE = 100;
```

### 不可变性

```java
// ✅ 优先使用record和final字段
public record MarketDto(Long id, String name, MarketStatus status) {}

public class Market {
  private final Long id;
  private final String name;
  // 只提供getter，不提供setter
}
```

### Optional使用

```java
// ✅ find*方法返回Optional
Optional<Market> market = marketRepository.findBySlug(slug);

// ✅ 使用map/flatMap而不是get()
return market
    .map(MarketResponse::from)
    .orElseThrow(() -> new EntityNotFoundException("Market not found"));
```

### Stream最佳实践

```java
// ✅ 使用Stream进行转换，保持管道简短
List<String> names = markets.stream()
    .map(Market::name)
    .filter(Objects::nonNull)
    .toList();

// ❌ 避免复杂的嵌套Stream；为清晰起见优先使用循环
```

### 异常处理

- 对领域错误使用非受检异常；用上下文包装技术异常
- 创建领域特定的异常（如 `MarketNotFoundException`）
- 避免宽泛的 `catch (Exception ex)`，除非重新抛出或集中记录

```java
throw new MarketNotFoundException(slug);
```

### 泛型和类型安全

- 避免原始类型；声明泛型参数
- 优先使用有边界的泛型以实现可复用工具

```java
public <T extends Identifiable> Map<Long, T> indexById(Collection<T> items) { ... }
```

---

## 项目结构

### MVC架构

```
src/main/java/com/example/app/
├── config/                    # 配置类
├── controller/                # 控制器
├── service/                  # 服务层
├── repository/               # 仓储层
├── entity/                   # 实体
├── dto/                      # 数据传输对象
│   ├── request/              # 请求DTO
│   └── response/             # 响应DTO
├── common/                   # 公共类
│   ├── exception/            # 异常定义
│   └── constant/            # 常量
└── util/                     # 工具类
```

### DDD架构

```
src/main/java/com/example/app/
├── config/                    # 配置类
├── controller/                # 控制器（适配层）
├── application/               # 应用层
│   ├── service/              # 应用服务
│   └── dto/                  # 应用DTO
├── domain/                   # 领域层
│   ├── entity/               # 领域实体
│   ├── valueobject/          # 值对象
│   ├── aggregates/           # 聚合根
│   ├── repository/           # 仓储接口
│   ├── domainevent/          # 领域事件
│   └── service/              # 领域服务
├── infrastructure/           # 基础设施层
│   ├── repository/           # 仓储实现
│   ├── converter/            # 转换器
│   └── persistence/         # 持久化
└── interfaces/               # 接口层
    ├── dto/                  # 接口DTO
    └── assembler/            # 组装器
```

---

## 格式化与风格

- 一致使用4个空格缩进
- 每个文件一个公开的顶层类型
- 保持方法简短且专注；提取辅助方法
- 成员顺序：常量、字段、构造函数、公开方法、受保护方法、私有方法

## 需要避免的代码坏味道

- 长参数列表 → 使用DTO/构建器
- 深层嵌套 → 提前返回
- 魔法数字 → 命名常量
- 静态可变状态 → 优先依赖注入
- 静默catch块 → 记录并处理或重新抛出

## 日志记录

```java
private static final Logger log = LoggerFactory.getLogger(MarketService.class);
log.info("fetch_market slug={}", slug);
log.error("failed_fetch_market slug={}", slug, ex);
```

## 空值处理

- 仅在不可避免时接受 `@Nullable`；否则使用 `@NonNull`
- 在输入上使用Bean Validation（`@NotNull`、`@NotBlank`）

## 测试期望

- JUnit 5 + AssertJ用于流畅的断言
- Mockito用于模拟；尽量避免部分模拟
- 优先确定性的测试；不使用隐藏的sleep

---

## 实现模板使用指南

详细实现模板和使用说明请参考 `REFERENCE/` 目录：

| 文件 | 适用架构 | 内容 |
|------|---------|------|
| MVC开发模板.md | MVC | Controller、Service、Repository、Entity、DTO完整实现 |
| DDD开发模板.md | DDD | 领域实体、聚合根、领域服务、仓储、应用服务实现 |
| 阿里开发规范速查.md | 通用 | 阿里Java开发规范要点速查 |

**记住**：保持代码有目的性、有类型、可观察。在证明必要之前，优先优化可维护性而非微优化。
