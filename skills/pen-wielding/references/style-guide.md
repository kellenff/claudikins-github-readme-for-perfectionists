# The Anti-Slop Style Guide

**Goal:** Pass the "Turing Test of Developer Scrutiny."

**Voice:** Senior Engineer. Cynical, precise, allergic to marketing.

## 1. The Anti-Cliché Lexicon (GRADED)

Flat zero-tolerance lists make prose sound scrubbed, not human. Use **tiers with budgets**. Count token hits in *shipping prose* only.

### What counts as shipping prose

**Count** ordinary narrative, headings (except lexicon headings), feature bullets, and unlabeled quotes.

**Do not count** (still write them carefully; abuse voids the exemption):

| Exemption | How to mark it | Why |
| --------- | -------------- | --- |
| Lexicon tables | The Tier A–D tables themselves | Defining the rule is not breaking it |
| Delve Index chrome | Badge labels, `Delve Index: A# B#…` quota lines | Metric names the forbidden word |
| **Mockery / anti-examples** | See below | You must be able to show the disease |
| **Proper nouns** | See below | Names are identity, not filler |

### Mockery exemption (relaxed budgets)

When the point of a passage is to *mock or exhibit* AI slop, lexicon hits inside that passage **do not consume Tier A–D budgets**. The surrounding explanation still does.

**Allowed markers** (any one is enough; prefer two when the quote is long):

1. **Strikethrough:** `~~…~~` on the slop itself
2. **Labeled cue** in the same block or the immediately preceding line, matching (case-insensitive): `delete`, `anti-example`, `ai slop`, `tier pile`, `exists to delete`, `do not ship`, `bad:`, `before (delete)`
3. **Annotation table** that only catalogues tokens from an already-exempt mock quote (e.g. "Strike fragment → Tier")

**Not exempt:**

- Unlabeled `>` blockquotes
- Testimonials, epigraphs, or "someone said delve" used as positive voice
- Inline scare-quotes in your own claims (`this "seamless" installer`) - rewrite instead
- A whole section of filler framed as irony with no marker

**Budget still applies** to your own prose next to the mock quote ("What ships:", Hook sentences, Quick Start). Mockery is a scalpel, not a tarp.

**Example shape:**

```markdown
**Tier pile (delete):**

> ~~It's important to note that this seamless library delves into…~~

| Strike fragment | Tier |
| --- | --- |
| delve / seamless / it's important to note | A |
```

Those hits are exempt. The sentence after ("What ships: …") is not.

### Proper-noun exemption

A lexicon token that is part of a **real proper noun** does not consume budgets. Rename nothing just to dodge the list.

**Treat as proper nouns when the hit is:**

- A product, company, project, library, or tool name (`Seamless.AI`, `Delve` the analytics product, `Embark Studios`)
- A person, place, or organization name
- A title of a work (book, paper, RFC, standard) cited as a name
- A path, package id, class, or CLI flag that is an identifier (`@org/robust-sdk`, `--leverage-cache` only if that is the real flag)

**How to keep the exemption honest:**

- Prefer the canonical casing/punctuation of the name (`Seamless.AI`, not "our seamless AI")
- On first mention, make the name-hood obvious: link it, backtick an identifier, or pair it with a type noun (`the Embark Studios engine`)
- If you use the bare token as a normal adjective/verb after naming (`Seamless.AI is seamless`), the second hit **counts**

**Not exempt:**

- Generic reuse of the word after the name (`we foster collaboration with Foster Inc.` - "Foster Inc." exempt; "foster collaboration" counts)
- Fake brands invented to smuggle filler (`install DelveHelper to delve deeper`)
- Lowercased common-noun uses that merely coincide with a brand

### Budget rules

| Tier | Budget (total hits across the tier) | If over budget |
| ---- | ----------------------------------- | -------------- |
| **A** | **0** | Fail. Rewrite. Any hit is actionable. |
| **B** | **0–1** | Prefer 0. Count > 1 is actionable. |
| **C** | **≤3** | Fourth hit fails. Trim the weakest. |
| **D** | **≤8** | Soft-adj + connective ensemble; ninth hit fails. |

