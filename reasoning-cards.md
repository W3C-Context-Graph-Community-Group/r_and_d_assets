---
data: cgvu:/reason?name=reasoning-card-examples&date-created=2026-09-22T13:18:00-06:00&source=https://www.irs.gov/pub/irs-regs/research_credit_basic_sec41.pdf
meaning: Explanation of reasoning cards using IRS section 41(a) example
structure: Unchanged, read-only
world: Give this to an LLM to teach it how to make reasoning cards. This document is a *resource* that a reasoning card would point to. `cgvu` stands for Context Graph Virtual URI. In our case, our pointer (in the `data` facet) and resource are one file.
---

# Reasoning Cards Primer
To make a decision, humans reason about the data they have. What they still need in order to read that data is context: the transition from an interpretation insufficient for the decision to one sufficient for it. A Spike has four facets: data, the anchoring observation, and three context facets beside it — meaning: the human-readable definition; structure: constraints, validation, schema; world: what an interpreter needs to use the meaning and structure in the situation at hand (who, what, why, where, when), written for the reader of the card. A Reasoning Card reuses the same four facets as a record format (spec §10): a card's facets are always filled and never DARK. Where a facet has nothing to say, this document uses the address cgvu:/not-applicable; that is a convention of these resources, not of the spec, and it is LIT like any other address.

## For human readers

This page turns the opening of 26 U.S.C. §41 into **reasoning cards**. Each card is a single step of reasoning, and its `cgvu:/` address is both its name and its instruction. The path names the handler that runs the card. The query string carries the variables the router passes to that handler. When a card needs another card, it refers to it by that card's `id`.

There are three kinds of card, and each kind's address lands in a different slot of the context object:

Statute cards transcribe one line of the regulation: irc-41-a, irc-41-a-1, irc-41-a-1-A, irc-41-a-1-B, irc-41-a-2 and irc-41-b. A reason entry points at one when a step applies that line.
Arithmetic cards are acts the regulation relies on but never writes down: math-excess ("the excess, if any"), math-percent and math-equivalence (are two readings the same?). Also pointed at from reason.
Policy and process cards run the decision. The chain (chain-rd-credit-2025) and the decision rule (decide-rd-credit) are intent-map policy (spec §8): the chain's address is intent-map[0]. The ask card (ask-qre-year) is an interrogative card (spec §10): asking it is an ASK route, the answer is a reason entry and an append to the DARK facet. The card the chain emits (result-rd-credit-2025) is a reason entry recording the chain's output; the decision reads it.

The figures themselves (QRE, base amount) are `data` on Spikes in `observe`. Their years and interpretations are appended to those Spikes' context facets.

Every table has the four facets: data, meaning, structure and world. The **structure** facet is the card's Lean 4 code, which builds on `Certainty.lean` and `Bits.lean`. The whole module is at the end of this page. It compiles with no `sorry`, and uses no axioms beyond `propext`, `Classical.choice` and `Quot.sound`.

With the worked example (QRE $1,000,000, base amount $600,000, basic research payments $50,000, all for 2025), the credit is $90,000 and the route is ACT. If you decline the question, the route is HALT, and the credit is never included on a guess.

## For agents
CRITICAL: 

1. Each Reasoning card is its own `cgvu` address, so we can only have one Reasoning Card per .MD file. This format will be just like this document: (A) YAML Frontmatter with the four facets of data, meaning, structure, and world and NOTHING ELSE, followed by (B) Any explanatory or other text helpful to add to the resource.
2. Each reasoning card is a **route** in URI-valid syntax, a route handler would manage routing to a resource, because these are virtual URIs, the routing can be in the same file.

````text
Use this document as a template to create Reasoning Cards if the user pastes regulation text.
Convert it into single-step reasoning cards, one card per step, in ONE Markdown file.

VOCABULARY (Context Graph Protocol)
- A Spike has four facets: data (the anchor) and three context facets: meaning, structure, world.
- Context is the transition from an interpretation insufficient for a decision to one sufficient
  for it. It is not "the missing data".
- A card's four facets are always filled (§10), so a card is never DARK. "" (DARK) appears only in
  observe facets. If a card facet does not apply, write cgvu:/not-applicable (a convention of this
  resource, not the spec; it is LIT because it is non-empty).
- Do not use the word "claim". The card a chain emits is a result card: cgvu:/result/value?...

ADDRESSES
- Every card address starts with "cgvu:/" (one slash). Never write "cgvu://".
- Shape: cgvu:/<handler-path>?key1=value1&key2=value2...  The path names the handler; the router
  reads the address and passes the query keys to that handler as variables. No reading looks
  inside an address; only the router does.
- Always include id=<short-kebab-id>. Refer to other cards by their id (e.g. via=math-excess).
  Lists are comma-separated (steps=a,b). Use hyphens for spaces. Percent-encode & = ? # if a value
  needs them.
- Statute cards also carry cite=<code>-<section>-<subsection>... (e.g. cite=26-usc-41-a-1).
- The file starts with front matter whose data line is
  cgvu:/resource?name=<name>&date-created=<ISO 8601>&source=<URL of the regulation>

WHERE EACH ADDRESS LANDS (D5; say this in the prose, and show it in the trace table)
- intent-map: the chain card and the decide rule (policy, §8), appended first.
- reason: statute cards, general (math) cards, ask cards, and the result card the chain emits (§10).
- observe: figures are Spike data; years, answers and interpretations are appended to the Spike's
  meaning / structure / world facets.
- decide: one entry per route dispatched; its address is cgp:// (D13) and carries its as-of index.

CARD KINDS (make all four)
1. Statute cards: exactly one per line of the supplied text (paragraph, subparagraph, definition
   heading). Do not invent content for text that was not supplied; if a definition stops at its
   heading, make the card anyway with status=heading-only and say its text was not supplied. The
   card is still LIT; the observe facet that would point to it stays DARK.
2. General cards: the acts the text relies on but does not state (arithmetic such as
   "excess (if any)", percentages, rounding, and equivalence of two readings). Path cgvu:/math/...
   Write them generically (a=nat&b=nat), not bound to this regulation.
3. At least one ASK card (cgvu:/ask/...): a question to the user that lights a DARK observe facet
   (usually world or meaning), with on-answer=light&on-decline=halt. It must feed a later card.
4. A chain card (cgvu:/chain/sequence?steps=...&into=...&emits=...&then=...), the result card it
   emits (cgvu:/result/value?..., carrying every value the decision needs), and a decide rule
   (cgvu:/decide/route?result=...&on-act=...&on-ask=...&on-halt=...).

