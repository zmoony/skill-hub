
```
# 技能：SQL 转 Java 实体
## 描述
自动将 MySQL/PostgreSQL 的建表语句转换成标准 Java 实体类
自动生成：字段、注释、类型映射、Lombok 注解、MyBatis-Plus 注解

## 规则
- 使用 @Data @NoArgsConstructor @AllArgsConstructor
- 数据库类型 → Java 类型自动映射
- 字段注释自动生成 Javadoc
- 主键自动识别 @TableId
- 驼峰命名自动转换
- 自动生成 @TableName("表名")
- 日期类型使用 LocalDateTime
```