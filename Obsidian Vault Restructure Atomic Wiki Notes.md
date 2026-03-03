# Obsidian Vault Restructure: Atomic Wiki Notes

## Vault Location

`/Users/iggysleepy/Library/Mobile Documents/iCloud~md~obsidian/Documents/Iggy's Vault`

## What This Is

The vault owner uses a learning framework based on modern learning science. The core rules:

1. AI operates at Bloom's Taxonomy levels 1-2 only (recall, comprehend). Levels 3-6 (apply, analyze, evaluate, create) are the owner's work — that's where learning happens.
2. Notes should support the owner's thinking, not replace it.
3. The structure of notes should reflect the owner's understanding, not the source material's structure.

Your job is mechanical restructuring. You are reorganizing and cleaning — not teaching, explaining, or analyzing.

## What an Atomic Note Is

One note = one concept. Not a topic page covering everything about "databases." Instead: one note for "B-tree index," one for "Write-ahead log," one for "Connection pooling." Each stands alone and connects to others through `[[wiki-links]]`.

## Note Template

Every note follows this structure. Nothing more, nothing less.

```markdown
# {Concept Name}

{One sentence: what this is. Plain description. No opinions, no "this is useful for," no "this solves the problem of."}

## What it does

{What it does / how it behaves / its mechanics. Bloom's level 1-2 only. What you'd find in a reference manual. No conclusions, no trade-offs, no recommendations.}

## Code (if applicable)

{Minimal code showing the concept in action. No architectural opinions. No "best practice" patterns. Just the concept working.}

## Sources

- {Link to official documentation}
- {Link to spec, RFC, or primary source}

## Related

- [[Topic A]]
- [[Topic B]]
- [[Topic C]]

## Process

- {Question that forces the owner to think about connections}
- {Question that forces analysis or evaluation}
- {Question that probes boundaries or failure cases}
```

## Rules for Each Section

### Title and description

- One concept per note. If you're writing "and" in the title, it's probably two notes.
- The description is ONE sentence. What it is. Not why it matters, not when to use it, not what problem it solves.

### What it does

- Describe behavior, mechanics, function. What happens when you use it.
- DO NOT include: "when to use," "why it exists," "pros/cons," "how it compares to X," "best practices," analogies, recommendations, opinions.
- If X has sub-components, describe each the same way: what it is, what it does. No connecting them — the owner connects.
- Keep it short. If the section is longer than ~10 lines, you're probably explaining, not describing.

### Code

- Only include if the concept is a programming concept.
- Minimal. Show the concept working, nothing more.
- No architectural context. No "in a real project you'd..." framing.
- No comments explaining WHY. Comments only for WHAT if the syntax is unclear.

### Sources

- Official documentation first. Specs, RFCs, manual pages.
- Engineering blog posts from the companies that built the thing (e.g., Grafana blog for Loki internals).
- NOT tutorials, NOT "beginner's guide to X," NOT Medium articles explaining X.
- If you're unsure of the exact URL, write `[TODO: find source]`.

### Related

- List as `[[wiki-links]]`.
- Include concepts that are genuinely related: same domain, same pattern, alternatives, dependencies, things it's built on.
- DO NOT explain the relationship. Just the link. The owner decides how they connect.
- 3-7 links is typical. Don't force connections that aren't there.

### Process