TABLE FORMAT (one table per card, nothing else under its header)
# <the card's full cgvu:/ address>
| facet | reading |
|---|---|
| data | `<the same address>` |
| meaning | plain-English statement of the one step, citing the line it comes from |
| structure | the card's Lean section (use <br> for newlines, &nbsp; for indentation, &#124; for |) |
| world | handler instructions, test vectors with expected outputs, what is out of scope |
After the cards, add a trace table (# cgvu:/trace/roll?...) showing one append per row, in order,
with the slot each append lands in.

LEAN (structure facet) — follow the Context Graph Protocol pattern
- Lean 4.19.0, core library only. No Mathlib. No sorry. Only axioms propext, Classical.choice, Quot.sound.
- One module: header comment, then `import Certainty` and `import Bits`, then `namespace Cards`,
  `open Certainty Foundation`. One `/-! ## <card address> -/` section per card, in dependency
  order. Doc comments on every definition. Name the emitted card's type `Result`.
- Money is whole dollars (Nat); "excess (if any)" is Nat truncated subtraction; percentages floor.
- The decision rule is a rule of the first kind: f : (Fin n → String) → Bool over the facets it reads.
  Bit F is `@decide (Settles f A x) (Classical.propDecidable _)`; route is `Foundation.route H F`.
  Prove for the ASK card: the asked facet is Essential, the rule is unsettled at the ask (route ASK),
  settled once answered (route ACT), and a decline gives HALT.
  Prove for the decide rule: it acts only when the route is ACT and Settles holds.
- End with a theorem that every card address passes wellFormedAddressB (by decide), then
  `end Cards` and one `#print axioms` line per theorem.
- Compile against the repository's own files, so the hashes match lean/BUILD_LOG.txt:
  put the module in ~/Documents/GitHub/w3c/proof/lean/ next to Certainty.lean, Types.lean and
  Bits.lean, build those in order (Certainty → Types → Bits), then `LEAN_PATH=. lean <file>`.
  Paste the full module and its #print axioms output at the end of the Markdown file.
  Do not say it compiles unless you ran it.

STYLE
- One card = one step. Short plain sentences. State what was omitted from the supplied text.
````

# cgvu:/reason/sum?id=irc-41-a&cite=26-usc-41-a&terms=irc-41-a-1,irc-41-a-2&out=research-credit&feeds=26-usc-38

| facet | reading |
|---|---|
| data | `cgvu:/reason/sum?id=irc-41-a&cite=26-usc-41-a&terms=irc-41-a-1,irc-41-a-2&out=research-credit&feeds=26-usc-38` |
| meaning | §41(a), the general rule: for purposes of §38, the research credit for the taxable year is the sum of the §41(a)(1) amount and the §41(a)(2) amount. |
| structure | <code>/-- §41(a): the research credit that feeds §38. -/<br>def credit (q : QRE) (b : BaseAmount) (brp : Nat) : Nat := a1 q b + a2 brp<br><br>theorem credit_example : credit ⟨"2025", 1000000⟩ ⟨"2025", 600000⟩ 50000 = 90000 := by<br>&nbsp;&nbsp;decide<br><br>theorem credit_when_no_excess {q : QRE} {b : BaseAmount} (brp : Nat)<br>&nbsp;&nbsp;&nbsp;&nbsp;(h : q.amount ≤ b.amount) : credit q b brp = a2 brp := by<br>&nbsp;&nbsp;unfold credit; rw [a1_zero_of_le h, Nat.zero_add]</code> |
| world | Handler: add the outputs of irc-41-a-1 and irc-41-a-2, and pass the total to the §38 general business credit. Test: QRE 1,000,000, base 600,000, payments 50,000 → 80,000 + 10,000 = 90,000. The supplied text omits current-law §41(a)(3) (energy research consortia), so it is not modelled. |

# cgvu:/reason/rate-of-excess?id=irc-41-a-1&cite=26-usc-41-a-1&rate-pct=20&over=irc-41-a-1-A&under=irc-41-a-1-B&via=math-excess,math-percent&out=incremental-credit

| facet | reading |
|---|---|
| data | `cgvu:/reason/rate-of-excess?id=irc-41-a-1&cite=26-usc-41-a-1&rate-pct=20&over=irc-41-a-1-A&under=irc-41-a-1-B&via=math-excess,math-percent&out=incremental-credit` |
| meaning | §41(a)(1): 20 percent of the excess (if any) of QRE for the taxable year over the base amount. It is zero when QRE does not exceed the base amount. |
| structure | <code>/-- The statutory rate in §41(a)(1) and (a)(2), in percent. -/<br>def rate41 : Nat := 20<br><br>/-- §41(a)(1) = math-percent 20 ∘ math-excess. -/<br>def a1 (q : QRE) (b : BaseAmount) : Nat := percent rate41 (excess q.amount b.amount)<br><br>theorem a1_zero_of_le {q : QRE} {b : BaseAmount} (h : q.amount ≤ b.amount) : a1 q b = 0 := by<br>&nbsp;&nbsp;unfold a1; rw [excess_zero_of_le h]; exact percent_zero _<br><br>theorem a1_mono_qre {q q' : QRE} {b : BaseAmount} (h : q.amount ≤ q'.amount) :<br>&nbsp;&nbsp;&nbsp;&nbsp;a1 q b ≤ a1 q' b :=<br>&nbsp;&nbsp;percent_mono (excess_mono_left h)<br><br>theorem a1_anti_base {q : QRE} {b b' : BaseAmount} (h : b.amount ≤ b'.amount) :<br>&nbsp;&nbsp;&nbsp;&nbsp;a1 q b' ≤ a1 q b :=<br>&nbsp;&nbsp;percent_mono (excess_anti_right h)<br><br>/-- The arithmetic never reads the year: "for the taxable year" needs its own card. -/<br>theorem a1_reads_no_year (y₁ y₂ y₁' y₂' : String) (q b : Nat) :<br>&nbsp;&nbsp;&nbsp;&nbsp;a1 ⟨y₁, q⟩ ⟨y₂, b⟩ = a1 ⟨y₁', q⟩ ⟨y₂', b⟩ :=<br>&nbsp;&nbsp;rfl</code> |
| world | Handler: call math-excess(QRE, base), then math-percent(20, result). Tests: (1,000,000, 600,000) → 80,000; (500,000, 600,000) → 0. The arithmetic reads no year (a1_reads_no_year), so the check that both figures are for the taxable year has to come from another card; ask-qre-year and math-equivalence supply it. |

# cgvu:/reason/input?id=irc-41-a-1-A&cite=26-usc-41-a-1-A&term=qualified-research-expenses&period=taxable-year&unit=usd&defined-by=irc-41-b

| facet | reading |
|---|---|
| data | `cgvu:/reason/input?id=irc-41-a-1-A&cite=26-usc-41-a-1-A&term=qualified-research-expenses&period=taxable-year&unit=usd&defined-by=irc-41-b` |
| meaning | §41(a)(1)(A): the qualified research expenses (QRE) for the taxable year. A whole-dollar figure together with the taxable year it belongs to. What counts as QRE is defined by §41(b). |
| structure | <code>/-- §41(a)(1)(A): QRE for a taxable year. -/<br>structure QRE where<br>&nbsp;&nbsp;year   : String<br>&nbsp;&nbsp;amount : Nat</code> |
| world | Handler: accept {year, amount}; reject a negative or fractional amount. The Spike for this figure starts with every context facet DARK. qre.world (the year) is lit by the ask-qre-year card. qre.meaning would point to irc-41-b. It stays DARK here because the supplied text stops at the §41(b) heading, so nothing has been appended to it. Test: {2025, 1,000,000}. |

# cgvu:/reason/input?id=irc-41-a-1-B&cite=26-usc-41-a-1-B&term=base-amount&period=taxable-year&unit=usd&defined-by=26-usc-41-c

| facet | reading |
|---|---|
| data | `cgvu:/reason/input?id=irc-41-a-1-B&cite=26-usc-41-a-1-B&term=base-amount&period=taxable-year&unit=usd&defined-by=26-usc-41-c` |
| meaning | §41(a)(1)(B): the base amount. A whole-dollar figure with the taxable year it was computed for. It is defined in §41(c), which is outside the supplied text. |
| structure | <code>/-- §41(a)(1)(B): the base amount, for the year it was computed. -/<br>structure BaseAmount where<br>&nbsp;&nbsp;year   : String<br>&nbsp;&nbsp;amount : Nat</code> |
| world | Handler: accept {year, amount} from the base-amount worksheet; the worksheet lights base.world. Test: {2025, 600,000}. The §41(c) computation is out of scope and is treated as a given input. |

# cgvu:/reason/rate?id=irc-41-a-2&cite=26-usc-41-a-2&rate-pct=20&of=basic-research-payments&defined-by=26-usc-41-e-1-A&via=math-percent&out=basic-research-credit

| facet | reading |
|---|---|
| data | `cgvu:/reason/rate?id=irc-41-a-2&cite=26-usc-41-a-2&rate-pct=20&of=basic-research-payments&defined-by=26-usc-41-e-1-A&via=math-percent&out=basic-research-credit` |
| meaning | §41(a)(2): 20 percent of the basic research payments determined under §41(e)(1)(A). |
| structure | <code>/-- §41(a)(2): 20 percent of the §41(e)(1)(A) amount (given). -/<br>def a2 (brp : Nat) : Nat := percent rate41 brp</code> |
| world | Handler: call math-percent(20, payments). Take the input only from the §41(e)(1)(A) determination, which is out of scope and treated as given. Tests: 50,000 → 10,000; 0 → 0. |

# cgvu:/reason/definition?id=irc-41-b&cite=26-usc-41-b&term=qualified-research-expenses&scope=26-usc-41&status=heading-only

| facet | reading |
|---|---|
| data | `cgvu:/reason/definition?id=irc-41-b&cite=26-usc-41-b&term=qualified-research-expenses&scope=26-usc-41&status=heading-only` |
| meaning | §41(b): "For purposes of this section", this subsection defines qualified research expenses. The supplied text stops at the heading, so only the definition's scope is transcribed. The card itself is LIT, since a card's facets are always filled (§10). What is missing is the definition's text, which the source did not supply. |
| structure | <code>/-- §41(b): the reading a LIT `qre.meaning` facet must match. Text not supplied. -/<br>def irc41b : String :=<br>&nbsp;&nbsp;"cgvu:/reason/definition?id=irc-41-b&amp;cite=26-usc-41-b&amp;term=qualified-research-expenses&amp;scope=26-usc-41&amp;status=heading-only"<br><br>/-- A QRE Spike whose `meaning` points to §41(b): FACET-MATCH on the card's address. -/<br>theorem qre_meaning_matches_41b (l : Log) (h : irc41b ∈ currentReading l) :<br>&nbsp;&nbsp;&nbsp;&nbsp;facetMatch l irc41b = true :=<br>&nbsp;&nbsp;(facetMatch_iff_mem l irc41b).mpr h</code> |
| world | This card's address is what a LIT qre.meaning must match (bit E). Until the text of §41(b)(1)–(4) is supplied, nothing appends this address to qre.meaning, so that facet stays DARK and nothing can certify that the QRE figure really is QRE. The declared rule in ask-qre-year does not read qre.meaning, so it does not block that decision. A rule that did read it would be blocked. Test: every address in this document is well-formed under D7 (all_cards_wellformed). |

# cgvu:/math/excess?id=math-excess&a=nat&b=nat&out=nat&floor=0

| facet | reading |
|---|---|
| data | `cgvu:/math/excess?id=math-excess&a=nat&b=nat&out=nat&floor=0` |
| meaning | General card, not a line of the statute. Returns the excess (if any) of a over b: a − b when a &gt; b, and 0 otherwise. The statute's phrase "the excess (if any) of" is this operation. |
| structure | <code>/-- "The excess (if any) of `a` over `b`": truncated subtraction. -/<br>def excess (a b : Nat) : Nat := a - b<br><br>theorem excess_zero_of_le {a b : Nat} (h : a ≤ b) : excess a b = 0 :=<br>&nbsp;&nbsp;Nat.sub_eq_zero_of_le h<br><br>theorem excess_add_of_le {a b : Nat} (h : b ≤ a) : excess a b + b = a :=<br>&nbsp;&nbsp;Nat.sub_add_cancel h<br><br>theorem excess_mono_left {a a' b : Nat} (h : a ≤ a') : excess a b ≤ excess a' b := by<br>&nbsp;&nbsp;unfold excess; omega<br><br>theorem excess_anti_right {a b b' : Nat} (h : b ≤ b') : excess a b' ≤ excess a b := by<br>&nbsp;&nbsp;unfold excess; omega</code> |
| world | Handler: truncated subtraction on whole numbers. Router passes a and b; returns one number. Tests: (1,000,000, 600,000) → 400,000; (500,000, 600,000) → 0; (600,000, 600,000) → 0. Never returns a negative number, so a shortfall under (a)(1) cannot reduce the (a)(2) amount. |

# cgvu:/math/percent?id=math-percent&rate-pct=nat&of=nat&out=nat&round=floor

| facet | reading |
|---|---|
| data | `cgvu:/math/percent?id=math-percent&rate-pct=nat&of=nat&out=nat&round=floor` |
| meaning | General card, not a line of the statute. Returns rate-pct percent of an amount, rounded down to the whole dollar. |
| structure | <code>/-- `r` percent of `a`, rounded down to the whole dollar. -/<br>def percent (r a : Nat) : Nat := a * r / 100<br><br>theorem percent_zero (r : Nat) : percent r 0 = 0 := by<br>&nbsp;&nbsp;simp [percent]<br><br>theorem percent_mono {r a a' : Nat} (h : a ≤ a') : percent r a ≤ percent r a' := by<br>&nbsp;&nbsp;have hm := Nat.mul_le_mul_right r h<br>&nbsp;&nbsp;unfold percent<br>&nbsp;&nbsp;generalize a * r = u at *<br>&nbsp;&nbsp;generalize a' * r = v at *<br>&nbsp;&nbsp;omega<br><br>/-- On whole hundreds the rounding never bites. -/<br>theorem percent_hundreds (r k : Nat) : percent r (100 * k) = r * k := by<br>&nbsp;&nbsp;unfold percent<br>&nbsp;&nbsp;rw [Nat.mul_assoc, Nat.mul_div_cancel_left _ (by decide : 0 &lt; 100), Nat.mul_comm]</code> |
| world | Handler: (of × rate-pct) ÷ 100, rounded down. Tests: (20, 400,000) → 80,000; (20, 50,000) → 10,000; (20, 0) → 0; (20, 7) → 1 (the floor). Every handler must round the same way; the Lean model fixes rounding down. |

# cgvu:/math/equivalence?id=math-equivalence&left=string&right=string&test=exact

| facet | reading |
|---|---|
| data | `cgvu:/math/equivalence?id=math-equivalence&left=string&right=string&test=exact` |
| meaning | General card, not a line of the statute. Are two readings the same? Exact string equality, the only comparison the instrument makes (bit E). The statute uses it implicitly: "for the taxable year" requires both figures to belong to the same year. |
| structure | <code>/-- Exact string equality: bit E (Bits.lean) lifted from entries to readings. -/<br>def equivalent (l r : String) : Bool := l == r<br><br>theorem equivalent_iff (l r : String) : equivalent l r = true ↔ l = r := by<br>&nbsp;&nbsp;unfold equivalent; exact beq_iff_eq<br><br>theorem equivalent_refl (s : String) : equivalent s s = true :=<br>&nbsp;&nbsp;(equivalent_iff s s).mpr rfl<br><br>theorem equivalent_symm {a b : String} (h : equivalent a b = true) : equivalent b a = true :=<br>&nbsp;&nbsp;(equivalent_iff b a).mpr ((equivalent_iff a b).mp h).symm<br><br>theorem equivalent_trans {a b c : String} (h₁ : equivalent a b = true)<br>&nbsp;&nbsp;&nbsp;&nbsp;(h₂ : equivalent b c = true) : equivalent a c = true :=<br>&nbsp;&nbsp;(equivalent_iff a c).mpr (((equivalent_iff a b).mp h₁).trans ((equivalent_iff b c).mp h₂))<br><br>/-- The card is bit E on the entries' addresses; nothing new is compared. -/<br>theorem equivalent_is_bitE (e₁ e₂ : Entry) : bitE e₁ e₂ = equivalent e₁.address e₂.address :=<br>&nbsp;&nbsp;rfl</code> |
| world | Handler: returns true exactly when left and right are the same exact string. No trimming, case-folding or date parsing: "2025" ≠ "FY2025". Tests: (2025, 2025) → true; (2025, 2024) → false; (2025, " 2025") → false. It is an equivalence relation (T6), so matching can be chained. |

# cgvu:/ask/value?id=ask-qre-year&facet=qre.world&answer-type=year&must-match=base.world&on-answer=light&on-decline=halt

| facet | reading |
|---|---|
| data | `cgvu:/ask/value?id=ask-qre-year&facet=qre.world&answer-type=year&must-match=base.world&on-answer=light&on-decline=halt` |
| meaning | Question card. Asks the user: "Which taxable year are your qualified research expenses for?" The answer lights the DARK qre.world facet. The declared rule (yearsMatch) passes only when the QRE year equals the base-amount year and both equal the credit year. |
| structure | <code>/-- Places of the declared rule: 0 = `qre.world`, 1 = `base.world`. -/<br>abbrev Place := Fin 2<br><br>/-- The declared rule, first kind (D15): both figures are for the credit year.<br>&nbsp;&nbsp;&nbsp;&nbsp;Built from the math-equivalence card. -/<br>def yearsMatch (creditYear : String) (x : Place → String) : Bool :=<br>&nbsp;&nbsp;equivalent (x 0) (x 1) &amp;&amp; equivalent (x 0) creditYear<br><br>/-- At the ask: `base.world` LIT from the worksheet, `qre.world` DARK. -/<br>def maskAtAsk : Place → Bool := fun i =&gt; i.val == 1<br><br>/-- After the reply: an answer lights every place; a decline lights nothing new. -/<br>def maskAfter (answered : Bool) : Place → Bool :=<br>&nbsp;&nbsp;if answered then fun _ =&gt; true else maskAtAsk<br><br>/-- Bit F of the first kind, as `Episode.F` in AskBound.lean. -/<br>noncomputable def F (cy : String) (A : Place → Bool) (x : Place → String) : Bool :=<br>&nbsp;&nbsp;@decide (Settles (yearsMatch cy) A x) (Classical.propDecidable _)<br><br>theorem F_iff (cy : String) (A : Place → Bool) (x : Place → String) :<br>&nbsp;&nbsp;&nbsp;&nbsp;F cy A x = true ↔ Settles (yearsMatch cy) A x :=<br>&nbsp;&nbsp;@decide_eq_true_iff _ (Classical.propDecidable _)<br><br>/-- The asked facet is essential: its reading alone can flip the verdict. -/<br>theorem qre_year_essential : Essential (yearsMatch "2025") (0 : Place) :=<br>&nbsp;&nbsp;⟨fun _ =&gt; "2025", "2024", by decide⟩<br><br>/-- With the base year matching, the DARK `qre.world` leaves the verdict open. -/<br>theorem unsettled_at_ask (x : Place → String) (hx : x 1 = "2025") :<br>&nbsp;&nbsp;&nbsp;&nbsp;¬ Settles (yearsMatch "2025") maskAtAsk x := by<br>&nbsp;&nbsp;intro hs<br>&nbsp;&nbsp;have agree : ∀ v, AgreeOn maskAtAsk x (upd x 0 v) := by<br>&nbsp;&nbsp;&nbsp;&nbsp;intro v i hi<br>&nbsp;&nbsp;&nbsp;&nbsp;have : i = 1 := by<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;revert hi; revert i; decide<br>&nbsp;&nbsp;&nbsp;&nbsp;subst this; rfl<br>&nbsp;&nbsp;have h₁ := hs (upd x 0 "2025") (agree _)<br>&nbsp;&nbsp;have h₂ := hs (upd x 0 "0000") (agree _)<br>&nbsp;&nbsp;have e₁ : yearsMatch "2025" (upd x 0 "2025") = true := by<br>&nbsp;&nbsp;&nbsp;&nbsp;simp [yearsMatch, upd, equivalent, hx]<br>&nbsp;&nbsp;have e₂ : yearsMatch "2025" (upd x 0 "0000") = false := by<br>&nbsp;&nbsp;&nbsp;&nbsp;simp [yearsMatch, upd, equivalent]<br>&nbsp;&nbsp;rw [e₁] at h₁; rw [e₂] at h₂; rw [← h₁] at h₂; exact Bool.noConfusion h₂<br><br>/-- Every facet LIT settles any rule. -/<br>theorem settled_all_lit {n : Nat} {α : Type} (f : (Fin n → α) → Bool) (x : Fin n → α) :<br>&nbsp;&nbsp;&nbsp;&nbsp;Settles f (fun _ =&gt; true) x := by<br>&nbsp;&nbsp;intro y hy<br>&nbsp;&nbsp;have : y = x := funext fun i =&gt; (hy i rfl).symm<br>&nbsp;&nbsp;rw [this]<br><br>/-- A base year that is not the credit year settles the rule without asking. -/<br>theorem mismatch_settles_without_asking (x : Place → String) (hx : x 1 ≠ "2025") :<br>&nbsp;&nbsp;&nbsp;&nbsp;Settles (yearsMatch "2025") maskAtAsk x := by<br>&nbsp;&nbsp;have off : ∀ z : Place → String, z 1 ≠ "2025" → yearsMatch "2025" z = false := by<br>&nbsp;&nbsp;&nbsp;&nbsp;intro z hz<br>&nbsp;&nbsp;&nbsp;&nbsp;unfold yearsMatch equivalent<br>&nbsp;&nbsp;&nbsp;&nbsp;cases h₀ : (z 0 == z 1) &lt;;&gt; cases h₁ : (z 0 == "2025") &lt;;&gt; simp_all<br>&nbsp;&nbsp;intro y hy<br>&nbsp;&nbsp;have hy1 : y 1 = x 1 := (hy 1 rfl).symm<br>&nbsp;&nbsp;rw [off x hx, off y (hy1 ▸ hx)]<br><br>theorem route_at_ask (x : Place → String) (hx : x 1 = "2025") :<br>&nbsp;&nbsp;&nbsp;&nbsp;route false (F "2025" maskAtAsk x) = .ASK := by<br>&nbsp;&nbsp;have : F "2025" maskAtAsk x = false := by<br>&nbsp;&nbsp;&nbsp;&nbsp;cases h : F "2025" maskAtAsk x with<br>&nbsp;&nbsp;&nbsp;&nbsp;&#124; false =&gt; rfl<br>&nbsp;&nbsp;&nbsp;&nbsp;&#124; true =&gt; exact absurd ((F_iff _ _ _).mp h) (unsettled_at_ask x hx)<br>&nbsp;&nbsp;rw [this]; rfl<br><br>theorem route_after_answer (cy : String) (x : Place → String) :<br>&nbsp;&nbsp;&nbsp;&nbsp;route false (F cy (maskAfter true) x) = .ACT := by<br>&nbsp;&nbsp;have : F cy (maskAfter true) x = true := (F_iff _ _ _).mpr (settled_all_lit _ x)<br>&nbsp;&nbsp;rw [this]; rfl<br><br>theorem route_after_decline (b : Bool) : route true b = .HALT := rfl</code> |
| world | Handler: show the question and wait. An answer appends the reading to qre.world, which moves it DARK → LIT. A decline sets bit H, and the route becomes HALT with this facet recorded. Governor: at the ask, ASK (route_at_ask); after an answer, ACT (route_after_answer); after a decline, HALT (route_after_decline). If the base year already fails to match the credit year, the rule is settled without asking, so zero questions are needed (mismatch_settles_without_asking, T10). This card is an instance of AskBound's Episode with n = 2 and α = String, so ask_loop_two_exits (T11) applies. |

# cgvu:/chain/sequence?id=chain-rd-credit-2025&steps=ask-qre-year,math-equivalence&into=irc-41-a&emits=result-rd-credit-2025&then=decide-rd-credit

| facet | reading |
|---|---|
| data | `cgvu:/chain/sequence?id=chain-rd-credit-2025&steps=ask-qre-year,math-equivalence&into=irc-41-a&emits=result-rd-credit-2025&then=decide-rd-credit` |
| meaning | Chain card. It runs two cards in sequence: first ask-qre-year (lights the QRE year), then math-equivalence (checks it against the base year and the credit year). Their output, together with the §41(a) computation, becomes a new card: result-rd-credit-2025. That result goes to decide-rd-credit. The chain's address is intent-map policy (intent-map[0] at t0), not a reason entry. |
| structure | <code>/-- What the chain is given: the two figures, the payments and the credit year. -/<br>structure Inputs where<br>&nbsp;&nbsp;qre        : QRE<br>&nbsp;&nbsp;base       : BaseAmount<br>&nbsp;&nbsp;brp        : Nat<br>&nbsp;&nbsp;creditYear : String<br><br>/-- The true reading at each place (the responder answers with it, as in AskBound). -/<br>def truth (i : Inputs) : Place → String :=<br>&nbsp;&nbsp;fun p =&gt; if p.val = 0 then i.qre.year else i.base.year<br><br>/-- The card the chain emits (result-rd-credit-2025). -/<br>structure Result where<br>&nbsp;&nbsp;credit  : Nat<br>&nbsp;&nbsp;verdict : Bool<br>&nbsp;&nbsp;halted  : Bool<br>&nbsp;&nbsp;lit     : Place → Bool<br>&nbsp;&nbsp;reading : Place → String<br><br>/-- ask-qre-year, then math-equivalence (inside `yearsMatch`), into irc-41-a. -/<br>def chain (i : Inputs) (answered : Bool) : Result where<br>&nbsp;&nbsp;credit  := credit i.qre i.base i.brp<br>&nbsp;&nbsp;verdict := yearsMatch i.creditYear (truth i)<br>&nbsp;&nbsp;halted  := !answered<br>&nbsp;&nbsp;lit     := maskAfter answered<br>&nbsp;&nbsp;reading := truth i<br><br>noncomputable def Result.route (c : Result) (cy : String) : Route :=<br>&nbsp;&nbsp;Foundation.route c.halted (F cy c.lit c.reading)<br><br>theorem chain_certified (i : Inputs) :<br>&nbsp;&nbsp;&nbsp;&nbsp;(chain i true).route i.creditYear = .ACT :=<br>&nbsp;&nbsp;route_after_answer _ _<br><br>theorem chain_blocked (i : Inputs) : (chain i false).route i.creditYear = .HALT := rfl<br><br>def example2025 : Inputs := ⟨⟨"2025", 1000000⟩, ⟨"2025", 600000⟩, 50000, "2025"⟩<br><br>theorem chain_certified_example :<br>&nbsp;&nbsp;&nbsp;&nbsp;(chain example2025 true).credit = 90000 ∧ (chain example2025 true).verdict = true ∧<br>&nbsp;&nbsp;&nbsp;&nbsp;(chain example2025 true).route "2025" = .ACT :=<br>&nbsp;&nbsp;⟨by decide, by decide, chain_certified example2025⟩</code> |
| world | Handler: run `steps` left to right, passing each card's output to the next. Compute `into`, then emit the `emits` card and route it to `then`. Each step appends one reason entry, and each append appends one trace entry (D6). Tests: answered → result with credit 90,000, verdict true and route ACT (chain_certified_example). Declined → route HALT (chain_blocked). |

# cgvu:/result/value?id=result-rd-credit-2025&of=irc-41-a&year=2025&value=90000&unit=usd&verdict=true&route=ACT&from=chain-rd-credit-2025

| facet | reading |
|---|---|
| data | `cgvu:/result/value?id=result-rd-credit-2025&of=irc-41-a&year=2025&value=90000&unit=usd&verdict=true&route=ACT&from=chain-rd-credit-2025` |
| meaning | Result card: the card the chain emits, not one written by hand. It is recorded as a reason entry (reason[3] at t10). It states: the 2025 research credit is $90,000, both figures are for 2025, and the Governor route is ACT. It is the input to the decision. |
| structure | <code>/-- The emitted result for the worked example. -/<br>def result2025 : Result := chain example2025 true<br><br>theorem result2025_values : result2025.credit = 90000 ∧ result2025.verdict = true := by<br>&nbsp;&nbsp;decide</code> |
| world | Handler: none. This card is data that the chain writes and the decision reads. The address carries every value the decision needs, so the router can pass it without a lookup. Status: the card's facets are always filled (§10); what depends on the route is whether the result is certified, which it is only if the chain's route is ACT. If the route is ASK or HALT, the chain still emits the result with that route, and the decision escalates or asks again. Lean: the fields of Result (credit, verdict, halted, lit, reading). |

# cgvu:/decide/route?id=decide-rd-credit&result=result-rd-credit-2025&rule=yearsMatch&on-act=include-in-26-usc-38&on-ask=ask-qre-year&on-halt=escalate

| facet | reading |
|---|---|
| data | `cgvu:/decide/route?id=decide-rd-credit&result=result-rd-credit-2025&rule=yearsMatch&on-act=include-in-26-usc-38&on-ask=ask-qre-year&on-halt=escalate` |
| meaning | Decide rule. It is intent-map policy (intent-map[1] at t1). It reads the result card and takes one of four actions: include the credit in the §38 computation, exclude it, ask again, or escalate. It includes the credit only when the route is ACT, the verdict is true and the credit is positive, so it never decides on a silent guess. |
| structure | <code>inductive Action &#124; includeInSec38 &#124; exclude &#124; askAgain &#124; escalate<br>deriving Repr, DecidableEq<br><br>noncomputable def decideOn (c : Result) (cy : String) : Action :=<br>&nbsp;&nbsp;match c.route cy with<br>&nbsp;&nbsp;&#124; .HALT =&gt; .escalate<br>&nbsp;&nbsp;&#124; .ASK  =&gt; .askAgain<br>&nbsp;&nbsp;&#124; .ACT  =&gt; if c.verdict &amp;&amp; decide (0 &lt; c.credit) then .includeInSec38 else .exclude<br><br>/-- Included only when certified: route ACT, rule settled, verdict true, credit &gt; 0. -/<br>theorem include_only_when_certified (c : Result) (cy : String)<br>&nbsp;&nbsp;&nbsp;&nbsp;(h : decideOn c cy = .includeInSec38) :<br>&nbsp;&nbsp;&nbsp;&nbsp;c.route cy = .ACT ∧ Settles (yearsMatch cy) c.lit c.reading ∧<br>&nbsp;&nbsp;&nbsp;&nbsp;c.verdict = true ∧ 0 &lt; c.credit := by<br>&nbsp;&nbsp;unfold decideOn at h<br>&nbsp;&nbsp;cases hr : c.route cy with<br>&nbsp;&nbsp;&#124; HALT =&gt; rw [hr] at h; exact Action.noConfusion h<br>&nbsp;&nbsp;&#124; ASK  =&gt; rw [hr] at h; exact Action.noConfusion h<br>&nbsp;&nbsp;&#124; ACT  =&gt;<br>&nbsp;&nbsp;&nbsp;&nbsp;rw [hr] at h<br>&nbsp;&nbsp;&nbsp;&nbsp;have hv : (c.verdict &amp;&amp; decide (0 &lt; c.credit)) = true := by<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cases hb : (c.verdict &amp;&amp; decide (0 &lt; c.credit))<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;· rw [hb] at h; exact Action.noConfusion h<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;· rfl<br>&nbsp;&nbsp;&nbsp;&nbsp;have hF : F cy c.lit c.reading = true := by<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;unfold Result.route Foundation.route at hr<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cases hH : c.halted &lt;;&gt; cases hF : F cy c.lit c.reading &lt;;&gt; simp_all<br>&nbsp;&nbsp;&nbsp;&nbsp;refine ⟨rfl, (F_iff _ _ _).mp hF, ?_, ?_⟩<br>&nbsp;&nbsp;&nbsp;&nbsp;· cases hc : c.verdict &lt;;&gt; simp_all<br>&nbsp;&nbsp;&nbsp;&nbsp;· cases hc : c.verdict &lt;;&gt; simp_all<br><br>theorem decide_example : decideOn result2025 "2025" = .includeInSec38 := by<br>&nbsp;&nbsp;unfold decideOn<br>&nbsp;&nbsp;rw [show result2025.route "2025" = .ACT from chain_certified example2025]<br>&nbsp;&nbsp;decide</code> |
| world | Handler: HALT → escalate to a person, with the declined facet recorded. ASK → re-issue the on-ask card. ACT with verdict true and credit &gt; 0 → include in §38. Any other ACT → exclude. When the router dispatches, it appends a decide entry. Per D13 that entry's address is navigable (cgp://) and carries the as-of trace index; the card itself stays cgvu:/. Tests: decide_example (worked example → include); include_only_when_certified. |

# cgvu:/trace/roll?id=roll-rd-credit-2025&object=chain-rd-credit-2025&order=per-object-trace&cards=ask-qre-year,math-equivalence,irc-41-a,result-rd-credit-2025,decide-rd-credit

This is the chain as it runs, one append per row and one trace index per append (D6). The chain and the decide rule go into `intent-map`. Ask, arithmetic and result cards go into `reason`. Figures and answers go into `observe`. qre.meaning, which would point to irc-41-b, stays DARK from start to finish. The decision is still certified, because the declared rule, yearsMatch, does not read that facet. The decide entries use `cgp://` because D13 requires a decide entry's address to be navigable. Every card remains `cgvu:/`.

| trace | slot | address appended | reading |
|---|---|---|---|
| t0 | intent-map[0] | `cgvu:/chain/sequence?id=chain-rd-credit-2025&steps=ask-qre-year,math-equivalence&into=irc-41-a&emits=result-rd-credit-2025&then=decide-rd-credit` | Policy: the chain this object applies. |
| t1 | intent-map[1] | `cgvu:/decide/route?id=decide-rd-credit&result=result-rd-credit-2025&rule=yearsMatch&on-act=include-in-26-usc-38&on-ask=ask-qre-year&on-halt=escalate` | Policy: the decide rule the chain hands its result to. |
| t2 | observe[0] (mint) | data: cgvu:/data/figure?name=qre&amount=1000000&unit=usd&source=user<br>meaning, structure, world: "" | QRE Spike minted. All three context facets are DARK. |
| t3 | observe[1] (mint) | data: cgvu:/data/figure?name=base-amount&amount=600000&unit=usd&source=worksheet<br>meaning, structure, world: "" | Base-amount Spike minted. All three context facets are DARK. |
| t4 | observe[1].world | `cgvu:/data/tax-year?value=2025&source=worksheet` | base.world is LIT. |
| t5 | reason[0] | `cgvu:/ask/value?id=ask-qre-year&facet=qre.world&answer-type=year&must-match=base.world&on-answer=light&on-decline=halt` | qre.world is DARK and essential, so F is false and H is false. |
| t6 | decide[0] | `cgp://acme/decide/rd-credit-2025/0` | Route ASK, as of t5 (route_at_ask). |
| t7 | reason[1] | `cgvu:/ask/reply?id=ask-qre-year&value=2025&by=user` | The user answers. |
| t8 | observe[0].world | `cgvu:/data/tax-year?value=2025&source=user` | qre.world is LIT. FACET-DARK flips at this one facet only (T2). |
| t9 | reason[2] | `cgvu:/math/equivalence?id=math-equivalence&left=2025&right=2025&test=exact&result=true` | The two years match. |
| t10 | reason[3] | `cgvu:/result/value?id=result-rd-credit-2025&of=irc-41-a&year=2025&value=90000&unit=usd&verdict=true&route=ACT&from=chain-rd-credit-2025` | The card the chain emits, recorded as a reason entry. |
| t11 | decide[1] | `cgp://acme/decide/rd-credit-2025/1` | Route ACT, as of t10. decide-rd-credit returns include-in-26-usc-38 (decide_example). |

# cgvu:/resource?name=reasoning-cards-lean&module=ReasoningCards&imports=Certainty,Bits&lean=4.19.0&core-only=true

The full module. Every structure facet above is one section of it.

```lean
/-
  ReasoningCards.lean — Reasoning Cards for 26 U.S.C. §41(a)–(b).
  Lean 4.19.0, core library only (no Mathlib). No proof holes.
  Builds on Certainty.lean (Settles, Essential, AgreeOn, upd) and Bits.lean
  (Route, route, bitE, wellFormedAddressB); goes after Bits in the build order.

  Each `/-! ## -/` section is one Reasoning Card; its header is the card's
  `cgvu:/` address (D7). Three kinds of card:
   * statute cards transcribe one line of §41 (irc-41-a …, irc-41-b);
   * general cards are acts the statute needs but does not state
     (math-excess, math-percent, math-equivalence);
   * process cards run the decision: an ASK (ask-qre-year), a chain
     (chain-rd-credit-2025), the card it emits (result-rd-credit-2025) and the
     decide rule that routes on that result (decide-rd-credit).
  Where each address lands (D5): the chain and the decide rule are intent-map
  policy; statute, general, ask and result cards are what `reason` entries point
  to; figures and answers are appended to `observe` facets.
  Amounts are whole US dollars (`Nat`); percentages floor to the dollar.
  Nothing here adds a field to Entry, Spike or CObj.
-/
import Certainty
import Bits

namespace Cards

open Certainty Foundation

/-! ## cgvu:/math/excess?id=math-excess&a=nat&b=nat&out=nat&floor=0 -/

/-- "The excess (if any) of `a` over `b`": truncated subtraction. -/
def excess (a b : Nat) : Nat := a - b

theorem excess_zero_of_le {a b : Nat} (h : a ≤ b) : excess a b = 0 :=
  Nat.sub_eq_zero_of_le h

theorem excess_add_of_le {a b : Nat} (h : b ≤ a) : excess a b + b = a :=
  Nat.sub_add_cancel h

theorem excess_mono_left {a a' b : Nat} (h : a ≤ a') : excess a b ≤ excess a' b := by
  unfold excess; omega

theorem excess_anti_right {a b b' : Nat} (h : b ≤ b') : excess a b' ≤ excess a b := by
  unfold excess; omega

/-! ## cgvu:/math/percent?id=math-percent&rate-pct=nat&of=nat&out=nat&round=floor -/

/-- `r` percent of `a`, rounded down to the whole dollar. -/
def percent (r a : Nat) : Nat := a * r / 100

theorem percent_zero (r : Nat) : percent r 0 = 0 := by
  simp [percent]

theorem percent_mono {r a a' : Nat} (h : a ≤ a') : percent r a ≤ percent r a' := by
  have hm := Nat.mul_le_mul_right r h
  unfold percent
  generalize a * r = u at *
  generalize a' * r = v at *
  omega

/-- On whole hundreds the rounding never bites. -/
theorem percent_hundreds (r k : Nat) : percent r (100 * k) = r * k := by
  unfold percent
  rw [Nat.mul_assoc, Nat.mul_div_cancel_left _ (by decide : 0 < 100), Nat.mul_comm]

/-! ## cgvu:/math/equivalence?id=math-equivalence&left=string&right=string&test=exact -/

/-- Exact string equality: bit E (Bits.lean) lifted from entries to readings. -/
def equivalent (l r : String) : Bool := l == r

theorem equivalent_iff (l r : String) : equivalent l r = true ↔ l = r := by
  unfold equivalent; exact beq_iff_eq

theorem equivalent_refl (s : String) : equivalent s s = true :=
  (equivalent_iff s s).mpr rfl

theorem equivalent_symm {a b : String} (h : equivalent a b = true) : equivalent b a = true :=
  (equivalent_iff b a).mpr ((equivalent_iff a b).mp h).symm

theorem equivalent_trans {a b c : String} (h₁ : equivalent a b = true)
    (h₂ : equivalent b c = true) : equivalent a c = true :=
  (equivalent_iff a c).mpr (((equivalent_iff a b).mp h₁).trans ((equivalent_iff b c).mp h₂))

/-- The card is bit E on the entries' addresses; nothing new is compared. -/
theorem equivalent_is_bitE (e₁ e₂ : Entry) : bitE e₁ e₂ = equivalent e₁.address e₂.address :=
  rfl

/-! ## cgvu:/reason/input?id=irc-41-a-1-A&cite=26-usc-41-a-1-A&term=qualified-research-expenses&period=taxable-year&unit=usd&defined-by=irc-41-b -/

/-- §41(a)(1)(A): QRE for a taxable year. -/
structure QRE where
  year   : String
  amount : Nat

/-! ## cgvu:/reason/input?id=irc-41-a-1-B&cite=26-usc-41-a-1-B&term=base-amount&period=taxable-year&unit=usd&defined-by=26-usc-41-c -/

/-- §41(a)(1)(B): the base amount, for the year it was computed. -/
structure BaseAmount where
  year   : String
  amount : Nat

/-! ## cgvu:/reason/rate-of-excess?id=irc-41-a-1&cite=26-usc-41-a-1&rate-pct=20&over=irc-41-a-1-A&under=irc-41-a-1-B&via=math-excess,math-percent&out=incremental-credit -/

/-- The statutory rate in §41(a)(1) and (a)(2), in percent. -/
def rate41 : Nat := 20

/-- §41(a)(1) = math-percent 20 ∘ math-excess. -/
def a1 (q : QRE) (b : BaseAmount) : Nat := percent rate41 (excess q.amount b.amount)

theorem a1_zero_of_le {q : QRE} {b : BaseAmount} (h : q.amount ≤ b.amount) : a1 q b = 0 := by
  unfold a1; rw [excess_zero_of_le h]; exact percent_zero _

theorem a1_mono_qre {q q' : QRE} {b : BaseAmount} (h : q.amount ≤ q'.amount) :
    a1 q b ≤ a1 q' b :=
  percent_mono (excess_mono_left h)

theorem a1_anti_base {q : QRE} {b b' : BaseAmount} (h : b.amount ≤ b'.amount) :
    a1 q b' ≤ a1 q b :=
  percent_mono (excess_anti_right h)

/-- The arithmetic never reads the year: "for the taxable year" needs its own card. -/
theorem a1_reads_no_year (y₁ y₂ y₁' y₂' : String) (q b : Nat) :
    a1 ⟨y₁, q⟩ ⟨y₂, b⟩ = a1 ⟨y₁', q⟩ ⟨y₂', b⟩ :=
  rfl

/-! ## cgvu:/reason/rate?id=irc-41-a-2&cite=26-usc-41-a-2&rate-pct=20&of=basic-research-payments&defined-by=26-usc-41-e-1-A&via=math-percent&out=basic-research-credit -/

/-- §41(a)(2): 20 percent of the §41(e)(1)(A) amount (given). -/
def a2 (brp : Nat) : Nat := percent rate41 brp

/-! ## cgvu:/reason/sum?id=irc-41-a&cite=26-usc-41-a&terms=irc-41-a-1,irc-41-a-2&out=research-credit&feeds=26-usc-38 -/

/-- §41(a): the research credit that feeds §38. -/
def credit (q : QRE) (b : BaseAmount) (brp : Nat) : Nat := a1 q b + a2 brp

theorem credit_example : credit ⟨"2025", 1000000⟩ ⟨"2025", 600000⟩ 50000 = 90000 := by
  decide

theorem credit_when_no_excess {q : QRE} {b : BaseAmount} (brp : Nat)
    (h : q.amount ≤ b.amount) : credit q b brp = a2 brp := by
  unfold credit; rw [a1_zero_of_le h, Nat.zero_add]

/-! ## cgvu:/reason/definition?id=irc-41-b&cite=26-usc-41-b&term=qualified-research-expenses&scope=26-usc-41&status=heading-only -/

/-- §41(b): the reading a LIT `qre.meaning` facet must match. Text not supplied. -/
def irc41b : String :=
  "cgvu:/reason/definition?id=irc-41-b&cite=26-usc-41-b&term=qualified-research-expenses&scope=26-usc-41&status=heading-only"

/-- A QRE Spike whose `meaning` points to §41(b): FACET-MATCH on the card's address. -/
theorem qre_meaning_matches_41b (l : Log) (h : irc41b ∈ currentReading l) :
    facetMatch l irc41b = true :=
  (facetMatch_iff_mem l irc41b).mpr h

/-! ## cgvu:/ask/value?id=ask-qre-year&facet=qre.world&answer-type=year&must-match=base.world&on-answer=light&on-decline=halt -/

/-- Places of the declared rule: 0 = `qre.world`, 1 = `base.world`. -/
abbrev Place := Fin 2

/-- The declared rule, first kind (D15): both figures are for the credit year.
    Built from the math-equivalence card. -/
def yearsMatch (creditYear : String) (x : Place → String) : Bool :=
  equivalent (x 0) (x 1) && equivalent (x 0) creditYear

/-- At the ask: `base.world` LIT from the worksheet, `qre.world` DARK. -/
def maskAtAsk : Place → Bool := fun i => i.val == 1

/-- After the reply: an answer lights every place; a decline lights nothing new. -/
def maskAfter (answered : Bool) : Place → Bool :=
  if answered then fun _ => true else maskAtAsk

/-- Bit F of the first kind, as `Episode.F` in AskBound.lean. -/
noncomputable def F (cy : String) (A : Place → Bool) (x : Place → String) : Bool :=
  @decide (Settles (yearsMatch cy) A x) (Classical.propDecidable _)

theorem F_iff (cy : String) (A : Place → Bool) (x : Place → String) :
    F cy A x = true ↔ Settles (yearsMatch cy) A x :=
  @decide_eq_true_iff _ (Classical.propDecidable _)

/-- The asked facet is essential: its reading alone can flip the verdict. -/
theorem qre_year_essential : Essential (yearsMatch "2025") (0 : Place) :=
  ⟨fun _ => "2025", "2024", by decide⟩

/-- With the base year matching, the DARK `qre.world` leaves the verdict open. -/
theorem unsettled_at_ask (x : Place → String) (hx : x 1 = "2025") :
    ¬ Settles (yearsMatch "2025") maskAtAsk x := by
  intro hs
  have agree : ∀ v, AgreeOn maskAtAsk x (upd x 0 v) := by
    intro v i hi
    have : i = 1 := by
      revert hi; revert i; decide
    subst this; rfl
  have h₁ := hs (upd x 0 "2025") (agree _)
  have h₂ := hs (upd x 0 "0000") (agree _)
  have e₁ : yearsMatch "2025" (upd x 0 "2025") = true := by
    simp [yearsMatch, upd, equivalent, hx]
  have e₂ : yearsMatch "2025" (upd x 0 "0000") = false := by
    simp [yearsMatch, upd, equivalent]
  rw [e₁] at h₁; rw [e₂] at h₂; rw [← h₁] at h₂; exact Bool.noConfusion h₂

/-- Every facet LIT settles any rule. -/
theorem settled_all_lit {n : Nat} {α : Type} (f : (Fin n → α) → Bool) (x : Fin n → α) :
    Settles f (fun _ => true) x := by
  intro y hy
  have : y = x := funext fun i => (hy i rfl).symm
  rw [this]

/-- A base year that is not the credit year settles the rule without asking. -/
theorem mismatch_settles_without_asking (x : Place → String) (hx : x 1 ≠ "2025") :
    Settles (yearsMatch "2025") maskAtAsk x := by
  have off : ∀ z : Place → String, z 1 ≠ "2025" → yearsMatch "2025" z = false := by
    intro z hz
    unfold yearsMatch equivalent
    cases h₀ : (z 0 == z 1) <;> cases h₁ : (z 0 == "2025") <;> simp_all
  intro y hy
  have hy1 : y 1 = x 1 := (hy 1 rfl).symm
  rw [off x hx, off y (hy1 ▸ hx)]

theorem route_at_ask (x : Place → String) (hx : x 1 = "2025") :
    route false (F "2025" maskAtAsk x) = .ASK := by
  have : F "2025" maskAtAsk x = false := by
    cases h : F "2025" maskAtAsk x with
    | false => rfl
    | true => exact absurd ((F_iff _ _ _).mp h) (unsettled_at_ask x hx)
  rw [this]; rfl

theorem route_after_answer (cy : String) (x : Place → String) :
    route false (F cy (maskAfter true) x) = .ACT := by
  have : F cy (maskAfter true) x = true := (F_iff _ _ _).mpr (settled_all_lit _ x)
  rw [this]; rfl

theorem route_after_decline (b : Bool) : route true b = .HALT := rfl

/-! ## cgvu:/chain/sequence?id=chain-rd-credit-2025&steps=ask-qre-year,math-equivalence&into=irc-41-a&emits=result-rd-credit-2025&then=decide-rd-credit -/

/-- What the chain is given: the two figures, the payments and the credit year. -/
structure Inputs where
  qre        : QRE
  base       : BaseAmount
  brp        : Nat
  creditYear : String

/-- The true reading at each place (the responder answers with it, as in AskBound). -/
def truth (i : Inputs) : Place → String :=
  fun p => if p.val = 0 then i.qre.year else i.base.year

/-- The card the chain emits (result-rd-credit-2025). -/
structure Result where
  credit  : Nat
  verdict : Bool
  halted  : Bool
  lit     : Place → Bool
  reading : Place → String

/-- ask-qre-year, then math-equivalence (inside `yearsMatch`), into irc-41-a. -/
def chain (i : Inputs) (answered : Bool) : Result where
  credit  := credit i.qre i.base i.brp
  verdict := yearsMatch i.creditYear (truth i)
  halted  := !answered
  lit     := maskAfter answered
  reading := truth i

noncomputable def Result.route (c : Result) (cy : String) : Route :=
  Foundation.route c.halted (F cy c.lit c.reading)

theorem chain_certified (i : Inputs) :
    (chain i true).route i.creditYear = .ACT :=
  route_after_answer _ _

theorem chain_blocked (i : Inputs) : (chain i false).route i.creditYear = .HALT := rfl

def example2025 : Inputs := ⟨⟨"2025", 1000000⟩, ⟨"2025", 600000⟩, 50000, "2025"⟩

theorem chain_certified_example :
    (chain example2025 true).credit = 90000 ∧ (chain example2025 true).verdict = true ∧
    (chain example2025 true).route "2025" = .ACT :=
  ⟨by decide, by decide, chain_certified example2025⟩

/-! ## cgvu:/result/value?id=result-rd-credit-2025&of=irc-41-a&year=2025&value=90000&unit=usd&verdict=true&route=ACT&from=chain-rd-credit-2025 -/

/-- The emitted result for the worked example. -/
def result2025 : Result := chain example2025 true

theorem result2025_values : result2025.credit = 90000 ∧ result2025.verdict = true := by
  decide

/-! ## cgvu:/decide/route?id=decide-rd-credit&result=result-rd-credit-2025&rule=yearsMatch&on-act=include-in-26-usc-38&on-ask=ask-qre-year&on-halt=escalate -/

inductive Action | includeInSec38 | exclude | askAgain | escalate
deriving Repr, DecidableEq

noncomputable def decideOn (c : Result) (cy : String) : Action :=
  match c.route cy with
  | .HALT => .escalate
  | .ASK  => .askAgain
  | .ACT  => if c.verdict && decide (0 < c.credit) then .includeInSec38 else .exclude

/-- Included only when certified: route ACT, rule settled, verdict true, credit > 0. -/
theorem include_only_when_certified (c : Result) (cy : String)
    (h : decideOn c cy = .includeInSec38) :
    c.route cy = .ACT ∧ Settles (yearsMatch cy) c.lit c.reading ∧
    c.verdict = true ∧ 0 < c.credit := by
  unfold decideOn at h
  cases hr : c.route cy with
  | HALT => rw [hr] at h; exact Action.noConfusion h
  | ASK  => rw [hr] at h; exact Action.noConfusion h
  | ACT  =>
    rw [hr] at h
    have hv : (c.verdict && decide (0 < c.credit)) = true := by
      cases hb : (c.verdict && decide (0 < c.credit))
      · rw [hb] at h; exact Action.noConfusion h
      · rfl
    have hF : F cy c.lit c.reading = true := by
      unfold Result.route Foundation.route at hr
      cases hH : c.halted <;> cases hF : F cy c.lit c.reading <;> simp_all
    refine ⟨rfl, (F_iff _ _ _).mp hF, ?_, ?_⟩
    · cases hc : c.verdict <;> simp_all
    · cases hc : c.verdict <;> simp_all

theorem decide_example : decideOn result2025 "2025" = .includeInSec38 := by
  unfold decideOn
  rw [show result2025.route "2025" = .ACT from chain_certified example2025]
  decide

/-! ## Every card address is well-formed (D7): `cgvu:/`, never `cgvu://`. -/

def cardAddresses : List String := [
  "cgvu:/math/excess?id=math-excess&a=nat&b=nat&out=nat&floor=0",
  "cgvu:/math/percent?id=math-percent&rate-pct=nat&of=nat&out=nat&round=floor",
  "cgvu:/math/equivalence?id=math-equivalence&left=string&right=string&test=exact",
  "cgvu:/reason/input?id=irc-41-a-1-A&cite=26-usc-41-a-1-A&term=qualified-research-expenses&period=taxable-year&unit=usd&defined-by=irc-41-b",
  "cgvu:/reason/input?id=irc-41-a-1-B&cite=26-usc-41-a-1-B&term=base-amount&period=taxable-year&unit=usd&defined-by=26-usc-41-c",
  "cgvu:/reason/rate-of-excess?id=irc-41-a-1&cite=26-usc-41-a-1&rate-pct=20&over=irc-41-a-1-A&under=irc-41-a-1-B&via=math-excess,math-percent&out=incremental-credit",
  "cgvu:/reason/rate?id=irc-41-a-2&cite=26-usc-41-a-2&rate-pct=20&of=basic-research-payments&defined-by=26-usc-41-e-1-A&via=math-percent&out=basic-research-credit",
  "cgvu:/reason/sum?id=irc-41-a&cite=26-usc-41-a&terms=irc-41-a-1,irc-41-a-2&out=research-credit&feeds=26-usc-38",
  "cgvu:/reason/definition?id=irc-41-b&cite=26-usc-41-b&term=qualified-research-expenses&scope=26-usc-41&status=heading-only",
  "cgvu:/ask/value?id=ask-qre-year&facet=qre.world&answer-type=year&must-match=base.world&on-answer=light&on-decline=halt",
  "cgvu:/chain/sequence?id=chain-rd-credit-2025&steps=ask-qre-year,math-equivalence&into=irc-41-a&emits=result-rd-credit-2025&then=decide-rd-credit",
  "cgvu:/result/value?id=result-rd-credit-2025&of=irc-41-a&year=2025&value=90000&unit=usd&verdict=true&route=ACT&from=chain-rd-credit-2025",
  "cgvu:/decide/route?id=decide-rd-credit&result=result-rd-credit-2025&rule=yearsMatch&on-act=include-in-26-usc-38&on-ask=ask-qre-year&on-halt=escalate"
]

theorem all_cards_wellformed : cardAddresses.all wellFormedAddressB = true := by
  decide

end Cards

/-! ## Audit, as in Audit.lean: statement and axioms of every card theorem. -/
#print axioms Cards.excess_zero_of_le
#print axioms Cards.excess_add_of_le
#print axioms Cards.excess_mono_left
#print axioms Cards.excess_anti_right
#print axioms Cards.percent_zero
#print axioms Cards.percent_mono
#print axioms Cards.percent_hundreds
#print axioms Cards.equivalent_iff
#print axioms Cards.equivalent_refl
#print axioms Cards.equivalent_symm
#print axioms Cards.equivalent_trans
#print axioms Cards.equivalent_is_bitE
#print axioms Cards.a1_zero_of_le
#print axioms Cards.a1_mono_qre
#print axioms Cards.a1_anti_base
#print axioms Cards.a1_reads_no_year
#print axioms Cards.credit_example
#print axioms Cards.credit_when_no_excess
#print axioms Cards.qre_meaning_matches_41b
#print axioms Cards.F_iff
#print axioms Cards.qre_year_essential
#print axioms Cards.unsettled_at_ask
#print axioms Cards.settled_all_lit
#print axioms Cards.mismatch_settles_without_asking
#print axioms Cards.route_at_ask
#print axioms Cards.route_after_answer
#print axioms Cards.route_after_decline
#print axioms Cards.chain_certified
#print axioms Cards.chain_blocked
#print axioms Cards.chain_certified_example
#print axioms Cards.result2025_values
#print axioms Cards.include_only_when_certified
#print axioms Cards.decide_example
#print axioms Cards.all_cards_wellformed
```

# cgvu:/resource?name=reasoning-cards-build&module=ReasoningCards&lean=4.19.0&sorry=0

Build it against the repository: put `ReasoningCards.lean` in `~/Documents/GitHub/w3c/proof/lean/`, next to `Certainty.lean`, `Types.lean` and `Bits.lean`. Then build Certainty → Types → Bits (or run `./build.sh`) and run `LEAN_PATH=. lean ReasoningCards.lean`. That way the hashes of the imported files match `lean/BUILD_LOG.txt`.

The output below comes from a build outside the repository. It used copies of those three files rebuilt from the proof listing PDF: same declarations, but blank lines lost, so different sha256. Exit code 0, `grep -c sorry` = 0. Output:

```text
'Cards.excess_zero_of_le' depends on axioms: [propext]
'Cards.excess_add_of_le' depends on axioms: [propext]
'Cards.excess_mono_left' depends on axioms: [propext, Quot.sound]
'Cards.excess_anti_right' depends on axioms: [propext, Quot.sound]
'Cards.percent_zero' depends on axioms: [propext]
'Cards.percent_mono' depends on axioms: [propext, Quot.sound]
'Cards.percent_hundreds' depends on axioms: [propext]
'Cards.equivalent_iff' does not depend on any axioms
'Cards.equivalent_refl' does not depend on any axioms
'Cards.equivalent_symm' does not depend on any axioms
'Cards.equivalent_trans' does not depend on any axioms
'Cards.equivalent_is_bitE' does not depend on any axioms
'Cards.a1_zero_of_le' depends on axioms: [propext]
'Cards.a1_mono_qre' depends on axioms: [propext, Quot.sound]
'Cards.a1_anti_base' depends on axioms: [propext, Quot.sound]
'Cards.a1_reads_no_year' does not depend on any axioms
'Cards.credit_example' does not depend on any axioms
'Cards.credit_when_no_excess' depends on axioms: [propext]
'Cards.qre_meaning_matches_41b' depends on axioms: [propext]
'Cards.F_iff' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.qre_year_essential' depends on axioms: [propext]
'Cards.unsettled_at_ask' depends on axioms: [propext]
'Cards.settled_all_lit' depends on axioms: [Quot.sound]
'Cards.mismatch_settles_without_asking' depends on axioms: [propext]
'Cards.route_at_ask' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.route_after_answer' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.route_after_decline' does not depend on any axioms
'Cards.chain_certified' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.chain_blocked' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.chain_certified_example' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.result2025_values' depends on axioms: [propext]
'Cards.include_only_when_certified' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.decide_example' depends on axioms: [propext, Classical.choice, Quot.sound]
'Cards.all_cards_wellformed' does not depend on any axioms
```
