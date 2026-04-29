---
name: "java-business-module-gen"
description: "为Spring Cloud微服务生成完整的Java业务模块，遵循WVA项目规范。当用户需要创建新业务功能、CRUD操作、API接口或完整服务模块时调用，包含实体类、Mapper、Service、Controller、DTO、校验、异常处理和单元测试。"
---

# Java业务模块生成器

本技能用于为WVA Cloud微服务平台生成生产级别的Java业务模块。它会根据项目的架构模式、编码规范和最佳实践，生成完整的、符合规范的代码。

## 何时使用

- 用户请求创建新的业务模块或功能
- 用户需要为新的实体生成CRUD操作
- 用户希望按照项目REST模式添加API接口
- 用户需要完整的服务层实现
- 用户要求按照WVA项目规范生成代码

## 项目架构概览

### 技术栈

| 技术 | 版本 | 用途 |
|---|---|---|
| Java | 1.8 | 运行环境 |
| Spring Boot | 2.7.18 | 应用框架 |
| Spring Cloud | 2021.0.8 | 微服务框架 |
| Spring Cloud Alibaba | 2021.0.6.0 | 服务发现与配置 |
| MyBatis-Plus | 3.5.4.1 | ORM框架 |
| PostgreSQL | 42.7.2 | 数据库驱动 |
| Druid | 1.2.21 | 连接池 |
| JWT (auth0) | 4.4.0 | 身份认证 |
| Knife4j | 4.5.0 | API文档 |
| Lombok | 1.18.30 | 代码生成 |
| Hutool | 5.8.23 | 工具库 |
| Fastjson2 | 2.0.43 | JSON处理 |

### 模块结构

```
wva-cloud/
├── wva-common/                     # 共享公共库
├── wva-gateway/                    # API网关
├── wva-system-management-service/  # 字典与主题管理
├── wva-user-center-service/        # 用户、部门、角色、菜单、认证
├── wva-operation-log-service/      # 操作日志服务
└── wva-business-service/           # 业务服务（目标模块）
```

### 包结构模式

每个微服务遵循以下分层架构：

```
com.wva.<service-name>/
├── config/          # Spring配置类
├── controller/      # REST API控制器
├── dto/             # 数据传输对象
├── entity/          # MyBatis-Plus实体类
├── mapper/          # MyBatis-Plus Mapper接口
├── service/         # Service接口
│   └── impl/        # Service实现类
└── *Application.java
```

## 代码生成规则

### 1. 实体类模式

**基类**：所有实体类必须继承 `wva-common` 中的 `BaseEntity`

**BaseEntity字段**（自动继承）：
- `id` - 主键（Long）
- `createTime` - 创建时间
- `createBy` - 创建人ID
- `updateTime` - 更新时间
- `updateBy` - 更新人ID
- `deleteFlag` - 逻辑删除标识（0=正常，1=已删除）
- `remark` - 备注

**实体类模板**：

```java
package com.wva.business.entity;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableName;
import com.wva.common.entity.BaseEntity;
import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * 业务实体类描述
 *
 * @author WVA
 * @version 1.0
 * @date 2026-04-22
 */
@Data
@EqualsAndHashCode(callSuper = true)
@TableName("wva_yangzhong.table_name")
public class BusinessEntity extends BaseEntity {

    /**
     * 字段描述
     */
    @TableField("field_name")
    private String fieldName;

    /**
     * 整型字段描述
     */
    @TableField("integer_field")
    private Integer integerField;

    /**
     * 长整型字段描述
     */
    @TableField("long_field")
    private Long longField;
}
```

**命名规范**：
- 表名：`wva_yangzhong.{table_name}`（下划线命名）
- 类名：大驼峰（如 `SensitiveEvent`）
- 字段名：Java中使用小驼峰，数据库中使用下划线
- 所有字段使用 `@TableField` 注解

### 2. DTO模式

为不同操作创建独立的DTO：

**创建/更新DTO**：

