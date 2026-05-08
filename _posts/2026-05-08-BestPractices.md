From 2026-05-05 09:09:56 UTC, e56548eac905e7f9c3a2206cc89e0ca1207ab3c9

## README.md

### Goals
- Anyone can find the most important rules of thumb to guide them toward success
- Everyone becomes more effective owners, making decisions that better serve their customers and the business
### Rules
- Do the right thing for your customers; whenever customer interests conflict with internal business concerns, favor the customer but not at the cost of the company's ability to continue serving them
- Don't let perfection be the enemy of good; deliver quickly, gather feedback, and iterate
- Fear is a signal that engineering rigor is missing and is unacceptable; if something feels scary or opaque, close that gap through provable mechanisms (e.g. measurement, tests) that reveal the properties of your system and its use cases, not by avoiding or working around it
- Focus on our expertise by investing effort where it creates unique value; for everything else, use established tools and industry standards unless they genuinely don't meet our needs
- You may deviate from a best practice if and only if you adequately document the reasons why so that others may understand what circumstances led to the decision (see documentation rules for details)
- You must ignore a best practice when following it would put customers at risk or threaten the company's ability to continue serving them:  best practices are meant to help; however, they do not override your responsibilities as an owner
- The only bar for creating a best practice is that it is broadly agreed to within the organization as a north star vision; there is no understanding that existing code or systems conform to it yet, as legacy work may predate these rules or reflect decisions made under different constraints
- Best practices should be removed if there are numerous equivalent or superior alternatives (highly-leveraged workarounds do not count)
- Best practices are hierarchical in nature (i.e. the best practice rules specified here apply to all other best practices and sections)

## code/README.md

### Goals
- Engineers can read any piece of code and immediately understand what it does
- Engineers can change any piece of code confidently, knowing exactly what it affects
### Rules
- Inline any abstraction unless it meets one of the following criteria:
  - It self-documents non-obvious relationships or hides significant complexity
  - It maintains state that the algorithm requires across iterations
  - It provides proven performance benefits (bottlenecks must be demonstrated first)
  - It has multiple, current implementations (e.g. subclasses of an interface)
- Variable names should:
  - Never duplicate type information
    - Bad examples: isValid, creationDate, configurationMap, userList 
    - Good examples: valid, createdOn, valueByKey, usersSortedByName
  - Never duplicate the names of the object they are contained within
    - Bad example: orderId within Order
    - Good example: id within Order
  - Clarify how they should be used/interpreted (especially in comparison with other dimensions that may be introduced in the future)
    - Bad example: customerId
    - Good example: consumerId (as it works alongside supplierId)
- Avoid vague verbs like `manage` or `handle` in function and class names since they obscure what the code actually does; instead, use names that describe the specific action or responsibility (e.g. `validateOrder`, `persistPayment`)
- Functions should do one thing; a name that requires "and" is a signal the function should be split (e.g. `validateAndPersist` should be two functions)
- Business logic should live at the top level of the call stack, not buried in lower layers; lower layers should be generic and unaware of the business context
- Utilities should only minimize boilerplate; they should accept configuration rather than deciding it themselves. For example, a `formatDate` utility should accept a format string rather than hardcoding it internally
- Exceptions should only communicate two things: whether the caller should retry and whether the fault is theirs or the system's. Avoid creating exception subclasses beyond what is needed to express these distinctions
- Consolidate logic into singular authorities instead of spreading logic in multiple locations:
  - Extract shared functions instead of duplicating logic
  - Encapsulate classes instead of using enums/switch statements
- Avoid shared mutable state; prefer immutable values and explicit dependencies over state that can be modified from multiple places
- Functions should only depend on what they use; prefer taking specific fields or values over large objects when only a subset is needed, which keeps the function usable in a wider range of contexts.
## code/test.md

