# DDD开发模板

基于阿里Java开发规范的DDD（领域驱动设计）架构实现模板，适用于业务复杂的中大型项目。

## 目录

1. [项目结构](#1-项目结构)
2. [领域层](#2-领域层)
3. [应用层](#3-应用层)
4. [基础设施层](#4-基础设施层)
5. [接口层](#5-接口层)
6. [领域事件](#6-领域事件)
7. [值对象](#7-值对象)

---

## 1. 项目结构

```
src/main/java/com/example/app/
├── config/                          # 配置类
├── controller/                      # 控制器（适配层）
├── application/                      # 应用层
│   ├── service/                     # 应用服务
│   ├── command/                     # 命令对象
│   ├── query/                       # 查询对象
│   └── dto/                         # 应用DTO
├── domain/                          # 领域层
│   ├── market/                      # 市场限界上下文
│   │   ├── entity/                  # 领域实体
│   │   │   └── Market.java
│   │   ├── valueobject/             # 值对象
│   │   │   └── MarketId.java
│   │   ├── aggregates/              # 聚合根
│   │   │   └── MarketAggregate.java
│   │   ├── repository/               # 仓储接口
│   │   │   └── IMarketRepository.java
│   │   ├── domainevent/             # 领域事件
│   │   │   └── MarketCreatedEvent.java
│   │   └── service/                 # 领域服务
│   │       └── MarketDomainService.java
│   └── shared/                      # 共享内核
│       └── BaseEntity.java
├── infrastructure/                  # 基础设施层
│   ├── repository/                  # 仓储实现
│   │   └── MarketRepositoryImpl.java
│   ├── converter/                   # 转换器
│   │   └── MarketConverter.java
│   └── persistence/                 # 持久化
│       ├── entity/                   # 持久化实体
│       │   └── MarketEntity.java
│       └── mapper/                  # MyBatis映射
├── interfaces/                      # 接口层
│   ├── dto/                         # 接口DTO
│   │   ├── request/
│   │   └── response/
│   └── assembler/                   # 组装器
└── common/                          # 公共类
    ├── exception/
    │   └── DomainException.java
    └── constant/
```

---

## 2. 领域层

### 2.1 聚合根

```java
/**
 * 市场聚合根
 * 聚合根负责维护聚合内部的一致性，所有对聚合的修改都必须通过聚合根完成
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class MarketAggregate extends BaseEntity<MarketId> {

    /**
     * 市场ID
     */
    private MarketId marketId;

    /**
     * 市场名称
     */
    private String name;

    /**
     * 市场slug
     */
    private String slug;

    /**
     * 市场状态
     */
    private MarketStatus status;

    /**
     * 私有构造函数，使用工厂方法创建
     */
    private MarketAggregate(MarketId marketId, String name, String slug, MarketStatus status) {
        this.marketId = marketId;
        this.name = name;
        this.slug = slug;
        this.status = status;
    }

    /**
     * 工厂方法：创建市场
     *
     * @param name  市场名称
     * @param slug  市场slug
     * @param status 市场状态
     * @return 市场聚合根
     */
    public static MarketAggregate create(String name, String slug, MarketStatus status) {
        // 业务规则校验
        Assert.hasText(name, "市场名称不能为空");
        Assert.hasText(slug, "市场slug不能为空");
        Assert.notNull(status, "市场状态不能为空");

        MarketAggregate market = new MarketAggregate(
            MarketId.generate(),
            name,
            slug,
            status
        );

        // 添加领域事件
        market.addDomainEvent(new MarketCreatedEvent(market.getMarketId(), name));

        return market;
    }

    /**
     * 激活市场
     */
    public void activate() {
        if (this.status == MarketStatus.ACTIVE) {
            return;
        }
        this.status = MarketStatus.ACTIVE;
        this.addDomainEvent(new MarketActivatedEvent(this.marketId));
    }

    /**
     * 禁用市场
     */
    public void deactivate() {
        if (this.status == MarketStatus.DISABLED) {
            return;
        }
        this.status = MarketStatus.DISABLED;
        this.addDomainEvent(new MarketDeactivatedEvent(this.marketId));
    }

    /**
     * 更新市场信息
     *
     * @param name  新名称
     * @param slug  新slug
     */
    public void update(String name, String slug) {
        Assert.hasText(name, "市场名称不能为空");
        Assert.hasText(slug, "市场slug不能为空");

        this.name = name;
        this.slug = slug;
        this.addDomainEvent(new MarketUpdatedEvent(this.marketId, name, slug));
    }

    @Override
    public MarketId getId() {
        return this.marketId;
    }
}
```

### 2.2 领域实体

```java
/**
 * 市场实体
 * 实体具有唯一标识，可以通过ID进行区分
 *
 * @author developer
 * @since 1.0.0
 */
@Data
public class MarketEntity {

    /**
     * 实体ID
     */
    private MarketId id;

    /**
     * 市场名称
     */
    private String name;

    /**
     * 市场slug
     */
    private String slug;

    /**
     * 创建时间
     */
    private LocalDateTime createdAt;

    /**
     * 更新时间
     */
    private LocalDateTime updatedAt;

    /**
     * 从聚合根转换
     */
    public static MarketEntity fromAggregate(MarketAggregate aggregate) {
        MarketEntity entity = new MarketEntity();
        entity.setId(aggregate.getMarketId());
        entity.setName(aggregate.getName());
        entity.setSlug(aggregate.getSlug());
        entity.setStatus(aggregate.getStatus());
        entity.setCreatedAt(aggregate.getCreatedAt());
        entity.setUpdatedAt(aggregate.getUpdatedAt());
        return entity;
    }
}
```

### 2.3 领域服务

```java
/**
 * 市场领域服务
 * 领域服务用于处理跨聚合的业务逻辑，或者不适合放在聚合根中的业务逻辑
 *
 * @author developer
 * @since 1.0.0
 */
@Service
public class MarketDomainService {

    /**
     * 检查slug唯一性
     *
     * @param repository 仓储
     * @param slug       slug
     * @param excludeId  排除的ID（用于更新时校验）
     */
    public void checkSlugUniqueness(IMarketRepository repository, String slug, MarketId excludeId) {
        Optional<MarketAggregate> existing = repository.findBySlug(slug);

        if (existing.isPresent()) {
            MarketAggregate market = existing.get();
            // 如果不是更新自身，则slug已存在
            if (excludeId == null || !market.getId().equals(excludeId)) {
                throw new DomainException("市场slug已存在: " + slug);
            }
        }
    }

    /**
     * 验证市场是否可以删除
     *
     * @param aggregate 市场聚合根
     */
    public void validateForDeletion(MarketAggregate aggregate) {
        // 在这里添加业务规则，比如检查是否有下游依赖
        // if (aggregate.hasDependentOrders()) {
        //     throw new DomainException("市场有关联订单，无法删除");
        // }
    }
}
```

### 2.4 仓储接口

```java
/**
 * 市场仓储接口
 * 仓储接口定义在领域层，具体实现在基础设施层
 *
 * @author developer
 * @since 1.0.0
 */
public interface IMarketRepository {

    /**
     * 根据ID查询
     *
     * @param id 市场ID
     * @return 市场聚合根
     */
    Optional<MarketAggregate> findById(MarketId id);

    /**
     * 根据slug查询
     *
     * @param slug 市场slug
     * @return 市场聚合根
     */
    Optional<MarketAggregate> findBySlug(String slug);

    /**
     * 保存聚合根
     *
     * @param aggregate 市场聚合根
     * @return 保存后的聚合根
     */
    MarketAggregate save(MarketAggregate aggregate);

    /**
     * 删除聚合根
     *
     * @param id 市场ID
     */
    void delete(MarketId id);

    /**
     * 检查slug是否存在
     *
     * @param slug 市场slug
     * @return 是否存在
     */
    boolean existsBySlug(String slug);
}
```

---

## 3. 应用层

### 3.1 应用服务

```java
/**
 * 市场应用服务
 * 应用服务负责协调领域对象完成业务流程，不包含业务逻辑
 *
 * @author developer
 * @since 1.0.0
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class MarketApplicationService {

    private final IMarketRepository marketRepository;
    private final MarketDomainService marketDomainService;
    private final ApplicationEventPublisher eventPublisher;

    /**
     * 创建市场
     *
     * @param command 创建命令
     * @return 市场ID
     */
    @Transactional
    public MarketId createMarket(CreateMarketCommand command) {
        // 1. 领域规则校验
        marketDomainService.checkSlugUniqueness(marketRepository, command.getSlug(), null);

        // 2. 创建聚合根
        MarketAggregate aggregate = MarketAggregate.create(
            command.getName(),
            command.getSlug(),
            command.getStatus()
        );

        // 3. 保存
        MarketAggregate saved = marketRepository.save(aggregate);

        // 4. 发布领域事件
        aggregate.getDomainEvents().forEach(eventPublisher::publish);

        log.info("创建市场成功, id={}, name={}", saved.getId(), saved.getName());

        return saved.getId();
    }

    /**
     * 更新市场
     *
     * @param id      市场ID
     * @param command 更新命令
     */
    @Transactional
    public void updateMarket(MarketId id, UpdateMarketCommand command) {
        // 1. 查询现有聚合根
        MarketAggregate aggregate = marketRepository.findById(id)
            .orElseThrow(() -> new DomainException("市场不存在"));

        // 2. 领域规则校验
        marketDomainService.checkSlugUniqueness(marketRepository, command.getSlug(), id);

        // 3. 更新
        aggregate.update(command.getName(), command.getSlug());

        // 4. 保存
        marketRepository.save(aggregate);

        // 5. 发布领域事件
        aggregate.getDomainEvents().forEach(eventPublisher::publish);

        log.info("更新市场成功, id={}", id);
    }

    /**
     * 删除市场
     *
     * @param id 市场ID
     */
    @Transactional
    public void deleteMarket(MarketId id) {
        // 1. 查询现有聚合根
        MarketAggregate aggregate = marketRepository.findById(id)
            .orElseThrow(() -> new DomainException("市场不存在"));

        // 2. 业务规则校验
        marketDomainService.validateForDeletion(aggregate);

        // 3. 删除
        marketRepository.delete(id);

        log.info("删除市场成功, id={}", id);
    }
}
```

### 3.2 命令对象

```java
/**
 * 创建市场命令
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CreateMarketCommand {

    @NotBlank(message = "市场名称不能为空")
    @Size(max = 100, message = "市场名称长度不能超过100")
    private String name;

    @NotBlank(message = "市场slug不能为空")
    @Size(max = 50, message = "市场slug长度不能超过50")
    @Pattern(regexp = "^[a-z0-9-]+$", message = "市场slug格式错误")
    private String slug;

    @NotNull(message = "市场状态不能为空")
    private MarketStatus status;
}
```

---

## 4. 基础设施层

### 4.1 仓储实现

```java
/**
 * 市场仓储实现
 * 负责将聚合根转换为持久化实体，并进行数据库操作
 *
 * @author developer
 * @since 1.0.0
 */
@Repository
@RequiredArgsConstructor
public class MarketRepositoryImpl implements IMarketRepository {

    private final MarketJpaRepository jpaRepository;
    private final MarketConverter converter;

    @Override
    public Optional<MarketAggregate> findById(MarketId id) {
        return jpaRepository.findById(id.getValue())
            .map(converter::toAggregate);
    }

    @Override
    public Optional<MarketAggregate> findBySlug(String slug) {
        return jpaRepository.findBySlug(slug)
            .map(converter::toAggregate);
    }

    @Override
    public MarketAggregate save(MarketAggregate aggregate) {
        MarketEntity entity = converter.toEntity(aggregate);
        MarketEntity saved = jpaRepository.save(entity);
        return converter.toAggregate(saved);
    }

    @Override
    public void delete(MarketId id) {
        jpaRepository.deleteById(id.getValue());
    }

    @Override
    public boolean existsBySlug(String slug) {
        return jpaRepository.existsBySlug(slug);
    }
}
```

### 4.2 转换器

```java
/**
 * 市场对象转换器
 * 负责不同层级对象之间的转换
 *
 * @author developer
 * @since 1.0.0
 */
@Component
public class MarketConverter {

    /**
     * 持久化实体转聚合根
     */
    public MarketAggregate toAggregate(MarketEntity entity) {
        if (entity == null) {
            return null;
        }

        MarketAggregate aggregate = new MarketAggregate();
        aggregate.setMarketId(new MarketId(entity.getId()));
        aggregate.setName(entity.getName());
        aggregate.setSlug(entity.getSlug());
        aggregate.setStatus(entity.getStatus());
        aggregate.setCreatedAt(entity.getCreatedAt());
        aggregate.setUpdatedAt(entity.getUpdatedAt());

        return aggregate;
    }

    /**
     * 聚合根转持久化实体
     */
    public MarketEntity toEntity(MarketAggregate aggregate) {
        if (aggregate == null) {
            return null;
        }

        MarketEntity entity = new MarketEntity();
        if (aggregate.getMarketId() != null) {
            entity.setId(aggregate.getMarketId().getValue());
        }
        entity.setName(aggregate.getName());
        entity.setSlug(aggregate.getSlug());
        entity.setStatus(aggregate.getStatus());
        entity.setCreatedAt(aggregate.getCreatedAt());
        entity.setUpdatedAt(aggregate.getUpdatedAt());

        return entity;
    }
}
```

### 4.3 持久化实体

```java
/**
 * 市场持久化实体
 * 对应数据库表
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Entity
@Table(name = "t_market")
public class MarketEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true, length = 50)
    private String slug;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private MarketStatus status;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    @Version
    private Long version;
}
```

---

## 5. 接口层

### 5.1 控制器

```java
/**
 * 市场接口控制器
 * 负责接收HTTP请求，调用应用服务，返回响应
 *
 * @author developer
 * @since 1.0.0
 */
@Slf4j
@RestController
@RequestMapping("/api/v1/markets")
@RequiredArgsConstructor
public class MarketController {

    private final MarketApplicationService applicationService;
    private final MarketQueryService queryService;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<Void> create(@Valid @RequestBody CreateMarketRequest request) {
        CreateMarketCommand command = toCommand(request);
        MarketId id = applicationService.createMarket(command);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(URI.create("/api/v1/markets/" + id.getValue()));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Void> update(
        @PathVariable Long id,
        @Valid @RequestBody UpdateMarketRequest request
    ) {
        UpdateMarketCommand command = toCommand(request);
        applicationService.updateMarket(new MarketId(id), command);
        return ResponseEntity.ok().build();
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        applicationService.deleteMarket(new MarketId(id));
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/{id}")
    public ResponseEntity<MarketResponse> getById(@PathVariable Long id) {
        return queryService.findById(new MarketId(id))
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    private CreateMarketCommand toCommand(CreateMarketRequest request) {
        return CreateMarketCommand.builder()
            .name(request.getName())
            .slug(request.getSlug())
            .status(request.getStatus())
            .build();
    }
}
```

---

## 6. 领域事件

### 6.1 领域事件基类

```java
/**
 * 领域事件基类
 *
 * @author developer
 * @since 1.0.0
 */
@Data
public abstract class DomainEvent {

    /**
     * 事件ID
     */
    private String eventId;

    /**
     * 发生时间
     */
    private LocalDateTime occurredOn;

    public DomainEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.occurredOn = LocalDateTime.now();
    }
}
```

### 6.2 具体领域事件

```java
/**
 * 市场创建事件
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class MarketCreatedEvent extends DomainEvent {

    /**
     * 市场ID
     */
    private MarketId marketId;

    /**
     * 市场名称
     */
    private String name;

    public MarketCreatedEvent(MarketId marketId, String name) {
        super();
        this.marketId = marketId;
        this.name = name;
    }
}
```

---

## 7. 值对象

### 7.1 值对象基类

```java
/**
 * 值对象基类
 * 值对象是不可变的，没有身份ID
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@EqualsAndHashCode
@Builder
public class MarketId {

    /**
     * ID值
     */
    private final Long value;

    private MarketId(Long value) {
        this.value = value;
    }

    /**
     * 生成新的ID
     */
    public static MarketId generate() {
        return new MarketId(IdGenerator.generate());
    }

    /**
     * 从值创建
     */
    public static MarketId of(Long value) {
        return new MarketId(value);
    }
}
```

---

## 附录：DDD核心概念

| 概念 | 说明 |
|------|------|
| **聚合根** | 聚合的根节点，负责维护聚合内部一致性 |
| **实体** | 具有唯一标识的对象 |
| **值对象** | 没有身份ID的不可变对象 |
| **领域服务** | 处理跨聚合或不适合放在聚合中的业务逻辑 |
| **仓储** | 负责聚合的持久化和查询 |
| **领域事件** | 领域中发生的事件，用于解耦 |
| **应用服务** | 协调领域对象完成业务流程 |
