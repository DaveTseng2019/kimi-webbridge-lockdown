# Changelog

## v0.1.1 — 2026-10-13

Final release. **Not recommended on Chrome 155+ / Edge 154+; the project is no longer maintained and the repository will be archived.** Documentation only — the scripts are unchanged.

- **Chrome 155 / Edge 154 warning** (#4, thanks [@DaveTseng2019](https://github.com/DaveTseng2019) for the measurements and the patch, following up on #3): from these versions a non-empty `runtime_blocked_hosts` disables `chrome.debugger.attach()` on **every** site (`Host access is restricted by policy.`), so the per-host lock becomes all-or-nothing.
- **Version boundary**: Chrome 155 / Edge 154 (Microsoft's ExtensionSettings page and the Chromium policy text say 154; the Chrome post says 155). Version 154 itself is marked as untested.
- **Caveat on older versions**: per-host blocking of `chrome.debugger` is best-effort even before 155 — Chrome states that origin-based filtering cannot securely restrict CDP.
- **AGENTS.md**: the agent checks the browser version in Step 0 and stops before applying anything on Chrome 155+ / Edge 154+; Step 4 accepts both error strings as "blocked". Contributor notes added for people changing the scripts.
- **README**: status banner at the top, a Requirements section (works on Chrome ≤154 / Edge ≤153; the temporary `--disable-features=ExtensionDebuggerStrictPolicyRestrictions` flag, removed in Chrome 160; do not downgrade a browser for this tool), and the recommended alternative: a separate browser profile for the agent, and no AI browser-control extension in the browser you use for critical sites.

---

最終版。**Chrome 155+／Edge 154+ 不建議使用；本專案停止維護，repo 將封存。** 只改文件，腳本沒有變動。

- **Chrome 155／Edge 154 警告**（#4，感謝 [@DaveTseng2019](https://github.com/DaveTseng2019) 實測並送 patch，接續 #3）：從這些版本起，只要 `runtime_blocked_hosts` 不是空的，`chrome.debugger.attach()` 在**所有**網站都失敗（`Host access is restricted by policy.`），逐 host 封鎖變成全有或全無。
- **版本界線**：Chrome 155／Edge 154（Microsoft ExtensionSettings 文件與 Chromium 政策說明寫 154，Chrome 官方文寫 155）。154 版本身標為未實測。
- **舊版的但書**：即使在 155 以前，逐 host 封鎖 `chrome.debugger` 也只是盡力而為——Chrome 官方說明 origin 過濾無法安全地限制 CDP。
- **AGENTS.md**：agent 在 Step 0 先查瀏覽器版本，遇到 Chrome 155+／Edge 154+ 就在寫入前停下；Step 4 兩種錯誤字串都算「已封鎖」。另加上給改腳本的人看的貢獻者注意事項。
- **README**：頂端狀態說明、相容性段（可用 Chrome ≤154／Edge ≤153；暫時旗標 `--disable-features=ExtensionDebuggerStrictPolicyRestrictions` 在 Chrome 160 移除；不要為了這個工具降版瀏覽器），以及建議的替代做法：給 agent 獨立的瀏覽器設定檔，開要害網站的瀏覽器不裝 AI 瀏覽器控制擴充。

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