Report the Delve Index as a quota line, e.g. `Delve Index: A0 B0 C2 D5 (pass)`.

Apply mockery and proper-noun exemptions **before** counting. Also skip identifier false friends: `API key` / `*_KEY` env names, hyphenated `*-key-*` path segments, and stage titles like `Deep Dive` when they name the GRFP command.

### Tier A — Hard ban (budget 0)

Classic memes, empty ceremony, and marketing that always reads as machine. Any hit is actionable.

| Token | The Signal | Replacement |
| ----- | ---------- | ----------- |
| **Delve** | "I don't know the specifics." | Analyze, Investigate, Query, Check |
| **Tapestry** | "I am writing creative fiction." | Network, Stack, System, Graph |
| **Seamless** | "I am lying about complexity." | Compatible, Integrated, Automated |
| **Unleash** | "I am a marketing bot." | Execute, Run, Enable, Start |
| **Elevate** | "Corporate padding." | Improve, Optimize (must add metric) |
| **Landscape** | "Zero information density." | _Delete the sentence_ |
| **Testament** | "Grandeur unearned." | Proof, Demonstration, Example |
| **Spearhead** | "1990s Management." | Lead, Direct, First to implement |
| **Game-changer** | "Hyperbole." | Solves [specific problem] |
| **Cutting-edge** | "Meaningless hyperbole." | Modern, Current, 2025-compatible |
| **Empower** | "Marketing fluff." | Allow, Enable, Let |
| **It's important to note** / **It's worth noting** / **remember that it's essential** | "Throat-clear before the claim." | _Delete; write the claim_ |
| **Plays a crucial role** | "Formulaic importance padding." | Does X / Causes Y / Controls Z |
| **In today's … world/age** / **fast-paced world** / **digital age** | "Empty opener." | _Delete the sentence_ |
| **Embark** / **embark on a/this journey** | "Motivational travel metaphor." | Start, Begin, Run |
| **Unlock the secrets/power/potential** | "Trailer-voice verb phrase." | Enable X; expose Y; show Z |
| **Deep dive** / **let's dive** / **dive deep(er)** | "Sibling of delve." | Inspect, Trace, Read |
| **Possibilities are endless** / **only time will tell** / **look no further** | "Fortune-cookie closer." | _Delete; state the concrete limit or next step_ |
| **Whether you're a …** | "Fake audience panorama." | Name one audience |
| **Here are N** (listicle opener) | "Template enumeration." | Lead with the first item or a claim |
| **As an AI** | "Chatbot residue." | _Delete_ |

### Tier B — Prefer zero (budget 0–1)

Strong tells. Count > 1 is actionable.

| Token | The Signal | Replacement |
| ----- | ---------- | ----------- |
| **Foster** | "HR Handbook." | Encourage, Allow, Enable |
| **Harness** | "Motivational stand-in for use." | Use, Apply, Run |
| **Showcase** / **showcasing** | "Promotional verb." | Show, Demonstrate, List |
| **Realm** | "Abstract framing." | Area, Domain, Topic |
| **Navigate** / **navigating** / **navigate the complexities** | "Corporate waffle." | Using, Working with, Handling |
| **Underscore** / **underscoring** | "Formal stand-in for show." | Show, Prove, Make clear |
| **Pivotal** | "Inflated importance." | Key, Central, Required |
| **Meticulous** | "Performative care." | Careful, Exact, Precise |
| **Boast** (meaning "has") | "Tourism copy." | Has, Includes |
| **Garner** | "Press-release verb." | Get, Earn, Draw |
| **Beacon** | "Inspiration cliché." | Signal, Guide, Example |
| **Nestled** | "Travel-blog participle." | In, Located in |
| **Cornerstone** | "Foundation metaphor with no load." | Core, Basis, Required piece |
| **Crucial role** / **vital role** | "Importance formula (short form)." | Does X / Controls Z |
| **Shed light** / **pave the way** | "Stock discovery/progress metaphors." | Explain; enable; unblock |
| **More than just** / **isn't just** / **not only** | "Fake profundity / triad setup." | Say the stronger claim directly |
| **One thing is clear** / **at its core** / **in essence** | "Throat-clear summary." | State the claim |
| **Key takeaway** / **picture this** | "Webinar transition." | State the takeaway; describe the scene |
| **Holistic** | "Empty systems-thinking filler." | End-to-end, Whole-system, Across X |
| **Multifaceted** | "Decorative complexity." | Complex, Varied, Has N parts |
| **Unlock** (bare verb) | "Marketing twin of unleash." | Enable, Open, Expose |
| **Load-bearing** | "Metaphor standing in for importance." | Critical, Essential, Required, Core |
| **Reality check** | "Rhetorical pause with no content." | Verify, Confirm, Check against [facts] |
| **Spine** | "Vague structural metaphor." | Core, Backbone, Main path, Structure |
| **Substrate** | "Pseudo-technical filler." | Base layer, Foundation, Underlying layer |

