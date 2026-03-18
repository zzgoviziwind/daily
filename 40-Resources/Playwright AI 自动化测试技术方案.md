# Playwright AI 自动化测试技术方案

> 基于 Playwright MCP Server 的意图驱动 UI 自动化测试方案
> 文档版本：1.0
> 生成时间：2026-03-14

---

## 一、方案概述

### 1.1 方案背景

传统的 UI 自动化测试面临以下痛点：
- **脚本维护成本高**：UI 元素变化导致脚本频繁失效
- **编写门槛高**：需要掌握编程技能和测试框架
- **执行结果不确定**：元素定位失败、超时等问题频发
- **与业务脱节**：测试脚本难以反映真实业务场景

本方案基于 **Playwright MCP Server**，采用**意图驱动**的测试自动化范式，通过自然语言描述测试意图，由大模型自动生成并执行测试脚本。

### 1.2 核心优势

| 传统自动化 | 意图驱动自动化 |
|-----------|---------------|
| 编写 XPath/CSS 选择器 | 描述"点击登录按钮" |
| 手动处理等待和重试 | 自动等待元素可交互 |
| 硬编码测试数据 | 动态生成测试数据 |
| 固定执行路径 | 智能适应 UI 变化 |

### 1.3 技术架构

```
┌─────────────────────────────────────────────────────────┐
│                    测试工程师                            │
│                   (自然语言描述)                          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│              大模型 (LLM)                                │
│   - 理解测试意图                                         │
│   - 规划测试步骤                                         │
│   - 调用 MCP 工具                                        │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│           Playwright MCP Server                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Tools:                                           │   │
│  │ - browser_navigate (导航)                         │   │
│  │ - browser_click (点击)                            │   │
│  │ - browser_fill (填写)                             │   │
│  │ - browser_snapshot (页面快照)                      │   │
│  │ - browser_assert (断言)                           │   │
│  └─────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│              Playwright 浏览器引擎                        │
│   - Chromium / Firefox / WebKit                        │
│   - 无头/有头模式                                        │
│   - 移动端模拟                                           │
└─────────────────────────────────────────────────────────┘
```

---

## 二、环境配置

### 2.1 基础环境要求

| 组件 | 版本要求 | 安装命令 |
|------|---------|---------|
| Node.js | v18+ | `node -v` |
| npm | v9+ | `npm -v` |
| Playwright | latest | `npm install -D @playwright/mcp` |
| Claude Code | latest | `npm install -g @anthropic/claude-code` |

### 2.2 安装 Playwright 浏览器

```bash
# 安装浏览器二进制文件
npx playwright install chromium

# 或者安装所有浏览器
npx playwright install
```

### 2.3 MCP 配置文件

在 Claude Code 配置文件中添加 Playwright MCP Server：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--headless"
      ]
    }
  }
}
```

### 2.4 高级配置（可选）

创建 `playwright-mcp-config.json`：

```json
{
  "browser": {
    "browserName": "chromium",
    "isolated": true,
    "launchOptions": {
      "headless": true,
      "channel": "chrome"
    },
    "contextOptions": {
      "viewport": {
        "width": 1280,
        "height": 720
      }
    }
  },
  "capabilities": ["core", "pdf", "vision"],
  "timeouts": {
    "action": 5000,
    "navigation": 60000
  },
  "network": {
    "allowedOrigins": ["https://your-app.com"]
  },
  "outputDir": "./playwright-output",
  "saveTrace": true,
  "saveVideo": {
    "width": 1280,
    "height": 720
  }
}
```

启动命令：
```bash
npx @playwright/mcp@latest --config playwright-mcp-config.json
```

---

## 三、核心测试能力

### 3.1 可用工具列表

| 工具名称 | 功能描述 | 使用场景 |
|---------|---------|---------|
| `browser_navigate` | 导航到指定 URL | 打开测试页面 |
| `browser_snapshot` | 获取页面 accessibility 快照 | 元素识别、页面分析 |
| `browser_click` | 点击页面元素 | 按钮点击、链接跳转 |
| `browser_fill` | 填写表单字段 | 输入用户名/密码 |
| `browser_type` | 模拟键盘输入 | 搜索框输入、富文本编辑 |
| `browser_select_option` | 选择下拉选项 | 选择国家/省份 |
| `browser_check` | 勾选复选框 | 同意条款、多选 |
| `browser_hover` | 悬停在元素上 | 触发下拉菜单 |
| `browser_drag` | 拖拽元素 | 文件上传、排序 |
| `browser_press_key` | 按键盘键 | Enter、Escape 等 |
| `browser_wait_for` | 等待条件满足 | 等待元素出现、等待文本 |
| `browser_evaluate` | 执行 JavaScript | 自定义逻辑、数据提取 |
| `browser_take_screenshot` | 截取屏幕截图 | 结果记录、缺陷复现 |

### 3.2 意图驱动测试示例

#### 示例 1：用户登录测试

**测试意图描述：**
```
测试用户登录功能：
1. 打开登录页面 https://example.com/login
2. 输入用户名 "test_user"
3. 输入密码 "test_pass"
4. 点击登录按钮
5. 验证跳转到仪表盘页面
```

**自动生成的执行步骤：**
```python
# 大模型根据意图自动规划并调用 MCP 工具

