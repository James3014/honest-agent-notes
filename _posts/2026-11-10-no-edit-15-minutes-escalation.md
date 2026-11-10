---
layout: post
title: "我們回放 70 次執行：第 15 分鐘還沒改檔的全部失敗，換強模型重來 15 題也沒救回"
description: "在 70 個歷史執行的樣本裡，第 15 分鐘還沒寫檔的 27 個全部沒解決；換較強的模型重跑 15 題，也是 0 題解決。"
date: 2026-11-10 09:36:42 +0800
---

在 70 個歷史執行裡，第 15 分鐘還沒寫檔的 27 個全部沒解決，但這個訊號只抓到 60 個失敗中的 27 個（45%）。我們拿 15 個被標記的任務做配對重跑，換較強模型和用原本的便宜模型重來，都是 0 題解決。所以在我們的樣本裡，這個訊號可以拿來提醒「這次大概不會成」，但換強模型重來沒有用。

## 我們怎麼量的

我們做兩件事。第一，回放（拿過去的執行紀錄套用新規則檢查，不重新跑模型）70 個可分類的歷史執行紀錄，用「第 15 分鐘還沒有任何寫檔動作」當標記。這裡的「寫檔動作」是指代理實際修改檔案，不是只讀取或輸出計畫。第二，從被標記的任務中取 15 個做配對試驗（同一題分成兩組、用不同做法重跑）：一組換成較強的 Claude 模型重來 15 分鐘，另一組用原本的便宜模型重來 15 分鐘，每組只跑一次。

配對試驗在 2026 年 10 月中完成。配對試驗的沙盒（讓代理在隔離環境裡執行的設定）與環境設定和原本的歷史執行不同，但兩組彼此相同。

## 數字

| 項目 | 數字 |
|---|---|
| 可分類的歷史執行 | 70 個 |
| 第 15 分鐘無寫檔而被標記 | 27 個 |
| 被標記的執行中解決問題 | 0 個 |
| 未被標記的執行 | 43 個 |
| 未被標記的執行中解決問題 | 10 個 |
| 失敗執行中被標記 | 27/60（45%） |
| 被標記且跑滿 30 分鐘上限 | 19 個 |
| 配對試驗的任務數 | 15 個 |
| 強模型重來後解決 | 0 題 |
| 便宜模型重來後解決 | 0 題 |
| 強模型組時間中位數 | 434 秒（預算 900 秒） |
| 強模型組自己停下來 | 14/15 |
| 強模型組真的改檔 | 14/15 |
| 便宜模型組時間中位數 | 905 秒（全部跑到 15 分鐘上限） |
| 次要比較：解決加部分解決（便宜／強） | 2 題／6 題 |
| 逐題比較：強模型較好／較差 | 5 題／2 題 |

歷史回放裡，10 個最後有解決的執行中，沒有任何一個被標記，所以沒有觀察到成功執行被誤擋。60 個失敗的執行有 27 個被標記，也就是有資格被換強模型重試；這 27 個裡有 19 個一路跑滿 30 分鐘上限。

配對試驗裡，兩組都是 0 題解決，所以在這組樣本裡，沒有證據支持把這個做法做成正式機制。這不是因為強模型沒動手：它 15 題中有 14 題真的改了檔案，也有 14 次自己停下來；便宜模型重來組則全部跑到 15 分鐘上限。也就是說，強模型有動手，時間也還夠，但換模型重來沒有把任何一題從未解決變成解決。

另有一個次要觀察：「解決加部分解決」的題數，便宜模型重來組是 2 題，強模型組是 6 題；逐題看，強模型在 5 題較好、2 題較差。這是沒有預先決定判準的次要觀察，不能當成主要結論。

## 限制

這篇不能推論到所有代理或所有模型。第一，配對試驗只有 15 個任務，每個任務、每組只跑一次；任一組多解決一題都在雜訊範圍內。第二，任務都是已有合併修復的舊問題，不是新題。第三，沙盒與環境設定和原本的歷史執行不同，而且強模型組同時換了模型和執行框架（包住模型、負責呼叫工具的那層程式），無法分開歸因。第四，「停掉而不換模型」可省約 15 分鐘的做法沒有測過，不能說有效。第五，70 個回放執行裡，有一部分正是當初用來設計這個訊號的資料。如果只看事先保留、沒參與設計的資料，39 個失敗執行中有 16 個（41.0%）被標記，事先定的判準只差一個任務就不會通過，所以訊號本身的證據也還薄。

## 你可以怎麼用

1. 把「15 分鐘內沒有任何寫檔」當成檢查點：看到這個訊號，先人工查看或準備停損，不要直接假設換更強模型就會好。直接停掉能省多少、有沒有誤殺，我們還沒測。
2. 在你的代理流程裡，把「讀取」和「實際修改檔案」分開記錄；這個訊號靠的是有沒有寫檔，不是有沒有在思考。
3. 若你想驗證升級模型的做法，先寫好要差幾題才算有效，再開始跑配對試驗。

## 出處

我們的內部實驗紀錄（歷史執行回放與配對試驗）。

## English summary

- In 70 classifiable historical runs, all 27 runs flagged for no file edit by minute 15 were unresolved; 10 of the 43 unflagged runs resolved. The flag caught only 27 of 60 failing runs (45%).
- None of the 10 resolved runs were flagged, so no successful runs were observed to be blocked.
- In a paired test of 15 flagged tasks, restarting with a stronger Claude model solved 0 tasks, same as restarting with the cheaper model (0).
- The stronger model edited files in 14 of 15 runs and stopped on its own in 14 of 15 (median 434 s of a 900 s budget); the cheaper restarts all ran to the 15-minute cap (median 905 s).
- Limits: only 15 old tasks, one run per task per arm; the strong arm changed model and scaffold together; stopping without escalation was not tested.

| Metric | Value |
|---|---|
| Classifiable historical runs | 70 |
| Flagged at minute 15 | 27 |
| Flagged runs resolved | 0 |
| Unflagged runs | 43 |
| Unflagged runs resolved | 10 |
| Failing runs flagged | 27/60 (45%) |
| Flagged runs hitting 30-min limit | 19 |
| Paired tasks | 15 |
| Strong-model restarts resolved | 0 |
| Cheap-model restarts resolved | 0 |
| Strong-model median time | 434 s (900 s budget) |
| Strong-model self-stopped | 14/15 |
| Strong-model edited files | 14/15 |
| Cheap-model median time | 905 s (all ran to the cap) |
| Solved + partial (cheap / strong) | 2 / 6 |
| Per-task better / worse (strong) | 5 / 2 |

---

*這篇由 AI 代理依我們的實驗紀錄起草，再由另一家模型逐條核對數字與出處後發佈。發現錯誤請[告訴我們](https://github.com/James3014/honest-agent-notes/issues)，我們會更正並標註。*
