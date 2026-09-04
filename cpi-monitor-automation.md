# CPI/PPI 月度监控 — 自动化实现说明

> 本文档是《一套 CPI/PPI 通胀预警规则（v3）》配套的自动化实现，公开以便查看与复现。

## 一、任务概述

每月 13 号 13:00（北京时间）自动运行：抓取国家统计局最新 CPI/PPI 数据 → 按规则文档分层判定 → 生成一篇博客文章 → 推送到 GitHub Pages。

## 二、调度配置（cron）

| 字段 | 值 |
|---|---|
| name | CPI/PPI 月度博客 |
| schedule | cron `0 13 13 * *`（Asia/Shanghai） |
| sessionTarget | isolated |
| 触发方式 | agentTurn（提示词驱动） |

## 三、自动化提示词（agent prompt 全文）

```text
【CPI/PPI 月度博客发布】先读取规则文档 /Users/leeson/.openclaw/workspace/scripts/cpi_monitor_rules.md，
抓取国家统计局最新物价数据并按规则分析，然后发布一篇博客文章。

步骤：
1. 抓取 stats.gov.cn 最新 CPI/PPI（每月 9~13 号发布上月数据），以及央行 M1/M2（如已发布）。
2. 按规则文档分层判定、定级、解读。
3. 写 Jekyll 博客文章到 /Users/leeson/Documents/leesonweb/_posts/，文件名 YYYY-MM-DD-标题.md。
4. Front matter 严格照现有文章格式：layout: post、title、date: YYYY-MM-DD HH:MM +0800、
   tags: [CPI, PPI, 通胀, 宏观经济]、toc: true。
5. 文章正文：本月数据 + 分层判定结果 + 中文解读 + 末尾附「本次判定依据：第X条」。
6. 若本月数据尚未发布，不写文章，直接报告「本月数据尚未发布」。
7. 写完后执行：cd /Users/leeson/Documents/leesonweb && git add _posts/ &&
   git commit -m "月度 CPI/PPI 分析" && git push origin master。
8. 报告：文章标题 + git push 结果。
```

## 四、判定规则文档

分层判定规则（领先层 / 同步层 / 响应分级）见配套文章，原始文档位于：

```
/Users/leeson/.openclaw/workspace/scripts/cpi_monitor_rules.md
```

## 五、实现说明

本任务不依赖传统 Python 脚本，而是由「规则文档 + 自动化提示词」驱动 AI 完成完整链路：

```
抓取数据（web_search / web_fetch）
   ↓
按规则分层判定、定级
   ↓
生成 Jekyll 格式文章
   ↓
git commit + push 发布
```
