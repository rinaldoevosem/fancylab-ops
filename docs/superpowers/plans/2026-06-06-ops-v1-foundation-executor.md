# fancylab-ops v1 — Plan 1 of 3: Repo Foundation + Email Executor

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish the `fancylab-ops` repo conventions (labels, directory layout, Bee brief stubs) and build the local Email Executor that, when an `action:email-reply` PR is merged, sends the reply via Gmail **exactly once** and closes the issue.

**Architecture:** The executor is a small standalone TypeScript program in `executor/` with its own deps and `.env`. It is built as pure, dependency-injected logic (parsers + an idempotency state machine) wrapped by thin GitHub (`@octokit/rest`) and Gmail (`googleapis`) adapters, driven by a poll loop. Pure logic is unit-tested with hand-rolled fakes via Node's built-in test runner (no test-framework dependency); adapters get creds-gated smoke checks. Exactly-once is enforced by a label state machine: a durable `sending` intent label is written **before** the send and a `sent` label **immediately after**; a PR found stuck in `sending` halts and alerts rather than auto-resending.

**Tech Stack:** TypeScript, `tsx` (run/loader), Node built-in `node:test`, `@octokit/rest`, `googleapis`, `dotenv`. npm. The `fancylab-ops` repo (separate from the hub).

**Repo workflow:** This work lands via a feature branch → PR → Rinaldo merges. Never commit to `main` directly.

**Prerequisite gate:** The live Gmail send (Task 9 smoke + real end-to-end) does not work until a `gmail.send`-scoped refresh token exists in `executor/.env`. Unit tests (Tasks 3–6) do **not** need it and should be built first.

---

## File Structure

```
fancylab-ops/
├── .github/
│   └── labels.json              # canonical label set (source of truth)
├── scripts/
│   └── setup-labels.sh          # idempotently applies labels.json via gh
├── bees/                        # Bee briefs (stubs now; filled in Plan 2)
│   ├── README.md
│   ├── email-scout.md
│   ├── forager.md
│   ├── architect.md
│   ├── builder.md
│   ├── worker.md
│   └── guard.md
├── sops/
│   └── README.md                # seed SOP folder Forager reads
├── outbox/
│   └── .gitkeep                 # per-flight reply files land here: outbox/NNNN-slug/reply.md
├── executor/
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env.example
│   ├── .gitignore
│   ├── src/
│   │   ├── types.ts             # ReplyDoc, ports, decisions
│   │   ├── parse-branch.ts      # parseIssueFromBranch()
│   │   ├── parse-reply.ts       # parseReplyDoc()
│   │   ├── decide.ts            # decideSend() state machine
│   │   ├── process.ts           # processReplyPR() orchestration (DI)
│   │   ├── github.ts            # Octokit adapter implementing GitHubPort
│   │   ├── gmail.ts             # googleapis adapter implementing GmailPort
│   │   └── poll.ts              # entrypoint: loop → process each merged reply PR
│   └── test/
│       ├── parse-branch.test.ts
│       ├── parse-reply.test.ts
│       ├── decide.test.ts
│       └── process.test.ts
└── docs/… (spec + this plan)
```

**Responsibilities:** `parse-*` and `decide` are pure functions (no I/O, fully unit-tested). `process.ts` is the orchestration that enforces exactly-once, tested with fakes. `github.ts`/`gmail.ts` are the only files that touch the network. `poll.ts` wires them together.

---

### Task 1: Repo foundation — labels, directories, brief stubs

**Files:**
- Create: `.github/labels.json`, `scripts/setup-labels.sh`, `sops/README.md`, `outbox/.gitkeep`, `bees/README.md` + 6 brief stubs, root `.gitignore`

- [ ] **Step 1: Create the feature branch**

```bash
cd /Users/melo/fancylab/fancylab-ops
git checkout -b plan1/foundation-executor
```

- [ ] **Step 2: Write the canonical label set**

Create `.github/labels.json`:

```json
[
  { "name": "source:email",       "color": "1d76db", "description": "Signal came from email" },
  { "name": "source:clickup",     "color": "1d76db", "description": "Signal came from ClickUp" },
  { "name": "source:slack",       "color": "1d76db", "description": "Signal came from Slack" },
  { "name": "facing:client",      "color": "5319e7", "description": "Client-facing" },
  { "name": "facing:internal",    "color": "5319e7", "description": "Team-facing" },
  { "name": "level:production",   "color": "0e8a16", "description": "Business operations" },
  { "name": "level:meta",         "color": "0e8a16", "description": "Hub/platform itself" },
  { "name": "priority:p1",        "color": "b60205", "description": "Urgent" },
  { "name": "priority:p2",        "color": "d93f0b", "description": "Soon" },
  { "name": "priority:p3",        "color": "fbca04", "description": "Whenever" },
  { "name": "action:email-reply", "color": "006b75", "description": "Has a live send-on-merge executor" },
  { "name": "action:manual",      "color": "c5def5", "description": "Watch-only; Rinaldo acts manually" },
  { "name": "action:none",        "color": "c5def5", "description": "No action needed" },
  { "name": "guard:pass",         "color": "0e8a16", "description": "Guard verdict: PASS" },
  { "name": "guard:block",        "color": "b60205", "description": "Guard verdict: BLOCK" },
  { "name": "sending",            "color": "fef2c0", "description": "Executor: send in progress (intent marker)" },
  { "name": "sent",               "color": "0e8a16", "description": "Executor: reply sent (done marker)" },
  { "name": "fyi",                "color": "ededed", "description": "No reply needed; auto-closed" }
]
```

- [ ] **Step 3: Write the idempotent label setup script**

Create `scripts/setup-labels.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
REPO="${REPO:-rinaldoevosem/fancylab-ops}"
FILE="$(dirname "$0")/../.github/labels.json"
count=$(jq 'length' "$FILE")
for i in $(seq 0 $((count-1))); do
  name=$(jq -r ".[$i].name" "$FILE")
  color=$(jq -r ".[$i].color" "$FILE")
  desc=$(jq -r ".[$i].description" "$FILE")
  # create, or update if it already exists (idempotent)
  gh label create "$name" --color "$color" --description "$desc" --repo "$REPO" 2>/dev/null \
    || gh label edit "$name" --color "$color" --description "$desc" --repo "$REPO"
  echo "ok: $name"
done
```

- [ ] **Step 4: Make it executable and run it**

Run:
```bash
chmod +x scripts/setup-labels.sh && ./scripts/setup-labels.sh
```
Expected: one `ok: <label>` line per label; no errors.

- [ ] **Step 5: Verify the labels exist on the remote**

Run: `gh label list --repo rinaldoevosem/fancylab-ops | grep -E 'action:email-reply|sending|sent'`
Expected: all three present.

- [ ] **Step 6: Create directory scaffolding and stubs**

Create `outbox/.gitkeep` (empty). Create `sops/README.md`:

```markdown
# SOPs (seed)

Standard operating procedures the Forager reads as source-of-truth when enriching an issue.
v1 is a seed; the full knowledge base is a later subsystem. Add one file per procedure,
named `NN-topic.md`, e.g. `01-client-reply-tone.md`.
```

Create `bees/README.md`:

```markdown
# Bee briefs

One file per Bee (Purple Bee role). v1 fills these in Plan 2. Each brief states the Bee's
single job, the artifact it reads, the artifact it writes, and its acceptance criteria.
See `../docs/specs/2026-06-06-fancylab-ops-design.md` §2.
```

Create six stub files `bees/{email-scout,forager,architect,builder,worker,guard}.md`, each containing exactly:

```markdown
# <Bee name> — brief

> STUB. Filled in Plan 2 (Email Flight + Guard). See docs/specs/2026-06-06-fancylab-ops-design.md.

**One job:** TBD in Plan 2.
```

(The `TBD` here is intentional — these are deliberate stubs scheduled for Plan 2, not plan placeholders.)

- [ ] **Step 7: Root .gitignore**

Create `.gitignore`:
```
node_modules/
*.log
.DS_Store
executor/.env
```

- [ ] **Step 8: Commit**

```bash
git add .github scripts sops outbox bees .gitignore
git commit -m "Repo foundation: labels, dirs, Bee brief stubs"
```

---

### Task 2: Executor project scaffold

**Files:**
- Create: `executor/package.json`, `executor/tsconfig.json`, `executor/.env.example`, `executor/.gitignore`

- [ ] **Step 1: package.json**

Create `executor/package.json`:

