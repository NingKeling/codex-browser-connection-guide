# Codex browser connection troubleshooting

[简体中文](README.zh-CN.md)

Browser works manually, but Codex cannot open or read pages? Start with the prompt below if you use Windows and a local HTTP system proxy. For other symptoms, use the table further down or give your AI the [standalone diagnostic reference](AI-REFERENCE.md).

This guide shares a workaround verified on one Windows + Edge setup. Similar errors can have different causes.

## 1. Copy this repair prompt

Use this branch when a local HTTP system proxy is enabled and Codex reports either of these errors:

- `nodeRepl.fetch request failed`
- `Unable to load browser request-header policy`

Copy the whole block into Codex and replace the error placeholder:

```text
I use Codex on Windows. My browser can access websites manually, but Codex browser operations fail with: [paste the exact error].

First check whether the browser-tool subprocess actually receives and uses my current local HTTP system proxy. Do not infer the cause from the error text alone. If the evidence points to missing proxy propagation, read the active system proxy address and port, and set HTTP_PROXY, HTTPS_PROXY, and NO_PROXY only for browser-related subprocesses.

If persistence requires a custom user-level launcher, create it locally with reviewable source before enabling it through CODEX_NODE_REPL_PATH. Select the official runtime using the app-provided NODE_REPL_NODE_PATH. Confirm that this build supports those interfaces; stop this branch if it does not. Retain runtime path and signature validation, timely stdin/stdout/stderr forwarding, arguments, and exit codes. Preserve existing proxy exclusions that this environment needs.

Before changing anything, record original values and a rollback procedure. Keep the original app package, authentication, TLS checks, browser permissions, and existing approval boundaries intact.

Work within my authorized scope and available tools. Tell me how to restart after saving my work. Once I have fully quit and reopened Codex from its normal shortcut, use the actual browser tool to open a public HTTPS test page, read its title and visible text, then close only that test tab and confirm it disappeared. Preserve my existing tabs. Report open, read, and close separately; do not claim success from configuration checks alone.
```

**What worked in the original case:** on October 15, 2026, a locally created adaptive launcher supplied the active proxy to browser-related subprocesses. After restarting Codex from its normal shortcut, the real Edge browser tool opened an HTTPS page, read its content, and closed the test tab.

Setting `CODEX_NODE_REPL_PATH` alone does not create a launcher. This is a guide for diagnosis and local adaptation, rather than a ready-made installer. The observed launcher hooks are version-dependent interfaces, not a documented compatibility promise for every release.

If the extension is missing or disabled, first follow the [official browser connection checks](https://learn.chatgpt.com/docs/chrome-extension#troubleshooting).

## 2. Restart and verify

Save your work, fully quit Codex, and reopen it from its normal shortcut. Then send:

```text
Using the current official browser tool and my chosen browser, create a temporary tab and open a public HTTPS page that I can access manually and the tool is permitted to visit.

Report the actual URL, page title, and a short passage of visible text. Close only the test tab, then confirm it is gone. Preserve existing tabs.

Report open, read, and close separately. If a step fails, retain the exact error and identify the earliest failed step. After a timeout, check whether the tab already exists before creating another.
```

The test passes only when all three page operations succeed after restart. A working extension side chat, process list, launcher receipt, or command-line HTTP request is supporting evidence, not browser acceptance.

## 3. Undo the change

Ask Codex to restore the recorded original user-level `CODEX_NODE_REPL_PATH` value and any other changes made during this repair. Remove that variable only if it did not exist beforehand. Save work and fully restart the app afterward.

If Codex cannot start, search Windows for "Edit environment variables for your account" and restore the user variable from your backup. If Windows still supplies the old value, save other work before signing out and back in.

## 4. Other errors and where to start

These are diagnostic directions, not claims that every report has the same cause.

| Error or symptom | Start here | Evidence |
| --- | --- | --- |
| Browser missing from the list or @ menu | Check the active browser profile, extension, and Settings → Computer Use. The browser should show Manage and its toggle should be on. | [Official guidance](https://learn.chatgpt.com/docs/chrome-extension#troubleshooting). |
| Extension installed, connection still fails | Try a new chat, restart the browser and app, and check updates. If needed, reinstall through Computer Use. | Official guidance; successful setup still needs a page test. |
| `Unable to load browser request-header policy` | With proxy-related evidence, check the browser subprocess environment. Otherwise retain the initialization error for diagnosis. | Original case verified the proxy branch; [#45678](https://github.com/openai/codex/issues/45678) reports the error without establishing its cause. |
| `Timed out waiting for Statsig values` | Retain initialization and network errors; do not assume a proxy cause. | Reference direction from earlier diagnostic material; no independent fix verified in the original case. |
| `nodeRepl.fetch request failed` during aggregate browser discovery | If current tool documentation supports it, select the intended browser directly, then test tab listing and page reading separately. | [Another user's workaround](https://github.com/openai/codex/issues/45340#issuecomment-5658617066); untested in the original case. |
| `js execution timed out; kernel reset, rerun your request` | Check whether the page was created; distinguish tab metadata from actual page access. | [A report where the page existed but access still timed out](https://github.com/openai/codex/issues/45340#issuecomment-5846770054). |
| In-app browser returns `browsers=[]` before any tab exists | Try a new chat, full restart, and update check; retain initialization errors if it persists. | [macOS in-app browser report](https://github.com/openai/codex/issues/53062); the Windows launcher is not a macOS fix. |
| Only one site fails, or access is denied | Check site restrictions and normal authorization. Compare another permitted public page only when access is allowed. | Official guidance and diagnostic comparison. |
| Works once, fails after restart or upgrade | Compare the effective launcher, proxy environment, and app-selected runtime before and after restart. Check for configuration regeneration. | [Configuration-overwrite report](https://github.com/openai/codex/issues/52956). Its app-repackaging method is separate from this guide. |

If unresolved, use `/feedback` in the app. Include versions, the exact error, the failed step, and the chat ID; sanitize relevant logs before sharing, following the official guidance.

## 5. Scope of the verified workaround

- Tested: Windows, Edge, and a single credential-free loopback HTTP system proxy. The recorded Windows package was `OpenAI.Codex 26.1007.9446.0`; this is not an About-dialog or CLI version.
- Encountered locally: both `nodeRepl.fetch request failed` and `Unable to load browser request-header policy`.
- Verified on October 15, 2026: normal-shortcut restart followed by opening, reading, and closing a real page.
- Unsupported by the original launcher: PAC, per-protocol proxy mappings, and proxy credentials. It required the local system proxy to remain enabled.
- The original implementation replaced `NO_PROXY` with `localhost,127.0.0.1,::1`; it did not merge existing exclusions. Adaptations must account for other exclusions the user's environment needs.
- A small-message buffering bug in the custom launcher was also fixed. That was a defect in the custom launcher, not an established upstream product defect.
- Compatibility after a real subsequent official app upgrade has not been verified. Current-runtime and simulated path-selection checks do not establish future compatibility.

For detailed implementation constraints and source links, see the [AI reference](AI-REFERENCE.md). The [original Chinese guide](https://wcnb8iaajf94.feishu.cn/docx/DcEyd4sCHokAUZxdqSvc1iw7nhc) is optional background; the English instructions here stand on their own.