```java
package com.wva.business.dto;

import lombok.Data;
import javax.validation.constraints.*;
import java.io.Serializable;

/**
 * 创建/更新DTO
 *
 * @author WVA
 * @version 1.0
 */
@Data
public class BusinessDTO implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 主键ID（更新时必填）
     */
    private Long id;

    /**
     * 必填字段示例
     */
    @NotBlank(message = "字段名称不能为空")
    @Size(max = 100, message = "字段名称长度不能超过100个字符")
    private String fieldName;

    /**
     * 数字字段示例
     */
    @NotNull(message = "数字字段不能为空")
    @Min(value = 0, message = "数字字段不能小于0")
    @Max(value = 100, message = "数字字段不能大于100")
    private Integer numberField;

    /**
     * 可选字段
     */
    private String optionalField;
}
```

**查询DTO**：

```java
package com.wva.business.dto;

import com.wva.common.result.PageRequest;
import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * 查询DTO
 *
 * @author WVA
 * @version 1.0
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class BusinessQueryDTO extends PageRequest {

    /**
     * 关键字搜索
     */
    private String keyword;

    /**
     * 状态筛选
     */
    private Integer status;

    /**
     * 其他筛选条件
     */
    private String filterField;
}
```

### 3. Mapper接口模式

```java
package com.wva.business.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.wva.business.entity.BusinessEntity;
import org.apache.ibatis.annotations.Mapper;

/**
 * 业务Mapper接口
 *
 * @author WVA
 * @version 1.0
 */
@Mapper
public interface BusinessMapper extends BaseMapper<BusinessEntity> {
    
    // 复杂查询方法可在此定义，并在XML中实现
    // List<BusinessVO> selectCustomList(@Param("query") BusinessQueryDTO query);
}
```

### 4. Service接口模式

```java
package com.wva.business.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.wva.business.dto.BusinessDTO;
import com.wva.business.dto.BusinessQueryDTO;
import com.wva.business.entity.BusinessEntity;
import com.wva.common.result.PageResult;

/**
 * 业务Service接口
 *
 * @author WVA
 * @version 1.0
 */
public interface BusinessService extends IService<BusinessEntity> {

    /**
     * 分页查询
     *
     * @param queryDTO 查询条件
     * @return 分页结果
     */
    PageResult<BusinessEntity> pageQuery(BusinessQueryDTO queryDTO);

    /**
     * 根据ID获取详情
     *
     * @param id 主键ID
     * @return 实体详情
     */
    BusinessEntity getDetail(Long id);

    /**
     * 创建
     *
     * @param dto 创建数据
     * @return 是否成功
     */
    boolean create(BusinessDTO dto);

    /**
     * 更新
     *
     * @param dto 更新数据
     * @return 是否成功
     */
    boolean update(BusinessDTO dto);

    /**
     * 删除
     *
     * @param id 主键ID
     * @return 是否成功
     */
    boolean delete(Long id);

    /**
     * 批量删除
     *
     * @param ids 主键ID列表
     * @return 是否成功
     */
    boolean deleteBatch(Long[] ids);
}
```

### 5. Service实现模式

