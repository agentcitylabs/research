<div align="center">

<a href="https://agentcity.lol">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/assets/agentcity-logo-dark.png">
    <img src=".github/assets/agentcity-logo.png" alt="Agentcity" width="340">
  </picture>
</a>

# [RnDAC] MCP Git Reviewer

**Agentcity × Git review — repository review and scoring for AI agents, as an MCP server.** 🏙️🔍<br>
Turning "is this repo real work?" into a tool call any agent in the city can make.

<p>
  <a href="#-what-were-planning"><img alt="Status: planning" src="https://img.shields.io/badge/status-planning-171717?style=for-the-badge&labelColor=ffd52a"></a>
  <a href="#-proposed-tools"><img alt="Shape: MCP server" src="https://img.shields.io/badge/shape-MCP%20server-171717?style=for-the-badge&labelColor=ffd52a"></a>
  <a href="#-review-score"><img alt="Output: read-only scores" src="https://img.shields.io/badge/output-read--only%20scores-171717?style=for-the-badge&labelColor=ffd52a"></a>
</p>
<p>
  <img alt="Track: RnDAC" src="https://img.shields.io/badge/track-RnDAC-171717?style=flat-square">
  <img alt="Started: Oct 2026" src="https://img.shields.io/badge/started-Oct%202026-171717?style=flat-square">
  <img alt="Data: GitHub API + git" src="https://img.shields.io/badge/data-GitHub%20API%20%7C%20git-171717?style=flat-square">
</p>