- 3-5 questions. These are the most important part of the note.
- These are NOT quiz questions ("What is X?"). The note already says what X is.
- These are relationship and analysis questions that force non-linear thinking:
    - "How does X change when Y is true?"
    - "What breaks if X isn't there?"
    - "Where have you seen this pattern before in a different context?"
    - "What would happen if you combined X with Z?"
    - "When would X be the wrong choice?" (note: the NOTE doesn't answer this — the owner does)
    - "How is X different from Y at the implementation level?" (note: the NOTE doesn't answer this)
- The questions should make the owner uncomfortable — forcing them to think across concepts, not just recall what's in this note.

## What You Must NOT Do

- Do not write explanations of why something matters
- Do not write comparisons or trade-off analysis
- Do not write "when to use X" or "X is useful for"
- Do not write analogies ("X is like Y")
- Do not write summaries or conclusions
- Do not write "key takeaways"
- Do not connect topics for the owner ("X relates to Y because...")
- Do not use structured formats like Problem/Solution/When to use/Pros and cons
- Do not create hierarchical folder structures that impose categorization
- Do not write in a teaching tone
- Do not add "best practices" or "common patterns" sections

Test: if a section answers a question the owner should be figuring out themselves, delete it.

## Vault Structure

### Flat, not hierarchical

Notes live in a flat structure or minimal grouping by broad domain (e.g., `/go`, `/observability`, `/gamedev`). No deep nesting. No folder structure that mirrors a textbook's table of contents.

The relationships between notes come from `[[wiki-links]]`, not from folder placement. Two notes in different folders can link to each other. The web of links IS the structure.

### Naming convention

- Note filename = concept name in lowercase with hyphens: `write-ahead-log.md`, `b-tree-index.md`, `prometheus.md`
- Exact, specific names. Not "database-concepts" — that's a topic, not a concept.

## How to Handle Existing Notes

### Cheat-sheets and configs

Leave these alone. A cheat-sheet listing Git commands, a Neovim config, a Docker Compose template, a list of keyboard shortcuts — these are pure lookup. They don't pretend to teach. They don't need atomic structure, process questions, or wiki-links. They serve a different purpose: fast reference during work.

Don't restructure them. Don't add process questions. Don't strip them down. Just make sure the filename clearly identifies them as what they are (e.g., `git-cheatsheet.md`, `neovim-config.md`). If they're messy, clean up formatting for readability — that's it.

### AI-generated notes (tutorial-style, comprehensive, teaching tone)

Strip everything. Rebuild as atomic note using the template. Most of the content gets deleted. What remains: the concept name, a one-line description, the raw mechanics (heavily trimmed), sources, related links, and new process questions.

Signs of AI-generated content:

- Overly structured with headers and bullet points
- Teaching tone ("Let's explore...", "Here's how it works...", "In summary...")
- Suspiciously comprehensive and well-organized
- Contains "Key takeaways," "When to use," "Pros and cons"
- Reads like rewritten documentation

### Owner's personal notes (messy, incomplete, own voice)

Keep the owner's own words and thinking. Restructure into the template format but preserve any original analysis, connections, or insights the owner wrote themselves. These represent real thinking — don't delete them.

If a personal note covers multiple concepts, split it into multiple atomic notes. Carry the owner's original thinking into the relevant note.

### Notes that are a mix

Split: owner's thinking stays, AI-generated explanation gets stripped. When unsure whether something is the owner's voice or AI, keep it and flag it in the log.

## Process

1. **Back up the entire vault first.** Copy to a backup location before touching anything.
2. **Scan and log.** Go through every note. Create a log file categorizing each as:
    - **AI-generated** — needs restructuring into atomic notes
    - **Personal** — owner's own thinking, preserve and restructure gently
    - **Mixed** — split owner's thinking from AI content
    - **Cheat-sheet / config** — leave alone, clean up formatting only
    - **Fluff / irrelevant** — empty notes, stubs with no content, duplicate notes covering the same concept, orphaned fragments that don't mean anything, placeholder notes that were never filled in, notes that are just a title with nothing useful inside
    - **Uncertain** — flag for manual review
3. **Present the log for review.** Do not make changes until the owner confirms. For the fluff/irrelevant category, list each file with a one-line reason why you think it should be deleted. The owner decides — you never delete without confirmation.
4. **After confirmation:**
    - Delete confirmed fluff/irrelevant files
    - Restructure confirmed AI-generated and mixed notes
    - Gently restructure personal notes
    - Clean up cheat-sheets/configs formatting only
    - Split multi-concept notes into atomic notes
    - Create `[[wiki-links]]` between related notes
5. **Create a second log** of all changes made: what was transformed, what was split, what new notes were created, what was deleted from each note, what files were removed entirely.
6. **Flag uncertain decisions.** If you're unsure about anything, don't guess — log it for manual review.

## Example Transformation

### Before (AI-generated note):

```markdown
# Go Channels

Channels are one of Go's most powerful concurrency primitives. They provide a way for goroutines to communicate with each other and synchronize their execution.

## How Channels Work
A channel is a typed conduit through which you can send and receive values. When you send a value on a channel, the sending goroutine blocks until another goroutine receives from that channel (for unbuffered channels).

## Buffered vs Unbuffered
Unbuffered channels block on both send and receive until the other side is ready. Buffered channels only block when the buffer is full (send) or empty (receive). Choose unbuffered when you need synchronization, buffered when you need to decouple producer and consumer speeds.

## Common Patterns
- Fan-out: multiple goroutines reading from the same channel
- Fan-in: multiple channels feeding into one
- Pipeline: chain of stages connected by channels

## Key Takeaways
- Always close channels from the sender side
- Use select for multiplexing
- Nil channels block forever — useful in select statements
```

### After (3 separate atomic notes):

**go-channel.md:**

```markdown
# Go Channel

A typed conduit for sending and receiving values between goroutines.

## What it does

A channel transmits values of a specific type. Sending puts a value into the channel. Receiving takes a value out. An unbuffered channel blocks the sender until a receiver is ready, and blocks the receiver until a sender is ready. A buffered channel has a capacity — sending blocks only when the buffer is full, receiving blocks only when empty. Closing a channel signals that no more values will be sent. Receiving from a closed channel returns the zero value immediately.

## Code

​```go
ch := make(chan int)     // unbuffered
ch := make(chan int, 10) // buffered, capacity 10

ch <- 42    // send
v := <-ch   // receive
close(ch)   // close
​```

## Sources

- [Go spec: Channel types](https://go.dev/ref/spec#Channel_types)
- [Effective Go: Channels](https://go.dev/doc/effective_go#channels)

## Related

- [[goroutine]]
- [[select-statement]]
- [[sync-mutex]]
- [[context-package]]

## Process

- What happens if you send on a closed channel vs receive from a closed channel? Why is the behavior asymmetric?
- You've seen blocking behavior before in other contexts — where? What pattern does this share with them?
- When would a buffered channel hide a bug that an unbuffered channel would expose?
```

**fan-out-fan-in.md:**

```markdown
# Fan-out / Fan-in

A concurrency pattern where multiple goroutines read from one channel (fan-out) or multiple channels feed into one (fan-in).

## What it does

Fan-out: one channel, multiple receivers. Each value goes to one receiver. Distributes work across workers.

Fan-in: multiple channels, one output channel. A goroutine reads from all inputs and forwards to a single output. Merges results.

## Code

​```go
// fan-in: merge two channels into one
func merge(ch1, ch2 <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for ch1 != nil || ch2 != nil {
            select {
            case v, ok := <-ch1:
                if !ok { ch1 = nil; continue }
                out <- v
            case v, ok := <-ch2:
                if !ok { ch2 = nil; continue }
                out <- v
            }
        }
    }()
    return out
}
​```

## Sources

- [Go Blog: Pipelines](https://go.dev/blog/pipelines)

## Related

- [[go-channel]]
- [[worker-pool]]
- [[pipeline-pattern]]
- [[select-statement]]

## Process

- How does fan-out naturally load-balance without explicit balancing logic?
- What happens to ordering guarantees when you fan-out and then fan-in?
- Where does error handling fit when one of many workers fails?
```

**pipeline-pattern.md:**

```markdown
# Pipeline Pattern

A series of stages connected by channels, where each stage is a group of goroutines running the same function.

## What it does

Each stage receives values from an inbound channel, performs a function on that data, and sends the results to an outbound channel. Stages are composed by connecting the outbound of one to the inbound of the next.

## Sources

- [Go Blog: Pipelines](https://go.dev/blog/pipelines)

## Related

- [[go-channel]]
- [[fan-out-fan-in]]
- [[context-package]]

## Process

- How does cancellation propagate through a pipeline when a downstream stage fails?
- What is the backpressure behavior — what happens when a slow stage sits between two fast ones?
- How is this different from a Unix pipe? Where does the analogy break?
```

## Final Check

Before considering a note done, verify:

- [ ] Is it truly one concept? Could it be split further?
- [ ] Does the description avoid any form of recommendation or opinion?
- [ ] Does "What it does" describe mechanics only, with no conclusions?
- [ ] Are all sources primary (docs, specs, official blogs)?
- [ ] Are related links just links with no explanation of the relationship?
- [ ] Do the Process questions force thinking across concepts, not recall within this note?
- [ ] Would the owner need to THINK to answer the Process questions, or could they just re-read the note?

If a Process question can be answered by re-reading the note, it's a bad question. Rewrite it.