---
name: anki-flashcards
description: >-
  Generate normalized, atomic Anki flashcards as CSV from course material,
  lecture slides, diagrams, or the content of a conversation. Use this skill
  whenever the user wants to turn notes, a slide, a screenshot, a diagram, or a
  chat discussion into Anki cards, build or extend a deck, or says things like
  "flashcard this", "make cards from this", "add this to Anki", even if they
  don't explicitly say "Anki" or "flashcard". Enforces evidence-based,
  low-friction card design and always outputs an importable CSV; it additionally
  produces an image-occlusion worklist when the material is best learned as a
  diagram.
---

# Anki Flashcards

Turn source material into a small deck of **atomic, normalized** Anki cards that
are smooth to review with low mental energy (e.g. on a phone during a commute).
The whole point is to remove friction: consistent format, no cards to rework, no
ambiguity to resolve at review time. Cards are built for active recall and assume
review in a spaced-repetition scheduler — Anki handles the scheduling, so the
job here is purely to write clean cards.

## One deck, tags do the sorting

Every card from a run goes into **a single deck** — never split across decks by
topic. All topic organization is carried by **hierarchical tags**, which are more
flexible than decks and keep the review experience to one queue with nothing to
navigate. Use the deck the user names; if they don't name one, default to a
single deck called `Flashcards`. This applies to occlusion cards too.

## Outputs

1. **A CSV file** — always. Contains every Basic and Cloze card. This is the
   primary deliverable.
2. **An image-occlusion worklist** — *only* when the material is best learned as
   a diagram: either the source contains a figure whose layout is the point, or
   it describes something whose meaning is carried by spatial arrangement or
   connection structure rather than by wording. Image-occlusion cards cannot be
   represented in CSV (they store mask geometry), so they are listed as a short
   worklist the user builds in Anki's native Image Occlusion editor. If the
   material isn't diagrammatic, do not produce this output at all — do not
   mention it.

## Allowed card types (and nothing else)

Only three types. Type-in-answer is deliberately excluded: exact-match grading
punishes typos and is miserable on a phone, which fights the low-friction goal.
Exact-syntax facts become Basic cards whose answer is the exact command/flag.