```json
{
  "name": "fancylab-ops-executor",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "node --import tsx --test test/*.test.ts",
    "poll": "tsx src/poll.ts",
    "poll:once": "ONCE=1 tsx src/poll.ts"
  },
  "dependencies": {
    "@octokit/rest": "^21.0.0",
    "dotenv": "^16.4.0",
    "googleapis": "^144.0.0"
  },
  "devDependencies": {
    "tsx": "^4.19.0",
    "typescript": "^5.6.0"
  }
}
```

- [ ] **Step 2: tsconfig.json**

Create `executor/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "types": ["node"]
  },
  "include": ["src", "test"]
}
```

- [ ] **Step 3: .env.example and .gitignore**

Create `executor/.env.example`:
```
# GitHub
GITHUB_TOKEN=
OPS_REPO=rinaldoevosem/fancylab-ops
# Gmail (send-scoped refresh token — see prerequisite)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REFRESH_TOKEN=
GMAIL_SENDER=rinaldo@evosem.com
# Poll cadence (ms)
POLL_INTERVAL_MS=120000
```

Create `executor/.gitignore`:
```
node_modules/
.env
```

- [ ] **Step 4: Install deps**

Run: `cd executor && npm install`
Expected: completes; `node_modules/` present.

- [ ] **Step 5: Commit**

```bash
git add executor/package.json executor/tsconfig.json executor/.env.example executor/.gitignore executor/package-lock.json
git commit -m "Executor scaffold: package, tsconfig, env template"
```

---

### Task 3: `parseIssueFromBranch` (pure, TDD)

**Files:**
- Create: `executor/src/parse-branch.ts`, `executor/test/parse-branch.test.ts`

- [ ] **Step 1: Write the failing test**

Create `executor/test/parse-branch.test.ts`:

```ts
import { test } from "node:test";
import assert from "node:assert/strict";
import { parseIssueFromBranch } from "../src/parse-branch.ts";

test("extracts issue number and slug from ops branch", () => {
  assert.deepEqual(parseIssueFromBranch("ops/42-acme-quote"), { issueNumber: 42, slug: "42-acme-quote" });
});

test("handles multi-digit and hyphenated slugs", () => {
  assert.deepEqual(parseIssueFromBranch("ops/1057-re-follow-up-on-invoice"),
    { issueNumber: 1057, slug: "1057-re-follow-up-on-invoice" });
});

test("returns null for non-ops branches", () => {
  assert.equal(parseIssueFromBranch("main"), null);
  assert.equal(parseIssueFromBranch("feature/x"), null);
  assert.equal(parseIssueFromBranch("ops/no-number"), null);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd executor && npm test`
Expected: FAIL — cannot find module `../src/parse-branch.ts`.

- [ ] **Step 3: Write minimal implementation**

Create `executor/src/parse-branch.ts`:

```ts
export function parseIssueFromBranch(branch: string): { issueNumber: number; slug: string } | null {
  const m = branch.match(/^ops\/(\d+)(?:-[\w-]+)?$/);
  if (!m) return null;
  const slug = branch.slice("ops/".length);
  return { issueNumber: Number(m[1]), slug };
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `cd executor && npm test`
Expected: PASS (3 tests).

- [ ] **Step 5: Commit**

```bash
git add executor/src/parse-branch.ts executor/test/parse-branch.test.ts
git commit -m "Executor: parseIssueFromBranch (TDD)"
```

---

### Task 4: `parseReplyDoc` (pure, TDD)

**Files:**
- Create: `executor/src/types.ts`, `executor/src/parse-reply.ts`, `executor/test/parse-reply.test.ts`

- [ ] **Step 1: Define types**

Create `executor/src/types.ts`:

```ts
export interface ReplyDoc {
  to: string;
  subject: string;
  threadId: string;
  inReplyTo?: string;
  body: string;
}

export interface MergedPR {
  prNumber: number;
  issueNumber: number;
  slug: string;
  mergeSha: string;
}

export interface GitHubPort {
  listMergedReplyPRs(): Promise<MergedPR[]>;
  getIssueLabels(issueNumber: number): Promise<string[]>;
  addLabel(issueNumber: number, label: string): Promise<void>;
  removeLabel(issueNumber: number, label: string): Promise<void>;
  closeIssue(issueNumber: number): Promise<void>;
  comment(issueNumber: number, body: string): Promise<void>;
  getFileAtRef(path: string, ref: string): Promise<string>;
}

export interface GmailPort {
  send(msg: ReplyDoc): Promise<{ messageId: string }>;
}

