
```makedown
# ElementPlus 编码规范（全局自动生效）
## 1. 组件使用规范
- 所有弹窗必须加 destroy-on-close
- 所有表单必须加 validate 校验
- 所有表格必须加 border、size="default"
- 所有按钮必须加 type="primary"/"default"/"danger"

## 2. 表单规范
- 输入框：clearable
- 下拉框：filterable、clearable
- 日期选择器：value-format、format
- 开关：v-model 绑定 boolean
- 表单禁用统一用 :disabled

## 3. 表格规范
- 序号列固定
- 操作列固定右侧
- 长文本省略 show-overflow-tooltip
- 分页必须显示总条数 layout="total, sizes, prev, pager, next, jumper"

## 4. 弹窗规范
- 标题统一：新增/编辑/详情
- 宽度合理：600px / 800px / 1000px
- 底部居中：取消、确定
- 关闭前清空表单

## 5. 样式规范
- scss 统一 scoped
- 组件间距 8px / 12px / 16px
- 表单 label-width 100px/120px
- 页面容器内边距 16px

## 6. 行为规范
- 提交前校验
- 提交后关闭弹窗
- 删除前提示
- 异步操作 loading
- 操作成功提示 ElMessage.success
```