---
name: "cert-study-map"
description: "Build a mobile last-minute study page for any IT certification: blueprint-aligned overview diagrams, flip cards, scenario quiz, numbers, readiness tracking."
---

# Cert Study Map

Builds one self-contained HTML study page for a certification exam (Cisco, AWS, Azure, CompTIA, etc.) and publishes it as an artifact. The page has four tabs: Overview (domain weights, readiness panel, deep-dive diagrams), Trigger cards (flip cards), Scenario quiz (with explanations and topologies), and Numbers (cheat-sheet tables). Content comes from the user's own study material plus the official exam blueprint.

Reference build: the "ENSLD Last-Minute Study Map" artifact (https://claude.ai/artifact/Aph2tajzrRhYTQBUtQzB8j). If it is reachable with read_artifact, reuse its CSS, layout, renderer and quiz/card logic as the template instead of rewriting them.

## Workflow

Set up a task list. Tell the user each draft is live as soon as it's published.

### 1. Pin the official blueprint first
- WebSearch for the exam code + "exam topics" + version. Prefer the vendor's own PDF/page and release notes. If the vendor URL can't be fetched, use a training partner page that reproduces the list verbatim (e.g. Global Knowledge), and confirm the version.
- Record every domain with its weight and every sub-objective and sub-item, using the official numbering.
- Check what changed between versions: topics added and **topics dropped**. Dropped topics must not appear on the page, even if older books still cover them.
- Record exam facts: question count, duration and pass score. If the vendor doesn't publish a cut score, say so and label third-party estimates as estimates.

### 2. Gather material
- If a Project is attached, use project_search on each sub-objective (5–6 searches per batch, run in parallel). Otherwise use files the user attached. Ask for material if there is none.
- Tag every fact with a short source label (e.g. OCG, CSLG, ARCH). When books disagree, show both values on the page and say which to expect on the exam.
- Search for material on every sub-objective, not just the familiar ones. Watch for blueprint vocabulary the books don't use (e.g. "platform abstraction" = StackWise/VSS) and add a card that maps the term.

### 3. Set coverage targets (state them on the page)
- Expected exam questions per sub-objective ≈ exam length × domain weight ÷ sub-objectives in that domain.
- Minimum per sub-objective: **5 cards, 5 quiz questions, 1 visual**. For a small domain (≤10 % weight) use 4 cards and 3 questions.
- Total quiz ≥ about two full exams of unique questions. Total cards ≥ 5 × sub-objectives.
- "Ready" rule: ≥85 % on **first-try** answers in every domain, with ≥80 % of the quiz seen. Present this as a heuristic, not a guarantee.

### 4. Data model (all in one `<script>`)
- `D` domains: `{key:{n:name,w:weight,c:color,t:[tags]}}`
- `C` cards: `[domain, requirement/trigger, solution, why, source, subObjective]`. Frame the front as a design requirement and the back as the solution plus a one-line reason.
- `Q` quiz: `[domain, scenario, options[4], correctIndex, explanation(with source), subObjective]`. Use scenario wording, plausible distractors, and a "why" in every explanation. Options are shuffled at render time.
- `OBJ` official sub-objective names keyed by official number. Every C/Q tag must exist in OBJ; validate there are no orphans.
- **Domain and tag must agree:** an item's domain key must be the domain its sub-objective belongs to (e.g. a 5.5 cloud card goes in Automation, not WAN, even if it feels WAN-ish). Derive or validate it from the tag. Otherwise domain bars and sub-objective rows disagree.
- **Stable progress keys:** key localStorage progress (cards known, first-try quiz results) by a hash of the item text, not by array index. Removing or reordering items must not shift the user's progress onto other items.
- Visual panels are `<details class="sec" data-d="domain" data-o="subObjective">` so coverage can be counted.

### 5. Page sections
- **Overview:** domain weight bars (tap → filtered cards), an "Am I ready to pass?" panel, a "How to read questions" signal-word panel, then deep-dive panels per domain. Each panel has an SVG diagram or comparison table plus a short `.mn` fact box.
- **"Am I ready to pass?" measures the user, never the content.** Lay it out like this:
  - A status banner at the top: red "Not ready yet: X of N domains meet the target" or green "Ready".
  - One row per domain: quiz seen (target ≥80 %), first-try correct (target ≥85 %), cards known, ✓/✗. Red = far off, orange = within 70 % of the target, green = met.
  - Below it, a sub-objective table with the user's first-try score per objective.
  - A reset button.
  - The **content check** (cards, quiz and visuals per domain vs the page's own targets) goes in a collapsed `<details>` labelled "Content check: does this page cover enough? (not your progress)". Never show content bars as the main green/red signal, because users read green as "I'm ready".
- **Cards:** filter chips, flip card, "Again" / "Got it". Progress lives in localStorage (always wrapped in try/catch). Cards with a diagram get a taller `.card.big`.
- **Quiz:** domain filter, shuffled options, Correct/Not quite plus the explanation, and a topology either with the question (when it's needed to answer) or in the explanation (when it shows why). Store first-try results per question index.
- **Numbers:** ports/IDs, addresses, timers, scale limits, thresholds, sizing tables, all with source tags and conflict notes. Where values form a pattern, show them as a **grid** rather than a list (e.g. AF classes × drop precedence with DSCP values, availability nines → downtime), plus the rule that generates them (AFxy = 8x + 2y).

### 6. Diagrams: reuse the tiny renderer
One `drawDG(key)` turns a spec into SVG (viewBox width 360, mobile-first):
`{h, z:[[x,y,w,h,label,color]], n:[[id,x,y,label]], l:[[a,b,style,label,offset]], t:[[x,y,text,color]]}`
Link styles: s solid, d dashed, x failed/blocked (red ✗), b good/green, f flow arrow (orange), ga flow arrow (green, e.g. the preferred or outbound direction), o orange, r red, p pink dashed, t teal, a arc (for tunnels over a core).
Rules:
- Clip line ends to box edges.
- Parallel links use offset ±6 (absolute y for horizontal links).
- Labels go above or below horizontal links, and beside vertical ones.
- Long text goes in `t` notes, not on diagonal links.

Map diagrams with `QD` (quiz scenario prefix → ["q"|"e", key]) and `CD` (card requirement prefix → key). To reuse a spec inside an overview panel, add `<div data-dg="key"></div>` under a short bold caption and fill all of them on load. Define the renderer, DG, QD and CD **at the top of the script**, before any render() call, to avoid temporal-dead-zone errors.
Add diagrams only where a topology, flow or comparison adds understanding. Leave definitions, ports and numbers as text.

Diagram types worth having in most design or architecture exams:
- **Placement:** where each tool or role sits on one path, e.g. QoS mark/police/queue/shape from endpoint to WAN edge, or STP root aligned with the FHRP active switch.
- **Failure and failover:** primary + backup and what triggers the switch.
- **Flow and loop prevention:** e.g. RPF, BGP transit.
- **Hierarchy and summarization:** e.g. /48 → /56 per site → /64.
- **Option comparisons drawn side by side:** e.g. Internet exit choices, ABR placement A vs B, without vs with a feature (no stubs vs stubs).
- **A "what's wrong here?" variant** shown with the question, with the correct version in the explanation or on the card.

**Hard-to-learn mechanisms get step-by-step diagrams.** The topics students struggle with most are mechanisms that run as invisible steps or follow direction and recipient rules. Typical examples:
- which messages cross which boundary (LSA types across areas)
- which knob steers which direction (outbound vs inbound path attributes)
- who receives what (route reflector rules)
- how a lookup or overlay resolves (LISP map-register/map-request → VXLAN, NHRP redirect)
- in what order components come up (SD-WAN: orchestrator → controllers → control protocol → data tunnels)
- how a tree or path changes over time (PIM register → shared tree → SPT)

Draw these with short numbered labels on the links (① ② ③) and one matching numbered note line per step underneath, so the reader follows the order. Put the step diagram in the domain's overview panel and link it to the related card and quiz explanation. Without reliable public data on what students find hard, choose by this mechanism test and tell the user that's the basis.

After the first full build, ask which blueprint items are "where does X go" or "what happens in which order" questions without a picture, and add those before anything else.

### 7. Validate before every publish
- Extract the script and run `node --check`.
- Use Playwright with the system Chrome (`executable_path="/usr/bin/google-chrome"`) at 390×844:
  - no page errors
  - `document.body.scrollWidth` == 390
  - all `details` opened and screenshotted
  - every DG key rendered once onto a single sheet and visually checked for overlapping labels
  - layout: every node box stays inside 0–360 (x ± w/2), no link crosses a `t` note, and two links never share a midpoint label
  - every `[data-dg]` placeholder filled (count the SVGs)
- Check counts per OBJ against the minimums. Fix any gaps, any orphans and any QD/CD prefixes that don't match an item.
- Check that each item's domain matches its sub-objective's domain. List any mismatches and fix them before publishing.
- After removing content (scope fixes), recount. Removals can push a sub-objective or domain under its minimum, so top it up with in-scope items.
- Simulate a few quiz answers and card clicks, then screenshot the readiness panel. It must show red/not-ready for a fresh user even when all content targets are met.
- Scope sweep: grep for dropped-version topics and for tools/media the blueprint doesn't list. Remove them, but keep them as wrong answer options where that's useful.

### 8. Publish
- Read the artifacts `guide` (pages) before the first publish, and call list_types. No artifact type fits an interactive study tool, so publish the hand-built HTML file with publish_files (title "<EXAM> Last-Minute Study Map", icon "cards").
- Republish to the same slug with base_version on every revision. Keep the localStorage key stable so the user keeps their progress.

## Language & tone
Write page content in the language the user asks for (default: the exam's language, usually English). Keep the chat summary short: what changed, counts, and any book conflicts or caveats. Don't recap the steps.

## Out of scope
No exam dumps or "real exam questions"; write original questions from the material. Don't predict pass/fail, only report coverage and first-try accuracy.