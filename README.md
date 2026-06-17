# ChatGPT Pro Codex Plugin Experiments

This repo is the focused live browser-control slice for the larger goal:
let Codex operate a persistent, logged-in ChatGPT website session through Chrome
DevTools Protocol, then turn that into a narrow plugin/tool surface.

It is not a production `ask_gpt_pro` tool yet. It is the live proof harness.
The target use is professional-level model collaboration: architecture and
system specs, feature decisions, research synthesis, planning across
agents/repos, and hard debugging strategy. ChatGPT Pro should be invited to
disagree, ask questions, propose different architecture, and name prerequisite
work when that is the right answer. Smoke prompts prove the transport; they are
not the main product.

## Demo

<video src="https://github.com/pauljunsukhan/chat-gpt-pro-codex-plugin/raw/main/docs/assets/plugin_v1_demo.mp4" controls width="100%"></video>

[Download the demo video](docs/assets/plugin_v1_demo.mp4)

## Shape

- `npm run chrome` opens `https://chatgpt.com/` in a normal Chrome window.
- The browser profile is `.devspace/chrome-profiles/chatgpt-pro`.
- CDP is exposed on `http://127.0.0.1:9222`.
- `npm run live:chatgpt` is the health check: it sends a unique smoke prompt
  and only passes when a new assistant message contains the expected token.
- `npm run chatgpt:call` is the real working path: it sends a natural prompt,
  waits for a normal assistant response, and saves prompt/answer receipts.
- `chatgpt:call` and `chatgpt:read` serialize through
  `~/.chatgpt-pro-codex/locks/browser-profile.lock/` so multiple agents do not
  write into the same ChatGPT profile at once.
- Session-registry writes serialize through
  `~/.chatgpt-pro-codex/projects/<projectId>/state.lock/` and use atomic JSON
  writes, so linked worktrees/agents do not lose room updates.
- Receipts and screenshots are written under `.devspace/runs/<run-id>/`.

Human login is deliberately visible. If ChatGPT asks for auth, complete login in
the opened Chrome window. Automation should not enter passwords, OTPs, solve
CAPTCHA, or inspect cookies/session storage.

## Quick Start

```bash
npm install
./bin/chatgpt-pro init
npm run setup:login
npm run login:check
npm run levels:list
npm run cdp:smoke
BROWSER_OBSERVER=1 npm run live:chatgpt
./bin/chatgpt-pro call --prompt-file=prompts/chatgpt-pro-system-improvement.md
```

If `login:check` or `live:chatgpt` exits with code `20` and
`auth.login_required`, finish login in the visible Chrome window and rerun:

```bash
npm run login:check
BROWSER_OBSERVER=1 npm run live:chatgpt
```

## Scripts

- `npm run chrome` launches a headed normal Chrome website window.
- `npm run chrome:headless` launches Chrome with the same target/profile in
  headless mode.
- `npm run chrome:debug` prints verbose target/profile/CDP output.
- `npm run cdp:smoke` verifies `/json/version` and `/json/list`.
- `npm run setup:login` opens/reuses the visible browser for manual login.
- `npm run login:check` verifies the profile is logged in without sending a
  prompt. It exits `20` with `auth.login_required` when manual login is needed.
- `npm run levels:list` reads the current account plan, active intelligence
  level, available intelligence labels, active model, and model labels from the
  website.
- `npm run levels:set -- --level=Pro` selects a live intelligence label.
- `npm run choices:set -- --model=5.4` selects a live model label.
- `npm run sessions:list` lists live ChatGPT website tabs.
- `npm run sessions:project` writes/prints the repo identity used for alias ownership.
- `npm run sessions:alias -- --name=main` binds the current tab to an alias.
- `npm run sessions:alias -- --name=main --rebind-alias` explicitly rebinds
  an alias to this repo when ownership has changed.
- `npm run sessions:alias -- --name=spec --rebind-alias --conversation-url=https://chatgpt.com/c/...`
  binds a pasted ChatGPT thread URL to a repo-owned alias.
- `npm run sessions:new -- --name=debug` creates a new named ChatGPT tab.
- `npm run context:bundle -- --name=focused` creates a repo monofile,
  manifest, and zip under `.devspace/context-bundles/`.
- `npm run history:export -- --alias=spec --last=20` exports visible ChatGPT
  conversation history, including user and assistant messages.
- `./bin/chatgpt-pro init` creates repo-local ChatGPT Pro state and installs
  `.codex/skills/chatgpt-pro-line/SKILL.md`.
- `./bin/chatgpt-pro context bundle --name=focused` is the packaged context
  bundle command.