export type SendDecision = "skip-already-sent" | "halt-stuck-sending" | "proceed";
```

- [ ] **Step 2: Write the failing test**

Create `executor/test/parse-reply.test.ts`:

```ts
import { test } from "node:test";
import assert from "node:assert/strict";
import { parseReplyDoc } from "../src/parse-reply.ts";

const SAMPLE = `---
to: jane@acme.com
subject: "Re: Q3 quote"
threadId: 18f2abc
inReplyTo: <msg-9@mail.acme.com>
---
Hi Jane,

Thanks for the note — confirming the Q3 scope is good on our side.

Best,
Rinaldo`;

test("parses front-matter and body", () => {
  const doc = parseReplyDoc(SAMPLE);
  assert.equal(doc.to, "jane@acme.com");
  assert.equal(doc.subject, "Re: Q3 quote");
  assert.equal(doc.threadId, "18f2abc");
  assert.equal(doc.inReplyTo, "<msg-9@mail.acme.com>");
  assert.ok(doc.body.startsWith("Hi Jane,"));
  assert.ok(doc.body.includes("Best,\nRinaldo"));
});

test("throws when a required field is missing", () => {
  const bad = `---\nto: x@y.com\n---\nhi`;
  assert.throws(() => parseReplyDoc(bad), /missing required field: subject/);
});

test("throws when front-matter fence is absent", () => {
  assert.throws(() => parseReplyDoc("no front matter here"), /missing front-matter/);
});
```

- [ ] **Step 3: Run test to verify it fails**

Run: `cd executor && npm test`
Expected: FAIL — cannot find module `../src/parse-reply.ts`.

- [ ] **Step 4: Write minimal implementation**

Create `executor/src/parse-reply.ts`:

```ts
import type { ReplyDoc } from "./types.ts";

function stripQuotes(v: string): string {
  const t = v.trim();
  return (t.startsWith('"') && t.endsWith('"')) || (t.startsWith("'") && t.endsWith("'"))
    ? t.slice(1, -1)
    : t;
}

