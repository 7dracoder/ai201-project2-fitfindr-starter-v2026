# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> The three tools and planning loop are implemented below.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr lets a user describe a secondhand clothing item, including an optional
size and maximum price. It searches the local listings and selects the best
match. It then uses the user's saved wardrobe to suggest an outfit and writes a
short fit-card caption. If there is no match, it stops and tells the user what
they can change in the search.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the local listings using description keywords and optional size and price filters.
- **Inputs:** `description` (`str`), `size` (`str | None`), and `max_price` (`float | None`). A size is matched as a complete size token, so `M` matches `S/M` but `S` does not match `US 9`.
- **Returns:** A best-match-first `list[dict]`, where each listing dict contains `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list.

### `suggest_outfit`

- **What it does:** Uses the new item and the user's wardrobe to suggest one or two outfits.
- **Inputs:** `new_item` (`dict`) and `wardrobe` (`dict` with an `items` list).
- **Returns:** A non-empty `str` containing concise outfit suggestions that name saved wardrobe pieces when available.
- **When it has nothing:** If the wardrobe has no items, returns general styling advice for the new item instead of failing.

### `create_fit_card`

- **What it does:** Turns an outfit suggestion and listing into a short social-media caption.
- **Inputs:** `outfit` (`str`) and `new_item` (`dict`).
- **Returns:** A two-to-four-sentence `str` that mentions the item, price, platform, and vibe.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a message explaining that a fit card could not be created.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, put a helpful
message in the session and stop. Otherwise, select the first result, call
`suggest_outfit`, and then call `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract phrases such as
`under $30` and `size M`; the remaining text becomes the description.

**What moves through the session:** `query` becomes `parsed`, then the search
output is saved in `search_results`. Its first item is saved as `selected_item`
and read from the session by `suggest_outfit`. That output is saved as
`outfit_suggestion` and read from the session by `create_fit_card`, whose output
is saved as `fit_card`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

Found: Graphic Tee — 2003 Tour Bootleg Style — $24.0 on depop

Outfit: Outfit 1: Y2K Grunge
- Base: Graphic Tee + Baggy straight-leg jeans (with Brown leather belt)
- Layer: Black cropped zip hoodie
- Footwear & Bag: Black combat boots + Black crossbody bag

Outfit 2: Off-Duty Streetwear
- Base: Graphic Tee (tucked) + Wide-leg khaki trousers (with Brown leather belt)
- Outerwear: Vintage black denim jacket
- Footwear & Bag: Chunky white sneakers + Black crossbody bag

Fit card: Scored this ultimate 2003 tour bootleg tee for just $24.00, and it’s
giving major Y2K grunge and off-duty streetwear energy. Seriously obsessed with
the faded graphic and vintage wash. Grab it now over on my depop before I change
my mind and keep it!
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([item['title'] for item in search_listings('graphic tee', max_price=30)])"
['Graphic Tee — 2003 Tour Bootleg Style', 'Y2K Baby Tee — Butterfly Print',
'Vintage Band Tee — Faded Grey', 'Vintage Graphic Hoodie — Faded Black',
'Mesh Long-Sleeve Top — Black', 'Low-Rise Cargo Pants — Khaki']
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Outfit 1: Casual & Sporty
- Top: White ribbed tank top
- Outerwear: Oversized grey crewneck sweatshirt
- Shoes: Chunky white sneakers
- Accessories: Black crossbody bag

Outfit 2: Edgy & Layered
- Top: White ribbed tank top
- Outerwear: Vintage black denim jacket
- Shoes: Black combat boots
- Accessories: Brown leather belt, Black crossbody bag
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('Pair them with a white ribbed tank and chunky white sneakers.', load_listings()[0]))"
Scored these vintage Levi's 501s for just $38.00 and I am obsessed. The medium
wash gives off such effortless off-duty model vibes. I’m listing them on depop
today, but honestly, I might just keep them.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked AI to complete the required tools while keeping the starter simple and changing only what the assignment required.
- *What came back:* It suggested a keyword search, two short model prompts, and a state-based loop.
- *What I changed:* I kept those pieces but weighted title and style-tag matches so a cheap, loosely related listing would not beat the item named in the query.

**Moment 2**

- *What I asked for:* I asked AI to wire the search result into the next tools without asking the user for the item again.
- *What came back:* It suggested storing each result in the session and reading it back in the next step.
- *What I changed:* I used a `next_step` loop and made size parsing require the word `size`, so text such as `midi` is not mistaken for size `M`.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
