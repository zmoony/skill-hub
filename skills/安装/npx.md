### 常见
```bash
npx skills add obra/superpowers planning-with-files vue-best-practices subagent-driven-development -g -f
npx skills add https://github.com/othmanadi/planning-with-files --skill planning-with-files -g
npx skills add  skill-creator findskill -g -f

npx skills list --all
npx skills list           # 列出所有技

```

### 查看已安装的技能
```bash
npx skills list           # 列出所有技能
npx skills list -g        # 只看全局技能
npx skills list -a claude-code  # 只看指定助手

```

### 搜索技能
```bash
npx skills find           # 交互式搜索
npx skills find react     # 按关键词搜索

```

### 更新技能
```bash
npx skills update                 # 更新所有技能
npx skills update frontend-design # 更新指定技能
npx skills update -g              # 只更新全局技能
npx skills check                  # 检查是否有更新

```

### 卸载技能
```bash
npx skills remove                      # 交互式选择卸载
npx skills remove frontend-design      # 卸载指定技能
npx skills remove frontend-design -g   # 从全局范围卸载
npx skills remove --all                 # 卸载所有技能
npx skills rm frontend-design           # 简写形式

```