- `./bin/chatgpt-pro history export --alias=spec` exports the full visible
  history for a bound ChatGPT thread.
- `./bin/chatgpt-pro call --alias=main --message-file=prompt.md` is the
  package-shaped command surface for agents.
- `./bin/chatgpt-pro call --alias=critic --fresh --message-file=prompt.md`
  opens a new unbound thread for clean review without moving the alias.
- `./bin/chatgpt-pro call --alias=critic --new --message-file=prompt.md`
  opens a new thread and binds the alias to it, preserving prior lineage.
- `./bin/chatgpt-pro read --alias=main` captures the newest response through
  the same global lock.
- `npm run chatgpt:call -- --alias=main --prompt-file=prompt.md` sends a
  conversational prompt to a named session and writes `assistant.md`; this is
  the local development alias for `chatgpt-pro call`.
- `npm run chatgpt:call -- --alias=main --message-file=prompt.md --response-mode=blocking`
  is the canonical packaged communication-line shape.
- `npm run chatgpt:call -- --context-dir=.devspace/context/current --prompt="..."`
  composes a titled context envelope from known files in that directory.
- `npm run chatgpt:read` waits for and captures the newest response in the
  current ChatGPT tab. This is useful when a long Pro response outlives an
  earlier runner.
- `npm run test:browser-lock` checks global lock acquire/wait/reclaim behavior.
- `npm run test:session-concurrency` checks concurrent project session-registry
  writes from multiple worker processes.
- `npm run test:cli-init` checks idempotent `chatgpt-pro init` behavior in a
  temp repo.
- `npm run test:message-anchor` runs deterministic fixture checks for binding
  a response to the newly sent user message.
- `npm run test:context-envelope` runs deterministic fixture checks for
  `--context-dir` section ordering, hashes, and oversized-file omission.
- `npm run test:non-interference` asserts the canonical call path does not use
  OS-level automation, CDP mouse dispatch, or CDP key dispatch.
- `npm run test:project-state` checks repo identity and session-registry
  normalization.
- `npm run live:chatgpt` runs the real website smoke proof.
- `npm run mcp:chrome` starts Chrome DevTools MCP against the same browser.
- `npm test` aliases the live smoke proof.

## Runtime Switches

- `BROWSER_TARGET_URL` defaults to `https://chatgpt.com/`.
- `BROWSER_PROFILE_NAME` defaults to `chatgpt-pro`.
- `BROWSER_POSTURE=headed|headless` controls visible vs. headless Chrome.
- `CHROME_REMOTE_DEBUGGING_PORT` defaults to `9222`.
- `BROWSER_OBSERVER=1` prints a run-inspector URL.
- `KEEP_OBSERVER=1` keeps that inspector alive after the run finishes.
- `CHATGPT_LEVEL` / `CHATGPT_INTELLIGENCE` selects a discovered intelligence
  label before a live smoke prompt.
- `CHATGPT_DEFAULT_LEVEL` defaults to `Pro` for `chatgpt-pro call`. Use
  `--no-default-pro` only for transport debugging.
- `CHATGPT_MODEL` selects a discovered model label before a live smoke prompt.
- `CHATGPT_RESPONSE_TIMEOUT_MS` controls how long to wait for the assistant
  response. Default: `240000`.
- `CHATGPT_NEW_CHAT=0` disables the default live-smoke behavior of starting a
  fresh ChatGPT page before prompting.
- `CHATGPT_NEW_CHAT_SETTLE_MS` waits after fresh-chat navigation before reading
  model/level menus. Default: `20000`.
- `CHATGPT_SESSION` selects a named ChatGPT session alias for `chatgpt:call`,
  `chatgpt:read`, and `live:chatgpt`.
- `CHATGPT_PROMPT` / `CHATGPT_PROMPT_FILE` provide the natural prompt for
  `chatgpt:call`.
- `CHATGPT_RESPONSE_MODE` defaults to `blocking`. Other modes are reserved and
  currently fail with `mode.unsupported`.
- `CHATGPT_CONTEXT_FILE` appends a text context artifact to a `chatgpt:call`
  prompt.
- `CHATGPT_CONTEXT_DIR` composes a stable context envelope from
  `codex-session-digest.md`, `chatgpt-history.md`, `repo-context.md` or
  `manifest.json`, `diff.patch`, and `test-output.txt`.
- `CHATGPT_ATTACH_REPO_CONTEXT=0` disables the default generated repo monofile
  attachment for `chatgpt-pro call`; `--no-repo-context` is the CLI equivalent.