# 1. 导航到登录页面
await browser_navigate("https://example.com/login")

# 2. 获取页面快照，识别元素
snapshot = await browser_snapshot()
# 识别到用户名输入框 ref="username_input"
# 识别到密码输入框 ref="password_input"
# 识别到登录按钮 ref="login_button"

# 3. 填写用户名
await browser_fill(ref="username_input", text="test_user")

# 4. 填写密码
await browser_fill(ref="password_input", text="test_pass")

# 5. 点击登录按钮
await browser_click(ref="login_button")

# 6. 等待页面跳转
await browser_wait_for(text="仪表盘")

# 7. 验证当前 URL
assert page.url == "https://example.com/dashboard"
```

#### 示例 2：电商购物车测试

**测试意图描述：**
```
测试电商网站购物车功能：
1. 访问电商网站首页
2. 搜索商品 "无线鼠标"
3. 将第一个商品添加到购物车
4. 验证购物车中商品数量
5. 进入购物车页面
6. 修改商品数量为 2
7. 验证总价更新
```

**执行流程：**
```
导航 → 快照分析 → 填写搜索 → 点击搜索 → 快照分析 →
点击添加购物车 → 等待提示 → 验证数量 → 点击购物车图标 →
快照分析 → 修改数量 → 等待计算 → 验证总价
```

---

## 四、测试框架设计

### 4.1 测试用例结构

```yaml
TestCase:
  name: "用户登录测试"
  id: "TC-LOGIN-001"
  description: "验证用户使用有效凭据可以成功登录"

  preconditions:
    - 用户已注册
    - 测试账号可用

  steps:
    - action: "navigate"
      target: "https://example.com/login"
      expected: "页面加载完成"

    - action: "fill"
      target: "用户名输入框"
      value: "test_user"
      expected: "输入框显示 test_user"

    - action: "fill"
      target: "密码输入框"
      value: "test_pass"
      expected: "输入框显示为掩码"

    - action: "click"
      target: "登录按钮"
      expected: "跳转到仪表盘"

  assertions:
    - type: "url"
      expected: "https://example.com/dashboard"

    - type: "text"
      target: "欢迎信息"
      expected: "欢迎，test_user"
```

### 4.2 Gherkin 格式支持

```gherkin
Feature: 用户登录功能

  Scenario: 使用有效凭据登录成功
    Given 用户已打开登录页面 "https://example.com/login"
    When 用户输入用户名 "test_user"
    And 用户输入密码 "test_pass"
    And 用户点击登录按钮
    Then 用户应该跳转到仪表盘页面
    And 页面应该显示欢迎信息 "欢迎，test_user"

  Scenario: 使用无效凭据登录失败
    Given 用户已打开登录页面 "https://example.com/login"
    When 用户输入用户名 "invalid_user"
    And 用户输入密码 "wrong_pass"
    And 用户点击登录按钮
    Then 页面应该显示错误提示 "用户名或密码错误"
    And 用户应该停留在登录页面
```

### 4.3 测试数据管理

```python
# test_data.py
class TestDataSource:
    """测试数据源管理"""

    @staticmethod
    def get_valid_user():
        return {
            "username": "test_user",
            "password": "test_pass",
            "email": "test@example.com"
        }

    @staticmethod
    def get_invalid_user():
        return {
            "username": "invalid_user",
            "password": "wrong_pass"
        }

    @staticmethod
    def generate_random_user():
        import uuid
        return {
            "username": f"user_{uuid.uuid4().hex[:8]}",
            "password": "Test123!",
            "email": f"test_{uuid.uuid4().hex[:8]}@example.com"
        }
```

---

## 五、测试执行流程

### 5.1 完整执行流程图

```
┌─────────────┐
│  测试意图   │
│  (自然语言) │
└──────┬──────┘
       ↓
┌─────────────┐
│  意图解析   │
│  (LLM)      │
└──────┬──────┘
       ↓
┌─────────────┐
│  步骤规划   │
│  (Task Plan)│
└──────┬──────┘
       ↓
┌─────────────┐
│  工具调用   │
│  (MCP Tools)│
└──────┬──────┘
       ↓
┌─────────────┐
│  浏览器操作  │
│  (Playwright)│
└──────┬──────┘
       ↓
