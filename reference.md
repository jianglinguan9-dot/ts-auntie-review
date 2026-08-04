# TS 审查参考规则

本文档为居委会大妈审 TS 的详细参考规则。核心审查用 SKILL.md 中的规则即可，遇到模糊 case 时查阅本文档。

## 类型安全详细规则

### any 的替代方案

| 场景 | 替代方案 |
|---|---|
| 不确定外部数据结构 | `unknown` + 类型守卫 |
| 第三方库无类型定义 | 写 `.d.ts` 声明文件 |
| 临时调试需要 any | 标记 `// TODO: 临时 any，调试后移除` |
| 函数参数需要接受任意类型 | 泛型 `<T>` 或 `unknown` |

### as 断言的合理 vs 滥用

可以接受但不够好（`as` 跳过编译期检查，无运行时验证）：
```typescript
// JSON 解析后断言——虽然常见，但没有运行时验证
const data = JSON.parse(str) as MyInterface
```

滥用：
```typescript
// 没有任何运行时检查就强制断言
const user = response as User  // 如果 response 是 any，应该先验证
```

正确做法（用类型守卫）：
```typescript
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'id' in obj && 'name' in obj
}

const data = JSON.parse(str)
if (isUser(data)) {
  // 这里 data 自动收窄为 User
}
```

### satisfies 操作符（TS 4.9+，推荐替代 as）

`satisfies` 会在编译期检查值是否符合类型，同时保留最精确的字面量类型。与 `as` 不同，`satisfies` 不会跳过检查——如果值不匹配类型，编译器会报错。

```typescript
// as：跳过检查，不安全的值也不报错
const color1 = "purple" as Color  // 无错误，但 "purple" 不在 Color 中

// satisfies：编译期验证，不匹配会报错
const color2 = "purple" satisfies Color  // Error: "purple" 不可赋值给 Color

// 实用场景：配置对象验证，保留字面量类型
const config = {
  environment: "development",
  port: 3000,
} satisfies Config  // 验证通过，且 environment 类型保留为 "development" 字面量
```

**优先级**：`satisfies` > 类型守卫 > `as`。只有在无法用前两者的场景（如 DOM 查询 `document.getElementById('app') as HTMLDivElement`）才用 `as`。

### 非空断言 `!` 的判断标准

- 合理：DOM 查询后确定存在的元素 `document.getElementById('app')!`（虽然最好还是检查）
- 滥用：`user!.profile!.address!.city` 连续非空断言，任何一层为 null 都会运行时崩溃
- 替代：可选链 `user?.profile?.address?.city ?? '未知'`

## 命名规范详细规则

### 驼峰命名法

| 类型 | 命名风格 | 示例 |
|---|---|---|
| 变量、函数 | camelCase | `userName`、`getUserId` |
| 类、接口、类型别名 | PascalCase | `UserService`、`User` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 枚举成员 | PascalCase 或 UPPER_SNAKE_CASE | `UserRole.Admin` 或 `UserRole.ADMIN` |
| 私有成员 | 前缀下划线（可选） | `_internalState` |
| 布尔变量/属性 | is/has/can/should 前缀 | `isLoading`、`hasPermission` |

### 接口命名

- 不推荐 `I` 前缀：`IUser` → `User`
- 例外：如果项目已有 `I` 前缀惯例，保持一致比改风格更重要

## 结构复杂度详细规则

### 嵌套深度阈值

| 深度 | 建议 |
|---|---|
| 1-2 层 | 正常 |
| 3 层 | 可以接受，建议考虑扁平化 |
| 4+ 层 | 需要重构：提前 return、提取子函数、用数组方法替代循环 |

### 函数行数阈值

| 行数 | 建议 |
|---|---|
| ≤ 20 行 | 理想 |
| 20-50 行 | 可接受 |
| 50-100 行 | 需要拆分 |
| > 100 行 | 必须拆分 |

### 圈复杂度参考

| 分数 | 级别 |
|---|---|
| 1-5 | 低复杂度，好 |
| 6-10 | 一般，可接受 |
| 11-15 | 偏高，建议重构 |
| 16+ | 高复杂度，必须重构 |

## 边界与安全详细规则

### null/undefined 处理决策树

1. 值可能为 null/undefined 吗？
   - 否 → 不需要处理
   - 是 → 进入 2