```java
package com.wva.business.service.impl;

import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.wva.business.dto.BusinessDTO;
import com.wva.business.dto.BusinessQueryDTO;
import com.wva.business.entity.BusinessEntity;
import com.wva.business.mapper.BusinessMapper;
import com.wva.business.service.BusinessService;
import com.wva.common.exception.BusinessException;
import com.wva.common.result.PageResult;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 业务Service实现
 *
 * @author WVA
 * @version 1.0
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class BusinessServiceImpl extends ServiceImpl<BusinessMapper, BusinessEntity> 
        implements BusinessService {

    @Override
    public PageResult<BusinessEntity> pageQuery(BusinessQueryDTO queryDTO) {
        LambdaQueryWrapper<BusinessEntity> wrapper = new LambdaQueryWrapper<>();
        
        // 关键字搜索
        if (StrUtil.isNotBlank(queryDTO.getKeyword())) {
            wrapper.like(BusinessEntity::getFieldName, queryDTO.getKeyword());
        }
        
        // 状态筛选
        if (queryDTO.getStatus() != null) {
            wrapper.eq(BusinessEntity::getStatus, queryDTO.getStatus());
        }
        
        // 排序
        if (StrUtil.isNotBlank(queryDTO.getOrderField())) {
            if ("asc".equalsIgnoreCase(queryDTO.getOrderType())) {
                wrapper.orderByAsc(getColumnMapping(queryDTO.getOrderField()));
            } else {
                wrapper.orderByDesc(getColumnMapping(queryDTO.getOrderField()));
            }
        } else {
            wrapper.orderByDesc(BusinessEntity::getCreateTime);
        }
        
        // 分页查询
        Page<BusinessEntity> page = new Page<>(queryDTO.getPageNum(), queryDTO.getPageSize());
        IPage<BusinessEntity> result = this.page(page, wrapper);
        
        return new PageResult<>(result.getCurrent(), result.getSize(), result.getTotal(), result.getRecords());
    }

    @Override
    public BusinessEntity getDetail(Long id) {
        BusinessEntity entity = this.getById(id);
        if (entity == null) {
            throw new BusinessException("数据不存在");
        }
        return entity;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean create(BusinessDTO dto) {
        BusinessEntity entity = new BusinessEntity();
        // DTO转Entity
        // entity.setFieldName(dto.getFieldName());
        
        boolean result = this.save(entity);
        if (!result) {
            throw new BusinessException("创建失败");
        }
        return true;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean update(BusinessDTO dto) {
        if (dto.getId() == null) {
            throw new BusinessException("ID不能为空");
        }
        
        BusinessEntity entity = this.getById(dto.getId());
        if (entity == null) {
            throw new BusinessException("数据不存在");
        }
        
        // DTO转Entity
        // entity.setFieldName(dto.getFieldName());
        
        boolean result = this.updateById(entity);
        if (!result) {
            throw new BusinessException("更新失败");
        }
        return true;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean delete(Long id) {
        BusinessEntity entity = this.getById(id);
        if (entity == null) {
            throw new BusinessException("数据不存在");
        }
        
        boolean result = this.removeById(id);
        if (!result) {
            throw new BusinessException("删除失败");
        }
        return true;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean deleteBatch(Long[] ids) {
        boolean result = this.removeByIds(java.util.Arrays.asList(ids));
        if (!result) {
            throw new BusinessException("批量删除失败");
        }
        return true;
    }

    /**
     * 获取字段映射
     */
    private com.baomidou.mybatisplus.core.toolkit.support.SFunction<BusinessEntity, ?> getColumnMapping(String orderField) {
        switch (orderField) {
            case "fieldName":
                return BusinessEntity::getFieldName;
            case "createTime":
                return BusinessEntity::getCreateTime;
            default:
                return BusinessEntity::getCreateTime;
        }
    }
}
```

### 6. Controller模式

```java
package com.wva.business.controller;

import com.wva.business.dto.BusinessDTO;
import com.wva.business.dto.BusinessQueryDTO;
import com.wva.business.entity.BusinessEntity;
import com.wva.business.service.BusinessService;
import com.wva.common.result.PageResult;
import com.wva.common.result.Result;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

/**
 * 业务Controller
 *
 * @author WVA
 * @version 1.0
 */
@Tag(name = "业务管理")
@RestController
@RequestMapping("/api/business/businesses")
@RequiredArgsConstructor
public class BusinessController {

    private final BusinessService businessService;

    @Operation(summary = "分页查询")
    @PostMapping("/page")
    @PreAuthorize("hasAuthority('business:query')")
    public Result<PageResult<BusinessEntity>> page(@RequestBody BusinessQueryDTO queryDTO) {
        PageResult<BusinessEntity> result = businessService.pageQuery(queryDTO);
        return Result.success(result);
    }

    @Operation(summary = "根据ID获取详情")
    @GetMapping("/{id}")
    @PreAuthorize("hasAuthority('business:query')")
    public Result<BusinessEntity> getDetail(
            @Parameter(description = "主键ID") @PathVariable Long id) {
        BusinessEntity entity = businessService.getDetail(id);
        return Result.success(entity);
    }

    @Operation(summary = "创建")
    @PostMapping
    @PreAuthorize("hasAuthority('business:create')")
    public Result<Void> create(@Validated @RequestBody BusinessDTO dto) {
        businessService.create(dto);
        return Result.success();
    }

    @Operation(summary = "更新")
    @PutMapping
    @PreAuthorize("hasAuthority('business:update')")
    public Result<Void> update(@Validated @RequestBody BusinessDTO dto) {
        businessService.update(dto);
        return Result.success();
    }

    @Operation(summary = "删除")
    @DeleteMapping("/{id}")
    @PreAuthorize("hasAuthority('business:delete')")
    public Result<Void> delete(
            @Parameter(description = "主键ID") @PathVariable Long id) {
        businessService.delete(id);
        return Result.success();
    }

    @Operation(summary = "批量删除")
    @DeleteMapping("/batch")
    @PreAuthorize("hasAuthority('business:delete')")
    public Result<Void> deleteBatch(@RequestBody Long[] ids) {
        businessService.deleteBatch(ids);
        return Result.success();
    }
}
```

