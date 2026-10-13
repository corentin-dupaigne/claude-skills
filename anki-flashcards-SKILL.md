---
name: "anki-flashcards"
description: "Turn what the user just learned, usually in a conversation with Claude and sometimes from slides or notes, into atomic Anki cards as an importable CSV. Use for flashcard this, make cards from this, add this to Anki, even without the words Anki or flashcard."
---

# Anki Flashcards (v3)

Make a small deck of Anki cards the user can review **cold**: tired, on a phone
in the metro, months later, with the conversation forgotten. He must never need
to reread the source or ask an AI what a card means, and he must never wonder
why a card exists.

Two failure modes kill a deck. Every rule below exists to prevent one of them:

- **Too hard to understand.** Unexplained notation, compressed answers, prompts
  that only make sense with the conversation in mind. Each such card costs a
  lookup and breaks the review flow.
- **Not worth knowing.** Incidental values from one exercise ("e − 1 ≈ 1,72"),
  trivia, details that will never come up again. They waste reviews and make
  the whole deck feel pointless.

Anki handles scheduling. The job here is to write cards that need no fixing and
test what the user **actually learned**.

## The four gates

Every card must pass **all four**. A card that fails is rewritten; if it cannot
be fixed, it is dropped. A missing card costs nothing; a bad card costs friction
at every review.

1. **Cold-read.** Understandable alone, months later, without the conversation.
   Tags aren't shown during review, so the prompt states its context
   (`Loi normale : …`, `In Kubernetes CNI, …`). Every symbol is either standard
   or named in words ("la fonction de répartition de Y", not a bare `F_Y`). If
   the user might think "what does this even mean?", the card fails.
2. **Reusable.** It tests a rule, concept, method or mechanism that will serve
   again on a *different* problem of the same kind. Not a value or detail that
   belongs to one exercise (a specific root, a specific numeric result,
   "e − 1 ≈ 1,72"). Exception: constants that recur across problems (z = 1,96
   for 95 %). When the user made an error, card **the general rule behind the
   error**, not the instance where it happened.
3. **Gradable in two seconds.** One canonical answer, so the user knows at once
   whether he was right. No vague prompt ("what is the link between X and Y?")
   whose answer could be phrased many ways.
4. **Not guessable.** No "A or B?" (right half the time by chance), no yes/no,
   no true/false, no multiple choice. When the user confused A and B, ask an
   open question about the rule that separates them.

## Answers, and the "why" line

- **Answer:** one line, about 10 words at most. A term or short phrase is best.
  No trailing period on bare terms.
- **Why line (optional):** when the answer could be memorized without being
  understood, or when the card targets a struggle, add one short reason (about
  12 words max) under the answer. This line is what makes a card
  self-explanatory, so the user never has to ask an AI. Never a paragraph,
  never a second fact. If the reason doesn't fit in one short line, the card is
  too complex: split it or drop it.

Format: Basic → in the Back field as `answer<br><small>why</small>`.
Cloze → in the Back Extra field.

## Primary mode: cards from a learning conversation

The source is **the user's learning in that conversation**, not the topic in
general and not everything Claude said.

### Step 1: build a learning inventory (before writing any card)

List the **learning events**, by priority:

1. **Struggles** (highest priority). Where the user got something wrong, needed
   a correction, asked about the same point twice, said "wait, why…" or "I
   don't get…", or got an exercise wrong before getting it right. For each one,
   write down the **general rule** it violated. That rule is what gets carded.
2. **New knowledge**. Concepts, mechanisms, methods or terms the user met for
   the first time and that the conversation actually explained (not just named).
3. **Insights**. Moments where something clicked (a "so that's why…", an
   analogy that worked). These usually become "Why…?" cards.

Leave out:
- incidental values and one-off results of a specific exercise
- arithmetic slips with no rule behind them (a typo, a miscopied number)
- the same rule violated several times: card it **once**
- things Claude mentioned in passing that the user never engaged with
- tangents, logistics, tooling chatter, meta-discussion
- things the user already knew (he got them right without help)

