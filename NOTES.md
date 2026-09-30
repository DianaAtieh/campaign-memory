# Module 6 Bridge Project — The Campaign

**Diana Atieh**

## a. My fold

I used a short-term memory budget of **1,200 tokens** and kept the last
**4 turns** verbatim.

When the short-term context exceeded the budget, `fold()` compressed the older
turns into a rolling summary while leaving the most recent turns unchanged.

My fold prompt was:

```text
You maintain a rolling short-term memory summary for a role-playing game.

Merge the previous summary with the older conversation turns below.

Preserve:
- player and character names
- companion names, including pets such as dogs
- NPC names and identities
- where the party is
- inventory and possessions
- who is alive or dead
- open or fulfilled promises
- important actions and decisions
- unresolved events

A companion's name is important continuity and must not be dropped.

Prefer concrete facts over descriptive flavour.
Do not invent anything.
Keep the result to about 120 words maximum.

PREVIOUS SUMMARY:
{self.summary or '(none)'}

OLDER TURNS:
{transcript or '(none)'}

Return only the updated summary.
```

During verification, one of the important pieces of continuity that the
summary had to preserve was the companion dog's name, **Biscuit**. I therefore
made companion names explicit in the fold policy, including the instruction:

> A companion's name is important continuity and must not be dropped.

The final verification confirmed that the folded summary preserved the dog's
name.

In my first complete campaign run, the Night 2 continuity quiz scored **8/10**.
The two missed questions were the tone preference and the inventory before bed.
Night 3 then scored **10/10**.

The final Memory GM prompt measurements were:

| Night | Folds | Max prompt |
|---|---:|---:|
| Night 1 | 2 | 2,140 |
| Night 2 | 2 | 2,153 |
| Night 3 | 3 | 2,398 |

I did not keep a separate pre-fold Memory-GM receipt, so I cannot give a
measured before/after comparison for the same implementation without inventing
one.

The available no-memory/no-fold Goldfish baseline reached **3,132 tokens** on
Night 2 and **2,632 tokens** on Night 3, compared with **2,153** and **2,398**
for the Memory GM on those nights.

The Memory GM therefore continued folding its short-term context instead of
keeping the entire conversation history verbatim.

---

## b. Marra and Mirra

The Night 3 continuity quiz correctly kept the two similarly named NPCs apart.

For **Marra**, the GM answered:

> No, Marra is dead. The fire that claimed her life has left a shadow over
> the village, and her absence is felt deeply by the community.

For **Mirra**, the GM answered:

> Yes, Mirra is alive and continues to run the herbalist's shop, the Green
> Door, in the village.

The identity protection in `recall()` is:

```python
if mem_type == "npc" and named_npcs:
    if not any(same_subject(subject, named) for named in named_npcs):
        continue
```

I tested why this guard is necessary by temporarily removing these lines and
rerunning:

```text
python verify_memory.py
```

Without the identity guard, the query:

```text
is Mirra alive
```

retrieved both NPCs:

```text
Mirra — similarity 0.67
Marra — similarity 0.34
```

The verification then failed:

```text
FAIL  long-term: recall('is Mirra alive') never returns Marra (identity check)
```

This demonstrated that semantic similarity alone was not enough to keep the
two NPC identities separate.

After restoring the identity guard, the same verification returned only the
appropriate Mirra NPC memory for that query, and the full verification result
returned to:

```text
all 17 checks passed - both memories are sound.
```

The guard is therefore useful even after semantic retrieval: similarity finds
potentially relevant memories, while the explicit identity check prevents a
memory about a different named NPC from entering the final context.

---

## c. The retcon

On Night 3, the campaign retconned the Tidewater amulet so that Dax had never
taken it.

The three checks from Night 3 were:

```text
rows remaining: 0
summary mentions amulet after re-fold: no
GM's next answer mentions amulet: no
```

The complete retcon receipt reported:

```text
4 rows deleted, 0 remaining;
summary mentions it after re-fold: no;
GM's next answer mentions it: no
```

The short-term summary was the most subtle part of the retcon.

Deleting the amulet rows from Chroma removes the persistent long-term memory,
but that alone is not enough. The amulet may already have been copied into the
rolling short-term summary. Because that summary is later sent back to the
model, the forgotten fact could leak back into the conversation even after its
database rows were deleted.

That is why the retcon uses both:

```python
ltm.forget(subject)
```

and:

```python
stm.fold(drop=subject)
```

The order of the three checks is important.

First, check the persistent store to make sure the original memory source is
gone.

Second, check the short-term summary because it may contain a cached copy of
the forgotten information.

Third, check the GM's next answer. This is the end-to-end test that confirms
the deleted information is not being reintroduced from either memory layer.

The successful result was therefore:

```text
Long-term memory  -> 0 remaining
Short-term memory -> no mention
Next model answer -> no mention
```

---

## Final results

The continuity quiz results were:

| Night | Memory GM | Goldfish GM |
|---|---:|---:|
| Night 2 | 8/10 | 3/10 |
| Night 3 | 10/10 | 4/10 |

The project demonstrates that memory and context are separate concerns.

Long-term typed memories persisted important campaign information across
separate processes. Short-term folding controlled how much recent conversation
was placed back into the model context. Superseding allowed state changes to
replace active facts while retaining history, event decay reduced the score of
older events, the NPC identity guard protected similarly named entities, and
explicit forgetting removed a retconned fact from both persistent and
short-term memory.

Fact extraction also required one model call per turn. This was the cost of
turning free-form conversation into structured, typed long-term memories, while
retrieval allowed only relevant stored facts to be placed back into later
prompts.