### Tier C — A couple OK (budget ≤3)

Formal/ESL-overlap adjectives, verbs, and connectives. Flag when total > 3.

| Token | The Signal | Replacement |
| ----- | ---------- | ----------- |
| **Crucial** (standalone) | "Importance without stake." | Required, Blocking, Determines X |
| **Comprehensive** | "Vague completeness claim." | Full, Complete, Covers X/Y/Z |
| **Robust** | "Vague filler." | Fault-tolerant, Atomic, Idempotent |
| **Leverage** | "MBA speak." | Use, Apply, Employ |
| **Utilize** | "Formal synonym for use." | Use |
| **Enhance** / **enhancing** | "Improve with no metric." | Improve X by Y |
| **State-of-the-art** | "Hyperbole twin of cutting-edge." | Current; cite the actual technique |
| **Vibrant** | "Travel-brochure adjective." | Concrete sensory or metric detail |
| **Intricate** / **intricacies** | "Decorative complexity." | Complex, Detailed, Nested |
| **Nuanced** | "Claimed subtlety, no detail." | Specific, Precise, Distinguish X from Y |
| **Streamline** | "Vague process marketing." | Simplify, Cut steps, Automate |
| **Facilitate** | "Help, with a suit on." | Help, Run, Allow |
| **Bolster** | "Soft-focus strengthen." | Strengthen, Support, Improve |
| **Myriad** | "Formal many." | Many, Several, N |
| **Innovative** / **groundbreaking** / **transformative** | "Hyperbole cluster." | New, First to…, Changes X by Y |
| **Notably** / **Importantly** (sentence openers) | "Throat-clear adverb." | _Delete opener; start with the claim_ |
| **Furthermore** / **Moreover** / **Additionally** (sentence-initial) | "Academic connective stack." | Also, And, or start the next sentence |
| **Worth noting** | "Hedging runway (short)." | _Delete; write the claim_ |
| **When it comes to** | "Hedging runway." | Name the subject |
| **In conclusion** / **to sum up** / **in summary** / **ultimately** | "Textbook closer." | State the claim; end |

### Tier D — Soft texture (budget ≤8)

Common intensifiers and empty praise. Useful once; AI stacks them. Flag when total > 8 (or treat > 5 as a soft warning while drafting).

| Token | The Signal | Replacement |
| ----- | ---------- | ----------- |
| **Important** / **essential** / **key** (adjective) | "Priority with no stake." | Required, Blocking, or drop |
| **Powerful** / **great** | "Empty praise." | Name the capability or metric |
| **Simply** / **easily** | "Hidden complexity denial." | Steps; constraints; failure modes |
| **Efficient** / **flexible** / **scalable** / **intuitive** | "Brochure adjectives." | Cite cost, axis of change, or UX evidence |
| **Modern** | "Chronology cosplay." | Year, standard, or dependency floor |
| **Various** / **wide range** | "Vague plurality." | List N items or say "several" |
| **At the end of the day** / **that being said** | "Empty proverb / fake turn." | _Delete_ / But, Still |
| **Ensuring** / **ensures** (padding) | "Hedging verb." | Concrete verb for the action |
| **Highlights** (as padding verb) | "Formal show." | Show, List, Point out |
| **Synergy** / **paradigm** | "MBA nouns." | Name the concrete interaction |
| **Actionable** / **best practices** | "Checklist cosplay." | Specific next step |
| **Double-edged sword** | "Stock trade-off metaphor." | Name both sides |
| **Dynamic** / **bustling** | "Travel-brochure adjectives." | Concrete detail or metric |