- **Basic** — the default. A question with one unambiguous answer.
- **Cloze** — only under the strict trigger below. Chosen for fast authoring
  from source text, never because it "retains better" (it doesn't, vs Basic).
- **Image Occlusion** — for diagrammatic material only; emitted to the worklist,
  not the CSV.

## Card-type decision procedure

Apply in order, per fact:

1. **Is it best tested on a diagram?** — its meaning is carried by spatial
   arrangement or connection structure rather than by wording → **image-occlusion
   worklist entry** (see below).
2. **Does it meet ALL of the Cloze conditions?** → **Cloze row.**
   - The source already states the fact as one clean declarative sentence, AND
   - the card is made by deleting exactly **one** load-bearing term (or one
     tightly-bound phrase), AND
   - the remaining sentence does **not** give the deleted term away (no synonym,
     no grammatical tell) and has exactly **one** correct answer.
   - If you are tempted to add a second, independent deletion (`c2`) for a
     *different* fact, that breaks atomicity → split into two cards instead.
3. **Otherwise → Basic row.** When in doubt, Basic. Basic gives less away than
   Cloze, which is why it is the default.
4. **If the fact cannot be made atomic and unambiguous by any of the above →
   drop it.** Emitting a shaky card the user must later fix is worse than
   omitting it. Short, clean decks beat exhaustive ones.

## Non-negotiable card rules

Every card must satisfy all of these, or it is dropped. Each rule has a short
reason so you can apply judgment on cases the rules don't spell out.

- **Force retrieval.** Every card is a genuine recall prompt — a question, or a
  deletion the reviewer fills from memory — never a statement to re-read, because
  retrieving from memory is what builds durable recall. No recognition: no
  multiple-choice, no "true or false", no "which of these…". The reviewer must
  *produce* the answer.
- **Don't give the answer away.** The prompt — and for Cloze, the words left in
  the sentence — must not telegraph the answer; delete or ask for the
  load-bearing term, not an incidental one. This is the single most important
  quality rule: it stops cards from decaying into sentence pattern-matching
  instead of real recall.
- **One retrieval per card — atomicity.** Exactly one fact, one answer.
  - **Never** write a card whose prompt asks to enumerate ("list the N…",
    "name all…", "what are the components of…"). Full-set recall is error-prone
    and high-friction — banned.
  - When the source presents a set of N peer items, generate **one card per
    item** that carries a distinct, testworthy fact (identity or role) — not one
    card for the whole set. Do **not** manufacture near-duplicate cards for
    members that carry no independent fact; prefer one clean conceptual card.
- **Calibrated difficulty.** One real but achievable retrieval step per card —
  not trivially obvious, not impossibly broad — since the useful effort is what
  strengthens memory.

## Content & style rules

- **Stay strictly inside the source.** Only make cards for concepts actually
  present in the provided material. No outside facts, no "enrichment", no cards
  that go beyond what was taught.
- **Test the concept, not the artifact.** For a slide or diagram, test what it
  teaches — never incidental properties of the image itself (e.g. never "which
  way does the arrow point", "what colour is the box").
- **Answers are one line.** A single term or short phrase, quiz-style. No
  explanatory sentences, no paragraphs, no trailing period on bare terms.
- **Keep decks short.** Core concepts only. Favor a tight deck over coverage.
- **Consistent phrasing.** Prefer canonical stems so review feels uniform:
  "What is X?" (definition), "What does X do?" (function), "Why does X …?"
  (rationale), "What/which … X?" (identification). Deviate only when a stem
  would distort the fact.

## CSV format

Emit a single UTF-8 `.csv` file. Use these exact header lines first (replace
`Flashcards` with the user's deck name if they gave one), then one row per card:

```
#separator:,
#html:true
#notetype column:1
#deck:Flashcards
#tags column:4
```

`#deck:` sets one deck globally for every row — that is what enforces the
single-deck rule. Each row then has **4 comma-separated columns, in this order**:

| Col | Meaning | Basic | Cloze |
|-----|---------|-------|-------|
| 1 | note type | `Basic` | `Cloze` |
| 2 | field 1 | Front (the question) | Text (sentence with `{{c1::…}}`) |
| 3 | field 2 | Back (the answer) | Back Extra (usually empty) |
| 4 | tags (space-separated, hierarchical `::`) | `linux::namespaces` | same |

Rules for valid CSV:
- **Quote any field** containing a comma, double-quote, or newline by wrapping it
  in double-quotes and doubling internal quotes (`"` becomes `""`). Otherwise
  leave unquoted.
- Because `#html:true` is set, a literal `<` or `>` in a field must be escaped as
  `&lt;` / `&gt;` so Anki doesn't read it as a tag.
- Keep every field on one physical line (no embedded newlines).
- Cloze deletions use `{{c1::answer}}`. Optional hint: `{{c1::answer::hint}}`.

**Tags carry all the sorting.** Assign a hierarchical tag matching the topic, so
cards self-sort by tag in searches, filters, and filtered decks — the user never
sorts by hand and never navigates multiple decks. Reuse a stable scheme, e.g.
`linux::namespaces`, `net::routing`, `k8s::cni`, `k8s::networking`. A card may
carry more than one tag when it genuinely spans topics.

Deliver the file and tell the user to import via **File → Import** in Anki
(recent versions auto-detect the header directives; the note types `Basic` and
`Cloze` are built in).

## Image-occlusion worklist (only when the material is diagrammatic)

When step 1 of the decision procedure fires and the source includes a usable
image, output a short list (markdown or plain text — not CSV) with one entry per
diagram. Use the same single deck as the CSV; tags carry the topic:

```
### Occlusion: <short name of the diagram>
- Source: <which slide/image>
- Deck: Flashcards (the run's single deck)   Tags: <hierarchical tags>
- Mask each of these labels separately (one card each, "Hide All, Guess One"):
  - <label 1>
  - <label 2>
  ...
- Build in Anki: Add -> Image Occlusion -> paste the image -> mask the labels above.
```

Only list labels that test a taught concept (structure identity / relationship),
never decorative parts of the figure.

If the material is diagrammatic but the source includes **no** usable image,
don't invent labels. Add a single-line flag instead — e.g. "This is spatial:
consider making or finding a diagram of it and occluding that." Keep it to one
line; don't expand it into a section.

## Worked examples

Header once, then rows. All rows share the one deck (from `#deck:`); the tags do
the topic sorting.

**Basic (definition — description in the prompt, term as the answer):**
```
Basic,"What isolates a process's view of network interfaces, routing, and firewall rules?",A network namespace,linux::namespaces
```

**Basic (function):**
```
Basic,What does the CNI ADD operation do?,Attaches a pod to the network,k8s::cni
```

**Cloze (clean source sentence, one load-bearing deletion, no tell):**
```
Cloze,The CNI {{c1::ADD}} operation attaches a pod to the network.,,k8s::cni
```

**Dropped (correctly omitted):** a slide bullet "A pod has one IP, shared by all
its containers, assigned by the CNI plugin at ADD time, reachable cluster-wide"
packs four facts. Do not make one card. Split into the atomic facts that each
carry a distinct point (e.g. "How many IPs does a pod have? -> One", "Which
component assigns a pod's IP? -> The CNI plugin"), and drop any that would be
near-duplicates or mere recognition.

**Enumeration (banned):** never emit
`Basic,List the three resources a network namespace isolates,...` — a list
prompt. If the individual resources each matter, test them one at a time;
otherwise keep the single conceptual card.

## Workflow when running this skill

1. Read the source (conversation, slide text, or image) and pull only the taught
   concepts.
2. For each concept, run the decision procedure; write Basic/Cloze rows, queue
   diagrammatic facts for the worklist, drop anything that can't be made atomic.
3. Write the CSV file with the exact header and columns above — one deck via
   `#deck:`, topics carried by tags.
4. If (and only if) any diagrammatic facts were queued, write the occlusion
   worklist.
5. Deliver the CSV (and worklist if present). Keep the deck tight; state the
   card count, the deck name, and the tags used so the user can import with
   confidence.
