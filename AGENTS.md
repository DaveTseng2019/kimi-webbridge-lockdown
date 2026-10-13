# AGENTS.md — guide a human through locking you out

You are an AI agent, and the human just asked you to set this tool up. Understand the situation: **you are configuring your own restraints.** The human uses you (or a tool like you) to control their real browser, and this repo blocks that control on the sites where a mistake costs real money. Run the process faithfully — a half-applied lockdown that *looks* applied is worse than none.

Everything below is a conversation script plus exact commands. The scripts support a non-interactive mode precisely so you can drive them; your job is the judgment around them: the interview, the verification, and honest reporting.

## Step 0 — Orient (read-only, no admin)

Identify OS and browser, then show the human where they stand:

```powershell
# Windows
.\windows\lockdown.ps1 -Action status
```
```bash
# macOS
./macos/lockdown.sh status
```

This lists every installed extension holding the `debugger` permission — likely including the one you drive — and what is currently blocked. Explain the headline finding to the human: the per-site "site access" whitelist in the extensions UI does **not** restrain these extensions; only this policy layer does (see README for the measurements).

**Then check the browser version before going any further** (`chrome://version` / `edge://version`). On **Chrome 155+ or Edge 154+**, stop here and tell the human plainly: once `runtime_blocked_hosts` is non-empty, the browser blocks the extension's `chrome.debugger` on **every** site, not just the listed ones (error `Host access is restricted by policy.`; see the README warning). The lock is now all-or-nothing — applying it disables you everywhere. Let the human decide whether they still want that, and point them to the README's alternative (a separate browser profile for the agent). Do not apply anything until they have decided. Official sources disagree on the exact first version (Chrome post: 155; Microsoft and the Chromium policy text: 154), and 154 itself was not measured, so treat 154 as affected.

## Step 1 — Interview: what should be off-limits?

Ask, one category at a time. Typical high-stakes categories:

- Online banking (each bank they actually use)
- Brokerage / securities accounts
- Mailbox (webmail is the account-recovery skeleton key)
- Anything else where an errant click moves money or leaks the archive of their life

**The one hard rule: never guess a host.** Not from your training data, not from a web search — bank login hosts move, stale indexes list dead ones, and personal vs corporate banking often live on different hosts of the same bank. A wrong host produces a lockdown that looks populated and protects nothing. Sources, in order:

1. `catalog/hosts.json` — every entry there was verified against the real login page (dated). If the human's bank is listed, use it.
2. Otherwise: **the human opens the login page themselves and pastes the URL** from the address bar. The scripts extract and validate the host (ports, userinfo, paths are stripped; junk is rejected loudly).

## Step 2 — Keep-open check

For each bank chosen, ask whether they use automation on *sibling* subdomains — credit-card event registration, account opening, forms (`card.`, `eopen.`, marketing sites). This tool blocks exact hosts only, never `*.domain` wildcards, exactly so those keep working. The catalog's `keep_open` fields list known siblings. Confirm the final list with the human before writing anything.

## Step 3 — Apply (admin required — the human stays in the loop)

Show the human the exact command before running it. Use the non-interactive form; pass `-Yes`/`--yes` only after they approved the final list in Step 2.

**Windows** — needs an elevated PowerShell. If you cannot elevate from your harness, hand the human this command to paste into an admin PowerShell (tell them: right-click PowerShell → 以系統管理員身分執行):

```powershell
.\windows\lockdown.ps1 -Action lock -Browser edge -ExtensionIds <id> -Hosts host1,host2 -Yes
```

**macOS** — the script calls `sudo`, which prompts for a password in the terminal; run it with the human present:

```bash
./macos/lockdown.sh lock chrome <id> host1,host2 --yes
```

Both scripts back up the current policy first, merge (never overwrite), and verify by reading the policy back. Multiple extension IDs are comma-separated — offer to include **every** debugger extension found in Step 0, not just yourself.

## Step 4 — Live verification (you are the perfect test instrument)

Policy read-back is not proof; the browser must actually enforce it. After the human **fully restarts the browser** (all windows AND background processes / Cmd+Q):

1. Confirm the policy page: `edge://policy` / `chrome://policy` must show `ExtensionSettings` — on macOS the level must read **Mandatory** (Recommended = no protection — a measured false positive; see the README table).
2. **Test the block:** try to open/attach to one blocked host through your own browser control. Expect failure: `Cannot attach to this target` on older browsers, `Host access is restricted by policy.` on Chrome 155+ / Edge 154+ — both mean blocked. You attempting and failing IS the proof.
3. **Test the control direction:** open an unblocked site the human normally automates. It must still work.

**Chrome 155+ / Edge 154+ (if the human chose to lock anyway after Step 0):** step 3 fails too — every host fails with `Host access is restricted by policy.` once `runtime_blocked_hosts` is non-empty (measured 2026-10-12; see the README warning). That is the browser's new behavior, not a broken lockdown. Tell the human plainly that the lock now disables you on every site, and let them decide whether to keep it. Do not rationalize it as a precise block.

Report both results verbatim. If the blocked host still opens, say so plainly and stop — do not rationalize a failed lockdown as success. (Check: browser fully restarted? policy level Mandatory? host exactly right?)

## Step 5 — Teach the exits

Show the human, so they are never stuck:

- **Unblock one site** (e.g. they need you to do something on a blocked site once): `-Action unblock` / `unblock` — removes selected hosts, keeps the rest. Re-lock afterwards with the same lock command.
- **Restore** any timestamped backup: `-Action restore` / `restore` (backups live in `~/.ai-browser-lockdown/backups/`).

## Honesty contract

- Never write a host you inferred. Address bar or verified catalog only.
- Never run the write step without having shown the final list.
- Never report protection without the Step 4 live test — in both directions.
- If any step fails, report the failure and what you tried. A confident-sounding non-answer is the worst outcome.