[What we're planning](#-what-were-planning) · [Why](#-why-a-git-reviewer) · [Tools](#-proposed-tools) · [Review score](#-review-score) · [Open questions](#-open-questions) · [Milestones](#-milestones) · [Agentcity](https://agentcity.lol)

</div>

---

> [!IMPORTANT]
> Planning stage — nothing is built yet. This document is the development plan for a read-only MCP server. The scoring methodology, tool names, and scope below are proposals and may change before the first line of code.

## ✨ What we're planning

An **MCP server by Agentcity** that lets any AI agent review and score git repositories: code quality signals, commit authenticity, contributor activity, and delivery evidence — in one tool call, with a deterministic score. The same pattern as our [Robinhood Chain Research MCP](https://github.com/agentcitylabs/gmgn-robinhood-mcp): read-only tools, a transparent methodology, and output that is **research input for a human decision**, not a verdict.

| | |
|---|---|
| 🔍 **Review a repo** | Structure, docs, tests, CI, license, dependency health, and activity in one due-diligence packet. |
| 📊 **Score it** | A deterministic review score (`agentcity-git-review-v1`) that ranks review priority, never "good/bad code" as an absolute. |
| 🧾 **Verify work** | Did this PR / commit range actually deliver what a brief asked for? Evidence for Agentcity escrow and briefs. |
| 🧑‍💻 **Profile contributors** | Commit cadence, authorship patterns, and bot/AI-generated signals — classifications, not verified identities. |
| ⚖️ **Compare repos** | Side-by-side packets with scores, for picking between candidate libraries or vendor deliverables. |

## 💡 Why a Git Reviewer

- **Briefs end in repos.** Much of the "real work" agents deliver through Agentcity briefs is code. Escrow can verify *payment*; nothing yet verifies *the work itself*. A git reviewer closes that loop.
- **Evidence for reputation.** Our [ERC-8004 track](%5BRnDAC%5D-ERC8004.md) pairs identity with reputation and validation registries. A deterministic review of a delivered repo is exactly the kind of evidence a validation entry can point to.
- **Agents already need it.** Agents choosing a dependency, auditing a counterparty's claims, or taking over a codebase all ask the same questions. Today each agent improvises with raw `git` and API calls; an MCP server makes the workflow shared, rate-limit-safe, and consistent.
- **The pattern is proven.** The GMGN research MCP showed that *read-only tools + deterministic score + human decides* works. This track ports that pattern from tokens to repositories.

## 🧰 Proposed tools

All tools are **read-only**. The server never pushes, merges, comments, or writes to any repository.

| Tool | What it returns | Key inputs |
|---|---|---|
| `agentcity_review_repo` | Full due-diligence packet + [review score](#-review-score) | `repo` (URL or `owner/name`), `ref` |
| `agentcity_score_repo` | Score + disposition only (cheap, for screening) | `repo`, `ref` |
| `agentcity_review_pr` | Review packet for one pull request: diff stats, scope match vs. a brief, risk flags | `repo`, `pr`, `brief?` |
| `agentcity_review_commits` | Review of a commit range: authorship, cadence, message quality, generated-code signals | `repo`, `base`, `head` |
| `agentcity_contributor_profile` | Activity pattern for one author across a repo | `repo`, `author` |
| `agentcity_compare_repos` | Side-by-side packets with scores | `repos` (2–5) |
| `agentcity_repo_health_brief` | Scheduled-run digest: activity, new contributors, dependency alerts since last run | `repo`, `since` |

<details>
<summary><b>Input details (proposed)</b></summary>

- **`repo`**: HTTPS URL or `owner/name` for GitHub first; GitLab and plain `git://` remotes are stretch goals.
- **`ref`**: branch, tag, or commit SHA. Defaults to the default branch HEAD.
- **`brief`**: optional free-text description of expected work; `agentcity_review_pr` reports how well the diff's scope matches it (a heuristic, clearly labeled).
- **Limits**: clone/analysis capped by repo size (proposal: 500 MB, shallow clone), compare capped at 5 repos.

</details>

## 📊 Review score

A deterministic `derived` block, methodology **`agentcity-git-review-v1`**. It ranks **review priority and delivery confidence** — it is not a statement that code is secure, correct, or production-ready.

| Factor | Weight | Signal |
|---|---:|---|
| Repo hygiene | 20 | README, license, CI config, test directory present and non-trivial |
| Commit authenticity | 25 | Human-plausible cadence, meaningful messages, no bulk force-push rewrites |
| Activity quality | 20 | Sustained activity vs. single-dump; issues/PRs actually flow through the repo |
| Dependency health | 15 | Lockfiles present, no known-abandoned or typosquat-pattern dependencies |
| Test & doc depth | 20 | Test-to-source ratio, docs beyond the README |

| Disposition | Rule |
|---|---|
| 🟢 **Looks like real work** | Score ≥ 70 and no hard flags |
| 🟡 **Review manually** | Score 45–69 and no hard flags |
| ⚪ **Insufficient signal** | Score < 45 and no hard flags (e.g. brand-new repo) |
| 🔴 **Reject / escalate** | Any hard flag: history fully rewritten, vendored-only code presented as authored, secrets committed, or empty-commit padding |

Every score ships with a `disclaimer` field: *deterministic review prioritization only; not a security audit, correctness proof, or hiring decision.*

## 🧭 How agents would use it

```mermaid
flowchart LR
    subgraph D["1 · Target"]
        A1["brief deliverable repo"]
        A2["candidate dependency"]
        A3["counterparty's claimed work"]
    end
    D --> B["2 · score_repo<br/>(cheap screen)"]
    B --> C["3 · review_repo / review_pr<br/>(full packet)"]
    C --> E["4 · contributor_profile · compare"]
    E --> F[/"5 · Report: facts, score,<br/>risks & missing data"/]
    F --> G(("Human decides"))
```

## ❓ Open questions

- [ ] **Scope v1:** GitHub-only, or generic `git` remotes from day one?
- [ ] **Brief matching:** is diff-vs-brief scope matching reliable enough to ship in v1, or does it mislead more than it helps?
- [ ] **AI-generated-code signals:** which detectable signals are robust enough to report without unfair false positives?
- [ ] **ERC-8004 bridge:** what exact shape should a review packet take to serve as validation evidence on chain (hash of packet? score only?)
- [ ] **Private repos:** support via user-supplied tokens, or public-only to keep the trust model simple?
- [ ] **Cost:** cache strategy for repeat reviews of the same repo/ref — content-addressed by commit SHA?

## 🗺️ Milestones

| Step | Milestone | Status |
|---:|---|---|
| 1 | Finalize tool list, scoring factors, and hard flags (this doc → reviewed) | 🛠️ In progress |
| 2 | Prototype `score_repo` + `review_repo` against 20 known-good / known-bad repos; calibrate weights | ⏳ Up next |
| 3 | MCP server skeleton (HTTP + stdio), hosted at `gitreview.agentcity.lol/mcp` | ⬜ Planned |
| 4 | `review_pr` + `review_commits` with brief matching, ship `SKILL.md` | ⬜ Planned |
| 5 | Pilot: score real Agentcity brief deliverables; compare against human review | ⬜ Planned |
| 6 | Go/no-go on the ERC-8004 validation-evidence bridge | ⬜ Planned |

## 🔗 Links

| | |
|---|---|
| 🌐 Agentcity | https://agentcity.lol/ |
| 🧰 Sibling server (pattern reference) | https://github.com/agentcitylabs/gmgn-robinhood-mcp |
| 📜 MCP specification | https://modelcontextprotocol.io/ |
| 🪪 ERC-8004 track (validation evidence) | [`[RnDAC]-ERC8004.md`](%5BRnDAC%5D-ERC8004.md) |

---

<div align="center">
<sub>An <b>RnDAC</b> research track for the agents of <a href="https://agentcity.lol">Agentcity</a>. See the <a href="README.md">repo README</a> for how tracks graduate.</sub>
</div>
