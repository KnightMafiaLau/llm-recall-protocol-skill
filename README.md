# llm-recall-protocol

A **self-evolving** Claude Code skill for measuring whether LLMs (Kimi / Doubao / DeepSeek / Qwen / Wenxin / GPT / etc.) actually recall your company on category-recommendation queries — and **why** they do or don't.

Most "AI visibility" tools just count mentions. This skill goes further:

1. **Multi-role question matrix** — generates queries from 5 user perspectives (普通用户 / 租赁商 / 开发者 / 投资人 / 行业观察) so you don't over-fit to one query type.
2. **Name-variant binding scoring** — splits "entity binding" into Chinese-only / English-only / combined-form, so you can detect asymmetric recall (e.g. an LLM recognizes the English name but not the Chinese one).
3. **Source attribution** — for each LLM response, asks "where did you learn this?" and classifies the cited sources by type (官网 / 综合媒体 / 行业媒体 / 社区 / 内容分发 / 评测).
4. **Per-LLM source-bias profile** — discovers each LLM's preferred source pool (e.g. Doubao loves 抖音/头条/CSDN/雪球, Wenxin loves 百家号), so you can target content distribution to the channels each LLM actually reads.
5. **Self-evolves the test library** — after each run, queries that have been recalled 3+ times get downweighted, persistent failures get amplified, new failure modes get auto-generated as new queries.

## Background

The methodology comes from a 3-week GEO experiment combined with internal channel-research data. The original protocol (multi-role + multi-industry + source-trace) was developed by 黄诗楚 in BridgeDP's GEO research project. This skill encodes that methodology into a repeatable, self-improving harness.

## What it produces (per run)

```
公司：桥介数物
测试日期：2026-05-07
本轮总分：32 / 100

LLM 排行：
1. Kimi      72/100  ↑8 (vs last run)
2. DeepSeek  60/100  ↑12
3. 豆包      32/100  ↓4 ⚠️
...

🔍 Source profile per LLM
- Kimi: top sources = [CSDN, 36氪, 知乎, 公众号, 头条]; bias = balanced
- 豆包: top sources = [抖音, 头条, CSDN, 雪球]; bias = strong ByteDance ecosystem
- ...

📌 Channel actions (auto-derived from source profiles)
- 豆包未召回 → optimize for ByteDance ecosystem: post on 头条 + 抖音 + 雪球
- 文心未召回 → optimize for Baidu ecosystem: post on 百家号 + 公众号
- ...

🔁 Self-evolution
- Promoted to "stable" (downweighted): query #3, #7
- Persistent failure (boosted): query #11
- New auto-generated query (from GPT failure mode): "RoboCraft 是宇树的硬件供应商吗？"
- Added to library; will run next round
```

## How self-evolution works

The skill maintains a `.recall_protocol_state.yml` file in the company's archive directory. Each run:

- Reads last 3 runs' results
- Detects pattern shifts (e.g. "Kimi started recalling on Query 7 last run, confirmed this run → mark stable")
- Reads LLM responses for new failure modes (e.g. "GPT classified as OEM in Query 8 — generate a defensive query")
- Adjusts query weights and adds new queries
- Writes back the updated state

So the test gets sharper over time without manual maintenance.

## Installation

```bash
curl -o ~/.claude/commands/llm-recall-protocol.md \
  https://raw.githubusercontent.com/KnightMafiaLau/llm-recall-protocol-skill/main/llm-recall-protocol.md
```

Use:

```
/llm-recall-protocol 桥介数物
```

This generates a test sheet. You manually run the queries on each LLM, paste responses back, then re-run the skill to get the analysis.

## Why per-LLM source profiling matters

A common GEO mistake: write great content, post it to one platform (say 知乎), and wonder why some LLMs still don't recall you.

Each major LLM has a **distinct source pool** that it preferentially crawls:

| LLM | Source bias | Why |
|---|---|---|
| 豆包 | 头条 / 抖音 / CSDN / 雪球 | Bytespider crawls within ByteDance ecosystem heavily |
| 文心一言 | 百家号 / 公众号 / 同花顺 | Baidu ecosystem |
| 通义千问 | 36氪 / IT之家 / 新华网 | Mainstream media integration |
| DeepSeek | IT之家 / 36氪 / CSDN / 澎湃 | Tech media + community |
| Kimi | 杂源 (掘金 / 知乎 / 公众号 / 头条) | Broad, less single-ecosystem bias |
| 元宝 | 腾讯云 / 公众号 | Tencent ecosystem |

If Doubao isn't recalling you, the fix isn't "write more content" — it's "post to ByteDance-friendly platforms." This skill makes that diagnosis explicit.

## Sister skills

- [geo-writing-skill](https://github.com/KnightMafiaLau/geo-writing-skill) — review drafts for GEO compliance.
- [llm-misclassification-defense-skill](https://github.com/KnightMafiaLau/llm-misclassification-defense-skill) — detect and fix LLM miscategorization.
- [agent-ready-skill](https://github.com/KnightMafiaLau/agent-ready-skill) — audit a website's agent-readiness.

## Credits

Methodology derived from internal GEO research by 黄诗楚 at BridgeDP, combined with field-tested patterns from a 3-week multi-LLM experiment.

## License

MIT — see [LICENSE](LICENSE).