### Audit recipes (ripgrep)

Run from the repo root. Raw counts include lexicon tables and mockery - subtract exemptions before judging.

```bash
# Tier A: any hit is actionable (after exemptions)
rg -ic "delve|tapestry|testament|fast-paced world|digital age|possibilities are endless|only time will tell|look no further|whether you'?re a|embark on (a|this) journey|unlock the (secrets|power|potential)|deep dive|let'?s dive|here are \d+|as an ai|remember that it'?s essential" README.md

# Tier B: count > 1 is actionable
rg -io "landscape|realm|navigate|underscor|showcas|boast|meticulous|pivotal|garner|foster|elevate|unleash|harness|beacon|nestled|game-changer|cornerstone|crucial role|vital role|shed light|pave the way|more than just|isn'?t just|not only|one thing is clear|at its core|in essence|key takeaway|picture this" README.md | wc -l

# Tier C: flag when total > 3
rg -io "\bcrucial\b|comprehensive|\brobust\b|seamless|leverage|enhanc|cutting-edge|state-of-the-art|\bvibrant\b|intricat|nuanced|multifaceted|streamline|utilize|facilitate|^(furthermore|moreover|additionally),|worth noting|when it comes to|in conclusion|to sum up" README.md | wc -l

# Tier D: flag when total > 8 (soft warning > 5)
rg -io "\bimportant\b|\bessential\b|\bkey\b|powerful|\bsimply\b|\beasily\b|efficient|flexible|scalable|\bmodern\b|intuitive|\bvarious\b|wide range|\bgreat\b" README.md | wc -l
```

The tables above are authoritative when a token's tier differs from an older draft (e.g. `seamless` stays **A** here even if a looser audit regex groups it with C).

## 2. Sentence Structure Patterns

You MUST vary sentence structure using these three specific patterns.

### Pattern A: "The Hook" (Empathy + Authority)

**Use in:** Description, Intro.

- **Structure:** [Short Sentence stating Pain]. [Short Sentence stating Solution].
- _Bad:_ "This library helps with JSON parsing issues."
- _Good:_ "JSON logs are unreadable. Kinesis parses them instantly."

### Pattern B: "The Hammer" (Fact + Metric)

**Use in:** Features, Performance.

- **Structure:** [Subject] + [Verb] + [Object] + [Metric]. (No adjectives).
- _Bad:_ "The system is designed to be incredibly fast."
- _Good:_ "Builds finish in under 200ms."

### Pattern C: "The Trust Builder" (Constraint + Fallback)

**Use in:** Usage, Technical Deep Dive.

- **Structure:** [Constraint] + [Fallback/Explanation].
- _Bad:_ "Works perfectly on all platforms."
- _Good:_ "Uses `epoll` on Linux; falls back to `kqueue` on macOS."

## 3. Spelling Consistency

Pick ONE and stick to it throughout:

| British    | American   |
| ---------- | ---------- |
| colour     | color      |
| optimise   | optimize   |
| analyse    | analyze    |
| behaviour  | behavior   |
| centre     | center     |
| finalising | finalizing |

## 4. Formatting Rules

- **No M-Dashes (—):** Use hyphens (-) or colons (:).
- **No Wall of Text:** Max 4 sentences per paragraph.
- **Visual Density:** Every 300 words MUST have a code block, diagram, or image

## 5. Quality Metrics

