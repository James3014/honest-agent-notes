---
layout: post
title: "token 都花到哪去了：3,928 份 Claude Code 紀錄的拆解"
description: "拆解一台機器上的 Claude Code 本機紀錄：按 token 數算，快取重讀佔 96.2%；按估計費用算，超過 20 萬 token 脈絡的呼叫佔 65.6%。"
date: 2026-10-23 20:36:01 +0800
---

我們拆解了一台電腦上的 Claude Code（一種能自主執行多步驟任務的 AI 代理）本機紀錄。按 token 數算，快取重讀（每次呼叫都重新讀入先前累積的脈絡）佔 96.2%，與之前用臨時腳本對同一批紀錄做的一次性人工分析（下稱「手動驗屍」）的 96.4% 吻合。以逐次呼叫計算，脈絡超過 200k（20 萬個 token）的呼叫佔全部估計費用的 65.6%。也就是說，按 token 數算，絕大部分是每次呼叫重新讀入的舊脈絡，不是模型新寫出來的內容；按估計費用算，超過 200k 的呼叫佔了大約三分之二。

## 我們怎麼量的

樣本是我們一台機器上於 2026-10-23 掃描的 Claude Code 本機工作階段紀錄（session，指使用者與代理從開始到結束的一段完整對話），共 3,928 個檔案、582 個 session、7.6 GB。量測工具是我們自己寫、已開源的 [token-autopsy](https://github.com/cairnwork/token-autopsy)，掃描只讀本機紀錄檔，不呼叫任何模型，也不連網。

所有金額都是依公開牌價估算的數值，不是帳單。訂閱制用戶實際付出的是額度，所以我們的主要指標是 token（計量模型處理文字量的基本單位）數量與占比。

## 數字

| 測量項目 | 數值與單位 | 統計口徑與說明 |
| :--- | :--- | :--- |
| 樣本規模 | 3,928 個檔案、582 個 session、7.6 GB | 2026-10-23 掃描單一機器的本機紀錄，不連網 |
| 快取重讀占比 | 佔全部 token 的 96.2% | 先前手動驗屍為 96.4%，兩者吻合 |
| 超過 200k 脈絡的呼叫（逐次呼叫計算） | 佔全部估計費用的 65.6%，佔全部 token 的 69.8% | 以單次呼叫切分 |
| 曾超過 200k 的檔案（整檔計算） | 310 個檔案，佔全部估計費用的 79.9% | 只要檔案內曾有一次超過 200k，整檔計入；與逐次呼叫口徑不同，不能直接比 |
| 輪詢（用 sleep 迴圈反覆檢查狀態） | 5,373 次呼叫，估計花費 $894，佔全部估計費用的 7.6% | 估計值，不是帳單 |
| 輪詢細項：sleep 迴圈 | 5,188 次呼叫 | 工具回報的分類細項 |
| 輪詢細項：60 秒內重複相同指令 | 241 次呼叫 | 工具回報的分類細項 |
| 子代理（subagent，由主代理派出的獨立分身）基底脈絡重讀 | 佔子代理估計費用的 11.8%，佔全部估計費用的 5.4% | 與手動驗屍的 47% 定義不同，不能比較 |
| 規則一：超過 200k 就重開或壓縮 | 估計可省 10.1%–20.2% | 工具標註為樂觀估計，沒算重開時寫摘要的成本 |
| 規則三：精簡基底脈絡 | 估計可省 1.3%–1.5% | 用一個假設比例推算，不是實測 |

## 限制

1. 只有一位作者、一台機器的紀錄，不能代表其他人的用法。
2. 工具目前（v0.1）只讀 Claude Code 的紀錄，不含 Codex 等其他工具。
3. 金額是牌價估算，不是帳單；訂閱用戶付的是額度。
4. 長脈絡有兩種口徑（逐次呼叫、整個檔案），算出來的比例不同，請看表格的說明，不要混著引用。
5. 節省比例是帶假設的估計，不是保證：規則一沒計入重開的摘要成本，規則三的 1.3%–1.5% 是用假設比例推算的。
6. 與手動驗屍相比，子代理基底等指標的定義不同，不能直接比較。
7. 「找檔案時間」的嚴格口徑只抓到 28 次、占比不到 0.1%，寬鬆上限卻是 58%，落差太大，我們不把它當結論。

## 你可以怎麼用

1. **脈絡超過 200k 就重開或壓縮。** 工具估計可省 10.1%–20.2%，但這是樂觀估計，沒算重開時寫摘要的成本，實際省多少要自己量。
2. **別讓代理自己寫 sleep 迴圈輪詢。** 我們的紀錄裡輪詢有 5,373 次呼叫，估計佔全部估計費用的 7.6%。改用明確的阻擋等待或事件通知，讓代理不必反覆檢查狀態。
3. **精簡每次都會被重新讀入的基底脈絡**，例如工具定義和專案說明檔。工具估計可省 1.3%–1.5%，但這是用假設比例推算的，不是實測，效果可能很小。

## 出處

- 開源工具與驗證紀錄：[token-autopsy](https://github.com/cairnwork/token-autopsy)（README.md 與 docs/VALIDATION.md）
- 手動驗屍的基準數據：我們的內部實驗紀錄

## English summary

- We scanned 3,928 Claude Code local log files (582 sessions, 7.6 GB) from a single machine on 2026-10-23, using our own open-source tool; the scan made no model calls and no network access.
- Cache re-read accounted for 96.2% of all tokens, consistent with an earlier manual audit (96.4%).
- Calls with context over 200k tokens accounted for 65.6% of estimated cost and 69.8% of tokens on a per-call basis; on a per-file basis, the 310 files that ever passed 200k account for 79.9% of estimated cost.
- Polling made 5,373 calls ($894 estimated, 7.6% of estimated cost); 5,188 were sleep-loops and 241 were repeated commands within 60 s (sub-categories as reported by the tool).
- Rule savings are estimates only: restarting at 200k is optimistic (it omits summary cost), and trimming base context is derived from an assumed ratio.

| Metric | Value | Notes |
| :--- | :--- | :--- |
| Sample size | 3,928 files, 582 sessions, 7.6 GB | Local logs scanned on 2026-10-23 |
| Cache re-read share | 96.2% of all tokens | Matches prior manual audit (96.4%) |
| Context >200k (per call) | 65.6% of estimated cost, 69.8% of tokens | Evaluated per API call |
| Context >200k (per file) | 310 files; 79.9% of estimated cost | Whole file counted if it ever passed 200k; not comparable to per-call |
| Polling calls | 5,373 calls, $894 (7.6% of estimated cost) | 5,188 sleep-loops, 241 repeated commands (sub-categories as reported by the tool) |
| Subagent base context re-read | 11.8% of subagent estimated cost, 5.4% of total estimated cost | Different definition from prior 47% manual audit |
| Rule 1 estimated savings | 10.1%–20.2% | Optimistic; omits summary cost on restart |
| Rule 3 estimated savings | 1.3%–1.5% | Derived from an assumed ratio, not measured |

---

*這篇由 AI 代理依我們的實驗紀錄起草，再由另一家模型逐條核對數字與出處後發佈。發現錯誤請[告訴我們](https://github.com/James3014/honest-agent-notes/issues)，我們會更正並標註。*
