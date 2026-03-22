# MVC开发模板

基于阿里Java开发规范的MVC架构实现模板，适用于业务逻辑相对简单的项目。

## 目录

1. [项目结构](#1-项目结构)
2. [Entity层](#2-entity层)
3. [Repository层](#3-repository层)
4. [Service层](#4-service层)
5. [Controller层](#5-controller层)
6. [DTO层](#6-dto层)
7. [异常处理](#7-异常处理)
8. [配置类](#8-配置类)

---

## 1. 项目结构

```
src/main/java/com/example/app/
├── config/                        # 配置类
│   └── WebConfig.java
├── controller/                   # 控制器层
│   └── MarketController.java
├── service/                      # 服务层
│   ├── MarketService.java
│   └── impl/
├── repository/                   # 仓储层
│   ├── MarketRepository.java
│   └── impl/
├── entity/                       # 实体层
│   └── Market.java
├── dto/                         # 数据传输对象
│   ├── request/
│   │   └── CreateMarketRequest.java
│   └── response/
│       └── MarketResponse.java
├── common/                       # 公共类
│   ├── exception/
│   │   ├── BusinessException.java
│   │   └── MarketNotFoundException.java
│   └── constant/
│       └── MarketStatus.java
└── util/                        # 工具类
    └── DateUtils.java

src/main/resources/
├── application.yml
└── mapper/                      # MyBatis映射文件（如使用）
```

---

## 2. Entity层

### 2.1 实体类

```java
/**
 * 市场实体
 * 对应数据库表：t_market
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Entity
@Table(name = "t_market")
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Market {

    /**
     * 主键ID
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    /**
     * 市场名称
     */
    @Column(name = "name", nullable = false, length = 100)
    private String name;

    /**
     * 市场slug
     */
    @Column(name = "slug", nullable = false, unique = true, length = 50)
    private String slug;

    /**
     * 市场状态
     */
    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    private MarketStatus status;

    /**
     * 创建时间
     */
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreatedDate
    private LocalDateTime createdAt;

    /**
     * 更新时间
     */
    @Column(name = "updated_at", nullable = false)
    @LastModifiedDate
    private LocalDateTime updatedAt;

    /**
     * 版本号（乐观锁）
     */
    @Version
    private Long version;

    /**
     * 是否删除
     */
    @Column(name = "deleted")
    private Boolean deleted;
}
```

### 2.2 枚举类

```java
/**
 * 市场状态枚举
 */
public enum MarketStatus {
    /**
     * 禁用
     */
    DISABLED,
    /**
     * 启用
     */
    ENABLED
}
```

---

## 3. Repository层

### 3.1 Repository接口

```java
/**
 * 市场仓储接口
 *
 * @author developer
 * @since 1.0.0
 */
public interface MarketRepository extends JpaRepository<Market, Long> {

    /**
     * 根据slug查询市场
     *
     * @param slug 市场slug
     * @return 市场信息
     */
    Optional<Market> findBySlug(String slug);

    /**
     * 根据slug查询未删除的市场
     *
     * @param slug 市场slug
     * @return 市场信息
     */
    Optional<Market> findBySlugAndDeletedFalse(String slug);

    /**
     * 根据状态查询市场列表
     *
     * @param status 市场状态
     * @return 市场列表
     */
    List<Market> findByStatusAndDeletedFalse(MarketStatus status);

    /**
     * 检查slug是否存在
     *
     * @param slug 市场slug
     * @return 是否存在
     */
    boolean existsBySlug(String slug);

    /**
     * 分页查询市场
     *
     * @param name   市场名称（模糊查询）
     * @param status 市场状态
     * @param page   页码
     * @param size   每页大小
     * @return 分页结果
     */
    Page<Market> findByNameContainingAndStatusAndDeletedFalse(
        String name,
        MarketStatus status,
        Pageable pageable
    );
}
```

### 3.2 自定义Repository实现

```java
/**
 * 市场仓储自定义实现
 *
 * @author developer
 * @since 1.0.0
 */
@Repository
@RequiredArgsConstructor
public class MarketRepositoryImpl implements MarketCustomRepository {

    private final EntityManager entityManager;

    @Override
    public List<Market> findByCriteria(MarketCriteria criteria) {
        StringBuilder jpql = new StringBuilder("SELECT m FROM Market m WHERE 1=1");
        List<Predicate> predicates = new ArrayList<>();

        if (StringUtils.hasText(criteria.getName())) {
            jpql.append(" AND m.name LIKE :name");
            predicates.add(criteria.getBuilder().like(criteria.getName()));
        }

        if (criteria.getStatus() != null) {
            jpql.append(" AND m.status = :status");
            predicates.add(criteria.getBuilder().equal(criteria.getStatus()));
        }

        return entityManager.createQuery(jpql.toString(), Market.class)
            .getResultList();
    }
}
```

---

## 4. Service层

### 4.1 Service接口

```java
/**
 * 市场服务接口
 *
 * @author developer
 * @since 1.0.0
 */
public interface MarketService {

    /**
     * 分页查询市场
     *
     * @param query 查询条件
     * @return 分页结果
     */
    Page<Market> queryByPage(MarketQuery query);

    /**
     * 根据ID查询市场
     *
     * @param id 市场ID
     * @return 市场信息
     */
    Optional<Market> findById(Long id);

    /**
     * 根据slug查询市场
     *
     * @param slug 市场slug
     * @return 市场信息
     */
    Optional<Market> findBySlug(String slug);

    /**
     * 创建市场
     *
     * @param request 创建请求
     * @return 创建的市场
     */
    Market create(CreateMarketRequest request);

    /**
     * 更新市场
     *
     * @param id      市场ID
     * @param request 更新请求
     * @return 更新的市场
     */
    Market update(Long id, UpdateMarketRequest request);

    /**
     * 删除市场
     *
     * @param id 市场ID
     */
    void delete(Long id);
}
```

### 4.2 Service实现

```java
/**
 * 市场服务实现
 *
 * @author developer
 * @since 1.0.0
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class MarketServiceImpl implements MarketService {

    private final MarketRepository marketRepository;
    private final MarketValidator validator;

    @Override
    @Transactional(readOnly = true)
    public Page<Market> queryByPage(MarketQuery query) {
        // 参数校验
        PageRequest pageRequest = PageRequest.of(
            query.getPage(),
            query.getSize(),
            Sort.by(Sort.Direction.DESC, "createdAt")
        );

        return marketRepository.findByNameContainingAndStatusAndDeletedFalse(
            query.getName(),
            query.getStatus(),
            pageRequest
        );
    }

    @Override
    @Transactional(readOnly = true)
    public Optional<Market> findById(Long id) {
        return marketRepository.findById(id);
    }

    @Override
    @Transactional(readOnly = true)
    public Optional<Market> findBySlug(String slug) {
        return marketRepository.findBySlugAndDeletedFalse(slug);
    }

    @Override
    @Transactional
    public Market create(CreateMarketRequest request) {
        // 参数校验
        validator.validate(request);

        // 业务校验
        if (marketRepository.existsBySlug(request.getSlug())) {
            throw new BusinessException("市场slug已存在");
        }

        // 构建实体
        Market market = Market.builder()
            .name(request.getName())
            .slug(request.getSlug())
            .status(request.getStatus())
            .deleted(false)
            .build();

        // 保存
        Market saved = marketRepository.save(market);
        log.info("创建市场成功, id={}, name={}", saved.getId(), saved.getName());

        return saved;
    }

    @Override
    @Transactional
    public Market update(Long id, UpdateMarketRequest request) {
        // 查询现有数据
        Market market = marketRepository.findById(id)
            .orElseThrow(() -> new MarketNotFoundException(id));

        // 业务校验
        if (!Objects.equals(market.getSlug(), request.getSlug())
            && marketRepository.existsBySlug(request.getSlug())) {
            throw new BusinessException("市场slug已存在");
        }

        // 更新属性
        market.setName(request.getName());
        market.setSlug(request.getSlug());
        market.setStatus(request.getStatus());

        // 保存
        Market updated = marketRepository.save(market);
        log.info("更新市场成功, id={}", id);

        return updated;
    }

    @Override
    @Transactional
    public void delete(Long id) {
        Market market = marketRepository.findById(id)
            .orElseThrow(() -> new MarketNotFoundException(id));

        // 逻辑删除
        market.setDeleted(true);
        marketRepository.save(market);

        log.info("删除市场成功, id={}", id);
    }
}
```

---

## 5. Controller层

### 5.1 Controller

```java
/**
 * 市场控制器
 *
 * @author developer
 * @since 1.0.0
 */
@Slf4j
@RestController
@RequestMapping("/api/v1/markets")
@RequiredArgsConstructor
public class MarketController {

    private final MarketService marketService;
    private final MarketConverter converter;

    /**
     * 分页查询市场列表
     */
    @GetMapping
    public ResponseEntity<Page<MarketResponse>> queryByPage(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) MarketStatus status,
        @RequestParam(defaultValue = "0") Integer page,
        @RequestParam(defaultValue = "20") Integer size
    ) {
        MarketQuery query = MarketQuery.builder()
            .name(name)
            .status(status)
            .page(page)
            .size(size)
            .build();

        Page<Market> result = marketService.queryByPage(query);
        Page<MarketResponse> response = result.map(converter::toResponse);

        return ResponseEntity.ok(response);
    }

    /**
     * 根据ID查询市场详情
     */
    @GetMapping("/{id}")
    public ResponseEntity<MarketResponse> getById(@PathVariable Long id) {
        return marketService.findById(id)
            .map(market -> ResponseEntity.ok(converter.toResponse(market)))
            .orElse(ResponseEntity.notFound().build());
    }

    /**
     * 根据slug查询市场
     */
    @GetMapping("/slug/{slug}")
    public ResponseEntity<MarketResponse> getBySlug(@PathVariable String slug) {
        return marketService.findBySlug(slug)
            .map(market -> ResponseEntity.ok(converter.toResponse(market)))
            .orElse(ResponseEntity.notFound().build());
    }

    /**
     * 创建市场
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<MarketResponse> create(@Valid @RequestBody CreateMarketRequest request) {
        Market market = marketService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(converter.toResponse(market));
    }

    /**
     * 更新市场
     */
    @PutMapping("/{id}")
    public ResponseEntity<MarketResponse> update(
        @PathVariable Long id,
        @Valid @RequestBody UpdateMarketRequest request
    ) {
        Market market = marketService.update(id, request);
        return ResponseEntity.ok(converter.toResponse(market));
    }

    /**
     * 删除市场
     */
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        marketService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 6. DTO层

### 6.1 请求DTO

```java
/**
 * 创建市场请求
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CreateMarketRequest {

    /**
     * 市场名称
     */
    @NotBlank(message = "市场名称不能为空")
    @Size(max = 100, message = "市场名称长度不能超过100")
    private String name;

    /**
     * 市场slug
     */
    @NotBlank(message = "市场slug不能为空")
    @Size(max = 50, message = "市场slug长度不能超过50")
    @Pattern(regexp = "^[a-z0-9-]+$", message = "市场slug只能包含小写字母、数字和连字符")
    private String slug;

    /**
     * 市场状态
     */
    @NotNull(message = "市场状态不能为空")
    private MarketStatus status;
}
```

### 6.2 响应DTO

```java
/**
 * 市场响应
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class MarketResponse {

    private Long id;
    private String name;
    private String slug;
    private MarketStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    /**
     * 从实体转换为响应对象
     */
    public static MarketResponse fromEntity(Market market) {
        return MarketResponse.builder()
            .id(market.getId())
            .name(market.getName())
            .slug(market.getSlug())
            .status(market.getStatus())
            .createdAt(market.getCreatedAt())
            .updatedAt(market.getUpdatedAt())
            .build();
    }
}
```

### 6.3 查询DTO

```java
/**
 * 市场查询条件
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class MarketQuery {

    /**
     * 市场名称（模糊查询）
     */
    private String name;

    /**
     * 市场状态
     */
    private MarketStatus status;

    /**
     * 页码
     */
    @Min(value = 0, message = "页码不能小于0")
    private Integer page = 0;

    /**
     * 每页大小
     */
    @Min(value = 1, message = "每页大小不能小于1")
    @Max(value = 100, message = "每页大小不能超过100")
    private Integer size = 20;
}
```

---

## 7. 异常处理

### 7.1 业务异常

```java
/**
 * 业务异常
 *
 * @author developer
 * @since 1.0.0
 */
@Data
public class BusinessException extends RuntimeException {

    /**
     * 错误码
     */
    private final String errorCode;

    /**
     * 错误消息
     */
    private final String errorMessage;

    public BusinessException(String errorMessage) {
        super(errorMessage);
        this.errorCode = "BUSINESS_ERROR";
        this.errorMessage = errorMessage;
    }

    public BusinessException(String errorCode, String errorMessage) {
        super(errorMessage);
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }
}
```

### 7.2 全局异常处理器

```java
/**
 * 全局异常处理器
 *
 * @author developer
 * @since 1.0.0
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理业务异常
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        log.warn("业务异常: errorCode={}, errorMessage={}", e.getErrorCode(), e.getErrorMessage());
        return ResponseEntity.badRequest()
            .body(new ErrorResponse(e.getErrorCode(), e.getErrorMessage()));
    }

    /**
     * 处理资源不存在异常
     */
    @ExceptionHandler(MarketNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(MarketNotFoundException e) {
        log.warn("资源不存在: id={}", e.getResourceId());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", e.getMessage()));
    }

    /**
     * 处理参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException e) {
        List<String> errors = e.getBindingResult().getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .toList();

        log.warn("参数校验失败: {}", errors);
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", String.join("; ", errors)));
    }

    /**
     * 处理未知异常
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        log.error("系统异常", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "系统异常，请稍后重试"));
    }
}
```

### 7.3 错误响应

```java
/**
 * 错误响应
 *
 * @author developer
 * @since 1.0.0
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ErrorResponse {

    /**
     * 错误码
     */
    private String errorCode;

    /**
     * 错误消息
     */
    private String errorMessage;
}
```

---

## 8. 配置类

### 8.1 分页配置

```java
/**
 * 分页配置
 *
 * @author developer
 * @since 1.0.0
 */
@Configuration
public class PageConfig {

    @Bean
    public PageHelper pageHelper() {
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "mysql");
        properties.setProperty("reasonable", "true");
        properties.setProperty("supportMethodsArguments", "true");
        pageHelper.setProperties(properties);
        return pageHelper;
    }
}
```

---

## 附录：常用依赖

```xml
<dependencies>
    <!-- Spring Boot -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- MySQL -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
