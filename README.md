# Delegate to Luna Max

讓 GPT-6 Luna 以固定 `max` 推理強度承擔有明確邊界的工作，主代理負責分工、重大決策與驗收。目的是減少高階主代理重複執行的工作，同時保留必要檢查。

## 使用方式

此 skill 允許自動選用，但自動判斷不保證每次都會觸發。要明確使用，可在要求中加入 `$delegate-to-luna-max`：

```text
$delegate-to-luna-max
修正 src/parser.py 的空輸入處理，可修改該檔與 tests/test_parser.py。
沿用現有介面，完成相關測試後停止。
```

```text
$delegate-to-luna-max
為 docs/install.md 和 docs/config.md 補上使用範例。
若工作彼此獨立可平行處理，每個代理只能修改分配到的文件。
```

主代理會略過不值得委派的極小任務；一般範圍明確的工作則優先交給 Luna。子代理固定指定 `model="gpt-6-luna"`、`reasoning_effort="max"`，預設不帶入整段對話歷史。

## 驗證方式

預設由主代理檢查一次實際差異與必要證據，搭配使用者、專案要求的檢查，以及確認變更行為所需的最小檢查。已通過的檢查只在後續修改影響結果、證據不足或存在具體風險時重跑。

本版本沒有 Adaptive 的 `verification=` 或 `effort=` 選項解析規則。需要按工作調整推理強度或指定驗證模式，可使用 [Delegate to Luna Adaptive](https://github.com/ccccchhhheeenng/delegate-to-luna-adaptive)。

## 安裝與更新

在 PowerShell 執行以下指令，安裝到 `$CODEX_HOME/skills`；未設定 `CODEX_HOME` 時使用 `$HOME/.codex/skills`。

```powershell
$skillRoot = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $HOME '.codex/skills'
}
New-Item -ItemType Directory -Force -Path $skillRoot | Out-Null
$skillPath = Join-Path $skillRoot 'delegate-to-luna-max'
git clone https://github.com/ccccchhhheeenng/delegate-to-luna-max.git $skillPath
```

也可將 clone 網址換成 GitLab：

```text
https://git.ccccchhhheeenng.com/Cheng/delegate-to-luna-max.git
```

選擇其中一個來源即可。若目錄已存在，先確認內容，不要覆蓋；既有 Git 安裝可在確認沒有待保留的未提交修改後更新：

```powershell
git -C $skillPath status --short
git -C $skillPath pull --ff-only
```

更新指令沿用上方的 `$skillPath`。GitLab 來源若需要登入，須先完成 Git 驗證。

## 分工與邊界

一般有明確範圍的實作、除錯、測試、文件與調查，優先交給 Luna。主代理先確認需求、風險、檔案範圍與驗收條件，再由 Luna 找既有做法並完成工作；主代理保留重大決策、整合與最後驗收。

- 同時最多 **3 個 Luna 子代理**，包含實作、調查與驗證角色。依獨立工作的數量使用 1、2 或 3 個，並受執行環境容量限制，不必開滿。
- 每條工作線都要明定允許讀取、允許修改、相依結果、必要檢查與停止條件。
- 平行修改必須有不同的檔案與可寫入資源；即使原始碼不同，共用產出檔案的檢查也要依序執行。
- Luna 可以在範圍內選擇既有做法，不能自行擴大需求、重設架構或另開子代理；額外授權由主代理處理。
- 避免主代理重做調查、重寫正確成果，或無理由重跑已通過的檢查。安全且範圍明確的小幅修正，優先交回同一個 Luna。
- 所有必要結果都要收齊、檢查，且不能留下仍在執行的子代理就結束回覆。

## 避免過度探索

預設只比較最多兩個有證據支持的調查假設。找到符合需求的既有做法後就採用；除非出現新需求、反證或檢查失敗，不重新討論已決定的方向。

指定檢查失敗後，先做一次範圍內修正並重測；仍失敗就回報部分成果與失敗證據，由主代理依重試規則決定下一步。完成驗收條件與指定檢查後立即停止。

這些是工作流程限制，不能硬性限制模型內部推理量或執行時間。並行可能縮短等待，也可能增加總用量；本專案不保證特定的速度或用量降幅。

## 環境需求與限制

需要能使用 skills、建立子代理，並明確指定 `gpt-6-luna` 與推理強度的 Codex 環境。實際可用模型、工具與並行容量以執行環境為準。無法使用指定模型時，主代理應說明並接手，不默默換成其他子代理模型。

這份 skill 不會更換主代理模型，也不會自行調整帳號額度或系統並行設定。需求不明、重大架構決策、高風險與不可逆操作由主代理負責；可獨立且安全的證據蒐集仍可交給 Luna。

## 規則來源

- [SKILL.md](SKILL.md)：實際委派、邊界、重試與驗收規則。
- [agents/openai.yaml](agents/openai.yaml)：顯示資訊與自動啟用設定。

README 提供使用說明；完整行為以 `SKILL.md` 為準。