### Goals
- Engineers can quickly read any test to understand exactly how the system behaves
- Engineers discover how their changes ripple through the system and whether they break core assumptions before they ship, without tests becoming a maintenance burden
### Rules
- When tests are difficult to write, treat it as a signal to refactor the production code; prefer smaller, more focused components with clear interfaces over adding test complexity. Tests are consumers of production interfaces just like any other client; a test that requires a complex mock is often really a client asking for a cleaner dependency injection point or more expressive return values
- When test validation logic grows complex (e.g. asserting 20 individual fields in multiple tests), treat it as a signal that production code is missing a concept -- that assertion is really asking for a method like `isEqual` on the production class; extract it there so both tests and other callers benefit
- Build a testing fabric where each layer tests different assumptions — the fabric gains strength from diversity, not repetition. Avoid tests that mirror production logic or only verify data routing (e.g. that a mocked value passes through unmodified, that a parameter is forwarded unchanged to a dependency); these share the code's blind spots. Focus instead on boundary conditions and transformations that will surface regressions
- Mocks should only be used when there is no alternative — typically for resources that are unavailable or prohibited in sandbox tests (e.g. disk, network, external services); if you find yourself mocking anything else, treat it as a signal to reconsider the test or the production design
  - For data, prefer factory-created randomized objects over mocks. Mocks assume only certain fields will be accessed, baking in white-box knowledge of the production code and will silently break during future refactors; a real randomized object will surface unexpected field accesses and remain valid as the code evolves
  - A test's job is to verify all meaningful combinations of inputs; mocks shortcut this by using white-box knowledge of which inputs the production code actually accesses, producing a test with artificially narrow coverage. If the combination space is too large to test, that is a signal to refactor the production code to reduce its inputs — not to use mocks to paper over it
- A test should read as a minimal specification: only fix the values relevant to the assertion and randomize everything else; incidental state should be invisible so readers immediately see what the test is actually proving
- Structure tests so each case is a distinct set of inputs and an expected output; the full suite should read like a table where adding a row means adding a use case and gaps in coverage are immediately visible
- When a gap surfaces at any test layer, fill it at that layer and every layer below it; the lower the layer that catches a regression, the faster and cheaper it is to diagnose
- Tests should be fully isolated; shared global state causes tests to interfere with each other, producing failures that depend on execution order and are difficult to diagnose
## deployments.md

### Goals
- Engineers are skeptical of every deployment and prove they work before reaching customers
- Rollbacks are safe, routine, and non-eventful because engineers invest in safety upfront
### Rules
- Deploy exploration rounds first; use metrics and shadow comparisons to gather information and validate assumptions before committing to a full rollout
- Prove functionality works through automated tests and CI/CD before any customer sees it; reacting is not acceptable, as it means customers are already affected
- Every deployment, regardless of implementation, must be observable; it must be easy to determine what is deployed, what is enabled, and what impact it is having
- Each deployment must update observability and alerting to verify whether the change is working or causing problems (see [observability.md], [alerts.md])
- Every change must be rollback safe: even in successful deployments, new code incrementally rolls out to hosts and old code continues to process state written by the new version
- Every state change (especially within APIs and DBs) must be preceded by forward-compatible changes; deployments should be broken into the following steps:
  - Expand: Add new schema with defaults; old code is unaware and continues to work unchanged
  - Shadow Writes: Write to both schemas; old remains source of truth. Backfill historical data if needed
  - Shadow Reads:  Read from both schemas but serve old; compare until diffs reach zero
  - Reverse Shadow: Serve reads from new schema; continue writing to old as safety net until parity is confirmed
  - Contract: Remove  old schema and comparison infrastructure; delay until you can prove nothing reads the old schema. This step cannot be undone
- When deploying, limit blast radius by rolling out changes incrementally (e.g. subsets of hosts, regions, customer partitions), measuring metrics against baselines rigorously, and then expand exponentially
- Rollbacks become more dangerous over time; previous state grows increasingly incompatible with the present as data accumulates, external services evolve, and invariants shift
- Roll back aggressively and immediately to protect customers; prefer rolling back deployments over patching, except for data changes, which are often safer to correct than reverse
- Before rolling forward again, add tests and metrics that would have caught the issue (see [post-mortems.md])
- For issues that can't be anticipated or measured in advance, prefer automatic remediation over manual intervention; automatic actions should be conservative and notify humans to verify
- Backups are for disasters only (data corruption, accidental deletes, irreversible failures), not for recovering from broken features
- Invest in CI/CD rather than temporary feature flags; every flag that isn't fully supported (tested, documented, maintained) is wasted effort that could have been used to protect every future deployment
- If you decide to use feature flags as a deployment mechanism:
  - Feature flag should have a clear purpose, owner, and expiration date; treat it as a feature until removed
  - Check flags at system boundaries (controllers, request handlers) not deep within business logic
  - Feature flags should be binary which code path runs (on/off); they are not configuration values like SLAs, limits, or thresholds 
