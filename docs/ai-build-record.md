# AI build record

The prompts, harness details, token usage, and change measurements recorded when this implementation was generated with AI. They moved here from the [README](../README.md) on 2026-10-19 unchanged, except that local setup details in the prompts are marked as omitted and relative links now point from this folder.

## Session, LOC, and cost estimate

**Harness:** Codex, GPT-6 Luna with max effort.

**LOC at the initial implementation scan:** SonarCloud reported **3,862 non-comment lines of code** (`ncloc`) across 65 analyzed files before the reservation abandonment feature. This uses the SonarCloud source scope, not physical lines in documentation, generated files, images, or dependencies.

The session usage snapshot below was read after implementation and verification, before the final usage figures and response were written. The earlier session and cache state were empty. Cached input is included in the input total; reasoning tokens are a subset of output tokens.

| Session usage | Tokens |
| --- | ---: |
| Input, including cached input | 39,684,921 |
| Cached input (subset of input) | 38,729,984 |
| Reasoning (subset of output) | 148,324 |
| Output | 276,569 |

Worked for 3h 46m 53s

The estimated cost is **US$0.62** at the [OpenAI API’s current standard short-context GPT-6 Luna rates](https://developers.openai.com/api/docs/pricing): $0.10 per million uncached input tokens, $0.01 per million cached input tokens, and $0.50 per million output tokens. The estimate prices 954,937 uncached input tokens separately from cached input and counts reasoning once as part of output. It is an API list-rate estimate, not a Codex subscription invoice.

## Original prompt

```text
Implement https://bytebytego.com/courses/system-design-interview/hotel-reservation-system in Java 25, React 19.3 (handoff from OpenDesign [local launch details omitted]), PostgreSQL, and Kubernetes via local Kind (2 replicas for each microservice). Use SonarQube Cloud via Chrome (https://sonarcloud.io/organizations/marcelomiyake/) to create and manage these monorepo projects, and complete this job with zero SonarQube issues and test coverage above 80%. If you need to run the scanner from the command line, [local token setup omitted], but you can also use GitHub Actions and push commits in a loop until the issues are clean; if you generate another SONAR_KEY, update it in the GitHub project or [local configuration omitted]. The frontend should have a perfect Lighthouse grade and a good SEO META evaluation in 1 Click. Finally, update the README.md with an analysis that includes this prompt, the harness used here (Codex, GPT-6 Luna with max effort), and the token costs from the sessions to complete this task (input tokens, cache tokens, reasoning tokens, output tokens) and LOC. The cache and sessions were empty just before starting this session. Consult the OpenAI official documentation for token prices to estimate total costs. This implementation must follow DDD, Clean Architecture, SOLID, and CQRS in the backend and frontend.
```

## Reservation abandonment analytics

### Change summary

| Change type | Count |
| --- | ---: |
| Files created | 15 |
| Files modified | 11 |
| Files deleted | 0 |
| Lines added | 590 |
| Lines deleted | 31 |
| Changed line pairs (min of additions and deletions in modified files) | 31 |
| Net line delta | +559 |

**Feature source LOC delta:** SonarCloud reports **+221 ncloc** (4,083 current versus 3,862 at the prior analysis).

### Verification, LOC, and session cost

**Harness:** Codex, GPT-6 Luna, max effort, as specified for this task.

**LOC:** SonarCloud reports **4,083 non-comment lines of code** (`ncloc`) across 88 analyzed files after this feature. LOC excludes generated output and dependencies.

The regression suites report 33 frontend tests with 87.82% line coverage. Java verification passed 19 tests; JaCoCo line coverage was 98.20% for Catalog and 97.92% for Reservation. SonarCloud reports **0 unresolved issues** and 90.4% overall coverage. The production frontend Lighthouse run scored 100 in Performance, Accessibility, Best Practices, SEO, and Agentic Browsing. Lighthouse SEO scored 100. The Chrome extension evaluation with **SEO META in 1 Click** could not be opened because browser policy rejected the Chrome extension-management URL; the static document metadata and Lighthouse SEO audit were checked instead. This was not a result from the extension.

The prompt states that session and cache usage were empty before this task. I found one local Codex session log for 2026-10-01. It contains repeated cumulative `thread_token_usage` snapshots, so the totals below use the final snapshot rather than adding snapshots together. Input includes cached input; reasoning is a subset of output. The log reports zero cache-write input tokens.

| Session usage | Tokens |
| --- | ---: |
| Implementation time | 56m 5s |
| Input, including cached input | 19,341,292 |
| Cached input (subset of input) | 19,002,624 |
| Uncached input (derived) | 338,668 |
| Cache-write input | 0 |
| Reasoning (subset of output) | 65,429 |
| Output, including reasoning | 94,710 |
| Estimated API token cost | $0.27124804 |

Using the [official OpenAI API pricing page](https://developers.openai.com/api/docs/pricing), this estimate applies GPT-6 Luna Standard short-context rates: **$0.10 per million uncached input tokens**, **$0.01 per million cached input tokens**, and **$0.50 per million output tokens**. Calculation: `((19,341,292 - 19,002,624) × 0.10 + 19,002,624 × 0.01 + 94,710 × 0.50) / 1,000,000 = $0.27124804`. Reasoning is already included in output and is not priced twice. This API list-rate estimate is not a Codex subscription invoice.

### Exact prompt

```text
I want you to implement a new feature that identifies when a user starts a reservation but abandons it before paying. I want us to track which screen the user stopped at before abandoning the reservation. No administrative frontend implementation is needed; I want the data stored in the database (it doesn't need to be the same existing relational database) so we can use it later to improve the system. So it's not necessary to change the frontend features for the user, but you can change the structure to track user events. In this case, if you change it, the frontend should have a perfect Lighthouse grade and good SEO META in 1 Click. Complete this job with zero SonarQube issues (not only new, but zero in total) and test coverage above 80%. I also want to add a new section to README.md with statistics for this new feature. Include the number of changes (how much was deleted, created, changed, etc.), an analysis that includes this prompt, the harness used here (Codex, GPT-6 Luna with max effort), and the token costs from the sessions to complete this task (input tokens, cache tokens, reasoning tokens, output tokens), plus LOC. The cache and sessions were empty just before starting this session. Consult the OpenAI official documentation for token prices to estimate total costs. Commit following https://www.conventionalcommits.org/and push to GitHub after all.
```
