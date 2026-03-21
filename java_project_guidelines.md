# 项目规范

## 一、项目基础信息规范

### 1.1 项目版本与迭代
*   **当前版本**：V1.0
*   **开发模式**：前后端分离，敏捷迭代。
*   **构建工具**：Maven（多模块聚合构建）。

### 1.2 技术栈规范
*   **JDK版本**：JDK 1.8
*   **核心框架**：Spring Boot (v2.6.4)
*   **分布式/RPC**：Dubbo (v3.0.2.1) + Zookeeper
*   **ORM框架**：MyBatis-Plus
*   **数据库**：MySQL 8.0.23
*   **缓存**：Redis
*   **消息队列**：RabbitMQ
*   **任务调度**：XXL-JOB
*   **API文档**：Knife4j

### 1.3 部署环境
*   **配置文件**：
    *   `application.yml`: 公共配置
    *   `application-dev.yml`: 开发环境
    *   `application-test.yml`: 测试环境
    *   `application-uat.yml`: 验收环境
    *   `application-prod.yml`: 生产环境

## 二、业务架构与流程规范

### 2.1 项目结构
采用maven多模块结构：

**{module}**: 父项目根目录，命名要符合业务特性，不得使用backend等不具备识别性的命名。

**{module}-service**: 业务逻辑层 和 数据访问层 (Mapper/XML)。

**{module}-api**: 接口定义层，包含 Dubbo 接口 (Service Interface) 和 DTO/VO。

### 2.2 关键业务流程规范
*   **跨模块调用**：必须通过 **Dubbo RPC** 接口调用，禁止跨模块直接查询数据库。
*   **数据隔离**：部分业务需支持数据权限隔离（如按部门/个人查看数据），需在 Service 层或 SQL 层处理。
## 三、技术架构规范

### 3.1 系统分层架构规范
遵循标准的 MVC + SOA 分层架构：
1.  **Controller 层 (`*.controller`)**：
    *   负责接收 HTTP 请求，参数校验，调用 Service 层。
    *   **禁止**：在 Controller 层直接编写复杂业务逻辑。
    *   **禁止**：在 Controller 层直接调用 Mapper/Dao 层。
    * 返回对象：统一使用  `ResultData` 包装 `VO` 对象，`ResultData`定义如下。
      ```
      {
          "code": "200",
          "msg": "请求成功",
          "data": {},
          "success": true
      }
      ```
    
      
2.  **Service 层 (`*.service` / `*.dubbo`)**：
    *   **Local Service (`*.service`)**：处理模块内部的具体业务逻辑，事务控制 (`@Transactional`)。
    *   **Dubbo Service (`*.dubbo` / `*ApiImpl`)**：实现 `*-api` 模块定义的接口，对外暴露服务，主要负责 DTO 转换和调用 Local Service。
3.  **Mapper/Dao 层 (`*.mapper`)**：
    *   负责与数据库交互，执行 SQL。
    *   推荐使用Mybatis-Plus预定义的方法操作数据库，简单条件使用LambdaQueryWrapper构建条件，复杂查询使用XML进行SQL编写。
    *   禁止物理删除数据，配置全局逻辑删除属性,操作数据时，使用`removeById`或者`removeByIds`方法，不使用`update`方法。
4.  **Entity/Model 层 (`*.entity` / `*.dto` / `*.vo`)**：
    *   **Entity**: 对应数据库表结构（`tb_*`）。
    *   **DTO (Data Transfer Object)**: Dubbo 接口传输对象，定义在 `*-api` 模块。
    *   **VO (View Object)**: 前端页面展示对象，Controller 返回。

### 3.2 接口设计规范
*   **API 命名**：RESTful 风格（推荐）或 语义化风格。
    *   查询：`GET /module/resource/{id}` 或 `/module/resource/list`
    *   新增：`POST /module/resource`
    *   修改：`POST /module/resource/update/{id}`
    *   删除：`POST /module/resource/delete/{id}`
*   **版本控制**：
    *   **禁止**：使用方法名区分版本（如 `list_new`），应采用 URL 版本号（`/v1/...`）或 Header 版本控制。
*   **参数校验**：
    *   使用 JSR-303 注解 (`@NotNull`, `@Size`) 在 DTO/VO 上进行校验。
*   **异常处理**：
    *   统一使用`ExceptionAdvice` 进行全局异常捕获。
    *   自定义业务异常需继承 `BaseException`,错误编码为`500`。

### 3.3 依赖与配置规范
*   **Maven 依赖**：
    *   模块间依赖仅引入 `*-api` 包，**禁止** 引入实现包。
    *   避免循环依赖。

### 3.4 模块目录结构规范
所有业务模块需严格遵循以下目录结构，保持风格统一：