## documentation.md

### Goals
- Anyone can discover important context without relying on tribal knowledge
- Authors document only what cannot be derived from the system or its version history
### Rules
- Carefully choose what to document, so that it doesn't become stale:
  - Documentation should never describe what or how, it should only describe why or add external context; if something requires documentation to understand what is happening, refactor it to be self-documenting
  - TODOs should never suggest solutions; instead they should call out why things are suboptimal, since potential solutions may change over time
- Place documentation alongside the authoritative source whose charter best aligns with its scope; refrain from describing exactly how or where it is called, as those specifics don't scale and quickly become stale.  For example:
  - Code:  TODOs describe why the code is not optimal
  - Pull-request:  The description explains why TODOs were added versus resolved as part of the change
  - Issue tracking software:  Explains why resolving the TODOs are a high-priority
- State each idea exactly once; consolidate similar ideas/sections and keep introductions high-level rather than previewing the detail that follows

## operations/README.md

### Goals
- Engineers understand production systems clearly without needing to be paged
- On-call is predictable and low-stress: few pages, clear SOPs, fast resolution, long quiet periods
### Rules
- When creating operational actions, choose carefully; when in doubt, prefer observability:
  - Alerting (see [alerts.md]): interrupts engineers and demands immediate action
  - Observability (see [observability.md]): always available and never interrupts
  - Use impact and urgency, based on Service Level Objectives (SLOs), to determine which to use (see [post-mortems.md] for details):

    | Severity | Signal | Trigger | Response                                                                        | Post-mortem |
    |----------|--------|---------|---------------------------------------------------------------------------------|-------------|
    | SEV-0 | Alert (page) | Major outage, catastrophic or irreversible harm (data loss, security breach), or significant trust or revenue loss | Page and escalate simultaneously; no grace period | Full required |
    | SEV-1 | Alert (page) | Significant service degradation, severe impact to a subset of users, or meaningful trust or revenue impact | Page immediately, escalate if not progressing within 30 min                     | Full required |
    | SEV-2 | Observability (issue) | Breached SLO, sustained burn, or early trust or revenue signals | Investigate during business hours, escalate if not progressing within 3-4 hours | Full or lite, async acceptable |
    | SEV-3 | Observability (issue) | Approaching SLO violation or anomaly | Investigate during business hours, escalate if not progressing within days      | Lite if recurring |
- Operational tooling should be treated as full-fledged features so they don't fail when we need them the most; kill switches, degraded modes, SOPs, alert configs, and dashboards should be tested and maintained like production code
- High-risk or complex subsystems need a way to be disabled or degraded without a deployment, so during wide-scale incidents we can shed complexity, recover faster, and investigate in isolation
- Never close a page or issue without at least one improvement action: fix the root cause, improve observability to make the next occurrence easier to diagnose, or refine the alert that fired
- Hold a quarterly alert review to assess signal quality — was each alert actionable, was paging justified, was the signal clean? Delete alerts that fail and convert them to observability if the signal is still worth tracking
## operations/alerts.md

### Goals
- On-call engineers know about every customer-impacting before customers do
- We maintain high signal quality with few pages, clear SOPs, fast resolution, and long quiet periods so engineers stay sharp and avoid burnout
### Rules
- Use alerts deliberately; only create or keep an alert if all the following are true (otherwise delete it or move it to the low-priority queue):
  - There is customer-visible impact (e.g. failed requests, data loss risk, security incidents)
  - There is a specific, immediate action the on-call engineer can take
  - Delaying action causes measurable harm or further escalation
  - The issue is not isolated to a transient single instance, host, or event
