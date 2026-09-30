# Module 6 Bridge Project — The Campaign
## Notes

### 1. Short-Term Memory and Folding

The short-term memory uses a token budget of 1,200 tokens and keeps the last 4 turns verbatim.

When the context exceeds the budget, older turns are folded into a rolling summary. The fold prompt asks the model to preserve important continuity such as character and companion names, locations, inventory, alive/dead status, promises, important actions, and unresolved events while keeping the summary compact.

Folds during the campaign:

- Night 1: 2 folds
- Night 2: 2 folds
- Night 3: 3 folds

This kept the prompt bounded instead of allowing the entire conversation history to grow continuously.

### 2. Memory vs. Goldfish

Continuity quiz results:

| Night | Memory GM | Goldfish GM |
|------|-----------|-------------|
| Night 2 | 8/10 | 3/10 |
| Night 3 | 10/10 | 4/10 |

The Memory GM performed substantially better because important facts from previous processes were stored persistently and retrieved when relevant.

The Goldfish GM had no persistent long-term memory and therefore forgot or incorrectly reconstructed several campaign facts.

### 3. Prompt Size

Maximum prompt sizes:

| Night | Memory GM | Goldfish GM |
|------|-----------|-------------|
| Night 1 | 2,140 | — |
| Night 2 | 2,153 | 3,132 |
| Night 3 | 2,398 | 2,632 |

The short-term folding policy helped keep the Memory GM's context bounded even while long-term campaign information remained available through retrieval.

### 4. Marra vs. Mirra

The memory system correctly kept the two similar NPC names separate.

By Night 3:

- Marra the smith was dead after the forge fire.
- Mirra the herbalist was alive.

The Night 3 continuity quiz answered both questions correctly. This demonstrates that the retrieval identity check prevented facts about Marra from being confused with facts about Mirra.

### 5. Retcon / Forgetting

On Night 3, the campaign retconned the Tidewater amulet:

> We never took the amulet from the shrine.

The forgetting operation produced:

- 4 amulet rows deleted
- 0 matching rows remaining
- Short-term summary mentioned the amulet after re-fold: no
- GM's next answer mentioned the amulet: no

This shows that forgetting affected both persistent long-term memory and the short-term summary, preventing the deleted information from leaking back into the conversation.

### 6. Result

The project demonstrated both short-term and long-term memory across three separate game-night processes.

Short-term folding controlled context growth, while persistent typed memories preserved campaign continuity across process restarts. Superseding kept newer state changes active without deleting historical rows, event decay reduced the importance of old events, citations exposed where remembered information came from, and explicit forgetting supported retcons.

The final Memory GM quiz scores were 8/10 and 10/10, compared with 3/10 and 4/10 for the Goldfish baseline.