```text
src/main/java/com/syz/{module}/
├── aspect/             # AOP切面（日志、权限、防重提交等）
├── config/             # 配置类（Spring, Shiro, Redis, Swagger等）
├── constant/           # 常量定义
├── controller/         # Web层控制器
│   └── {business}/     # 复杂模块可按业务子包划分（如 award, company）
├── core/               # 模块核心基类（BaseController, ExceptionAdvice）
├── convert/            # 对象转换器（Entity <-> DTO/VO）
├── dubbo/              # Dubbo接口实现层（实现 *-api 定义的接口，命名 *ApiImpl）
├── entity/             # 数据库实体类 (Entity) 及 视图对象 (VO)
│   ├── {Entity}.java   # 数据库映射对象
│   └── {Entity}VO.java # 前端展示对象（现状：VO常与Entity同包）
├── mapper/             # MyBatis DAO接口
├── service/            # 本地业务服务接口
│   └── impl/           # 本地业务服务实现
└── utils/              # 模块私有工具类

src/main/resources/
├── mapper/             # MyBatis XML 映射文件
├── static/             # 静态资源（HTML/JS/CSS，仅限非前后端分离页面）
├── templates/          # Thymeleaf 模板文件（按功能模块分子目录）
├── application.yml     # 公共配置
└── application-{env}.yml # 环境差异化配置
```

**特别说明**：
1.  **Dubbo实现**：Dubbo服务的实现类必须放在 `dubbo` 包下，而非 `service` 包，以区分本地事务服务和远程RPC服务。
2.  **Entity与VO**： `VO` 类建议创建独立 `vo` 包；若维护旧模块，遵循现有风格。
3.  **Mapper XML**：XML文件必须放在 `resources/mapper` 目录下，且路径与 Mapper 接口包名保持映射关系。

## 四、代码规范
*   **强制遵守**：严格遵守 `Alibaba Java Coding Guidelines`中的指引进行编码。当我的个性化规范与`Alibaba Java Coding Guidelines`中的规范冲突时，以我的规范为主。
### 4.1 命名规范
*   **类名**：UpperCamelCase，如 `DeptController`。
*   **方法名**：lowerCamelCase，如 `selectDeptList`。
*   **变量名**：lowerCamelCase，见名知意。
*   **常量**：UPPER_CASE_WITH_UNDERSCORES，如 `CACHE_EXPIRE_TIME`。
*   **包名**：`com.syz.{module}.{layer}`，如 `com.syz.system.controller`。
*   **Mapper 方法命名**：
    *   查询列表：`select*List`
    *   查询单个：`select*ById` / `select*ByCode`
    *   新增：`insert*`
    *   修改：`update*`
    *   删除：`delete*`

### 4.2 注释规范
*   **类注释**：必须包含 `@author` (生成器/作者), `@date`, 类的职责描述。
*   **方法注释**：Public 方法必须包含 Javadoc，说明 `@param`, `@return`, `@throws`。
    *   *AI适配*：注释应清晰描述业务逻辑，方便 AI 理解上下文。
*   **代码内注释**：复杂逻辑需添加单行注释 (`//`)，解释“为什么这样做”。

### 4.3 代码格式规范
*   **缩进**：4个空格。
*   **行宽**：建议 120 字符。
*   **括号**：`if/else/for` 等语句块必须使用大括号 `{}`，即使只有一行代码。
*   **工具类**：优先使用 `hutool`类库中的工具类。

### 4.4 异常处理规范
*   **捕获原则**：只捕获能处理的异常，无法处理的异常抛出给上层或全局异常处理器。
*   **日志记录**：异常捕获后必须记录日志，并且根据需要确定日志级别 (`log.error`/`log.warn`)，包含堆栈信息。
*   **禁止**：捕获 `Exception` 后不做任何处理（吞异常）。
*   **禁止**：使用`Sytem.out.println`进行日志输出。

### 4.5 编码规范
1.  **DTO/VO 转换**： 禁止使用 `BeanUtil.copy` 或 `MapStruct` 进行对象转换，严禁将 Entity 直接返回给前端，应使用明确的get/set方法进行取值与赋值。
3.  **注释要求**：要求生成的代码必须包含符合 Javadoc 规范的注释。
## 附则
*   **枚举使用**：状态值、类型值必须定义枚举类 (`Enum`)，禁止在代码中使用魔术数字（Magic Number）。
*   **日志规范**：使用 `slf4j` + `logback`。日志打印需包含关键参数（如订单号、用户ID），方便排查问题。
### 4.6 数据库规范
当需要进行数据库设计时,参考数据库设计的要求与最佳实践：[database_guidelines.md](D:\workspace\coding-guidelines\database_guidelines.md)