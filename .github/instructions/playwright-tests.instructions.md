---
applyTo: 'tests/**'
description: 'Playwright 测试文件（tests/ 目录）的恒定约束：文件命名、分组上限、test 粒度、test.step 分步、POM 边界、超时常量、提交卫生。'
---

# 测试文件约定

## 文件与目录

- 测试文件一律以 `.spec.ts` 结尾；被 `setup` project 匹配的初始化文件用 `.setup.ts`
- 冒烟测试放在 `tests/smoke/`，目录不存在就创建

## 分组

- 一个 spec 文件对应 1 个页面 / 功能 / 业务流
- 顶层 `test.describe` **有且只有 1 个**
- 子 `test.describe` 0~3 个，超 3 个先试**换一个分组维度**（按角色改成按结果，或反之），换不动才拆文件
- 每个子 describe 含 2~8 个**逻辑** test，超 8 个先**再切一层场景维度**（「失败」拆成「校验失败 / 权限失败」），切不动才拆文件
- 成组的判据：组内 test 的**差异维度同一**（同一操作的成功/失败、同一操作在不同角色/权限下的表现）；「创建用户 / 删除用户 / 编辑用户」是并列流程，**不成组**，test 直接写在顶层
- 分组名笼统到能装下任何操作（「用户操作」「功能测试」）是并列流程的信号
- 子 describe 只在**提供场景信息**时存在：只含 1 个逻辑 test、名称与文件名 / 顶层 describe 同义、名称是「其他 / 杂项 / 补充」这类杂物容器 —— 这三种都不建这层分组，test 直接写在顶层
- 整个文件**逻辑** test 总数 ≤ 16，超出则沿场景维度拆新文件；参数化（`for` / `test.each`）若各 case 共享同一验收条件、仅数据不同，计为 1 个逻辑 test，两处上限同一口径
- `tests/smoke/` 下的 spec 可以只有 1 个 test

## test 粒度与分步

- 一个 `test` = 一个业务上可观察、**可独立失败**（一个不成立时，另一个仍可能成立）的最小验收条件及其操作集合；必然同生共死的断言留在同一个 test 里
- 每个 test 至少有一个断言（POM 的 `expectX()` 也算）；没有断言的不是 test 而是前置操作，并入真正做验收的那个 test 或搬进 `beforeEach`
- test 的**每一条执行路径**上都要有断言：`if` 包住断言时块外需有无条件兜底，否则把分支条件提到 test 层面拆成两个 test
- test 名写**验收结论**（「提交后订单出现在列表中」），不写操作（「点击提交按钮」）
- 每个测试步骤用 `test.step` 包裹，一个 step = 一个业务动作阶段，不是一行操作
- 关键步骤加中文注释，写**为什么这样断言 / 为什么是这个顺序**；删掉注释不会导致误解就不加，不复述 `click` / `fill` 本身
- test 之间不得有顺序依赖；共享前置搬进 `beforeEach` 或 fixture。仅当共享状态真的**不可重建**（外部系统、一次性资源）才用 `test.describe.serial`，并加注释写明为何不可重建

## 边界

- spec 内**不得出现** `page.locator(...)` / `getByRole(...)` / `getByTestId(...)` / CSS 选择器，元素访问一律经由 `src/pages/` 下的 POM
- 验收断言写在 spec，同步等待留在 POM 的 `waitForX()`（POM 形态见 `review-pom`）
- 不用 `page.waitForTimeout`
- 不写超时字面量，一律继承 `playwright.config.ts` 的全局预算；确需偏离时在 `src/utils/timeouts.ts` 新增具名常量并在注释中写明为何必须偏离（偏离值的唯一来源，`tests/timeouts.ts` 仅为兼容转发）
- 提交前清掉 `test.only`（它静默少跑其他 test，`forbidOnly` 只在 CI 生效，本地一路绿灯到合并）
- `test.skip` / `.fixme` 必须带一行注释说明原因与恢复条件

## 相关 skill

创建 spec → `create-spec`；审查 spec 组织 → `review-spec`；创建 POM → `create-pom`；审查 POM → `review-pom`。
