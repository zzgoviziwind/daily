# AI 重塑测试开发系统实践 - 详细拓展版总结

> 课程：极客时间《AI 重塑测试开发系统实践》
> 讲师：陈磊（前京东测试架构师，阿里云、华为云 MVP）
> 共 18 讲，已完结

---

## 第一部分：大模型基础篇（01-03 讲）

### 第 01 讲：读懂 Attention：测试工程师驾驭 LLM 的第一步

#### 1.1 Attention 机制详解

Attention（注意力）机制是大模型理解上下文的核心，它让模型能够动态关注输入序列中最重要的部分。

**Self-Attention 工作原理：**
```
输入序列 → Query/Key/Value 向量 → 注意力分数 → 加权求和 → 输出
```

对测试工程师的启示：大模型不是死记硬背，而是理解上下文关系，这解释了为什么它能处理未见过的测试场景。

#### 1.2 Tokens 计算与成本估算

| 语言 | 1 token 约等于 |
|------|--------------|
| 英文 | 0.75 单词 ≈ 4 字符 |
| 中文 | 1-2 字符 |
| 混合 | 2-3 字符 |

**实际案例：**
```
1000 字的中文测试用例 ≈ 500-1000 tokens
一次 API 测试请求 + 响应 ≈ 200-500 tokens
生成完整测试报告 ≈ 2000-5000 tokens
```

#### 1.3 超参数调优指南

| 参数 | 作用 | 测试场景推荐值 |
|------|------|---------------|
| **temperature** | 控制输出创造性 | 0.1-0.3（测试需要确定性） |
| **top_k** | 采样候选词数量 | 40-50 |
| **top_p** | 累积概率阈值 | 0.8-0.9 |
| **max_tokens** | 最大输出长度 | 根据用例复杂度设定 |

---

### 第 02 讲：写好提示词：精准唤醒大模型测试能力

#### 2.1 提示词 4 要素框架

```markdown
# 你是谁（角色设定）
你是一名从业 15 年的资深测试工程师

# 你在哪（上下文）
正在为一个电商平台的订单模块设计测试用例

# 要干什么（任务目标）
根据需求文档生成覆盖正常/异常/边界场景的测试用例

# 怎么告诉我（输出格式）
以表格形式输出，包含：用例编号、测试场景、前置条件、操作步骤、预期结果
```

#### 2.2 思维链（Chain of Thought）实战

**等价类测试用例生成 Prompt：**
```python
system_message = """你是一名资深测试工程师，下面你会用等价类测试用例设计方法设计测试用例
请根据下面的业务描述设计接口参数的入参：
- 有效等价类：满足业务规则的输入
- 无效等价类：违反业务规则的输入
- 边界值：最小值、最大值、空值"""
```

#### 2.3 角色催眠技巧

通过强化角色设定，让大模型表现更专业：

```
❌ 普通提示："生成测试用例"
✅ 催眠提示："你是一名专注于金融系统测试的资深 QA，曾主导过多个亿元级交易系统的测试设计，现在请..."
```

---

### 第 03 讲：精准评估大模型：量化表现的关键指标

#### 3.1 大模型评估维度

| 维度 | 指标 | 说明 |
|------|------|------|
| **准确性** | Exact Match | 完全匹配率 |
| **相关性** | ROUGE-L | 内容覆盖度 |
| **流畅度** | Perplexity | 语言流畅程度 |
| **多样性** | BLEU | 词汇变化度 |

#### 3.2 ROUGE-L 详解

ROUGE-L 基于最长公共子序列（LCS）评估生成内容与参考答案的相似度：

```
ROUGE-L = LCS(生成内容，参考答案) / 参考答案长度
```

**测试场景应用：** 评估 AI 生成的测试用例是否覆盖了需求文档中的关键点

#### 3.3 BLEU 详解

BLEU 通过计算 n-gram 精度评估生成内容的多样性：

```
BLEU = BP × exp(Σwₙlogpₙ)
```

**测试场景应用：** 评估 Query 改写后的多样性，BLEU 越低表示变化越多

---

## 第二部分：RAG 与 MCP 进阶篇（04-08 讲）

### 第 04 讲：让 RAG 更聪明：掌握分块的"核心刀法"

