
```makedown
# Vue 企业级编码规范
## 命名规范
- 组件：大驼峰 PascalCase
- 页面组件：XXXPage
- 公共组件：XXXCom
- 方法：小驼峰，动词开头

## 结构顺序
- defineProps / defineEmits
- 变量
- computed
- watch
- 方法
- 生命周期

## 格式规范
- 缩进 2 空格
- 标签属性换行
- 单文件 ≤ 300 行
- 方法 ≤ 80 行
- 样式必须 scoped

## 业务规范
- template 无复杂表达式
- API 统一管理
- 禁止直接操作 DOM
- async/await 异步
- 全局 loading、错误处理
```