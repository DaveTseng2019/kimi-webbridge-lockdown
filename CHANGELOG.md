# Changelog

## v0.1.0 — 2026-08-09

Initial release.

- **The lock**: writes the `ExtensionSettings` → `runtime_blocked_hosts` enterprise policy for Microsoft Edge / Google Chrome — the only per-host control that stops `chrome.debugger`-based AI browser-control extensions (the per-site "site access" UI does not; see README for measurements).
- **Windows (tested)**: interactive PowerShell tool — detects `debugger`-permission extensions across all profiles, merges (never overwrites) into the HKLM policy, timestamped backup before every write, read-back verification, guided live test.
- **macOS (tested)**: policy written to `/Library/Managed Preferences` (sudo), the only layer macOS browsers treat as *Mandatory*. Measured on real hardware (macOS 26.5.2 / Chrome 151): user-level `defaults write` loads as *Recommended* and blocks nothing, while the same payload in Managed Preferences yields `Cannot attach to this target` on blocked hosts with control hosts unaffected — hence the verification standard "level must read Mandatory".
- **unblock**: remove selected hosts later; every other host, sibling setting, extension, and policy is preserved.
- **restore**: one command back to any timestamped backup.
- **Agent-guided mode**: AGENTS.md scripts the whole flow for an AI agent to guide the human — interview which sites to protect (hosts only from the user's address bar or the verified catalog, never from model memory), keep-open check, non-interactive apply (`-Browser/-ExtensionIds/-Hosts/-Yes`; positional `lock|unblock <browser> <extids> <hosts> --yes` on macOS), then the agent proves the lock by attempting to attach to a blocked host and failing.
- **Catalog**: verified Taiwan bank/broker/Gmail login hosts (address-bar confirmed, dated), with keep-open sibling subdomains documented. Exact hosts only, no wildcards.
- **Tests/CI**: 44 PowerShell logic tests (Windows PowerShell 5.1 + PowerShell 7) and 54 bash logic tests (macOS system bash 3.2), mutation-verified; shellcheck; gitleaks secret scan over full history.

---

初版發佈。

- **鎖的本體**：寫入 Edge／Chrome 的 `ExtensionSettings` → `runtime_blocked_hosts` 企業政策——唯一擋得住 `chrome.debugger` 型 AI 瀏覽器控制擴充的逐 host 控制（擴充設定裡的逐站「網站存取權」擋不住；實測見 README）。
- **Windows（已實測）**：互動式 PowerShell 工具——全 profile 偵測帶 `debugger` 權限的擴充、合併寫入 HKLM（絕不覆蓋）、寫入前自動備份、讀回驗證、引導實測。
- **macOS（已實測）**：政策寫入 `/Library/Managed Preferences`（sudo），macOS 唯一當成「強制」的層。實機量測（macOS 26.5.2／Chrome 151）：user 層 `defaults write` 只是「建議採用」、完全不擋；同樣內容放進 Managed Preferences 才會 `Cannot attach to this target`、對照 host 不受影響——所以驗證標準是「等級必須是強制」。
- **unblock**：之後可只解除選定 host，其餘封鎖與設定全數保留。
- **restore**：一鍵回到任一時間戳備份。
- **Agent 引導模式**：AGENTS.md 讓 AI agent 帶人跑完全程——訪談要保護哪些網站（host 只准來自網址列或已驗證目錄）、keep-open 確認、非互動執行，最後 agent 以「自己 attach 被封 host 而失敗」證明鎖有效。
- **目錄**：實地驗證過的台灣網銀／券商／Gmail 登入 host（附驗證日期與該保持開放的子網域）。只收精確 host，不用 wildcard。
- **測試／CI**：PowerShell 44 條（PS 5.1＋PS 7）＋bash 54 條（macOS 系統 bash 3.2），含變異驗證；shellcheck；gitleaks 全 history 掃描。
