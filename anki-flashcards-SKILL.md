---
name: anki-flashcards
description: Turn what the user just learned, usually in a conversation with Claude and sometimes from slides or notes, into atomic Anki cards as an importable CSV. Use for flashcard this, make cards from this, add this to Anki, even without the words Anki or flashcard.
---

# Anki Flashcards (v2)

Make a small deck of **atomic, normalized** Anki cards that review smoothly with
low energy (e.g. on a phone in the metro). Anki handles the scheduling. The job
here is to write cards that need no fixing and test what the user **actually
learned**.

## Primary mode: cards from a learning conversation

Usually the user calls this skill at the end of a conversation where Claude
taught them a new topic. The source is **the user's learning in that
conversation**, not the topic in general and not everything Claude said.

### Step 1: build a learning inventory (before writing any card)

Go through the conversation and list the **learning events**, grouped by
priority:

1. **Struggles** (highest priority). Places where the user got something wrong,
   misunderstood something that had to be corrected, asked about the same point
   more than once, said "wait, why…" or "I don't get…", or got an exercise or
   Socratic question wrong before getting it right.
2. **New knowledge**. Concepts, mechanisms, terms or commands the user asked
   about or clearly met for the first time, and that the conversation actually
   explained (not just named).
3. **Insights**. Moments where something clicked: a "so that's why…", a
   connection the user made, or an analogy that made the idea work for them.

Leave out:
- things Claude mentioned in passing that the user never engaged with
- tangents and side remarks
- things the user already knew (they explained them correctly without help)
- logistics, tooling chatter, and meta-discussion

A short conversation can give 5 cards. A long session usually gives 10–25. Never
pad. If the inventory is thin, the deck is small.

### Step 2: turn each event into card(s)

- **Use the conversation's own framing.** Use the same terms, the same examples,
  and the same analogy if that analogy is what made it click. A card should feel
  like *their* conversation, not a textbook. Don't add facts, numbers or detail
  that the conversation didn't contain.
- **Struggles get targeted cards.** Write the card so it tests the exact point
  of confusion, i.e. the distinction the user got wrong. For example, if the
  user mixed up a namespace and a cgroup, ask "Which mechanism limits a
  process's CPU, a namespace or a cgroup?" rather than a generic definition
  card. A struggle can get **up to 2 cards from different angles** (e.g. the
  "what" and the "why"). Never phrase the wrong idea as if it were true.
- **New knowledge** gets one card per atomic fact.
- **Insights** usually become a "Why…?" card whose answer is the short reason
  the user came to understand.

### Step 3: deliver with the inventory visible

When delivering, list in 3–6 short lines **which struggles were carded**, so the
user can spot anything missing at a glance. Don't list every card.

## Secondary mode: slides, notes, images

When the source is course material rather than a conversation, skip the
inventory and pull only the concepts that are taught. Everything below still
applies. Stay strictly inside the source: no outside facts, no "enrichment".
Test the concept, not the picture (never "which way does the arrow point").

## One deck, tags do the sorting

All cards go into **one deck**: the one the user names, otherwise `Flashcards`.
Topics are carried only by hierarchical tags (`k8s::cni`, `net::5g::nr`). Never
split into subdecks. Reuse a stable tag scheme across runs (`linux::namespaces`,
`net::routing`, `k8s::networking`). A card may carry several space-separated
tags when it really spans topics.

## Card types: pick per fact, by the shape of the fact

There is no default type. Pick the type that matches the fact's shape:

| Fact shape | Type |
|---|---|
| Concept, function, rationale ("what does / why does…") | **Basic** |
| A two-way pair that is useful in both directions: acronym ↔ expansion, term ↔ one-line meaning, command ↔ effect | **Basic (optional reversed card)** with Add Reverse = `y` |
| A fact that reads naturally as one sentence with one key term; a set of peer items; an ordered sequence or process | **Cloze** |
| Meaning carried by layout or connections (a diagram) | **Image Occlusion** (worklist, not CSV) |

For reversed notes, write **bare items on both sides, not questions**: Front
`IPAM (networking)`, Back `IP Address Management`. The reverse card shows the
Back as the prompt, so a question-shaped Front would give the answer away. Put
the context in parentheses on the Front.

Type-in-answer is excluded: exact-match grading punishes typos on a phone.
Exact-syntax facts become Basic cards whose answer is the exact command or flag.

### Cloze rules

- **You may write the sentence yourself.** The source doesn't need to contain a
  clean sentence already. Rephrase the fact into one, using the conversation's
  wording.
- **The sentence must not give the deletion away.** No synonym, no grammatical
  tell, exactly one correct answer per deletion.
- **Several deletions in one note are fine:** `c1`, `c2`, … each produce a
  *separate* card, so every card is still one retrieval. Use this for:
  - **Sets of peers**: one note, one cN per item that carries its own fact.
  - **Ordered sequences / steps**: one note listing the steps in order, with
    each step as its own cN (overlapping cloze). The visible neighbours give the
    position. This replaces "list the steps" cards.
- Keep a note to 5 deletions at most. Split longer sequences into stages.

### Image Occlusion

Use it only when the source includes a usable figure whose layout is the point.
Write a short worklist instead of CSV rows (format below). If the material is
spatial but there's no image, add one line suggesting a diagram, nothing more.
If nothing is diagrammatic, don't mention occlusion at all.

