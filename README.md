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
> The three tools and planning loop are implemented. Try FitFindr with python app.py ask "vintage graphic tee under $30".
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

FitFindr will search the provided clothing listings using a description, size, and maximum price. It will use a matching item and the user's wardrobe to suggest an outfit, then generate a short fit-card caption. If no listings match, it will stop and explain what the user could change.

### Setup and data notes

- Environment check: 10 passed, 0 failed.
- Dataset: 40 listings and 10 wardrobe items.
- Listings include `id`, `title`, `description`, `style_tags`, `size`, and `price`.
- Sizes have different formats, such as `M`, `S/M`, and `W30 L30`; some brands are null.
- The planning loop connects the three tools. Try it with `python app.py ask 'vintage graphic tee under $30'`.


---

## Tool Inventory

### `search_listings(description: str, size: str | None = None, max_price: float | None = None)`

- **What it does:** Searches the clothing listings for description keywords and optional size and price filters.
- **Returns:** A list of matching listing dictionaries, best match first. Each listing includes its `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.
- **When it has nothing:** Returns an empty list (`[]`).

### `suggest_outfit(new_item: dict, wardrobe: dict)`

- **What it does:** Suggests outfits using a selected listing and the user’s wardrobe.
- **Inputs:** `new_item` (`dict`), `wardrobe` (`dict`).
- **Returns:** A non-empty string with one or two outfit suggestions.
- **When the wardrobe is empty:** Returns general styling advice for the item.

### `create_fit_card(outfit: str, new_item: dict)`

- **What it does:** Writes a short social caption about the selected item and outfit.
- **Returns:** A two-to-four-sentence caption mentioning the item, its price, its platform, and its style.
- **When the outfit is empty:** Returns a descriptive message instead of raising an error.

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

**Branch rule:** If `search_listings` returns no matches, `run_agent()` stores an error message in the session and returns before calling the other tools. Otherwise, it stores the first match as `session["selected_item"]`, passes it to `suggest_outfit`, then passes the outfit and selected item to `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract the size and price limit. The remaining text becomes the search description.

**What moves through the session:** `run_agent()` stores the parsed description, size, and price in `session["parsed"]`, then stores search results in `session["search_results"]`. If there is a match, the first listing goes in `session["selected_item"]`; the outfit suggestion and fit card are stored in `session["outfit_suggestion"]` and `session["fit_card"]`. The loop checks its iteration count with `trace.check_iterations()`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask "vintage graphic tee under $30"

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two practical, wearable outfits using your new Y2K butterfly baby tee and pieces from your wardrobe:

### Outfit 1: High-Contrast Streetwear (Y2K Meets Baggy Denim)
*The fitted, cropped silhouette of the baby tee balances out the volume of the baggy jeans for an authentic early-2000s look.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans, dark wash (`w_001`)
*   **Outerwear:** Vintage black denim jacket (`w_006`)
*   **Shoes:** Chunky white sneakers (`w_007`)
*   **Bag:** Black crossbody bag (`w_010`)

### Outfit 2: Casual Earth Tones
*Playing on the softer pink and purple tones in the butterfly graphic by pairing the tee with relaxed neutrals.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers (`w_002`)
*   **Accessory:** Brown leather belt (`w_009`) worn with the trousers
*   **Shoes:** Chunky white sneakers (`w_007`)
*   **Bag:** Black crossbody bag (`w_010`)

  Fit card: Score this Y2K Baby Tee — Butterfly Print for just $18.00! It’s giving major early-2000s vibes, perfect for pairing with baggy denim and a vintage black jacket for an authentic streetwear look. Grab it now over on depop before it's gone.