#### 4.1 RAG 系统架构

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  用户 Query  │ ──→ │  检索模块   │ ──→ │  向量数据库  │
└─────────────┘     └─────────────┘     └─────────────┘
                            ↓                     ↓
                    ┌─────────────┐     ┌─────────────┐
                    │ Embedding   │     │   Chunks    │
                    │   Model     │     │  (分块)     │
                    └─────────────┘     └─────────────┘
                            ↓
                    ┌─────────────┐
                    │  大模型生成  │
                    │   回答      │
                    └─────────────┘
```

#### 4.2 8 种 Chunk 分割策略详解

| 策略 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **固定长度分割** | 均匀分布的文本 | 实现简单 | 可能切断语义 |
| **段落/句子分割** | 结构清晰的文档 | 保持语义完整 | 粒度不均 |
| **语义分割** | 高质量内容 | 语义连贯性好 | 计算成本高 |
| **递归字符分割** | 通用场景 | 平衡效率与质量 | 需要调参 |
| **代理分割** | 复杂文档 | 智能化高 | 依赖额外模型 |
| **标题感知分割** | 技术文档 | 结构对齐 | 依赖标题质量 |
| **表格感知分割** | 数据密集型 | 保持表格完整 | 处理复杂 |
| **代码感知分割** | 代码文档 | 保持语法完整 | 需语法解析 |

#### 4.3 分割策略选择建议

```python
def choose_chunk_strategy(doc_type, content_size):
    if doc_type == "API 文档":
        return "标题感知分割 + 代码感知分割"
    elif doc_type == "需求文档":
        return "段落分割 + 语义分割"
    elif doc_type == "测试用例":
        return "表格感知分割"
    elif content_size > 100000:
        return "递归字符分割"
    else:
        return "固定长度分割"
```

---

### 第 05 讲：超越 Function Call：MCP 为何更胜一筹？

#### 5.1 MCP vs Function Call

| 对比维度 | Function Call | MCP |
|---------|--------------|-----|
| **标准化** | 各厂商私有协议 | 统一开放协议 |
| **发现机制** | 硬编码 | 动态资源发现 |
| **传输方式** | HTTP 为主 | STDIO + SSE |
| **生态** | 封闭 | 开源社区驱动 |

#### 5.2 MCP 核心组件

**Tools（工具）：**
```json
{
  "name": "search_database",
  "description": "搜索测试用例数据库",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {"type": "string"}
    }
  }
}
```

**Resources（资源）：**
```python
@mcp.resource(
    uri="file:///project/test-cases.md",
    title="测试用例库",
    description="历史测试用例集合"
)
def get_test_cases():
    return read_file("test-cases.md")
```

**Prompts（提示词模板）：**
```python
@mcp.prompt()
def generate_test_case(requirement: str) -> str:
    return f"请为以下需求生成测试用例：{requirement}"
```

---

### 第 06 讲：测试提效 20%：需求条目化与 RAG 实践

#### 6.1 需求条目化流程

```
原始需求文档 → 需求解析 → 条目化存储 → 向量嵌入 → RAG 检索
```

#### 6.2 测试用例生成智能体架构

```python
class TestCaseGenerator:
    def __init__(self, rag_client, llm_client):
        self.rag = rag_client
        self.llm = llm_client

    def generate(self, requirement):
        # 1. 检索相似历史用例
        similar_cases = self.rag.search(requirement, top_k=5)

        # 2. 构建提示词
        prompt = self._build_prompt(requirement, similar_cases)

        # 3. 生成测试用例
        test_cases = self.llm.generate(prompt)

        # 4. 过滤与去重
        return self._filter(test_cases)
```

#### 6.3 实际效果数据

| 指标 | 改进前 | 改进后 | 提升 |
|------|-------|-------|------|
| 用例设计时间 | 4 小时/需求 | 1 小时/需求 | 75%↓ |
| 用例覆盖率 | 70% | 92% | 31%↑ |
| 缺陷逃逸率 | 15% | 8% | 47%↓ |

---

### 第 07 讲：告别手写接口测试：RAG+Text2SQL 自动生成

#### 7.1 系统架构

```
API 文档 → 文档解析 → 向量库
                      ↓
用户提问 → Text2SQL → 查询向量库 → 生成测试脚本
```

#### 7.2 Text2SQL 核心 Prompt

```sql
-- 将自然语言转换为测试脚本查询
输入："生成用户登录 API 的测试用例"
输出：
SELECT template, parameters
FROM api_test_templates
WHERE api_name = 'user_login'
  AND test_type IN ('success', 'failure', 'boundary')