## Card rules (each card must pass all of them, or it is dropped)

- **Force retrieval.** The user must produce the answer. No multiple choice, no
  true/false, no "which of these" (except a two-option contrast that targets a
  real struggle, as in Step 2).
- **Don't give the answer away.** The prompt must not contain the answer's key
  word or a near-synonym of it.
- **One retrieval per card.** Never a "list / name all…" Basic card. Use cloze
  sets or sequences instead.
- **Context in the prompt.** Tags aren't shown during review, so every prompt
  must be understandable on its own months later. Start with a short context
  when it's needed: `In 5G NR, …`, `In Kubernetes CNI, …`.
- **Answers fit on one line, up to about 10 words.** A term or a short phrase is
  best. A short reason is fine for "why" cards. No paragraphs, no trailing
  period on bare terms.
- **Calibrated difficulty.** One real but achievable retrieval step: not
  trivially obvious, not impossibly broad.
- **Drop rather than ship a shaky card.** If a fact can't be made atomic and
  unambiguous, omit it. A card the user has to fix later costs more than a
  missing one.
- **Canonical stems** so review feels uniform: "What is X?", "What does X do?",
  "Why does X …?", "Which X …?". Deviate only when a stem would distort the fact.

## CSV format

UTF-8 `.csv`. Header lines first:

```
#separator:,
#html:true
#guid column:1
#notetype column:2
#deck:Flashcards
#tags column:6
```

6 columns per row, always (leave unused fields empty):

| Col | Basic | Basic (optional reversed card) | Cloze |
|---|---|---|---|
| 1 guid | id | id | id |
| 2 notetype | `Basic` | `Basic (optional reversed card)` | `Cloze` |
| 3 | Front | Front | Text with `{{c1::…}}` |
| 4 | Back | Back | Back Extra (usually empty) |
| 5 | *(empty)* | `y` (Add Reverse) | *(empty)* |
| 6 tags | `k8s::cni` | … | … |

- **guid:** a stable id so re-importing a fixed file updates cards instead of
  duplicating them. Use the first 10 hex chars of sha1(tag + "|" + field 3 as
  first written). Keep a card's guid unchanged when you revise its wording in
  the same session.
- Quote any field that contains a comma, double-quote or newline, and double
  any internal quotes. Escape literal `<` `>` as `&lt;` `&gt;`. No embedded
  newlines.
- Cloze syntax: `{{c1::answer}}`, optionally with a hint: `{{c1::answer::hint}}`.

Import: **File → Import** in Anki. The three note types are built in.

## Image-occlusion worklist format

```
### Occlusion: <diagram name>
- Source: <which slide/image>
- Deck: <deck>   Tags: <tags>
- Mask separately (Hide All, Guess One): <label 1>, <label 2>, …
- Build: Add → Image Occlusion → paste image → mask the labels above.
```

Only mask labels that test a taught concept.

## Examples

Conversation excerpt: *the user thought a pod's IP was assigned by the kubelet;
Claude corrected it (the CNI plugin assigns it during ADD); the user then asked
why containers in a pod can reach each other on localhost; later the user
learned the CNI call order.* (The guids below are placeholders. Compute real
ones as described above.)

```
#separator:,
#html:true
#guid column:1
#notetype column:2
#deck:Flashcards
#tags column:6
a1f3c9e2b0,Basic,"In Kubernetes, which component assigns a pod's IP: the kubelet or the CNI plugin?",The CNI plugin,,k8s::cni
7c2e91d4aa,Basic,"In Kubernetes, why can containers in the same pod talk over localhost?",They share one network namespace,,k8s::networking
4be0d2f71c,Cloze,"In Kubernetes CNI, the plugin assigns the pod IP during the {{c1::ADD}} operation.",,,k8s::cni
9d03aa61e5,Cloze,"Pod network setup order: kubelet → {{c1::container runtime}} → {{c2::CNI plugin}} → {{c3::IPAM plugin}}",,,k8s::cni
e6a7b8c901,Basic (optional reversed card),IPAM (networking),IP Address Management,y,net::ipam
```

Bad cards (never emit):
- `List the resources a network namespace isolates` → enumeration. Use a cloze set.
- `What isolates a process's network view?` → `network namespace`: the prompt gives the answer away.
- A card about a fact Claude mentioned once in a tangent the user never engaged with.

## Workflow

1. Detect the mode (conversation vs. course material).
2. Conversation mode: build the learning inventory (struggles → new knowledge
   → insights). Material mode: list the taught concepts.
3. For each item, pick the card type by the shape of the fact and write the
   card(s).
4. **Self-check every card** against the card rules: context present, nothing
   given away, one retrieval, one-line answer, inside the source. Fix or drop
   any card that fails. If every card came out the same type, re-check each
   fact's shape once. Uniformity is fine only if the facts really are uniform.
5. Write the CSV, then validate it with a quick script: every row parses to
   exactly 6 fields, every Cloze row has at least one `{{cN::`, guids are
   unique, and there are no raw `<` or `>` outside HTML.
6. Write the occlusion worklist, only if one is needed.
7. Deliver the file(s) with: card count, deck, tags used, and (conversation
   mode) the 3–6 line list of struggles covered.
