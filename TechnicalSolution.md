# 🛠️ InfoGather Agent 技术方案书

## 1. 系统架构概览 (System Architecture)

系统采用 **“生产者-消费者” (Producer-Consumer)** 模型，配合 **微内核架构**。

* **输入端**：多源爬虫、RSS 订阅、API 接口。
* **处理端**：LLM（大语言模型）负责清洗、摘要、评分。
* **交互端**：微信接口（接收指令 + 推送日报）。

### 架构图 (Mermaid)

```mermaid
graph TD
    User((用户)) <-->|微信消息/指令| WeChat_Gateway[微信网关层]
    WeChat_Gateway -->|NLP意图识别| Command_Parser[指令解析器]
    Command_Parser -->|更新配置| User_DB[(用户配置数据库)]
    
    subgraph "核心处理引擎 (Core Engine)"
        Scheduler[定时任务调度器 APScheduler] -->|触发任务| Crawler_Manager[爬虫管理器]
        Crawler_Manager -->|抓取请求| Sources{数据源: RSS/Web/API}
        Sources -->|原始HTML/JSON| Cleaner[清洗模块]
        Cleaner -->|纯净文本| LLM_Processor[AI处理中心 (LLM)]
        
        LLM_Processor -->|1. 内容摘要| Content_Store[(内容库)]
        LLM_Processor -->|2. 质量评分| Content_Store
        LLM_Processor -->|3. 信任度校验| Content_Store
    end

    subgraph "投递服务 (Delivery Service)"
        Digest_Engine[日报生成引擎] -->|查询Top-N| Content_Store
        Digest_Engine -->|读取偏好| User_DB
        Digest_Engine -->|生成排版| Template_Renderer[模板渲染]
        Template_Renderer -->|发送请求| WeChat_Gateway
    end

```

---

## 2. 技术栈选型 (Tech Stack)

| 模块 | 推荐技术 | 理由 |
| --- | --- | --- |
| **编程语言** | **Python 3.10+** | 生态丰富，NLP与爬虫库最全。 |
| **Web 框架** | **FastAPI** | 高性能，易于处理微信 Webhook 回调。 |
| **数据库** | **PostgreSQL** (pgvector) | 存储用户关系，支持向量检索（未来用于语义去重）。 |
| **任务队列** | **Celery + Redis** | 异步处理爬虫任务和耗时较长的 LLM 推理。 |
| **爬虫工具** | **Crawlee** (Python版) 或 **Scrapy** | 现代化爬虫框架；对于动态网页使用 Playwright。 |
| **AI/NLP** | **OpenAI API / Claude 3.5** | 用于摘要、评分和意图识别（也可本地部署 Ollama/Llama3）。 |
| **微信接口** | **WeChatPy** 或 **Weco** | 封装好的微信 SDK，支持企业微信和公众号。 |

---

## 3. 核心功能模块详细设计

### 3.1 微信交互层 (Interaction Layer)

**目标**：解决“个人号易被封”的问题，同时实现自然语言交互。

* **接入方案建议**：
* **方案 A (推荐 - 稳定)**：**企业微信 (WeChat Work)** 自建应用。
* 优点：API 开放能力强，支持主动发消息，无封号风险，消息展示类似普通微信聊天。
* 实现：用户需加入企业（个人可注册企业微信），Agent 作为应用存在。


* **方案 B (实验 - 简单)**：**itchat-uos / wechaty** (基于 Pad 协议)。
* 优点：直接用个人微信号，体验最原生。
* 缺点：极高风险被微信封禁，不稳定。




* **NLP 指令解析**：
* 使用 LLM Function Calling 解析自然语言。
* *输入*："以后只要 AI 的新闻，别发政治的"
* *输出 JSON*：`{"action": "update_prefs", "add_topics": ["AI"], "block_topics": ["politics"]}`



### 3.2 智能采集层 (Smart Information Gathering)

**目标**：全网抓取，去噪。

* **数据源配置 (`trusted_sources.txt`)**：
* 支持 RSS Feed (最快，最干净)。
* 支持特定网站 Sitemap 爬取。
* 支持 Google Search API / Bing Search API (针对关键词主动搜索)。


* **去重机制**：
* 计算文章标题和内容的 SimHash 或 Embedding 相似度，避免推送重复新闻。



### 3.3 质量与信誉评分系统 (Scoring Engine)

这是本产品的核心壁垒。

* **评分公式**：


1. **信誉分 ()**：基于域名白名单。例如 `nature.com` = 100, `random-blog.xyz` = 40。
2. **内容质量分 ()**：调用 LLM 进行评估。
* Prompt: *"请阅读以下文章，基于信息密度、客观性、逻辑清晰度打分（0-100）。如果是标题党或软广，直接打 0 分。"*


