# kimi-webbridge-lockdown

[繁體中文](README.zh-TW.md)

Lock AI browser-control extensions out of your bank, broker, and mailbox — while letting them automate everything else.

> **Status: not recommended on Chrome 155+ / Edge 154+, and no longer maintained.**
> On these versions the browser turns this tool's per-site block into an all-or-nothing switch: the AI extension loses `chrome.debugger` on **every** site, not just your bank (details in [the warning below](#the-problem)).
> There is no policy key that brings per-site debugger blocking back. The repository is archived as of v0.1.1; see [Requirements](#requirements) for the versions it still works on.
>
> **What to do instead:** give the agent its **own browser profile** (or a separate browser) that is never logged in to your bank, broker, or mailbox, and install the AI extension only there. Keep **no** AI browser-control extension in the browser you use for those critical sites. The protection then comes from separation, not from a host list — nothing to enumerate, and no browser version can undo it.

## Requirements

| Browser | Works as described | Notes |
|---|---|---|
| Google Chrome | **154 and earlier** | 155+ is all-or-nothing ([Chrome post](https://developer.chrome.com/blog/debugger-enterprise-policy-restrictions)). The Chromium policy text names 154, and 154 itself was not tested — treat 154 as uncertain. |
| Microsoft Edge | **153 and earlier** | 154+ is all-or-nothing ([Microsoft ExtensionSettings](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-browser-policies/extensionsettings): "As of Microsoft Edge version 154 …"). |

- Windows: Windows PowerShell 5.1 or PowerShell 7, admin rights to write the policy. macOS: system bash, `sudo`.
- **Temporary escape hatch:** starting the browser with `--disable-features=ExtensionDebuggerStrictPolicyRestrictions` restores the old per-host behavior, but the Chrome post says this flag **will be removed in Chrome 160**. It only applies when the browser is launched with that flag (taskbar, autostart, or a link opening the browser can bypass it), and whether Edge honors the same feature name was not verified.
- **Do not downgrade your browser to use this tool.** An old browser stops receiving security updates, which is a far bigger risk than the one this tool addresses. If you are already on an older version for other reasons, the tool still works as documented below — with the best-effort caveat in the warning.

## The problem

AI coding agents can drive your real browser through extensions (Kimi WebBridge, Claude in Chrome, and similar). Some of these use the `chrome.debugger` API with `<all_urls>` host permissions: they get **your logged-in identity on every site, all at once**.

The counter-intuitive part, confirmed by hands-on testing (2026-08-09, Edge on Windows):

> **The per-site "site access" whitelist in the extensions UI does NOT stop debugger-based extensions.** That UI only withholds `host_permissions` (content scripts, webRequest). The `debugger` API permission is granted globally — the extension can still attach to any tab, read the page, and run JavaScript, even on sites you removed from its list.

The only per-host control that actually works is the **enterprise policy layer**: `ExtensionSettings` → `runtime_blocked_hosts`. With it set, the extension gets `Cannot attach to this target` on blocked sites and works normally everywhere else. Verified in both directions on a real setup (13 blocked hosts refused, unblocked automation targets unaffected). *(Older browsers only — see the warning below.)*

> **⚠ From Chrome 155 / Edge 154, this policy no longer blocks per host.**
> Measured on Windows on 2026-10-12 with the Claude in Chrome extension:
>
> - If an extension's `runtime_blocked_hosts` is **non-empty**, its `chrome.debugger.attach()` fails on **every site**.
> - The error is `Host access is restricted by policy.`, not the older `Cannot attach to this target`.
> - The unblocked `example.com` fails too, even with only 3 bank hosts on the list.
>   Measured on Chrome 155.0.8059.39. Edge 155.0.4283.45 was not tested while the block was in place.
> - After the block is removed and the browser is fully restarted, `example.com` works again on both Chrome and Edge.
> - Screenshots fail too, with the same error (observed on Chrome on 2026-10-10).
>
> According to the official post, adding `runtime_allowed_hosts` does not help (not tested here):
> [Stricter enterprise policy enforcement for chrome.debugger in Chrome 155](https://developer.chrome.com/blog/debugger-enterprise-policy-restrictions)
>
> Official sources disagree on the first affected version: the Chrome post says Chrome 155, while
> [Microsoft's ExtensionSettings page](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-browser-policies/extensionsettings)
> and the Chromium policy definition say 154. Version 154 itself was not tested here.
>
> So on Chrome 155+ / Edge 154+, this tool has only two outcomes: the extension **cannot** use the debugger anywhere, or it can use it **everywhere**.
> "Works normally everywhere else" above and the "both directions" live test below apply to earlier versions only.
> Even there, treat per-host blocking as best-effort, not a guarantee: the same Chrome post explains that CDP
> "operates beneath the web platform's origin model, [so] origin-based filtering cannot securely restrict it" —
> which is why Chrome switched to all-or-nothing.
> macOS was not tested on 155.

This tool makes that policy a guided, reversible, one-command setup.

## What it does

1. **Detects** installed extensions holding the `debugger` permission (the risky class), across all browser profiles.
2. **Lets you pick** what to protect: a starter catalog of verified Taiwan bank / broker / Gmail login hosts, plus any host you add (paste the login-page URL — the host is extracted for you).
3. **Backs up** the current policy, then **merges** (never overwrites) your blocked hosts into `ExtensionSettings`.
4. **Verifies** by reading the policy back, and walks you through the live test: blocked site must fail to attach, normal site must still work.
5. **Unblocks** selected hosts later (need automation on a blocked site once? remove just that host, keep the rest) and **restores** any backup with one command.

## Let your AI agent run this for you

If you use an AI browser-control tool, you already have an assistant that can drive terminals — so let it guide you through its own lockdown. Tell your agent:

> Read AGENTS.md in this repo and walk me through locking my critical sites.

[AGENTS.md](AGENTS.md) scripts the whole flow for the agent: interview you about which banks/brokers/mailboxes to protect (with the hard rule that hosts come from your address bar or the verified catalog — never from the model's memory), confirm keep-open subdomains, run the tool in its non-interactive mode, and then **prove the lock works by trying to break it** — the agent attaching to a blocked host and failing is the verification. Both scripts take parameters (`-Browser/-ExtensionIds/-Hosts/-Yes` on Windows, positional `lock chrome <id> <hosts> --yes` on macOS) exactly for this.

## Usage

### Windows (Edge + Chrome) — tested

Run in an **elevated** PowerShell (the policy lives in HKLM):

```powershell
git clone https://github.com/drpwchen/kimi-webbridge-lockdown
cd kimi-webbridge-lockdown\windows
.\lockdown.ps1              # interactive menu: lock / unblock / status / restore
.\lockdown.ps1 -Action status    # read-only, no admin needed

# non-interactive (agent-guided; see AGENTS.md)
.\lockdown.ps1 -Action lock    -Browser edge -ExtensionIds <id> -Hosts host1,host2 -Yes
.\lockdown.ps1 -Action unblock -Browser edge -ExtensionIds <id> -Hosts host1 -Yes
```

After writing, **fully quit and restart the browser** (all windows and background processes), check `edge://policy` / `chrome://policy` shows `ExtensionSettings`, then live-test both directions.

### macOS (Edge + Chrome) — tested

The policy must live in `/Library/Managed Preferences`, so the script calls `sudo`:

```bash
cd kimi-webbridge-lockdown/macos
./lockdown.sh                    # interactive menu: lock / unblock / status / restore
./lockdown.sh status             # read-only, no admin needed

# non-interactive (agent-guided; see AGENTS.md)
./lockdown.sh lock    chrome <id> host1,host2 --yes
./lockdown.sh unblock chrome <id> host1 --yes
```

After writing, **fully quit and restart the browser** (Cmd+Q, all windows), open
`chrome://policy` / `edge://policy`, and confirm `ExtensionSettings` shows **level
`Mandatory`** — then live-test both directions.

> **`defaults write` does not work, and fails silently.** Measured on macOS 26.5.2 /
> Chrome 151 with a real `debugger`-permission extension (2026-08-09):
>
> | Where the policy is written | `chrome://policy` | `chrome.debugger.attach` on a blocked host |
> |---|---|---|
> | `defaults write com.google.Chrome` | level **Recommended**, status *Valid* | **succeeds** — the extension attached and read the page |
> | `/Library/Managed Preferences/…` (root:wheel, 644) | level **Mandatory**, scope *Machine* | **`Cannot attach to this target.`** |
>
> An unblocked control host kept working in both runs, so the block is precise rather
> than a broken browser. The dangerous part is the middle column: the user-defaults
> policy is listed as *Valid* on `chrome://policy`, so a naive "ExtensionSettings
> appears on the policy page" check would tell you that you are protected when you
> are not. That is why the check is "level must read Mandatory", not
> "ExtensionSettings appears".

If your Mac is MDM-managed, your MDM may overwrite `/Library/Managed Preferences`;
deliver the same `ExtensionSettings` payload through the MDM instead.

## Design decisions

- **Exact hosts, never `*.domain` wildcards.** Blocking `*.esunbank.com.tw` would also kill `card.esunbank.com.tw` — the credit-card / event-registration subdomains are often exactly what you *want* automation to keep doing. Precision is the point.
- **Merge, don't clobber.** Existing `ExtensionSettings` content (other extensions, other settings on the same extension) is preserved; blocked-host lists are unioned. Every write is preceded by a timestamped backup under `~/.ai-browser-lockdown/backups/`.
- **Catalog entries are verified, or they don't ship.** Every host in `catalog/hosts.json` was confirmed by opening the real login page and copying the host from the address bar. Search-engine memory is not a source — stale indexes list dead hosts (we found one), and personal vs corporate banking often live on different hosts.

## Limitations — read this

- **A blacklist only blocks what you enumerate.** New bank account → come back and add its host. This is inherent to the mechanism.
- **Admin on the machine can undo it.** This protects against your AI agent wandering into your bank with your logged-in session; it does not protect against malicious local software or anyone with admin rights.
- The deeper residual risk of AI browser control is **prompt injection**: an agent reading a hostile page while your critical accounts are logged in elsewhere in the same browser. This tool removes the worst targets from its reach; the habit of not mixing untrusted browsing with logged-in critical sessions is still on you.
- An airtight alternative: run automation in a **separate browser profile that is never logged in** to anything critical. More setup friction, no enumeration problem.

## Contributing catalog entries

PRs adding bank/broker host groups are welcome, with one hard rule: **every host must be verified by actually opening the login page and copying the host from the address bar**, and the entry must carry the verification date. Please also note sibling subdomains that should stay open (credit-card portals, account opening, marketing sites).

## License

MIT