ORDER BY coverage DESC
LIMIT 10
```

#### 7.3 生成的测试脚本示例

```python
@pytest.mark.parametrize("username,password,expected", [
    ("valid_user", "valid_pass", 200),
    ("invalid_user", "any_pass", 401),
    ("", "valid_pass", 400),
    ("valid_user", "", 400),
])
def test_user_login(username, password, expected):
    response = requests.post(
        "/api/login",
        json={"username": username, "password": password}
    )
    assert response.status_code == expected
```

---

### 第 08 讲：MCP 赋能测试：打造大模型的"智能手脚"

#### 8.1 MCP Server 开发流程

```python
from mcp.server import FastMCP

mcp = FastMCP("TestAssistant")

@mcp.tool()
def run_test_suite(suite_name: str) -> dict:
    """执行测试套件"""
    result = pytest.main([suite_name, "--json"])
    return parse_result(result)

@mcp.tool()
def get_coverage_report() -> str:
    """获取覆盖率报告"""
    return generate_coverage_html()

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

#### 8.2 配置示例

```json
{
  "mcpServers": {
    "test-assistant": {
      "command": "python",
      "args": ["mcp_test_server.py"],
      "env": {
        "TEST_ENV": "staging",
        "LOG_LEVEL": "INFO"
      }
    }
  }
}
```

---

## 第三部分：实战篇（09-15 讲）

### 第 09 讲：开源的 Playwright MCP Server：意图驱动测试自动化

#### 9.1 Playwright MCP 配置

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--headless",
        "--browser=chromium"
      ]
    }
  }
}
```

#### 9.2 意图驱动测试示例

```
用户指令："测试用户登录流程"

执行步骤：
1. 导航到登录页面
2. 填写用户名和密码
3. 点击登录按钮
4. 验证跳转成功
```

#### 9.3 测试脚本自动生成

```python
# 大模型根据意图生成的 Playwright 代码
async def test_login(page):
    await page.goto("https://example.com/login")
    await page.fill("#username", "test_user")
    await page.fill("#password", "test_pass")
    await page.click("#login-btn")
    await expect(page).to_have_url("https://example.com/dashboard")
```

---

### 第 10 讲：解锁测试新姿势：打造你的 MCP Server

#### 10.1 从 API 到 MCP Server 的转换

```python
# 输入：API 定义
# 输出：MCP Server 代码

import requests
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("api-wrapper")

@mcp.tool()
async def get_user_info(user_id: int) -> dict:
    """获取用户信息"""
    response = requests.get(f"https://api.example.com/users/{user_id}")
    return response.json()

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

#### 10.2 API 转 MCP Server 提示词

详见课程中的完整 Prompt 设计，包含：
- 角色设定（资深 Python 开发工程师）
- MCP 协议规范
- 错误处理要求
- 日志记录规范

---

### 第 11 讲：5 种 Agentic 模式赋能自动化测试

#### 11.1 反思（Reflexion）

```python
def reflexion_loop(task, max_iterations=3):
    for i in range(max_iterations):
        result = agent.execute(task)
        feedback = evaluate(result)
        if feedback.passed:
            return result
        task = refine_task(task, feedback)
    return result
```

**测试场景：** 测试用例质量迭代优化

#### 11.2 提示链（Chain of Thought）

```python
def chain_of_thought(requirement):
    # 步骤 1：分析需求
    analysis = llm("分析以下需求的测试点：" + requirement)

    # 步骤 2：设计测试场景
    scenarios = llm("基于分析设计测试场景：" + analysis)

    # 步骤 3：生成测试用例
    test_cases = llm("生成详细测试用例：" + scenarios)

    return test_cases
```

**测试场景：** 复杂需求的分步测试分析

#### 11.3 规划（Planning）

```python
def planning_agent(test_request):
    plan = llm("制定测试计划：" + test_request)
    tasks = parse_plan(plan)

    for task in tasks:
        execute_task(task)

    return generate_report()
```

**测试场景：** 测试任务分解与排期

#### 11.4 并行处理（Parallelization）

```python
from concurrent.futures import ThreadPoolExecutor

def parallel_test_execution(test_cases):
    with ThreadPoolExecutor(max_workers=5) as executor:
        results = list(executor.map(run_test, test_cases))
    return analyze_results(results)
```

**测试场景：** 多浏览器并发测试

#### 11.5 路由（Routing）