┌─────────────┐
│  结果验证   │
│  (Assertions)│
└──────┬──────┘
       ↓
┌─────────────┐
│  生成报告   │
│  (Report)   │
└─────────────┘
```

### 5.2 执行示例代码

```python
# test_runner.py
import asyncio
from playwright.async_api import async_playwright

class IntentDrivenTest:
    def __init__(self, intent_description):
        self.intent = intent_description
        self.page = None
        self.context = None

    async def setup(self):
        """初始化浏览器"""
        playwright = await async_playwright().start()
        browser = await playwright.chromium.launch(headless=True)
        self.context = await browser.new_context()
        self.page = await self.context.new_page()

    async def execute_intent(self):
        """执行测试意图"""
        # 调用 LLM 解析意图并生成执行计划
        plan = await self.llm_generate_plan(self.intent)

        # 执行每个步骤
        for step in plan:
            result = await self.execute_step(step)
            if not result.success:
                return result

        return TestResult(passed=True)

    async def execute_step(self, step):
        """执行单个步骤"""
        action = step['action']
        target = step.get('target')
        value = step.get('value')

        if action == 'navigate':
            await self.page.goto(target)
        elif action == 'click':
            await self.page.click(target)
        elif action == 'fill':
            await self.page.fill(target, value)
        elif action == 'assert_text':
            text = await self.page.text_content(target)
            assert value in text

        return StepResult(success=True)

    async def teardown(self):
        """清理资源"""
        await self.context.close()
```

---

## 六、测试报告与可视化

### 6.1 报告结构

```json
{
  "test_run_id": "TR-20260314-001",
  "test_case": "用户登录测试",
  "status": "passed",
  "duration_ms": 3420,
  "steps": [
    {
      "step": 1,
      "action": "navigate",
      "target": "https://example.com/login",
      "status": "passed",
      "duration_ms": 850,
      "screenshot": "screenshots/step_1.png"
    },
    {
      "step": 2,
      "action": "fill",
      "target": "username_input",
      "value": "test_user",
      "status": "passed",
      "duration_ms": 120,
      "screenshot": "screenshots/step_2.png"
    }
  ],
  "trace_file": "traces/TR-20260314-001.zip",
  "video_file": "videos/TR-20260314-001.webm"
}
```

### 6.2 Allure 报告集成

```python
# allure_integration.py
import allure

@allure.feature("用户登录")
@allure.story("有效凭据登录")
@allure.title("使用有效凭据登录成功")
@allure.severity(allure.severity_level.CRITICAL)
def test_login_with_valid_credentials():
    """测试使用有效凭据登录"""

    with allure.step("打开登录页面"):
        browser_navigate("https://example.com/login")

    with allure.step("输入用户名"):
        browser_fill(ref="username", text="test_user")

    with allure.step("输入密码"):
        browser_fill(ref="password", text="test_pass")

    with allure.step("点击登录按钮"):
        browser_click(ref="login_btn")

    with allure.step("验证跳转"):
        assert page.url.endswith("/dashboard")

    with allure.step("验证欢迎信息"):
        assert "欢迎" in page.content()
```

---

## 七、最佳实践

### 7.1 元素定位策略

| 优先级 | 定位方式 | 说明 |
|-------|---------|------|
| 1 | `aria-ref` | Playwright MCP 专用，基于 accessibility |
| 2 | `role + name` | 语义化定位，如 `button[name="提交"]` |
| 3 | `testid` | 开发预留的测试标识 |
| 4 | `CSS 选择器` | 备选方案 |
| 5 | `XPath` | 最后选择 |

### 7.2 等待策略

```python
# 推荐：使用智能等待
await browser_wait_for(text="欢迎")  # 等待文本出现
await browser_wait_for(time=2)       # 等待固定时间（不推荐）

# Playwright 内置智能等待
# - 元素可交互时自动执行
# - 自动重试直到超时
```

### 7.3 错误处理

```python
# 错误处理最佳实践
try:
    await browser_click(ref="submit_btn")
except TimeoutError:
    # 截取失败截图
    await browser_take_screenshot(filename="error.png")
    # 记录详细错误
    logs = await browser_console_messages()
    # 重新尝试或报告失败
    raise
```

### 7.4 数据驱动测试

```python
@pytest.mark.parametrize("username,password,expected", [
    ("valid_user", "valid_pass", "success"),
    ("invalid_user", "any_pass", "failure"),
    ("", "valid_pass", "failure"),
    ("valid_user", "", "failure"),
])
async def test_login(username, password, expected):
    """数据驱动的登录测试"""
    await browser_navigate("https://example.com/login")
    await browser_fill(ref="username", text=username)
    await browser_fill(ref="password", text=password)
    await browser_click(ref="login_btn")

    if expected == "success":
        await browser_wait_for(text="仪表盘")
    else:
        await browser_wait_for(text="登录失败")