## 异常处理

### 自定义异常类

所有异常位于 `com.wva.common.exception`：

1. **BusinessException** - 业务逻辑错误（错误码500）
   ```java
   throw new BusinessException("错误信息");
   ```

2. **AuthException** - 认证错误（错误码401）
   ```java
   throw AuthException.tokenInvalid();
   throw AuthException.loginError();
   ```

3. **ParamValidException** - 参数验证错误（错误码400）
   ```java
   throw new ParamValidException("参数错误");
   ```

4. **ForbiddenException** - 授权错误（错误码403）
   ```java
   throw new ForbiddenException("无权限访问");
   ```

### 错误码

来自 `ErrorCodeConstants`：
- 200: 成功
- 400: 参数错误
- 401: 未授权
- 403: 禁止访问
- 404: 未找到
- 500: 业务错误
- 1xxx: 用户/认证领域
- 2xxx: 角色领域
- 3xxx: 部门领域
- 6xxx: 字典领域
- 9xxx: 系统错误

## 校验规则

### javax.validation注解

- `@NotBlank` - 字符串不能为空
- `@NotNull` - 对象不能为null
- `@Size(min, max)` - 字符串/集合大小校验
- `@Min`、`@Max` - 数字范围校验
- `@Email` - 邮箱格式校验
- `@Pattern` - 正则表达式校验

### 校验分组

使用校验分组区分创建和更新：

```java
public interface CreateGroup {}
public interface UpdateGroup {}

@NotNull(message = "ID不能为空", groups = UpdateGroup.class)
private Long id;

@NotBlank(message = "名称不能为空", groups = {CreateGroup.class, UpdateGroup.class})
private String name;
```

## 单元测试模式

### 测试结构

```java
package com.wva.business.service;

import com.wva.business.dto.BusinessDTO;
import com.wva.business.entity.BusinessEntity;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

@SpringBootTest
class BusinessServiceTest {

    @Autowired
    private BusinessService businessService;

    @Test
    void testCreate() {
        BusinessDTO dto = new BusinessDTO();
        dto.setFieldName("测试数据");
        
        boolean result = businessService.create(dto);
        assertThat(result).isTrue();
    }

    @Test
    void testGetDetail() {
        Long id = 1L;
        BusinessEntity entity = businessService.getDetail(id);
        assertThat(entity).isNotNull();
    }

    @Test
    void testDeleteNonExistent() {
        Long id = 999999L;
        assertThrows(BusinessException.class, () -> {
            businessService.delete(id);
        });
    }
}
```

### 测试框架

- **JUnit 5** - `org.junit.jupiter.api.Test`
- **Spring Boot Test** - `@SpringBootTest`
- **AssertJ** - 流式断言
- **Maven Surefire** - 测试运行器

## API文档

### Knife4j注解

- `@Tag(name = "模块名称")` - Controller级别标签
- `@Operation(summary = "接口描述")` - 方法级别操作
- `@Parameter(description = "参数描述")` - 参数描述

### API路径规范

模式：`/api/{service}/{resources}`

示例：
- `/api/user/users` - 用户管理
- `/api/business/businesses` - 业务管理
- `/api/dict/types` - 字典类型

### HTTP方法

- `POST /page` - 分页查询（POST带请求体）
- `GET /{id}` - 根据ID获取
- `POST` - 创建
- `PUT` - 更新
- `DELETE /{id}` - 删除
- `DELETE /batch` - 批量删除

## 数据库规范

### 表命名

- 前缀：业务表使用 `wva_yangzhong_`
- 格式：`wva_yangzhong_{table_name}`（下划线命名）
- 示例：`wva_yangzhong_sensitive_event`

### 公共字段