```python
def routing_agent(request):
    if "UI" in request:
        return ui_test_agent(request)
    elif "API" in request:
        return api_test_agent(request)
    elif "性能" in request:
        return performance_test_agent(request)
    else:
        return general_test_agent(request)
```

**测试场景：** 按模块分发测试任务

---

### 第 12 讲：借力打力：大模型生成 QA Pairs 提升 RAG 应用测试效率

#### 12.1 QA Pairs 生成流水线

```
文档输入 → 分块处理 → LLM 生成 → 过滤验证 → JSON 输出
```

#### 12.2 LangChain QAGenerationChain

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

PROMPT_TEMPLATE = """
You are an expert assistant tasked with generating question-and-answer pairs from a given text.
Based on the following text, please generate a list of QA pairs.
Output should be a valid JSON object containing a "qa_pairs" key.

Text:
---
{text}
---
"""

class QAGenerator:
    def __init__(self, llm):
        prompt = ChatPromptTemplate.from_template(PROMPT_TEMPLATE)
        parser = JsonOutputParser()
        self.chain = prompt | llm | parser

    def generate(self, doc_path):
        docs = load_document(doc_path)
        chunks = self._split_documents(docs)

        qa_pairs = []
        for chunk in chunks:
            result = self.chain.invoke({"text": chunk.page_content})
            qa_pairs.extend(result["qa_pairs"])

        return qa_pairs
```

#### 12.3 质量验证机制

```python
def validate_qa_pairs(qa_pairs, threshold=0.7):
    validated = []
    for qa in qa_pairs:
        # 语义相似度检查
        similarity = calculate_similarity(qa['question'], qa['answer'])
        if similarity > threshold:
            validated.append(qa)
    return validated
```

---

### 第 13 讲：Query 改写：大模型测试的数据倍增器

#### 13.1 三种改写方法对比

| 方法 | 原理 | 优点 | 缺点 | 适用场景 |
|------|------|------|------|---------|
| **基于大模型** | LLM 生成变体 | 创造性高 | 可能幻觉 | 探索性测试 |
| **词汇表改写** | 规则替换 | 稳定可控 | 缺乏灵活 | 医疗/法律 |
| **同义词改写** | LLM+ 规则 | 平衡灵活与稳定 | 实现复杂 | 通用场景 |

#### 13.2 基于大模型的改写代码

```python
class LLMRewriter:
    def __init__(self, llm):
        self.llm = llm
        self.system_prompt = """
你是一名从业 20 年的资深测试开发专家，专注于 NLP、LLM 相关技术测试。
请分析用户输入的测试数据，改写并返回多样化的 query 变体。
返回格式：JSON list，包含 10 条新 query。
"""

    def rewrite(self, query, reference):
        prompt = f"{self.system_prompt}\n\n{{\"query\":\"{query}\",\"reference\":\"{reference}\"}}"
        response = self.llm.invoke(prompt)
        return parse_json(response)
```

#### 13.3 词汇表改写代码

```python
import itertools
import jieba

class GlossaryRewriter:
    def __init__(self, glossary, max_combos=100):
        self.glossary = glossary  # [["买", "购买", "选购"], ["手机", "电话"]]
        self.max_combos = max_combos

    def rewrite(self, query):
        words = list(jieba.cut(query))
        synonym_lists = [self._get_synonyms(w) for w in words]
        combos = list(itertools.product(*synonym_lists))[:self.max_combos]
        return ["".join(combo) for combo in combos]
```

#### 13.4 ROUGE-L + BLEU 归一化验证

```python
def rouge_l_bleu_normalized(rewritten_queries, original_query, rouge_weight=0.7):
    """
    ROUGE-L 越高越好（语义相似度）
    BLEU 越低越好（词汇差异度），使用 (1-BLEU) 使其越高越好
    """
    scored_queries = []
    for rq in rewritten_queries:
        rouge_l = calculate_rouge_l(rq, original_query)
        bleu = calculate_bleu(rq, original_query)
        score = rouge_weight * rouge_l + (1 - rouge_weight) * (1 - bleu)
        scored_queries.append((score, rq))

    return max(scored_queries, key=lambda x: x[0])[1]
```

---

### 第 14 讲：AI 赋能测试左移：用大模型下好需求"先手棋"

#### 14.1 测试左移核心实践

```
传统流程：需求 → 开发 → 测试 → 发布
                    ↑
               测试介入太晚

左移流程：需求 → 开发 → 测试 → 发布
          ↑
     测试从需求开始介入
