# Skills 与自动化

**用时**：1.5 小时 | **难度**：⭐⭐ 进阶

## 目标

创建可复用的 Skills（技能模块），为 Claude 提供领域专属知识。将重复问题的解决方案打包封装。

---

## 你将学到

- 什么是 Skills，以及它们为何强大
- 使用 SKILL.md 创建技能
- Skills 的前置元数据
- 自动调用 Skills
- 构建知识库
- 将 Skills 打包到项目中

---

## 什么是 Skills？

**Skills（技能模块）**是可复用的知识模块，用于教授 Claude 如何完成特定事项。

### 示例：测试技能

与其每次会话都解释你的测试方法，不如创建一个技能：

```Markdown
# 我们项目的测试最佳实践

## 框架：Jest

## 风格
- 描述性名称："should validate email with + symbols"
- Arrange-Act-Assert 模式
- Mock 外部依赖
- 测试行为，而非实现

## 覆盖率目标
最低 80%

```

现在，每当 Claude 协助测试时，它会读取此技能并遵循你的方法。

### Skills vs 智能体

| 方面 | Skills（技能模块） | 智能体 |
|-|-|-|
| 用途 | 传授知识 | 执行任务 |
| 范围 | 领域知识 | 专业工作流 |
| 持久性 | 每次会话记住 | 显式调用 |
| 自动调用 | 是（可选） | 仅手动 |
| 示例 | "我们如何写测试" | "测试写作智能体" |

---

## 创建你的第一个 Skill

Skills 是 `.claude/skills/` 中的 Markdown 文件。

### 基本结构

````Markdown
---
name: testing-standards
description: Our project's testing practices and conventions
triggers: [test, testing, jest, spec]
auto_invoke: false
keywords: [jest, unit-test, integration-test, mocking]
version: 1.0.0
---

# 测试规范

## 框架
Jest + @testing-library/react

## 文件组织
- 测试文件与源码同目录
- 命名：`[Component].test.tsx`
- 测试数据放 `__fixtures__/`
- Mock 放 `__mocks__/`

## 测试结构（AAA）
1. **Arrange（准备）**：设置测试数据
2. **Act（执行）**：调用函数/组件
3. **Assert（断言）**：检查结果

## 示例

```typescript
describe('validateEmail', () => {
  it('should accept valid email addresses', () => {
    // Arrange
    const email = 'user@example.com';

    // Act
    const result = validateEmail(email);

    // Assert
    expect(result).toBe(true);
  });
});

````

## 覆盖率要求

- 目标：最低 80%
- 关键路径：100%
- 覆盖类型：行覆盖、分支覆盖、函数覆盖

## Mock 策略

- 外部 API：使用 jest.mock()
- 数据库：使用测试 fixtures
- 计时器：使用 jest.useFakeTimers()

## 运行测试

```Bash
npm test                 # 运行所有测试
npm test -- --coverage   # 带覆盖率报告
npm test -- --watch      # 监听模式

```

```Plain Text

### 文件位置

```

my-project/└── .claude/└── skills/└── testing-standards.md

````Plain Text

---

## Skills 功能特性

### 触发词

当 Claude 看到特定关键词时自动调用该技能：

```markdown
---
triggers: [test, jest, spec, coverage, mock]
---

````

如果 Claude 看到"add tests to this function"，它会自动读取测试技能。

### 自动调用

```Markdown
---
auto_invoke: true
---

```

设为 `true` 时，Claude 在会话开始时加载该技能（无需你请求）。用于关键规则。

### 关键词

帮助 Claude 的搜索找到该技能：

```Markdown
---
keywords: [testing, jest, unit-test, mocking, assertions]
---

```

### 版本

追踪技能版本：

```Markdown
---
version: 1.0.0
---

```

技能有重大变更时更新版本号。

---

## 常见 Skills 模式

### 模式 1：编码规范

````Markdown
---
name: python-standards
triggers: [python, flask, django]
---

# Python 编码规范

## 风格
- PEP 8 合规（最大 100 字符）
- 所有函数加类型提示
- Google 格式文档字符串

## 测试
- pytest 用于单元测试
- 最低覆盖率 80%
- Mock 外部依赖

## 文件组织
src/
├── models/
├── services/
├── controllers/
└── tests/

## 导入
```python
# ✅ 好：精确导入
from models import User
from services.auth import authenticate

# ❌ 不好：通配符导入
from models import *

````

````Plain Text

### 模式 2：领域知识

```markdown
---
name: payment-processing
description: Payment system rules and edge cases
auto_invoke: true
---

# 支付处理规则

## PCI 合规
- 绝不记录卡号
- 使用 Tokenization（Stripe）
- 加密敏感数据
- 审计所有交易

## 常见问题
1. 部分收费：使用指数退避重试
2. 货币换算：始终保留 2 位小数
3. 时区处理：所有时间以 UTC 存储

## 边界情况
- 卡被拒绝：提供清晰的错误信息
- 卡过期：建议更新支付方式
- 3D Secure：处理验证流程

````

### 模式 3：流程文档