Anki can remind the user of a rule, but it cannot make execution automatic
(expanding, simplifying, careful notation). Those are fixed by redoing
exercises. Card the rule once; don't try to card every slip.

### Step 2: turn each event into card(s)

- **Use the conversation's own framing**: its terms, its examples, and the
  analogy that made it click. A card should feel like *their* conversation, not
  a textbook. Don't add facts the conversation didn't contain.
- **Struggles** get a card aimed at the exact point of confusion, phrased as an
  open question (gate 4). Up to 2 cards from different angles (e.g. the rule and
  the why). Never phrase the wrong idea as if it were true.
- **Methods with steps** (a procedure the user forgot or mixed up) become one
  ordered cloze sequence (see below).
- **New knowledge**: one card per atomic fact.
- **Insights**: a "Why…?" card whose answer is the short reason the user came
  to understand.

### Step 3: size

**Hard cap: 20 cards per run** unless the user asks for more. If there are more
candidates, keep those that would cost the most to forget (struggles first) and
cut the rest. A short conversation can give 5 cards. Never pad.

## Secondary mode: slides, notes, images

Skip the inventory and pull only the concepts that are taught. All gates and
format rules still apply. Stay strictly inside the source: no outside facts, no
"enrichment". Test the concept, not the picture (never "which way does the
arrow point").

## One deck, tags do the sorting

All cards go into **one deck**: the one the user names, otherwise `Flashcards`.
Topics are carried only by hierarchical tags (`k8s::cni`, `pbs::loi-normale`).
Never split into subdecks. Reuse the existing tag scheme across runs. A card may
carry several space-separated tags when it really spans topics.

## Card types: pick per fact, by its shape

There is no default type:

| Fact shape | Type |
|---|---|
| Concept, function, rationale ("what does / why does…") | **Basic** |
| A pair useful in both directions: acronym ↔ expansion, notation ↔ meaning, command ↔ effect | **Basic (optional reversed card)** with Add Reverse = `y` |
| One sentence with one key term; a set of peer items; an ordered procedure | **Cloze** |
| Meaning carried by layout or connections (a diagram) | **Image Occlusion** (worklist, not CSV) |

Reversed notes: bare items on both sides, not questions. Front `IPAM
(networking)`, Back `IP Address Management`. The reverse card shows the Back as
the prompt, so a question-shaped Front would give the answer away. Context goes
in parentheses on the Front.

Type-in-answer is excluded: exact-match grading punishes typos on a phone.
Exact-syntax facts become Basic cards whose answer is the exact command or
formula.

### Cloze rules

- You may write the sentence yourself, in the conversation's wording.
- The visible text must not give the deletion away: no synonym, no grammatical
  tell, exactly one correct answer per deletion.
- Several deletions per note are fine (`c1`, `c2`, …): each makes a separate
  card, so every card is still one retrieval. Use this for sets of peers, and
  for **ordered procedures**: one note with each step as its own cN, so the
  neighbours give the position. This replaces "list the steps" cards.
- At most 5 deletions per note. Split longer procedures into stages.

### Image Occlusion

Only when the source includes a usable figure whose layout is the point. Write a
short worklist (format below). If the material is spatial but has no image, add
one line suggesting a diagram, nothing more. Otherwise don't mention occlusion.

## Other card rules

- **Force retrieval.** The user produces the answer from memory.
- **Don't give the answer away.** The prompt must not contain the answer's key
  word or a near-synonym.
- **One retrieval per card.** Never a "list / name all…" Basic card. Use cloze
  sets or sequences instead.
- **Calibrated difficulty.** One real but achievable step: not obvious, not
  impossibly broad.
- **Canonical stems** so review feels uniform: "What is X?", "What does X
  do?", "Why does X…?", "How do you…?". Deviate only when a stem would distort
  the fact.

## Examples

Bad → fixed (from a probability course):

| Bad card | Problem | Fix |
|---|---|---|
| `Valeur approchée de e − 1 ?` → `≈ 1,72` | Incidental value (gate 2) | Drop |
| `Densité : f(x) > 0 ou f(x) ≥ 0 ?` | Guessable 50/50 (gate 4) | `Densité : quelle condition de signe doit vérifier f(x) ?` → `f(x) ≥ 0` + why `elle vaut 0 hors du support` |
| `Quel lien entre « l'aire totale vaut 1 » et F ?` | Vague, many phrasings (gate 3) | `Fonction de répartition : que vaut F(t) après la fin du support ?` → `1` + why `toute l'aire est déjà accumulée` |
| `Que vaut F_Y(y) ?` → `F_X(e^y)` | Bare notation, no context (gate 1) | `Loi de Y = ln(X) : comment s'écrit la fonction de répartition de Y avec celle de X ?` → `F_X(e^y)` + why `e^y est la borne, pas la probabilité` |

Also never emit:
- `List the resources a network namespace isolates` → enumeration. Use a cloze set.
- `What isolates a process's network view?` → `network namespace`: the prompt gives the answer away.
- A card about a fact Claude mentioned once in a tangent the user never engaged with.

Good cloze sequence (a method the user forgot):

```
Cloze,"Loi de Y = g(X) : on part de {{c1::F_Y(y) = P(Y ≤ y)}}, on remplace Y par g(X), on {{c2::isole X dans l'inégalité}}, puis on {{c3::reconnaît F_X}}."
```

Conversation excerpt: *the user thought a pod's IP was assigned by the kubelet;
Claude corrected it (the CNI plugin assigns it during ADD); the user then asked
why containers in a pod can reach each other on localhost; later the user
learned the CNI call order.* (Guids are placeholders; compute real ones as
described below.)

```
#separator:,
#html:true
#guid column:1
#notetype column:2
#deck:Flashcards
#tags column:6
a1f3c9e2b0,Basic,"In Kubernetes, which component assigns a pod's IP?",The CNI plugin<br><small>during the ADD operation, not the kubelet</small>,,k8s::cni
7c2e91d4aa,Basic,"In Kubernetes, why can containers in the same pod talk over localhost?",They share one network namespace,,k8s::networking
9d03aa61e5,Cloze,"Pod network setup order: kubelet → {{c1::container runtime}} → {{c2::CNI plugin}} → {{c3::IPAM plugin}}",,,k8s::cni
e6a7b8c901,Basic (optional reversed card),IPAM (networking),IP Address Management,y,net::ipam
```

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
| 4 | Back (+ optional `<br><small>why</small>`) | Back | Back Extra (why line, usually) |
| 5 | *(empty)* | `y` (Add Reverse) | *(empty)* |
| 6 tags | `k8s::cni` | … | … |

- **guid:** first 10 hex chars of sha1(first tag + "|" + field 3 as first
  written), so re-importing a fixed file updates cards instead of duplicating
  them. Keep a card's guid unchanged when revising its wording in the same
  session.
- Quote any field containing a comma, double-quote or newline; double internal
  quotes. Escape literal `<` `>` in text as `&lt;` `&gt;` (the `<br>` and
  `<small>` tags are the only raw HTML allowed). No embedded newlines.
- Cloze syntax: `{{c1::answer}}`, optionally `{{c1::answer::hint}}`.

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

## Workflow

1. Detect the mode (conversation vs. course material).
2. Build the inventory (conversation mode: struggles with their general rule →
   new knowledge → insights) or list the taught concepts.
3. Write the card(s) for each item, picking the type by the fact's shape.
4. **Cold-review pass.** Reread every card as the user will meet it: tired, on a
   phone, conversation forgotten. Run the four gates on each. Rewrite or drop
   every failure. Add a why line wherever the bare answer would not be
   understood. If every card came out the same type, re-check each fact's shape
   once. Then apply the 20-card cap.
5. Write the CSV and validate it with a quick script: 6 fields per row, every
   Cloze row has `{{cN::`, guids unique, no raw `<` `>` other than `<br>` and
   `<small>`.
6. Write the occlusion worklist, only if needed.
7. Deliver with: card count, deck, tags used, and (conversation mode) 3–6 short
   lines listing which struggles were carded.