- Every alert must be actionable and self-contained; alerts should always contain:
  - What is broken
  - How bad it is: severity, scope, and impact
  - What to do first
  - Who owns the response
  - How to mitigate or roll back
  - When to escalate
  - Good: `SEV2 - Checkout error rate > 5% for 5m (20% of users). First action: rollback release_checkout_v3. Owner: Payments. SOP: <link>.  Escalate to: #payments-lead if not resolved in 30m`
  - Bad: "Checkout errors spiking - check Datadog"
- If an alert is noisy, the metric is wrong; look for invariants that hold regardless of traffic level and never need recalibrating such as conditions that should always or never be true (e.g., auth tokens should never be issued without a valid session)
- Alerts should convey how fast a problem is worsening and when it becomes critical, not just that an arbitrary threshold was crossed (e.g., "ingestion lag consuming buffer at 3x the normal rate and will be exhausted in 20m", not "lag > 500ms")
- Avoid alert storms by consolidating to a single page for a single issue; don't page per region or downstream dependency
- Before shipping an alert, verify that a sleep-deprived engineer can act safely at 3am using pre-planned operational levers, not by deploying new code
- When responding, assume that the issue is bigger than it appears; actively prove that the alarm isn't a symptom of a larger issue before closing


## operations/observability.md

### Goals
- Everyone understands production without lying to themselves
- Engineers design observability upfront because lost data is irreversible; critical data must always be available when needed
### Rules
- Never expose PII or confidential data in any observability signal; assume every log will be leaked, subpoenaed, or audited
- Each signal type serves a distinct purpose; using the wrong one increases cost and harms debuggability:
  - Logs [long-retention]: For engineers, finance, compliance, and auditors to view immutable facts that the company definitely did, captured with full context at that instant.  Never include time or duration (see other signal types for those)
  - Spans [short-retention]: For engineers to view task duration and causality, including parent/child relationship trees (called traces)
  - Metrics [long-retention via aggregation]: For anybody to track trends and drive alerts via the four golden signals (latency, traffic, errors, and saturation)
- Model every entity with both an id and a versionId from the start; observability is a first-class design concern, not something retrofitted later
- Propagate both an entity's id and versionId across all service boundaries so any event can be correlated to the exact entity state it acted on
- Never use sampling; instead, aggregate by distinct type, which preserves data completeness. Critical financial paths are the exception, where every individual event should be retained
- Design metrics around finite, discrete sets of values defined in advance (like histogram buckets, e.g. status code, region, error type); unbounded dimensions (e.g. user.id, request.id) cause cost explosions and belong in traces instead
- Operational/infrastructure failures belong in spans (status = ERROR); business, security, and audit failures belong in logs as immutable facts. Don't duplicate across both without a strong reason
- Use attribute names as defined by integrated tools and partners (e.g. `http.request.method`, `db.system` from OpenTelemetry); only define custom attributes for domain concepts they don't cover (see [top-level rules](../README.md))
- Log levels are alerting contracts:
  - Error: page immediately if seen across multiple hosts (systemic failure)
  - Warn: page if sustained over a long period (degraded but recovering)
  - Info: never pages; records significant, expected events worth retaining (service started, payment created)
  - Never use trace/debug; use spans instead
- A dashboard should be a single page that answers "is the system ok" at a glance; it should show only the most important business and customer health indicators
- Prefer capture/replay testing over pre-emptive debug logging; logs are never at the right level of detail, whereas replay lets you investigate at any depth
- If purging data is necessary, delete low-criticality normal paths first and critical paths last; never delete errors or anomalous transactions (unexpected latency, retries, unexpected status codes)
- 
## operations/post-mortems.md