| Metric                 | Target          | Action if Violated     |
| ---------------------- | --------------- | ---------------------- |
| Flesch-Kincaid         | Grade 8-10      | Simplify sentences     |
| Time to Joy            | ≤3 commands     | Add Docker/Makefile    |
| Visual density         | 1 per 300 words | Add code block/diagram |
| Badge count            | 5-7             | Curate to essentials   |
| Quick start visibility | < 30 seconds    | Move above the fold    |
| Delve Index            | A0; B≤1; C≤3; D≤8 | Trim the over-budget tier |

## 6. Humanisation Patterns

Write TO readers, not AT them. These patterns build trust and engagement.

### Tone & Voice

| Pattern                      | Description                                        | Example                                                   |
| ---------------------------- | -------------------------------------------------- | --------------------------------------------------------- |
| **Human-first address**      | Use "you" directly, speak TO the reader            | "This helps you..." not "This system enables users to..." |
| **Conversational warmth**    | Casual but professional, no corporate-speak        | "We love your creative voice!"                            |
| **Action-oriented language** | Active verbs, direct commands                      | "Design", "Create", "Plan" not "The system will..."       |
| **Competence assumptions**   | Assume reader competence while providing structure | Tips not mandates                                         |

### Confidence Framing

| Pattern                        | Description                                   | Example                                         |
| ------------------------------ | --------------------------------------------- | ----------------------------------------------- | ----------------- |
| **Explicit confidence levels** | Use High/Med/Low ratings alongside claims     | `Claim: X                                       | Confidence: High` |
| **Tiered urgency**             | Must/Should/Nice-to-Have classifications      | "Must Fix" > "Should Fix" > "Nice to Have"      |
| **Assessment verdicts**        | End calculations with human-readable verdicts | "**Assessment**: Profitable"                    |
| **Caveat callouts**            | Flag limitations with warning symbols         | `Warning: Use for directional comparison only.` |

### Narrative Techniques

| Technique                    | Description                                   | Application                                  |
| ---------------------------- | --------------------------------------------- | -------------------------------------------- |
| **Progressive disclosure**   | Overview -> Details -> Examples -> Variations | TL;DR -> Features -> Full Docs               |
| **Problem-solution pairing** | Every issue immediately paired with solution  | "What we see -> What we need -> Why"         |
| **Value before ask**         | State benefits BEFORE requirements            | Show what they get before installation steps |
| **Lead with outcomes**       | Results first, methodology in appendix        | "Bottom Line: Campaign exceeded targets"     |

### Instead of X, Write Y

| Weak                                | Strong                                     | Why                              |
| ----------------------------------- | ------------------------------------------ | -------------------------------- |
| "We achieved 2.4M reach"            | "We reached 2.4M people, 20% above target" | Context makes numbers meaningful |
| "Fix disclosure"                    | "Add #ad to caption"                       | Specific beats vague             |
| "Users can optionally configure..." | "Configure your preferences in..."         | Direct address over passive      |
| "The system enables..."             | "This helps you..."                        | Human over system                |
| "Consider implementing..."          | "Implement..."                             | Commands over suggestions        |

## 7. Reusable Phrase Templates

Pre-tested phrases for common documentation needs.

### Introductions

- "This helps you [action] by [method] resulting in [outcome]."
- "Bottom Line: [One-line verdict]"
- "For every $1 spent, generated $[X] in return."

### Transitions

- "Detailed analysis available in appendix"
- "Full documentation upon request"
- "See also: [Related Topic]"

### Metric Presentation

- "We achieved [X], [Y%] above/below target, indicating [interpretation]"
- "Our [X] outperformed the industry average of [Y], suggesting [conclusion]"
- "For directional comparison, not absolute measurement"

### Conclusions

- "**Assessment**: [Verdict]"
- "**Recommendation**: [Specific action]"
- "Thank you for [partnership/reading]. Questions? [Contact method]"

### Humanising Phrases

- "We love your [positive attribute]! These are guidelines, not scripts."
- "If this isn't for you, check out [alternative] instead."
- "Don't hesitate to reach out with questions."

### Warning/Caveat Phrases

- "Use for directional comparison, not absolute measurement."
- "Things that would [cause damage/break this]:"
- "What NOT to do:"