3. **时效分 ()**：发布时间距今越近分数越高。



### 3.4 个性化生成与投递 (Curation & Delivery)

**目标**：千人千面。

* **生成逻辑**：
1. 遍历用户订阅列表。
2. 查询数据库中今日抓取的、Tag 匹配的、分数 > 60 的文章。
3. 按分数排序，取 Top N。
4. **摘要生成**：LLM 生成 50 字以内的 "One-liner" 摘要。


* **排版渲染**：
* 生成美观的文本卡片（Markdown 风格转微信支持的格式）。
* 如果使用企业微信，可发送 **图文消息卡片 (Rich Media Card)**，包含封面图。



---

## 4. 数据库设计 (Schema Design)

简化的 SQL 设计思路：

```sql
-- 用户表
CREATE TABLE users (
    wechat_id VARCHAR PRIMARY KEY,
    is_active BOOLEAN DEFAULT TRUE,
    delivery_time TIME DEFAULT '07:30',
    frequency VARCHAR DEFAULT 'daily'
);

-- 订阅偏好表
CREATE TABLE preferences (
    user_id VARCHAR REFERENCES users(wechat_id),
    keyword VARCHAR, -- e.g., "AI", "Climate"
    type VARCHAR,    -- "include" or "exclude"
    created_at TIMESTAMP
);

-- 文章库
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR,
    url VARCHAR UNIQUE,
    content_text TEXT,
    summary TEXT,
    source_domain VARCHAR,
    trust_score INT,
    quality_score INT,
    publish_date TIMESTAMP,
    vector_embedding VECTOR(1536) -- 用于语义匹配
);

```

---

## 5. 部署与运行流程 (Deployment)

### 步骤 1: 环境准备

```bash
# 使用 Docker 快速部署 PostgreSQL 和 Redis
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres
docker run -d -p 6379:6379 redis

```

### 步骤 2: 配置文件 (`config/settings.yaml`)

```yaml
wechat:
  corp_id: "ww123456..."  # 企业微信ID
  corp_secret: "xk82..."
  agent_id: 10001

llm:
  provider: "openai"
  api_key: "sk-..."
  model: "gpt-4o-mini" # 性价比高，适合大量摘要任务

scheduler:
  default_time: "07:30"
  timezone: "Asia/Shanghai"

```

### 步骤 3: 核心代码逻辑 (`main.py` 伪代码)

```python
from apscheduler.schedulers.blocking import BlockingScheduler
from agents import CrawlerAgent, CuratorAgent, DeliveryAgent

def daily_workflow():
    # 1. 爬取
    print("开始全网抓取...")
    raw_data = CrawlerAgent.run(sources_list)
    
    # 2. 清洗入库
    for article in raw_data:
        score, summary = CuratorAgent.evaluate(article)
        if score > 50:
            Database.save(article, score, summary)
            
    # 3. 分发
    users = Database.get_active_users()
    for user in users:
        digest = CuratorAgent.compile_digest(user.preferences)
        DeliveryAgent.send_wechat(user.wechat_id, digest)

if __name__ == "__main__":
    scheduler = BlockingScheduler()
    # 每天凌晨 2 点抓取，早上 7:30 推送
    scheduler.add_job(daily_workflow, 'cron', hour=2) 
    scheduler.start()

```

---

## 6. 难点解决方案

### Q1: 如何防止 LLM 费用过高？

* **方案**：不要把整篇文章扔给 LLM。先用传统的 NLP 库（如 `jieba` 或 `BeautifulSoup`）提取正文，截取前 2000 个 token。
* **方案**：在爬取阶段先用规则（正则表达式）过滤掉明显的广告和垃圾网站，不送入 LLM 处理。

### Q2: 微信消息长度限制怎么办？

* **方案**：InfoGather Agent 发送的是 **"摘要日报"**。每条新闻只显示：`[标题] + [一句话摘要] + [链接]`。用户点击链接跳转原网页阅读详细内容。

### Q3: 爬虫被反爬怎么办？

* **方案**：
* 优先使用 RSS（无反爬）。
* 对于必须爬取的网站，使用无头浏览器（Playwright）并集成代理池（Proxy Pool）。
* 设置随机 User-Agent 和 请求间隔（Sleep）。



---

## 7. 总结与后续建议

这个方案的核心在于**利用 LLM 进行非结构化数据的结构化处理（摘要与评分）**。

**建议的第一步 (MVP)**：

1. **不搞复杂的爬虫**：只支持 RSS 源订阅。
2. **不搞复杂的 NLP 指令**：只支持 `/sub keyword` 固定指令。
3. **接入企业微信**：这是最稳健的通道。

这个架构既满足了你 "无需手动检查多个来源" 的痛点，也保证了 "信息质量" 和 "隐私控制"。
