
```makedown
# MyBatis / MyBatis-Plus 企业规范
## 1. Mapper 规范
- 接口继承 BaseMapper
- 不允许手写复杂 SQL，提倡条件构造器
- 禁止 Select *，必须指定字段

## 2. XML 规范
- 开启驼峰自动映射
- SQL 关键字大写
- 动态 SQL 使用 trim 代替 where
- 批量操作必须使用 foreach
- 禁止出现 1=1 这种万能条件

## 3. MyBatis-Plus 规范
- 查询用 QueryWrapper
- 更新用 UpdateWrapper
- Lambda 方式优先使用 LambdaQueryWrapper
- 分页必须使用 Page
- 逻辑删除必须配置 3.0+
- 自动填充 create_time、update_time

## 4. 禁止行为
- 禁止在业务层拼接 SQL
- 禁止超大结果集不加分页
- 禁止循环内查询数据库
- 禁止事务中包含长耗时操作

## 5. 推荐写法
- 查询列表：list(QueryWrapper)
- 查询单条：getOne(QueryWrapper)
- 分页：page(Page, QueryWrapper)
```