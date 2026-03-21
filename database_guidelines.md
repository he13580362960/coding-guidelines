## 数据库设计规范
### 调试环境
*   **数据库版本**：Mysql 8.0.23。
*   **开发环境依赖**：使用本地mysql实例作为开发环境调试使用，ip=127.0.0.1,账号是root,密码是root。

### 数据库基础规范
*   **存储引擎**：InnoDB。
*   **字符集**：utf8mb4。
*   **表前缀**：`tb_{module}_`，如 `tb_system_dept`。

### 表结构规范
*   **主键**：必须包含 `id` 字段，通常为 `varchar` (UUID/Snowflake) 或 `bigint`。
*   **通用固定字段**（所有表必须包含）：
    ```sql
    `is_delete` smallint DEFAULT '0' COMMENT '是否删除 0:否 1:是',
    `gmt_create` datetime DEFAULT NULL COMMENT '创建时间',
    `gmt_modified` datetime DEFAULT NULL COMMENT '修改时间',
    `creater_name` varchar(50)  DEFAULT NULL COMMENT '创建人名称',
    `modifier_name` varchar(50)  DEFAULT NULL COMMENT '修改人名称',
    `last_updated_time` timestamp not null default current_timestamp on update current_timestamp comment '最后修改时间'
    ```
*   **字段命名**：lower_case_with_underscores，如 `dept_name`。
*   **非空约束**：核心业务字段应尽量设置为 `NOT NULL`，并提供默认值。
*   **禁止JSON存储**：核心业务字段应尽量避免使用 JSON 类型存储，而是使用独立的表存储关联数据。
*   **禁止last_updated_time生成到业务代码中**：last_updated_time不体现在业务代码中，仅由数据库自主更新，禁止手动更新。

### 索引与存储规范
*   **固定索引**（所有表必须包含）：
    ```sql
    KEY `idx_last_updated_time` (`last_updated_time`) USING BTREE
    ```
    *用途：用于数据同步（ETL）及增量更新检测。*
*   **索引命名**：`idx_{table_short}_{field}`。
*   **唯一索引**：`uk_{table_short}_{field}`。
*   **外键**：**禁止** 使用数据库物理外键，关联关系在代码层面维护。
## 附则
*   **枚举使用**：状态值、类型值必须定义枚举类 (`Enum`)，禁止在代码中使用魔术数字（Magic Number）。
*   **日志规范**：使用 `slf4j` + `logback`。日志打印需包含关键参数（如订单号、用户ID），方便排查问题。
