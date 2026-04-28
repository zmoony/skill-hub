
```makedown
# API 接口企业级规范
## 1. 请求规范
- GET：查询
- POST：新增
- PUT：修改
- DELETE：删除
- 接口路径使用短横线：/user/list

## 2. 参数规范
- GET 使用 @RequestParam
- POST/PUT 使用 @RequestBody
- 必须做参数校验：@NotBlank/@NotNull/@Length
- 分页参数固定：current、size

## 3. 返回规范
{
  "code": 200,    // 200成功 500异常
  "msg": "成功",
  "data": {}
}
- 成功统一 code=200
- 失败统一返回错误信息
- 列表返回带分页 total/records

## 4. 状态码规范
- 200 成功
- 400 参数错误
- 401 未登录
- 403 无权限
- 500 服务异常

## 5. 接口文档
- 所有接口必须加 @ApiOperation
- VO/DTO 必须加字段说明
- 必填字段标注清楚
```