### Goals
- Customers never experience the same failure twice: every incident is a commitment to ship root cause fixes and prevent a class of failures, not just this one
- Engineers find the process lightweight, worth their time, and effective at reducing operational pain
### Rules
- Treat post-mortems as an opportunity, not a chore; incidents create rare moments of business attention and urgency that engineers can use to drive improvements and address long-standing issues that would otherwise be deprioritized
- Complex systems fail due to systemic causes, not individual mistakes; assign ownership to drive fixes faster, not to blame or punish
- Given that large issues fail at multiple layers, make sure the systems are hardened at every layer and don't rely on best intentions; if the only actions are "Repair" (see below), the incident will recur in a different shape and we still may continue to have single points of failure
- Only draft once the issue is mitigated but do it quickly (within 3 business days) before the pain and memory fades; a good-enough post-mortem shipped fast beats a perfect one written too late
- Write only enough to understand what happened, what failed, and what's being fixed; lengthy write-ups are only warranted for wide-impact or gross-negligence situations where you need to rally priority or drive culture shifts
- The severity of the incident determines the depth of the write-up (see [README.md] for which severity requires which):
  - Full: impact, scope (affected customers and businesses), timeline with exact timestamps, detection, mitigation, root cause and contributing factors, what went well/poorly, action items; may take 30 minutes or more to read
  - Lite: impact (1–2 lines), scope (brief), trigger/cause, mitigation, action items (1–3); readable in less than 15 minutes
- Action items must be concrete and completable, not large initiatives (which are slow to land, often change scope, and may never address the specific failure); if the path forward is unclear, the action item is to create a plan, not execute one
- A post-mortem isn't closed until all the action items are shipped or formally deferred with rationale (signed off by a manager); Each action item must be tracked in task software with: 
  - a single owner
  - what it addresses: 
    - Detection: catching the issue earlier
    - Mitigation: reducing the time-to-recover
    - Repair: fixing the immediate defect
    - Prevention: stopping this class of failure
  - a priority (P0/P1/P2)
  - a due date
  - a verifiable done condition


## operations/standard-operating-procedures.md

### Goals
- The most junior, sleep-deprived engineer at 3am can find the right SOP, follow the steps, and fix the issue without making things worse
- Best practices are encoded in tools and SOPS so that good outcomes are repeatable, not accidental
### Rules
- The most dangerous, error-prone, and repetitive operations should have an SOP, not just incidents; encoding how work gets done is the first step toward replacing direct, unguarded production access with tools that encode best practices and let engineers act confidently
- A good SOP names its trigger (the alert, symptom, or cadence that activates it), is actionable without interpretation, and is safe by default; if it requires judgment to use, it's a draft
- Every SOP must validate before acting and stop loudly when checks fail; an SOP that continues past a failed check is more dangerous than no SOP at all
- Include only enough prose to orient the operator — the steps are the intent; use checklists to track progress, since doing a step twice or out of order can be as dangerous as skipping it
- Every user of an SOP is its owner and must immediately fix any wrong steps; a bad step left in place is a trap for the next engineer who follows it
- Resolve first, then improve; after every execution, strengthen or automate at least one part of the SOP
- Improve SOPs incrementally: rough shared doc in the place engineers are most likely to look → version-controlled with PR review → individual steps replaced by scripts → automated trigger with observability; a fully mature SOP is a link to the automation that replaced it
- Allow engineers to be better owners by showing them the full impact of actions; access controls and confirmation dialogs undermine this — click-fatigue turns off the critical thinking we rely on to keep production safe
- Tools must limit blast radius through execution design: break risky operations into steps small enough to verify impact and stop if needed (e.g. show scope and proposed changes before executing, validate against a single record before operating in bulk)
- Improve operational tools when they are too restrictive, unintuitive, or klunky — don't bypass them; recovering from major incidents always costs more than the friction you avoided
- All production actions must leave an audit trail: who executed it, when it ran, what it changed, and why (ticket, incidentId, or reason message); compliance obligations (SOC 2, PCI) are met by the tools and systems that enact SOPs, not by the SOPs themselves
- Engineers must never access PII or PHI beyond what the task strictly requires; queries must filter or redact sensitive fields, and any access to raw sensitive data must be justified, time-bounded, and covered by an audit trail
- Operational and on-call actions should err on the side of over-logging; incidents are chaotic and a strong paper trail may be the only way to find your way back if things go wrong
