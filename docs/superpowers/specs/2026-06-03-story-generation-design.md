# 最终挑战：AI 故事生成 + 模板引擎 设计方案

## 概述

将"最终挑战"页面从固定模板填充改造为动态故事生成系统。每次词汇学习（30 词）结束后，生成一篇包含全部所学词汇的英文故事。

## 核心要求

1. 故事必须使用本次学习的全部 30 个单词/短语
2. 除目标词外，其他单词尽可能简单（CEFR A2-B1 级别）
3. 每次生成的故事不同
4. 中段长度 400-600 词
5. 保留段落翻译功能

## 架构

```
学习完成 → goToChallenge()
              ↓
     ┌─ hasApiKey()? ────────────┐
     ↓                            ↓
  generateWithDeepSeek()    generateWithTemplate()
  (deepseek-chat model)     (20+ 框架 × 随机组合)
     ↓                            ↓
     └── 成功? ──No──→  generateWithTemplate()
         ↓
    统一渲染: renderStoryArticle()
```

## 模块设计

### 1. DeepSeek API 故事生成器

- **端点**: `https://api.deepseek.com/chat/completions`
- **模型**: `deepseek-chat`
- **认证**: `Authorization: Bearer <apiKey>`
- **超时**: 30 秒 (AbortController)
- **重试**: 1 次
- **输出**: `{ title: string, paragraphs: [{en: string, cn: string}] }`

**System Prompt 要点**:
- 使用全部 30 个词（提供词列表，含中英释义）
- 其他词用简单英语（像给初中生写故事）
- 400-600 词，8-10 段
- 返回 JSON，每段带英文和中文翻译

### 2. 模板故事引擎 (Fallback)

- **20+ 套故事框架**: 覆盖不同主题（侦探、科幻、旅行、创业、环保、校园……）
- **随机化元素**: 主角名(20+)×地点(15+)×季节×冲突类型×结局
- **智能填词**: 根据词语的词性 (verb/noun/adjective/phrase) 填入合适语法位置
- **简单词汇约束**: 框架中非目标词使用 BNC 高频 2000 词
- **框架格式**: 预分段，每段含英文 + 中文翻译

### 3. API Key 管理

- 存储: `localStorage('deepseek_api_key')`
- UI: 封面页右侧齿轮图标 ⚙️
- 设置面板: 输入框 + 保存/清除按钮 + 状态提示
- 无 Key 时: 静默使用模板引擎

### 4. UI 改动

- **加载状态**: 生成中显示旋转动画 + "正在用 AI 生成你的专属故事..."
- **错误处理**: 生成失败 → 提示 → 自动降级模板引擎
- **故事展示**: 保持不变（标题 + 词汇高亮红色 + 普通词点击查词 + 段落翻译）

## 文件改动

| 文件 | 改动范围 |
|------|---------|
| `index.html` | CSS: 加载动画、设置面板样式。HTML: 设置面板 DOM。JS: generateWithDeepSeek()、generateWithTemplate()、故事框架库、API Key 管理函数、改造 renderArticle() |
| `dictionary.js` | 不修改 |

## API 响应格式

```json
{
  "title": "The Lighthouse Keeper's Secret",
  "paragraphs": [
    {
      "en": "On a small island far from the city, an old man named Thomas worked as a lighthouse keeper...",
      "cn": "在远离城市的一个小岛上，一位名叫托马斯的老人担任灯塔看守人..."
    },
    ...
  ]
}
```

## 验收标准

1. 有 API Key 时，调用 DeepSeek 生成故事成功，30 个词全部高亮出现
2. 无 API Key 时，模板引擎生成故事成功，30 个词全部出现
3. API 调用失败时，自动降级到模板引擎
4. 生成的故事情节连贯、可读
5. 段落翻译功能正常
6. API Key 设置/清除功能正常
7. 词汇高亮可点击跳转学习