```

---

## 八、CI/CD 集成

### 8.1 GitHub Actions 配置

```yaml
# .github/workflows/playwright-tests.yml
name: Playwright AI Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'

    - name: Install Playwright
      run: |
        npm install -D @playwright/mcp
        npx playwright install chromium

    - name: Run Tests
      run: |
        npx @playwright/mcp --headless
        # 或运行测试脚本
        python test_runner.py

    - name: Upload Test Results
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-output/
```

### 8.2 Docker 容器化

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/playwright/python:v1.40.0

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "test_runner.py"]
```

```bash
# 构建并运行
docker build -t playwright-ai-tests .
docker run --rm \
  -v $(pwd)/output:/app/output \
  playwright-ai-tests
```

---

## 九、常见问题与解决方案

### 9.1 元素定位失败

**问题：** MCP 无法找到目标元素

**解决方案：**
1. 使用 `browser_snapshot` 重新获取页面状态
2. 检查元素是否在视口内，需要时滚动
3. 等待元素加载完成
4. 尝试使用 role + name 组合定位

### 9.2 执行超时

**问题：** 操作超时导致测试失败

**解决方案：**
1. 增加超时时间配置
2. 检查网络请求是否被阻塞
3. 使用 `browser_wait_for` 等待前置条件
4. 添加重试机制

### 9.3 动态内容处理

**问题：** 页面内容动态加载导致识别失败

**解决方案：**
```python
# 等待动态内容加载
await browser_wait_for(text="加载完成")

# 或者使用轮询
for i in range(5):
    snapshot = await browser_snapshot()
    if "目标内容" in snapshot:
        break
    await asyncio.sleep(1)
```

### 9.4 认证状态保持

**问题：** 每次测试都需要重新登录

**解决方案：**
```json
{
  "browser": {
    "userDataDir": "./browser-profile",
    "contextOptions": {
      "storageState": "./auth-state.json"
    }
  }
}
```

---

## 十、方案评估

### 10.1 适用场景

| 场景 | 适用度 | 说明 |
|------|-------|------|
| **回归测试** | ⭐⭐⭐⭐⭐ | 重复执行，高 ROI |
| **冒烟测试** | ⭐⭐⭐⭐⭐ | 快速验证核心功能 |
| **探索性测试** | ⭐⭐⭐⭐ | AI 可自动发现边缘场景 |
| **视觉测试** | ⭐⭐⭐⭐ | 结合截图对比 |
| **性能测试** | ⭐⭐ | 需要额外工具支持 |
| **安全测试** | ⭐⭐ | 需要专业工具配合 |

### 10.2 预期收益

| 指标 | 改进前 | 改进后 | 提升 |
|------|-------|-------|------|
| 测试编写时间 | 4 小时/用例 | 0.5 小时/用例 | 88%↓ |
| 测试维护成本 | 高 | 低 | 70%↓ |
| 测试覆盖率 | 60% | 85% | 42%↑ |
| 缺陷发现率 | 基准 | +35% | 35%↑ |

### 10.3 投资回报分析

```
年度收益计算：

传统自动化：
- 脚本编写：100 用例 × 4 小时 × ¥200/小时 = ¥80,000
- 脚本维护：100 用例 × 1 小时/月 × 12 月 × ¥200/小时 = ¥240,000
- 总计：¥320,000/年

意图驱动自动化：
- 脚本编写：100 用例 × 0.5 小时 × ¥200/小时 = ¥10,000
- 脚本维护：100 用例 × 0.3 小时/月 × 12 月 × ¥200/小时 = ¥72,000
- MCP 工具成本：¥20,000/年
- 总计：¥102,000/年

年度节省：¥320,000 - ¥102,000 = ¥218,000
ROI: 216%
```

---

## 十一、总结与建议

### 11.1 核心优势总结

1. **降低门槛**：自然语言描述测试意图，无需编程技能
2. **提高效率**：自动生成测试步骤，减少手工编写时间
3. **增强覆盖**：AI 可发现人工容易忽略的边缘场景
4. **易于维护**：意图描述稳定，不随 UI 变化而失效

### 11.2 实施建议

1. **从小做起**：选择 1-2 个核心功能开始试点
2. **逐步扩展**：验证效果后逐步扩大应用范围
3. **人员培训**：培训测试人员掌握意图描述技巧
4. **持续优化**：收集反馈，不断优化提示词和执行策略

### 11.3 技术路线图

```
Phase 1 (1-2 月): 环境搭建，试点项目
         ↓
Phase 2 (3-4 月): 核心功能覆盖，流程标准化
         ↓
Phase 3 (5-6 月): 全面推广，CI/CD 集成
         ↓
Phase 4 (7-12 月): 智能化提升，预测性测试
```

---

*文档结束*