```Markdown
---
name: code-review-checklist
triggers: [review, pull request, pr]
---

# 代码审查清单

## 提交审查前
- [ ] 本地测试通过
- [ ] 没有 console.log 语句
- [ ] 代码中没有密钥
- [ ] 提交信息清晰

## 安全检查
- [ ] 无 SQL 注入漏洞
- [ ] 无 XSS 漏洞
- [ ] 无暴露的 API Key
- [ ] 输入已验证

## 性能
- [ ] 无 N+1 查询
- [ ] 无死循环
- [ ] 加载时间可接受

## 测试
- [ ] 已添加单元测试
- [ ] 集成测试已更新
- [ ] 覆盖率 >80%

```

---

## 打包 Skills

你可以将多个相关技能打包在一起。

### 项目技能包

```Plain Text
my-project/
└── .claude/
    └── skills/
        ├── testing-standards.md
        ├── api-design.md
        ├── database-patterns.md
        └── security-checklist.md

```

在 CLAUDE.md 中引用它们：

```Markdown
## 可用 Skills
我们的自定义技能会自动加载：
- **testing-standards**：我们如何写测试
- **api-design**：REST API 规范
- **database-patterns**：常用查询与迁移
- **security-checklist**：安全审查流程

```

### 向团队分发 Skills

要与团队共享技能，将其纳入 Git 版本控制：

```Bash
git add .claude/skills/
git commit -m "Add testing and API design skills"
git push

```

团队成员检出项目后自动获得这些技能。

---

## 练习：创建领域技能

### 场景

你在构建一个电商网站，希望 Claude 理解你的产品数据模型。

### 第一步：创建技能

```Bash
cat > .claude/skills/product-data-model.md << 'EOF'
---
name: product-data-model
description: E-commerce product data structure and rules
triggers: [product, catalog, sku, price, inventory]
auto_invoke: false
version: 1.0.0
---

# 产品数据模型

## 核心实体

### Product（产品）

```

{id: UUID,name: string,slug: string,  // URL 友好description: string,category_id: UUID,created_at: timestamp,updated_at: timestamp}

```Plain Text

### SKU（库存单位）

```

{id: UUID,product_id: UUID,sku: string,  // 如 "BLUE-XL-001"price: decimal,  // 始终保留 2 位小数cost: decimal,inventory: integer,weight: float,  // 单位：kg}

````Plain Text

### 库存规则
- 下单时减少库存
- 退货时增加库存
- 低库存提醒：< 5 件
- 补货水平：按产品设置

## 常用查询

### 获取产品及其所有 SKU
```sql
SELECT p.*, s.* 
FROM products p 
JOIN skus s ON p.id = s.product_id 
WHERE p.slug = ?

````

### 检查库存

```SQL
SELECT sum(inventory) FROM skus WHERE product_id = ?

```

## 边界情况

1. 缺货：返回 404 或"不可用"
2. 变体选择：按 SKU 显示价格
3. 价格变动：在 SKU 中更新，不在 Product 中更新EOF

````Plain Text

### 第二步：在 CLAUDE.md 中引用

```markdown
## Skills
- **product-data-model**：了解我们的产品结构

````

### 第三步：使用它

在会话中：

```Plain Text
Add a query to find all products with low inventory (< 5 units)

```

Claude 会：

1. 读取 product-data-model 技能
2. 理解你的数据库结构
3. 写出正确的 SQL

---

## 最佳实践

### 应该做

✅ 为重复性内容创建技能

✅ 保持技能专注（每个技能一个领域）

✅ 对技能进行版本控制

✅ 在技能中包含示例

✅ 需求变化时更新技能

✅ 与团队共享技能

### 不应该做

❌ 为一次性知识创建技能（改用 CLAUDE.md）

❌ 技能写得太长（> 500 行 = 拆分成多个技能）

❌ 用技能存放临时指令（改用 CLAUDE.md 或 AGENT.md）

❌ 把技能当作全面的文档

---

## Skills 生命周期

1. **创建**：识别重复模式或领域知识
2. **文档化**：用示例编写技能内容
3. **测试**：在会话中使用，验证 Claude 是否遵守
4. **精化**：根据反馈更新
5. **分享**：提交到 Git，供团队使用
6. **维护**：随实践演进持续更新

---

## 验证：以下都满足说明你已准备好

✓ 已创建至少一个自定义技能

✓ 理解触发词和自动调用

✓ 知道 Skills 和智能体的区别

✓ 能解释何时用技能 vs CLAUDE.md

✓ 已在真实会话中测试过你的技能

---

## 下一步

**模块 06：Hooks 与事件**涵盖：

- 自动化系统事件的响应
- 提交前验证
- 操作后通知
- 构建安全的自动化

这将教你如何在无需手动干预的情况下自动化重复任务。

---

**完成模块 05？** → 进入 模块 06：Hooks 与事件

---

> 来源：飞书 · AI Spark 知识库 ｜ 原文（最新版）：<https://lcnniolukk80.feishu.cn/wiki/GfnnwNIKWijnTfkO0ZJcdZEvnWh> ｜ 归档：2026-06-04