所有表必须包含：
- `id` - BIGINT PRIMARY KEY
- `create_time` - TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- `update_time` - TIMESTAMP DEFAULT CURRENT_TIMESTAMP
- `create_by` - BIGINT（创建人ID）
- `update_by` - BIGINT（更新人ID）
- `is_deleted` - SMALLINT DEFAULT 0（0=正常，1=已删除）
- `remark` - VARCHAR(500)

### 实体映射

```java
@TableName("wva_yangzhong.table_name")
public class EntityName extends BaseEntity {
    @TableField("column_name")
    private String columnName;
}
```

## 安全与授权

### 权限字符串

格式：`{module}:{action}`

示例：
- `business:query` - 查询权限
- `business:create` - 创建权限
- `business:update` - 更新权限
- `business:delete` - 删除权限

### Controller中使用

```java
@PreAuthorize("hasAuthority('business:query')")
public Result<PageResult<BusinessEntity>> page(...) {
    // ...
}
```

## 生成工作流

调用时，遵循以下步骤：

1. **收集需求**
   - 询问用户实体名称和描述
   - 确定所需字段及其类型
   - 明确校验规则
   - 澄清业务逻辑需求

2. **生成数据库结构**
   - 创建符合规范的SQL DDL
   - 包含所有公共字段
   - 添加索引和约束

3. **生成实体类**
   - 继承BaseEntity
   - 添加所有字段映射
   - 包含正确的注解

4. **生成DTO**
   - 创建用于创建/更新的DTO
   - 创建用于分页查询的QueryDTO
   - 添加校验注解

5. **生成Mapper**
   - 创建Mapper接口
   - 继承BaseMapper
   - 按需添加自定义方法

6. **生成Service**
   - 创建Service接口
   - 实现包含完整CRUD的Service
   - 添加异常处理
   - 包含事务管理

7. **生成Controller**
   - 创建REST Controller
   - 添加所有CRUD端点
   - 包含授权控制
   - 添加API文档

8. **生成单元测试**
   - 创建测试类
   - 添加基本测试用例
   - 覆盖CRUD操作
   - 测试异常场景

9. **提供使用说明**
   - 解释如何集成
   - 展示API示例
   - 文档化配置需求

## 质量检查清单

在交付生成的代码前，验证：

- [ ] 实体类继承BaseEntity
- [ ] 所有字段都有@TableField注解
- [ ] 表名使用wva_yangzhong前缀
- [ ] DTO包含校验注解
- [ ] Service接口定义所有操作
- [ ] Service实现包含@Transactional
- [ ] 异常处理完整
- [ ] Controller使用@PreAuthorize
- [ ] API文档注解存在
- [ ] 遵循命名规范
- [ ] 正确使用Lombok注解
- [ ] 使用@RequiredArgsConstructor构造器注入
- [ ] 返回Result<T>包装器
- [ ] 分页使用PageResult
- [ ] 单元测试覆盖基本场景

## 示例

### 示例1：简单实体

**用户需求**："创建一个敏感事件管理模块，包含事件名称、事件类型、事件级别、发生时间、描述等字段"

**生成文件**：
1. `SensitiveEvent.java` - 实体类
2. `SensitiveEventDTO.java` - 创建/更新DTO
3. `SensitiveEventQueryDTO.java` - 查询DTO
4. `SensitiveEventMapper.java` - Mapper
5. `SensitiveEventService.java` - Service接口
6. `SensitiveEventServiceImpl.java` - Service实现
7. `SensitiveEventController.java` - Controller
8. `SensitiveEventServiceTest.java` - 单元测试

### 示例2：带关联的复杂实体

**用户需求**："创建人员管理模块，需要关联部门、角色，支持多对多关系"

**生成文件**：
- 所有基础文件外加：
- 关联实体类
- 用于复杂查询的自定义Mapper XML
- 用于响应数据的VO类
- 批量操作支持

## 注意事项

- 始终使用中文注释描述字段
- 遵循项目中现有的代码风格
- 尽可能使用Hutool工具类（StrUtil、DateUtil等）
- 优先使用MyBatis-Plus内置方法而非自定义SQL
- 使用LambdaQueryWrapper进行类型安全的查询
- 始终包含逻辑删除支持
- 添加正确的Javadoc，包含@author、@version、@date
- 使用`lombok.extern.slf4j.Slf4j`进行日志记录
- 不要在响应中暴露密码或敏感字段