0 model calls this session, 2 served from cache

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(x['id'], x['title'], x['price']) for x in search_listings('graphic tee', max_price=30)])"
[('lst_002', 'Y2K Baby Tee — Butterfly Print', 18.0), ('lst_006', 'Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('lst_017', 'Mesh Long-Sleeve Top — Black', 15.0), ('lst_033', 'Vintage Band Tee — Faded Grey', 19.0), ('lst_011', 'Low-Rise Cargo Pants — Khaki', 27.0), ('lst_015', 'Vintage Graphic Hoodie — Faded Black', 26.0)]

```

```
$ (.venv) C:\Users\SN\CP_201\W3\ai201-project2-fitfindr-starter-v2026>python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here are two practical, everyday outfits featuring your new Vintage Levi's 501s:

### Outfit 1: Clean & Casual Streetwear
* **Base:** Vintage Levi's 501 Jeans + **White ribbed tank top** (w_003) tucked in.
* **Layer:** **Oversized grey crewneck sweatshirt** (w_004) worn over top.
* **Footwear:** **Chunky white sneakers** (w_007).
* **Accessories:** **Black crossbody bag** (w_010).
* **Why it works:** The fitted white tank provides a clean contrast to the oversized grey crewneck and straight-leg fit of the 501s. It's an easy, comfortable off-duty look.

### Outfit 2: Edgy Vintage Contrast
* **Top:** **White ribbed tank top** (w_003) + **Black cropped zip hoodie** (w_005) worn unzipped.
* **Outerwear:** **Vintage black denim jacket** (w_006) for a layered double-denim-adjacent vibe.
* **Footwear:** **Black combat boots** (w_008) — let the hem of the 501s break right at the top of the boots.
* **Accessories:** **Brown leather belt** (w_009) to anchor the waist.
* **Why it works:** Pairing medium-wash blue denim with black outerwear and boots creates a balanced, high-contrast streetwear look that leans into the 501s' vintage aesthetic.

```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Just scored these Vintage Levi's 501 Jeans — Medium Wash on depop for $38.00! They feature a classic look with light fading at the knees that adds to the vintage vibe. Style them simply with jeans and white sneakers for an effortless streetwear outfit.

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked ChatGPT to help me finish the three blank acceptance criteria and their reasons, and tell me where to put them in `criteria.md`.
- *What came back:* It suggested measurable checks for passing the same listing between tools, including key details in a fit card, and respecting a price limit.
- *What I changed:* I used those ideas to fill in criteria 3–5 and wrote reasons tied to the session, model-generated captions, and listing prices.


- *What I asked for:* I asked ChatGPT to review my completed criteria and tell me whether they were clear.
- *What came back:* It said the targets were measurable and suggested formatting the code references and using a specific price example for criterion 5.
- *What I changed:* I kept criterion 5 general so it applies to any maximum price, and noted the formatting suggestion for a final cleanup.

### Unit 4 — MCP assistance

- **What I asked for:** I asked ChatGPT to explain the terminal
  commands and guide me through moving `search_listings` to MCP.
- **What came back:** It provided the MCP wrapper and the replacement
  search call in `agent.py`, with explanations of the inputs and roles.
  It also helped identify that my saved agent still used the direct call.
- **What I changed and checked:** I added the wrapper, updated and saved
  the agent's import and search call, and added comments to explain the
  code. I ran `python mcp_client.py` to check registration and an
  end-to-end query to check that the connected agent still worked.
  The outfit and fit-card responses were served from cache.

### Unit 4 — Failure handling and trace assistance

I asked ChatGPT to explain the failure tests and help add
`ModelUnavailable` handlers and `trace.step()` calls. It supplied
code and reviewed my edits, including catching a missing final
`return session`. I ran the commands locally and checked the
actual output. It also helped format those results in this README.

### Unit 4 — Evaluation assistance

ChatGPT helped me configure five scenarios and add selected-item IDs
and returned listing prices to the evaluation report. I ran the
25 baseline tries locally with caching disabled. 

It reviewed the saved outputs, suggested verdict explanations, and identified an
unsupported ownership claim that my original criteria did not check.
I used that evidence to document a tighter future target.

ChatGPT also suggested the caption-perspective instructions and helped
compare the before and after reports, including the ownership diagnostic
and the service-unavailable error. I applied the edit and ran the
after evaluation locally.

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
| 1. Matching query completes all three tools and returns a fit card | At least 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before suggest_outfit and names what to change | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item ID matches the new_item ID passed to suggest_outfit | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes the selected title, price, and platform | At least 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. With max_price=30, every returned listing costs at most $30 | Every returned listing; checked in five tries | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

Baseline report: [Full before-run output](results/run_2026-10-07_2052_before.md).

All 25 tries ran with caching disabled, using 40 model calls.
Each criterion was scored using its five corresponding tries.
No acceptance criteria were revised.

**Real output from one try**, pasted as text, naming the file and function
that produced it:

Criterion 4, Try 1, from `results/run_2026-10-07_2052_before.md`.
The caption was returned by `tools.py::create_fit_card()`, called by
`agent.py::run_agent()`, and recorded by `run_eval.py::write_report()`.

```text
Scored this Y2K Baby Tee — Butterfly Print for just $18.00! I love styling it with baggy straight-leg jeans and chunky sneakers for an easy, high-contrast streetwear look. Grab it now over on depop before it's gone!
```
### Evidence from one try per criterion
These excerpts come from Try 1 of each corresponding scenario in `results/run_2026-10-07_2052_before.md`, recorded by `run_eval.py::write_report()` from `agent.py::run_agent()`

**Criterion 1 — all three tools complete**

The trace shows `search_listings` through MCP, followed by
`tools.py::suggest_outfit()` and `tools.py::create_fit_card()`.

```text
[1] search_listings (via MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    Matches found: select the first listing.
[2] suggest_outfit
      in:  new_item_id=lst_002; wardrobe_items=10
      out: Here are two ways to style your new Y2K butterfly baby tee using pieces already in your closet:  ### Outfit 1:…
      →    Use the selected item stored in the session.
[3] create_fit_card
      in:  new_item_id=lst_002; outfit=Here are two ways to style your new Y2K butterfly baby tee using pieces already in…
      out: Channel early-2000s street style with this Y2K Baby Tee — Butterfly Print, featuring a fitted crop and cute gr…
      →    Use the outfit suggestion and the same selected item.
```

The resulting fit card from `tools.py::create_fit_card()`:

```text
Channel early-2000s street style with this Y2K Baby Tee — Butterfly Print, featuring a fitted crop and cute graphic. Pair it with baggy denim and chunky sneakers for an authentic look, or layer it under a black denim jacket with combat boots. This vintage piece is available now for $18.00 on depop.
```

**Criterion 2 — empty search stops before the outfit tool**

`mcp_server.py::search_listings()` returned an empty list.
`agent.py::run_agent()` stopped and supplied advice.

```text
- stopped early: yes — No listings matched. Try broader keywords, a different size, or a higher price limit.
- selected_item: (none)
- search_results: 0
```

```text
[1] search_listings (via MCP)
      in:  {'description': 'qzxvplmnonexistent', 'size': None, 'max_price': None}
      out: [] (empty)
      →    No matches: stop before suggest_outfit.
```

**Criterion 3 — the selected item matches the outfit tool input**

`agent.py::run_agent()` stored the selected item and passed it to
`tools.py::suggest_outfit()`. These report and trace excerpts show
the same ID.

```text
- selected_item_id: lst_002
```

```text
[2] suggest_outfit
      in:  new_item_id=lst_002; wardrobe_items=10
      out: Here are two ways to style your new Y2K butterfly baby tee using pieces already in your wardrobe:  ### Outfit …
      →    Use the selected item stored in the session.
```

**Criterion 4 — the fit card includes title, price, and platform**

See the Criterion 4, Try 1 caption immediately above this subsection,
returned by `tools.py::create_fit_card()`. It includes
`Y2K Baby Tee — Butterfly Print`, `$18.00`, and `depop`.

**Criterion 5 — every returned listing respects the $30 limit**

The parsed limit came from `agent.py::run_agent()`. The listing prices
came from `tools.py::search_listings()`, exposed through
`mcp_server.py::search_listings()` and called through MCP.

```text
- parsed_max_price: 30.0
- returned_listing_prices: [('lst_002', 18.0), ('lst_006', 24.0), ('lst_033', 19.0), ('lst_015', 26.0), ('lst_017', 15.0), ('lst_003', 22.0), ('lst_011', 27.0), ('lst_012', 20.0), ('lst_013', 30.0), ('lst_014', 12.0)]
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
| 1 | Matching query completes all three tools and returns a fit card | At least 4/5 | MET (5/5) | Each trace shows search through MCP, suggest_outfit, and create_fit_card in order, followed by a non-empty fit card. |
| 2 | Impossible query stops before suggest_outfit and names what to change | 5/5 | MET (5/5) | Each trace contains only search. Each response suggests broader keywords, a different size, or a higher price limit. |
| 3 | Selected item ID matches the outfit tool input | 5/5 | MET (5/5) | selected_item_id and the traced new_item_id are both lst_002 in every try. |
| 4 | Fit card includes selected title, price, and platform | At least 4/5 | MET (5/5) | Each fit card includes Y2K Baby Tee — Butterfly Print, $18.00, and depop. |
| 5 | Every returned listing respects max_price=30 | Every returned listing; checked in five tries | MET (5/5) | Each try has parsed_max_price=30.0 and ten returned prices ranging from $12 to $30. None exceeds $30. |


**Diagnoses**

The likely mechanism is ambiguous perspective in the
`create_fit_card()` prompt. It asks for a natural social-media caption
without specifying that the item is a listing the user is considering.
Although it says not to invent details, the model still adopted a
seller's voice in the observed output. I will test whether an explicit
instruction about ownership and perspective reduces this behavior.

All five original criteria were met, so there are no missed criteria
to diagnose.

Criterion 4 set too low a standard for caption accuracy: it checked
whether the title, price, and platform appeared, but did not check
whether the caption invented ownership. In the baseline report,
Criterion 3, Try 4 includes this sentence:

> Head over to my depop to snag it before it's gone.

The problem appears in the model output returned by
`tools.py::create_fit_card()`, the third tool in the loop. The caption
contains the required listing details but adds an unsupported claim
that the speaker owns the depop account.

A specific tighter future target would require the selected title,
price, and platform, with no unsupported claims that the user bought,
owns, or is selling the item, in 5 of 5 tries.

This is a proposed future target. I have not changed the original
criterion or rescored the baseline against it.



## Deliberate Failure Checks

### Empty search

Command: `python app.py ask "qzxvplmnonexistent"`

Message from `agent.py::run_agent()`, displayed by `app.py::_ask_one()`:

```text
No listings matched. Try broader keywords, a different size, or a higher price limit.
```

The trace confirmed that the agent stopped before `suggest_outfit`.
No model calls were made.

### Empty wardrobe

Command: `python app.py ask "vintage graphic tee under $30" --empty-wardrobe`

`suggest_outfit()` in `tools.py` returned general styling advice:

```text
Since a baby tee is fitted and cropped, the main styling rule is to balance the proportions with your bottoms. Here are two easy, wearable ways to style it using common wardrobe staples:
```

The run completed with an outfit and fit card, using two fresh model calls.
The advice did not claim that the suggested pieces were already owned.

### Model unavailable

I disabled caching and supplied a deliberately invalid GEMINI_API_KEY
inside one Python process. I did not modify the real key in `.env`.

Before adding the agent handler, `app.py::main()` displayed the
`ModelUnavailable` exception without a traceback.

After adding handlers around both model-calling tools in
`agent.py::run_agent()`, the same test returned:

```text
Could not generate outfit suggestions The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com.
```

The failed call was in `tools.py::suggest_outfit()`, through
`generate.py::generate()`. The agent stored the message in
`session["error"]` and stopped before generating a fit card.
One model call was attempted.

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
python app.py ask "vintage graphic tee under $30" --trace
[1] search_listings (via MCP)
      in:  {'description': 'vintage graphic tee', 'size': None, 'max_price': 30.0}
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    Matches found: select the first listing.
[2] suggest_outfit
      in:  new_item_id=lst_002; wardrobe_items=10
      out: Here are two practical, wearable outfits using your new Y2K butterfly baby tee and pieces from your wardrobe: …
      →    Use the selected item stored in the session.
[3] create_fit_card
      in:  new_item_id=lst_002; outfit=Here are two practical, wearable outfits using your new Y2K butterfly baby tee and…
      out: Score this Y2K Baby Tee — Butterfly Print for just $18.00! It’s giving major early-2000s vibes, perfect for pa…
      →    Use the outfit suggestion and the same selected item.

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two practical, wearable outfits using your new Y2K butterfly baby tee and pieces from your wardrobe:

### Outfit 1: High-Contrast Streetwear (Y2K Meets Baggy Denim)
*The fitted, cropped silhouette of the baby tee balances out the volume of the baggy jeans for an authentic early-2000s look.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Baggy straight-leg jeans, dark wash (`w_001`)
*   **Outerwear:** Vintage black denim jacket (`w_006`)
*   **Shoes:** Chunky white sneakers (`w_007`)
*   **Bag:** Black crossbody bag (`w_010`)

### Outfit 2: Casual Earth Tones
*Playing on the softer pink and purple tones in the butterfly graphic by pairing the tee with relaxed neutrals.*

*   **Top:** Y2K Butterfly Baby Tee
*   **Bottoms:** Wide-leg khaki trousers (`w_002`)
*   **Accessory:** Brown leather belt (`w_009`) worn with the trousers
*   **Shoes:** Chunky white sneakers (`w_007`)
*   **Bag:** Black crossbody bag (`w_010`)

  Fit card: Score this Y2K Baby Tee — Butterfly Print for just $18.00! It’s giving major early-2000s vibes, perfect for pairing with baggy denim and a vintage black jacket for an authentic streetwear look. Grab it now over on depop before it's gone.

0 model calls this session, 2 served from cache

```

**Empty search**

```
python app.py ask "qzxvplmnonexistent" --trace
[1] search_listings (via MCP)
      in:  {'description': 'qzxvplmnonexistent', 'size': None, 'max_price': None}
      out: [] (empty)
      →    No matches: stop before suggest_outfit.

  No listings matched. Try broader keywords, a different size, or a higher price limit.

0 model calls this session

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->
Search now runs through MCP. The matching trace shows search,
outfit suggestions, and fit-card generation in order. The empty-search
trace shows only search, confirming that the loop stops before
`suggest_outfit`. The matching run used cached model responses;
these traces demonstrate execution flow, not repeated evaluation.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**
I added explicit perspective and ownership instructions to the prompt
in `tools.py::create_fit_card()`. The caption should recommend a listing
the user is considering without claiming a purchase or sale.

**Which failure it was meant to fix:**

The baseline caption for Criterion 3, Try 4 said "my depop",
an unsupported ownership claim. The original five criteria all passed,
but they did not check this aspect of caption accuracy.

**How I will measure it:**

I will rerun the same five scenarios with five tries each and caching
disabled, scoring against the unchanged original criteria.

As an additional diagnostic check, I will compare the five captions
from the Criterion 4 scenario before and after. A caption passes this
check if it includes the selected title, price, and platform without
claiming the speaker bought, owns, listed, or is selling the item.

In the baseline, Try 1 and Try 3 fail this additional check because
they begin "Scored this", implying a completed purchase.
Try 2, Try 4, and Try 5 pass: "Score this" in Try 5 is an invitation,
not a claim that a purchase already happened. The baseline is 3/5.

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Matching query completes all three tools and returns a fit card | At least 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. Impossible query stops before suggest_outfit and names what to change | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. Selected item ID matches the new_item ID passed to suggest_outfit | 5/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. Fit card includes the selected title, price, and platform | At least 4/5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. With max_price=30, every returned listing costs at most $30 | Every returned listing; checked in five tries | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

After report: [Full after-run output](results/run_2026-10-07_2227_after.md).

All 25 tries ran with caching disabled, using 40 model calls.
Criterion 3, Try 4 passed the ID check: selected_item_id and the
suggest_outfit input were both lst_002. Caption generation later
failed with a 503 UNAVAILABLE response. This does not invalidate
the earlier state check, but the overall run did not finish.

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->
The ownership diagnostic improved in this sample, from 3/5 before
to 5/5 after. I checked the five captions from the Criterion 4
scenario in each report using the same rule.

| Ownership diagnostic | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Result |
|---|---|---|---|---|---|---|
| Before | FAIL | PASS | FAIL | PASS | PASS | 3/5 |
| After | PASS | PASS | PASS | PASS | PASS | 5/5 |

The two baseline failures said "Scored this", implying a completed
purchase. All five after captions retained the title, price, and
platform without claiming ownership, purchase, or sale.

The original five criteria remained MET. These results suggest the
prompt change helped for this query; five tries on one selected item
do not establish that it will work for every listing.


---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->
No original criterion was missed in its assigned five tries.
However, Criterion 3, Try 4 could not generate a caption because
the model service returned 503 UNAVAILABLE.

The failure occurred in tools.py::create_fit_card(), through
generate.py::generate(). The agent caught ModelUnavailable,
stored an error message, and returned without a fit card.
The caption-prompt improvement does not address service availability.
A future improvement could retry transient service errors with a
limited number of attempts.

The matching scenarios all selected the same listing, lst_002.
Other items, query phrasings, and sizes need broader testing.


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

## Unit 4- MCP Move

I registered `search_listings` in `mcp_server.py` and changed
`agent.py::run_agent()` to call it through `mcp_client.call_tool()`.
The other two tools still run directly.

`python mcp_client.py` listed the registered tool and its inputs.
An end-to-end query, `vintage graphic tee under $30`, selected the
same Y2K Baby Tee — Butterfly Print for $18.00 on depop and returned
an outfit and fit card. The two model responses came from the build
cache; this was a connection check, not the repeated evaluation.