2. 在哪一层处理最合理？
   - 调用方 → 用可选链 `?.` + 空值合并 `??`
   - 函数内部 → 在入口处 early return
   - 类型定义 → 用可选属性 `field?: type` 而非 `field: type | undefined`

### 异常处理标准

```typescript
// 好的做法
try {
  await operation()
} catch (error) {
  if (error instanceof SpecificError) {
    // 处理已知错误
    logger.error('操作失败', error)
    throw new UserFriendlyError('操作失败，请重试')
  }
  throw error // 未知错误往上抛
}

// 坏的做法
try {
  await operation()
} catch (e) {
  // 静默吞掉，什么都没做
}

// 更坏
try {
  await operation()
} catch (e) {
  console.log(e)  // 只打了个 log 就不管了
}
```

## 死代码检测详细规则

### 废弃函数

- 函数被定义但在当前文件和项目的可见范围内没有被调用
- 被注释掉的函数调用（不算被"使用"）
- 只在测试中使用的导出函数：不算死代码，但建议标注 `// @internal`

### 无用 import

```typescript
// 死代码
import { unusedFunc, usedFunc } from 'lib'  // unusedFunc 从未使用

// 可被 tree-shaking 的类型 import
import type { UserType } from 'types'  // 如果只用做类型标注，用 import type
```

### 不可达代码

```typescript
function getStatus(code: number): string {
  if (code === 200) return 'ok'
  if (code === 404) return 'not found'
  return 'error'
  console.log('这行永远不会执行')  // 不可达代码
}
```

### 注释代码块

- 超过 5 行的被注释代码应删除，git 历史会保留
- 1-2 行的临时注释可以接受

## TS 专属习俗详细规则

### type vs interface 决策

| 场景 | 推荐 |
|---|---|
| 对象类型 | `interface`（可扩展、可合并） |
| 联合类型 | `type`（interface 不支持） |
| 工具类型/映射类型 | `type` |
| 需要声明合并 | `interface` |
| 元组类型 | `type` |

一致性原则：同一项目中，同类型定义应统一使用 `type` 或 `interface`。

### enum 的问题

所有 enum（除 `const enum`）的共性问题：
1. 生成了运行时代码（增加 bundle 体积），`const enum` 会被内联但有一些坑
2. 字符串 enum 和数字 enum 都会生成运行时对象

数字 enum 专属问题：
1. 数字 enum 的 0 是 falsy，`if (role)` 判断会出错（`Active = 0` 被判为"没有"）
2. 数字 enum 可以反向映射 `UserRole[0]` → `"Admin"`，容易意外触发

> **TS 5.8 新增**：`--erasableSyntaxOnly` flag 会直接禁止 `enum` 声明和构造器参数属性（`constructor(public x: number)`）。如果项目开了此 flag，enum 会被完全禁止，应改用联合类型或 `as const`。

替代方案：
```typescript
// 联合类型（推荐）
type UserRole = 'admin' | 'user' | 'guest'

// 字符串 enum（次选，如果需要枚举对象）
enum UserRole {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST',
}

// as const（最轻量）
const UserRole = {
  Admin: 'ADMIN',
  User: 'USER',
  Guest: 'GUEST',
} as const
type UserRole = typeof UserRole[keyof typeof UserRole]
```

### readonly 使用场景

- 配置对象：`readonly config: Config`
- 函数参数中不应被修改的对象
- React 组件的 props（React.PropsWithChildren 默认 readonly）
- 类中不应在外部修改的属性：`private readonly`

### import type（TS 4.9+）

只用于类型标注的 import 应使用 `import type`，编译后会被完全移除，不产生运行时代码。

```typescript
// 好：编译后完全移除
import type { User, Product } from './types'

// 坏：混合导入，User 是类型但会保留在运行时
import { User, ProductService } from './types'
```

混合导入时可以用内联 `type` 修饰符：
```typescript
import { ProductService, type User } from './types'
```

### 泛型约束

```typescript
// 无约束（坏）——T 可以是任何东西，没有类型安全
function getProp<T>(obj: T, key: string) {
  return (obj as any)[key]
}

// 有约束（好）——T 必须是对象，key 必须是 T 的键
function getProp<T extends Record<string, unknown>>(obj: T, key: keyof T) {
  return obj[key]
}
```
