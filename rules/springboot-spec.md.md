
```makedown
# SpringBoot 企业级开发规范
## 1. 项目结构
- com.公司名.项目名
  - controller：API 接口层
  - service：业务逻辑层
  - mapper：数据访问层
  - entity/model：实体类
  - vo：视图返回对象
  - dto：参数接收对象
  - config：配置类
  - util：工具类
  - exception：全局异常
  - constant：常量

## 2. 分层职责
- Controller：参数校验、返回封装、权限校验
- Service：核心业务、事务控制、组合调用
- Mapper：数据操作，不写业务
- VO：返回给前端
- DTO：接收前端参数

## 3. 配置规范
- 禁用硬编码，统一放入 application.yml
- 多环境区分：dev/test/prod
- 配置项使用驼峰命名
- 敏感配置放入配置中心/环境变量

## 4. 全局规范
- 统一返回 Result.success() / Result.fail()
- 全局异常捕获 @RestControllerAdvice
- 事务使用 @Transactional(rollbackFor = Exception.class)
- 日志使用 SLF4J，不使用 System.out
- 接口必须定义接口文档注解
```