export function parseReplyDoc(markdown: string): ReplyDoc {
  const m = markdown.match(/^---\n([\s\S]*?)\n---\n([\s\S]*)$/);
  if (!m) throw new Error("missing front-matter (expected --- fenced block)");
  const [, fm, body] = m;
  const fields: Record<string, string> = {};
  for (const line of fm.split("\n")) {
    const idx = line.indexOf(":");
    if (idx === -1) continue;
    fields[line.slice(0, idx).trim()] = stripQuotes(line.slice(idx + 1));
  }
  for (const req of ["to", "subject", "threadId"]) {
    if (!fields[req]) throw new Error(`missing required field: ${req}`);
  }
  return {
    to: fields.to,
    subject: fields.subject,
    threadId: fields.threadId,
    inReplyTo: fields.inReplyTo || undefined,
    body: body.trim(),
  };
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd executor && npm test`
Expected: PASS (parse-branch + parse-reply tests).

- [ ] **Step 6: Commit**

```bash
git add executor/src/types.ts executor/src/parse-reply.ts executor/test/parse-reply.test.ts
git commit -m "Executor: types + parseReplyDoc (TDD)"
```

---

### Task 5: `decideSend` idempotency state machine (pure, TDD)

**Files:**
- Create: `executor/src/decide.ts`, `executor/test/decide.test.ts`

- [ ] **Step 1: Write the failing test**

Create `executor/test/decide.test.ts`:

```ts
import { test } from "node:test";
import assert from "node:assert/strict";
import { decideSend } from "../src/decide.ts";

test("already sent -> skip", () => {
  assert.equal(decideSend(["action:email-reply", "sent"]), "skip-already-sent");
});

test("sent wins even if sending also present", () => {
  assert.equal(decideSend(["sending", "sent"]), "skip-already-sent");
});

test("stuck in sending (crashed mid-send) -> halt, never auto-resend", () => {
  assert.equal(decideSend(["action:email-reply", "sending"]), "halt-stuck-sending");
});

test("clean -> proceed", () => {
  assert.equal(decideSend(["action:email-reply", "guard:pass"]), "proceed");
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd executor && npm test`
Expected: FAIL — cannot find module `../src/decide.ts`.

- [ ] **Step 3: Write minimal implementation**

Create `executor/src/decide.ts`:

```ts
import type { SendDecision } from "./types.ts";

// Exactly-once guard. `sent` is the done-marker; `sending` is the intent-marker
// written BEFORE the send. A PR found in `sending` without `sent` crashed
// mid-send and MUST NOT auto-resend — a human confirms whether it went out.
export function decideSend(labels: string[]): SendDecision {
  if (labels.includes("sent")) return "skip-already-sent";
  if (labels.includes("sending")) return "halt-stuck-sending";
  return "proceed";
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `cd executor && npm test`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add executor/src/decide.ts executor/test/decide.test.ts
git commit -m "Executor: decideSend state machine (TDD)"
```

---

### Task 6: `processReplyPR` orchestration (DI, TDD — the exactly-once core)

**Files:**
- Create: `executor/src/process.ts`, `executor/test/process.test.ts`

- [ ] **Step 1: Write the failing test (fakes record call order)**

Create `executor/test/process.test.ts`:

```ts
import { test } from "node:test";
import assert from "node:assert/strict";
import { processReplyPR } from "../src/process.ts";
import type { GitHubPort, GmailPort, MergedPR, ReplyDoc } from "../src/types.ts";

const PR: MergedPR = { prNumber: 7, issueNumber: 42, slug: "42-acme", mergeSha: "abc" };
const REPLY = `---\nto: a@b.com\nsubject: Re: hi\nthreadId: t1\n---\nbody text`;

function fakeGitHub(initialLabels: string[]) {
  const calls: string[] = [];
  let labels = [...initialLabels];
  const gh: GitHubPort = {
    async listMergedReplyPRs() { return [PR]; },
    async getIssueLabels() { return labels; },
    async addLabel(_i, l) { calls.push(`add:${l}`); labels.push(l); },
    async removeLabel(_i, l) { calls.push(`remove:${l}`); labels = labels.filter(x => x !== l); },
    async closeIssue() { calls.push("close"); },
    async comment() { calls.push("comment"); },
    async getFileAtRef() { return REPLY; },
  };
  return { gh, calls };
}

function fakeGmail(opts: { throws?: boolean } = {}) {
  const calls: string[] = [];
  const gmail: GmailPort = {
    async send(_m: ReplyDoc) {
      calls.push("send");
      if (opts.throws) throw new Error("smtp boom");
      return { messageId: "m1" };
    },
  };
  return { gmail, calls };
}

test("happy path: sending added BEFORE send, sent added AFTER send, then close", async () => {
  const { gh, calls: g } = fakeGitHub(["action:email-reply", "guard:pass"]);
  const { gmail, calls: m } = fakeGmail();
  const alerts: string[] = [];
  const res = await processReplyPR(PR, gh, gmail, async (msg) => { alerts.push(msg); });
  assert.equal(res.status, "sent");
  // order: add:sending must come before send; add:sent after send; close last
  assert.deepEqual(g, ["add:sending", "add:sent", "comment", "remove:sending", "close"]);
  assert.deepEqual(m, ["send"]);
  assert.deepEqual(alerts, []);
});

test("already sent -> no send, no alert", async () => {
  const { gh, calls: g } = fakeGitHub(["sent"]);
  const { gmail, calls: m } = fakeGmail();
  const res = await processReplyPR(PR, gh, gmail, async () => {});
  assert.equal(res.status, "skipped");
  assert.deepEqual(m, []);
  assert.deepEqual(g, []); // no label churn
});

test("stuck in sending -> halt + alert, never sends", async () => {
  const { gh } = fakeGitHub(["sending"]);
  const { gmail, calls: m } = fakeGmail();
  const alerts: string[] = [];
  const res = await processReplyPR(PR, gh, gmail, async (msg) => { alerts.push(msg); });
  assert.equal(res.status, "halted");
  assert.deepEqual(m, []);
  assert.equal(alerts.length, 1);
});

test("send throws -> leaves sending, never marks sent, alerts, does not close", async () => {
  const { gh, calls: g } = fakeGitHub(["action:email-reply"]);
  const { gmail } = fakeGmail({ throws: true });
  const alerts: string[] = [];
  const res = await processReplyPR(PR, gh, gmail, async (msg) => { alerts.push(msg); });
  assert.equal(res.status, "error");
  assert.ok(g.includes("add:sending"));
  assert.ok(!g.includes("add:sent"));
  assert.ok(!g.includes("close"));
  assert.equal(alerts.length, 1);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd executor && npm test`
Expected: FAIL — cannot find module `../src/process.ts`.

- [ ] **Step 3: Write minimal implementation**

Create `executor/src/process.ts`:

```ts
import { decideSend } from "./decide.ts";
import { parseReplyDoc } from "./parse-reply.ts";
import type { GitHubPort, GmailPort, MergedPR } from "./types.ts";

export type ProcessResult =
  | { status: "sent"; messageId: string }
  | { status: "skipped" }
  | { status: "halted" }
  | { status: "error"; error: string };

type Alert = (message: string) => Promise<void>;

export async function processReplyPR(
  pr: MergedPR,
  gh: GitHubPort,
  gmail: GmailPort,
  alert: Alert,
): Promise<ProcessResult> {
  const labels = await gh.getIssueLabels(pr.issueNumber);
  const decision = decideSend(labels);
  if (decision === "skip-already-sent") return { status: "skipped" };
  if (decision === "halt-stuck-sending") {
    await alert(`Issue #${pr.issueNumber} (PR #${pr.prNumber}) stuck in 'sending' — manual check needed; not auto-resent.`);
    return { status: "halted" };
  }

  // Durable intent marker BEFORE the send.
  await gh.addLabel(pr.issueNumber, "sending");
  try {
    const raw = await gh.getFileAtRef(`outbox/${pr.slug}/reply.md`, pr.mergeSha);
    const doc = parseReplyDoc(raw);
    const { messageId } = await gmail.send(doc);
    // Done marker IMMEDIATELY after send — before any secondary write.
    await gh.addLabel(pr.issueNumber, "sent");
    await gh.comment(pr.issueNumber, `✅ Sent reply (Gmail message ${messageId}). — fancylab-ops executor`);
    await gh.removeLabel(pr.issueNumber, "sending");
    await gh.closeIssue(pr.issueNumber);
    return { status: "sent", messageId };
  } catch (err) {
    const error = err instanceof Error ? err.message : String(err);
    // Leave 'sending' in place so the next poll HALTS rather than resending.
    await alert(`Issue #${pr.issueNumber} (PR #${pr.prNumber}) send failed: ${error}. Left in 'sending' for manual check.`);
    return { status: "error", error };
  }
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `cd executor && npm test`
Expected: PASS (all suites). Confirm the order assertion in the happy-path test passes — this is the exactly-once guarantee.

- [ ] **Step 5: Commit**

```bash
git add executor/src/process.ts executor/test/process.test.ts
git commit -m "Executor: processReplyPR exactly-once orchestration (TDD)"
```

---

### Task 7: GitHub adapter (Octokit → GitHubPort)

**Files:**
- Create: `executor/src/github.ts`

No unit test (thin network adapter — verified by the Task 9 smoke check). Keep it a literal translation of the port.

- [ ] **Step 1: Implement the adapter**

Create `executor/src/github.ts`:

```ts
import { Octokit } from "@octokit/rest";
import { parseIssueFromBranch } from "./parse-branch.ts";
import type { GitHubPort, MergedPR } from "./types.ts";

export function makeGitHub(token: string, repo: string): GitHubPort {
  const [owner, name] = repo.split("/");
  const octo = new Octokit({ auth: token });

  return {
    async listMergedReplyPRs(): Promise<MergedPR[]> {
      // Merged PRs labeled action:email-reply whose issue is still open.
      const prs = await octo.paginate(octo.pulls.list, {
        owner, repo: name, state: "closed", per_page: 50, sort: "updated", direction: "desc",
      });
      const out: MergedPR[] = [];
      for (const pr of prs) {
        if (!pr.merged_at) continue;
        if (!pr.labels.some((l) => (typeof l === "string" ? l : l.name) === "action:email-reply")) continue;
        const parsed = parseIssueFromBranch(pr.head.ref);
        if (!parsed) continue;
        const issue = await octo.issues.get({ owner, repo: name, issue_number: parsed.issueNumber });
        if (issue.data.state !== "open") continue;
        out.push({ prNumber: pr.number, issueNumber: parsed.issueNumber, slug: parsed.slug, mergeSha: pr.merge_commit_sha! });
      }
      return out;
    },
    async getIssueLabels(issueNumber) {
      const r = await octo.issues.listLabelsOnIssue({ owner, repo: name, issue_number: issueNumber });
      return r.data.map((l) => l.name);
    },
    async addLabel(issueNumber, label) {
      await octo.issues.addLabels({ owner, repo: name, issue_number: issueNumber, labels: [label] });
    },
    async removeLabel(issueNumber, label) {
      await octo.issues.removeLabel({ owner, repo: name, issue_number: issueNumber, name: label }).catch(() => {});
    },
    async closeIssue(issueNumber) {
      await octo.issues.update({ owner, repo: name, issue_number: issueNumber, state: "closed" });
    },
    async comment(issueNumber, body) {
      await octo.issues.createComment({ owner, repo: name, issue_number: issueNumber, body });
    },
    async getFileAtRef(path, ref) {
      const r = await octo.repos.getContent({ owner, repo: name, path, ref });
      if (!("content" in r.data)) throw new Error(`not a file: ${path}`);
      return Buffer.from(r.data.content, "base64").toString("utf8");
    },
  };
}
```

- [ ] **Step 2: Type-check**

Run: `cd executor && npx tsc --noEmit`
Expected: no errors.

- [ ] **Step 3: Commit**

```bash
git add executor/src/github.ts
git commit -m "Executor: Octokit GitHub adapter"
```

---

### Task 8: Gmail adapter (googleapis → GmailPort)

**Files:**
- Create: `executor/src/gmail.ts`

- [ ] **Step 1: Implement the adapter**

Create `executor/src/gmail.ts`:

```ts
import { google } from "googleapis";
import type { GmailPort, ReplyDoc } from "./types.ts";

export function makeGmail(opts: {
  clientId: string; clientSecret: string; refreshToken: string; sender: string;
}): GmailPort {
  const auth = new google.auth.OAuth2(opts.clientId, opts.clientSecret);
  auth.setCredentials({ refresh_token: opts.refreshToken });
  const gmail = google.gmail({ version: "v1", auth });

  return {
    async send(msg: ReplyDoc) {
      const headers = [
        `From: ${opts.sender}`,
        `To: ${msg.to}`,
        `Subject: ${msg.subject}`,
        msg.inReplyTo ? `In-Reply-To: ${msg.inReplyTo}` : "",
        msg.inReplyTo ? `References: ${msg.inReplyTo}` : "",
        'Content-Type: text/plain; charset="UTF-8"',
        "",
        msg.body,
      ].filter(Boolean).join("\r\n");
      const raw = Buffer.from(headers).toString("base64url");
      const res = await gmail.users.messages.send({
        userId: "me",
        requestBody: { raw, threadId: msg.threadId },
      });
      return { messageId: res.data.id! };
    },
  };
}
```

- [ ] **Step 2: Type-check**

Run: `cd executor && npx tsc --noEmit`
Expected: no errors.

- [ ] **Step 3: Commit**

```bash
git add executor/src/gmail.ts
git commit -m "Executor: googleapis Gmail send adapter"
```

---

### Task 9: Poll loop + entrypoint, and a creds-gated smoke check

**Files:**
- Create: `executor/src/poll.ts`

- [ ] **Step 1: Implement the entrypoint**

Create `executor/src/poll.ts`:

```ts
import "dotenv/config";
import { makeGitHub } from "./github.ts";
import { makeGmail } from "./gmail.ts";
import { processReplyPR } from "./process.ts";

function env(name: string): string {
  const v = process.env[name];
  if (!v) throw new Error(`missing env: ${name}`);
  return v;
}

async function alert(message: string) {
  // v1: stderr is the alert channel; a richer channel (Slack/email) comes later.
  console.error(`[ALERT] ${message}`);
}

async function tick() {
  const gh = makeGitHub(env("GITHUB_TOKEN"), env("OPS_REPO"));
  const gmail = makeGmail({
    clientId: env("GOOGLE_CLIENT_ID"),
    clientSecret: env("GOOGLE_CLIENT_SECRET"),
    refreshToken: env("GOOGLE_REFRESH_TOKEN"),
    sender: env("GMAIL_SENDER"),
  });
  const prs = await gh.listMergedReplyPRs();
  console.log(`[poll] ${prs.length} merged reply PR(s) to consider`);
  for (const pr of prs) {
    const res = await processReplyPR(pr, gh, gmail, alert);
    console.log(`[poll] issue #${pr.issueNumber} -> ${res.status}`);
  }
}

const interval = Number(process.env.POLL_INTERVAL_MS ?? 120000);
if (process.env.ONCE) {
  tick().catch((e) => { console.error(e); process.exit(1); });
} else {
  console.log(`[poll] starting; interval ${interval}ms`);
  const run = () => tick().catch((e) => console.error("[poll] tick error:", e));
  run();
  setInterval(run, interval);
}
```

- [ ] **Step 2: Type-check the whole project**

Run: `cd executor && npx tsc --noEmit`
Expected: no errors.

- [ ] **Step 3: Run the full unit suite once more**

Run: `cd executor && npm test`
Expected: PASS — all pure-logic + orchestration tests green.

- [ ] **Step 4: Creds-gated smoke check (only after the prerequisite is met)**

> Requires `executor/.env` filled with a `gmail.send`-scoped refresh token and `GITHUB_TOKEN`.
> If the prerequisite isn't done yet, **skip this step** and note it in the PR; the loop is otherwise complete.

Manual smoke (does not send to a client):
1. Create a throwaway issue + branch + `outbox/<n>-smoke/reply.md` with `to:` set to your own address, label the PR `action:email-reply`, merge it.
2. Run: `cd executor && npm run poll:once`
3. Expected: log shows `issue #<n> -> sent`; you receive the test email; the issue gets `sent` + a receipt comment + is closed.
4. Run `npm run poll:once` AGAIN. Expected: `issue #<n> -> skipped` (no second email). **This is the exactly-once proof.**

- [ ] **Step 5: Commit**

```bash
git add executor/src/poll.ts
git commit -m "Executor: poll loop + entrypoint"
```

---

### Task 10: Open the PR

- [ ] **Step 1: Push the branch**

Run: `git push -u origin plan1/foundation-executor`

- [ ] **Step 2: Open the PR for Rinaldo to review/merge**

Run:
```bash
gh pr create --base main --head plan1/foundation-executor \
  --title "Plan 1: repo foundation + email executor (exactly-once send)" \
  --body "Implements docs/superpowers/plans/2026-06-06-ops-v1-foundation-executor.md. Pure-logic + exactly-once orchestration are unit-tested; Gmail/GitHub adapters are creds-gated. The live send smoke (Task 9 step 4) is pending the gmail.send re-consent prerequisite."
```
Expected: PR URL printed. This is the first real fancylab-ops PR — Rinaldo reviews and merges.

---

## Self-Review

**Spec coverage (Plan 1 portion of spec §5):**
- Component 1 (repo structure: labels, `outbox/`, `sops/` seed, `executor/`, Bee brief stubs) → Tasks 1–2 ✓
- Component 6 (Executor: merged `action:email-reply` PR → Gmail send → close; **exactly-once**) → Tasks 3–9 ✓
- Spec §5 exactly-once requirement (sending-before / sent-after / halt-on-stuck) → Task 6 tests assert call order ✓
- Components 2,4,5 (Email Scout, Email Flight, Guard) → **Plan 2** (out of scope here, by design)
- Components 3,7,8 (ClickUp Scout, Hub Inbox, scheduling) → **Plan 3**

**Placeholder scan:** The only `TBD`s are the deliberate Bee-brief stubs (Task 1 Step 6), explicitly scheduled for Plan 2 — flagged as intentional, not plan gaps. No `add error handling`-style hand-waving: every code step ships real code; the send failure path is concretely implemented and tested.

**Type consistency:** `GitHubPort`/`GmailPort`/`ReplyDoc`/`MergedPR`/`SendDecision` defined in `types.ts` (Task 4) and used identically in `decide.ts`, `process.ts`, `github.ts`, `gmail.ts`. `parseReplyDoc(markdown)` single-arg signature consistent across Task 4 def and Task 6 usage. `processReplyPR(pr, gh, gmail, alert)` signature consistent across Task 6 def and Task 9 call.

**Resolved spec §8 open questions (for this plan's scope):** executor = standalone script in `executor/` with its own `.env`, poll cadence 120s (configurable); dedup/idempotency = label state machine (`sending`/`sent`) + issue-open filter, no separate store; PR↔issue link = branch name `ops/NNNN-slug` (no "Closes #" so merge doesn't auto-close — the executor closes after send); alert channel = stderr in v1.

## Open questions deferred to Plan 2 (do not block Plan 1)
- Flight orchestration (how Forager→…→Ship run as subagents per issue).
- Exact Guard automated checks and BLOCK-routing mechanism.
- Voice-brief reuse vs. trimmed copy in `bees/worker.md`.