- `CHATGPT_LOCK_TIMEOUT_MS` controls how long `chatgpt:call` / `chatgpt:read`
  wait for the global browser-profile lock. Default: `600000`.
- `CHATGPT_STALE_LOCK_TTL_MS` controls when a lock heartbeat is considered
  stale and eligible for dead-owner reclaim. Default: `900000`.

## Choice Discovery

The model and intelligence names are read from the live website, not from a
repo-local enum. Current commands:

```bash
npm run levels:list
npm run levels:set -- --level=Pro
npm run choices:set -- --model=5.4
CHATGPT_LEVEL=Pro CHATGPT_MODEL=5.5 BROWSER_OBSERVER=1 npm run live:chatgpt
```

The list output separates two meanings of "Pro":

- `account.planLabel` / `account.isPro`: subscription/account state read from
  the profile menu.
- `intelligence.options`: live thinking/intelligence choices exposed by the
  composer dropdown.

Live smoke starts a fresh ChatGPT page by default so proof prompts do not
inherit old chat context. For real collaboration, use named rooms:

```bash
npm run sessions:alias -- --name=main
npm run chatgpt:call -- --alias=main --prompt-file=prompt.md
CHATGPT_SESSION=main npm run chatgpt:read
```

Use an existing alias when prior reasoning matters. Use a fresh chat for clean
critique, independent work, or health checks.

Aliases are repo-owned. The local project pointer is stored in
`.devspace/state/chatgpt-project.json`; git repos also store
`chatgpt-pro.projectId` in local git config so linked worktrees share the same
rooms. Alias records live in
`~/.chatgpt-pro-codex/projects/<projectId>/chatgpt-sessions.json` and include
`projectId`, `repoRoot`, room lineage, recent runs, and fresh-thread records.
If an alias belongs to a different repo, calls fail with
`session.alias_project_mismatch` unless `--rebind-alias` is supplied.

Room/thread policy:

```bash
./bin/chatgpt-pro call --alias=main --prompt="..."          # continue main
./bin/chatgpt-pro call --alias=debug --prompt-file=bug.md   # focused failure
./bin/chatgpt-pro call --alias=critic --fresh --prompt="..." # clean unbound review
./bin/chatgpt-pro call --alias=critic --new --prompt="..."   # replace critic room
./bin/chatgpt-pro call --alias=scratch --fresh --prompt="..." # disposable
```

`--fresh` records a new thread without mutating the alias. `--new` requires an
alias and archives the previous active thread for that alias in lineage.

Do not open a fresh tab for every normal call. For multi-agent use, create or
bind a repo-owned alias for that agent/task, then reuse it:

```bash
./bin/chatgpt-pro call --alias=critic --new --prompt="Start a clean critic room."
./bin/chatgpt-pro call --alias=critic --prompt-file=next-review.md
```

The browser/profile remains globally locked while a call is active, but the
conversation room is selected per repo-owned alias.

## Conversational Calls

`chatgpt:call` is the current prototype of the future plugin surface. It writes:

- `prompt.md`
- `assistant.md`
- `transcript.md` with titled sent/received sections for session-history capture
- `receipt.json` and `receipt.md`
- `final.png`
- `snapshot.json`
- `console.json`
- `network.json`

The call receipt includes prompt hashes, response hashes, detected Pro/model
state, desired intelligence/model, lock owner/wait/held data, conversation URL,
response character count, the completion detector used, and
`messageAnchor.responseBoundToSentPrompt`. Alias calls also include
`roomTarget.targetBoundToRoom` so a caller can prove the browser target matched
the repo room before text was sent or read.
By default, `chatgpt:call` prints the exact `## Message Sent To ChatGPT Pro` /
`## Message Received From ChatGPT Pro` block that should be pasted into the
Codex session. Do not summarize, paraphrase, trim, or rewrite that block.
Set `CHATGPT_THREAD_ECHO=0` only for machine consumers.
Receipts include `threadEcho` hashes for the transcript, sent prompt, and
received message. Verify them with:

```bash
./bin/chatgpt-pro transcript verify --receipt=.devspace/runs/<run-id>/receipt.json
```
The response reader deliberately waits through transient states like
`Pro thinking` and `Stop answering`.

The call path is text-only. It does not use ChatGPT voice, dictation,
microphone, audio capture, or spoken commands.

Packaged agent behavior lives in
[.codex/skills/chatgpt-pro-line/SKILL.md](.codex/skills/chatgpt-pro-line/SKILL.md).
Agents using the line should repeat the same headings in the live Codex thread:

```md
## Message Sent To ChatGPT Pro

...

## Message Received From ChatGPT Pro

...
```

For now the line is blocking: Codex sends, waits, reads, records the transcript,
then continues. Event/interrupt mode is reserved but intentionally not
implemented.

When `--context-dir` is supplied, `chatgpt:call` includes recognized files in a
fixed order and records included/omitted files, hashes, sizes, and section
budgets in `receipt.json`. Oversized files are represented by path/hash/size
instead of pasted wholesale.

When no `--context-dir` is supplied, `chatgpt:call` generates a repo monofile by
default. The bundle includes:

- `repo-context.md`: repo structure, LOC, file contents, and source-map tables
- `manifest.json`: machine-readable file and directory records with line ranges
  into `repo-context.md`
- `repo-context.zip`: zip of both artifacts

For human-driven ChatGPT work, bind the thread URL and export its visible
history into the repo context space:

```bash
./bin/chatgpt-pro sessions alias --name=spec --rebind-alias --conversation-url=https://chatgpt.com/c/...
./bin/chatgpt-pro history export --alias=spec --last=20
./bin/chatgpt-pro history export --alias=spec
```

The exporter writes `chatgpt-history.md` and `history.json` under
`.devspace/chatgpt-history/<alias>/...`; pass that directory with
`--context-dir` when the next Pro call should work from the human/ChatGPT
conversation.

See [docs/chatgpt-call-contract.md](docs/chatgpt-call-contract.md) for the
session-history digest, context tiers, session-room model, and next failure
codes.

## Artifacts

Each live run writes:

- `receipt.json` and `receipt.md`
- `run.json`
- `final.png`
- `snapshot.json`
- `console.json`
- `network.json`

The observer link shows current phase, target, CDP URL, artifacts, console
warnings/errors, and failed requests.

## Codex MCP Integration

The project `.codex/config.toml` starts `chrome-devtools-mcp` against
`http://127.0.0.1:9222`. Start Chrome first:

```bash
npm run chrome
```

Then a Codex subagent can use the `chrome_devtools` MCP server to operate that
same visible browser profile.

## Current Boundary

Proven:

- persistent dedicated ChatGPT website profile
- visible manual login boundary
- account plan detection (`account.isPro`)
- live intelligence/model choice discovery and selection
- CDP attach to the live website
- prompt send
- fresh assistant-output readback
- conversational `chatgpt:call` with prompt/answer artifacts
- current-response recovery with `chatgpt:read`
- message anchoring from sent user prompt to following assistant response
- global browser-profile lock around live call/read
- per-project state lock and atomic writes around room registry mutation
- packaged `chatgpt-pro-line` skill contract
- repo-scoped alias ownership with `projectId`
- git worktree-shared room identity and separate-repo isolation
- room target verification/repair by ChatGPT conversation URL
- default repo monofile context bundles with manifest/source-map line ranges
- visible ChatGPT conversation history export by alias
- non-interference check for the canonical call path
- receipt/screenshot/console/network artifacts

Still to design:

- file upload into the ChatGPT composer
- automatic Codex session-history digest extraction from the host thread
- plugin-global browser registry and `chatgpt:doctor`
- error taxonomy for rate limits, auth expiry, and model-side refusals

## Failure Codes

- `auth.login_required`, process exit `20`: the persistent profile needs human
  login. Run `npm run setup:login`, complete auth in the visible browser, then
  run `npm run login:check`.
- `chatgpt.composer_missing`, process exit `1`: ChatGPT loaded but the composer
  is not visible or selectors drifted.
- `chatgpt.response_timeout`, process exit `1`: the prompt was sent, but no
  matching assistant token stabilized before `CHATGPT_RESPONSE_TIMEOUT_MS`.
- `lock.busy`, process exit `1`: another run owns the browser profile lock and
  this run was configured not to wait.
- `lock.timeout`, process exit `1`: another run did not release the browser
  profile lock before `CHATGPT_LOCK_TIMEOUT_MS`.
- `state_lock.busy`, process exit `1`: another process owns the project
  session-registry lock and this run was configured not to wait.
- `state_lock.timeout`, process exit `1`: another process did not release the
  project session-registry lock before the state lock timeout.
- `composer.input_mismatch`, process exit `1`: ChatGPT's composer did not
  contain the intended prompt after insertion, so the command refused to send.
- `chrome.cdp_unavailable`, process exit `1`: Chrome/CDP is not reachable.
- `session.alias_project_mismatch`, process exit `1`: the requested alias
  belongs to another repo. Use a different alias or pass `--rebind-alias`
  deliberately.