```

#### 14.2 开卡（KO）与验卡（DC）

**开卡（Kick Off）流程：**
```
1. Scrum Master 将需求移入待开发列表
2. 大模型自动补充验收条件
3. 测试/产品/开发三方评审
4. 补充验收条件
5. 进入开发
```

**验卡（Desk Check）流程：**
```
1. 开发完成功能
2. 测试/产品/开发三方验收
3. 按验收条件逐一验证
4. 通过后进入测试环节
```

#### 14.3 测试左移智能体 Prompt

```python
def 测试左移专家 ():
    """
    你是一名从业 15 年的软件测试专家，专注于测试左移实践。
    从合理性、可行性、兼容性、可测试性四个维度补齐条目化需求。
    """
    补齐原则 = [
        "明确：需求条目必须清晰易懂",
        "具体：提供详细的补充描述",
        "可测：每条目应包含可验证的验收条件",
        "一致：条目间无矛盾",
        "完整：覆盖功能性、非功能性、性能和安全需求"
    ]

# 输出格式
"""
需求名称：用户登录后查看订单列表
AC1：
  - Given：用户已成功登录
  - When：用户点击"订单"菜单
  - Then：系统显示订单列表页面
AC2：
  - Given：用户未登录
  - When：用户尝试访问订单页面
  - Then：系统跳转到登录页面
...
"""
```

#### 14.4 大模型友好度评估

| 维度 | 评估标准 | 友好信号 |
|------|---------|---------|
| 模块化程度 | 能否拆解为独立任务 | 可细分为 3-5 个子任务 |
| 遗留系统依赖性 | 是否依赖老旧系统 | 无 COBOL 等老旧系统依赖 |
| 迭代频率 | 更新频率 | 每周更新适合 AI 辅助 |

---

### 第 15 讲：从狂热到理性：大模型在测试内部落地的实战复盘

#### 15.1 组织推广三阶段模型

```
        采纳率
          ↑
    60% ──┤     开悟之坡
          │    ╱
          │   ╱
          │  ╱
          │ ╱          绝望之谷
          │╱    ╲______╱
          │     ╲
          │      ╲
          │       ╲
          │        ╲
    0% ───┴─────────╲───────→ 时间
          愚昧之巅
```

#### 15.2 各阶段特征与应对

| 阶段 | 特征 | 应对策略 |
|------|------|---------|
| **愚昧之巅** | 热情高涨，快速采纳 | 简化演示，降低门槛 |
| **绝望之谷** | 期望落空，弃用 | 快速响应反馈，解决痛点 |
| **开悟之坡** | 理性使用，稳定收益 | 提供最佳实践，持续优化 |

#### 15.3 禀赋效应应用

> 人们一旦拥有某物就会高估其价值

**应用策略：**
1. 让团队成员"拥有"自己的智能体/MCP Server
2. 鼓励在内部论坛推广自己的作品
3. 利用成就感驱动持续优化

#### 15.4 MCP Server 封装避坑指南

| 问题 | 错误做法 | 正确做法 |
|------|---------|---------|
| **API 暴露** | 直接将平台 API 转为 MCP | 整合相关 API，二次加工 |
| **返回量大** | 返回全部日志 | 只返回错误部分 |
| **命名随意** | `tool_1`, `api_handler` | `jira_search`, `browser_click` |
| **无语义内容** | 返回 `id=10001` | 过滤或转换为有意义文本 |
| **错误码** | `{errorCode: "90001"}` | `{errorMessage: "数据库超时"}` |

---

## 第四部分：核心观点与展望

### 测试工程师角色演进

```
缺陷猎手 → 风险守护者 → AI 测试架构师
   ↓            ↓              ↓
发现 Bug    预防风险      设计智能体
```

### 大模型与测试平台关系

> "大模型和软件测试结合的形态既不会淘汰测试平台，也不会淘汰测试智能体，而是相辅相成的关系。"

```
┌─────────────┐     ┌─────────────┐
│  测试平台   │ ←─→ │  测试智能体  │
│ (数据存储)  │     │ (执行任务)  │
└─────────────┘     └─────────────┘
        ↑                   ↓
        └───────────────────┘
           数据循环增强
```

### 未来展望

1. **零缺陷交付**：AI 驱动的预测性缺陷注入
2. **端到端自治**：测试流程全自动执行
3. **人机协作**：测试工程师聚焦高价值设计

---

*文档生成时间：2026-03-14*
