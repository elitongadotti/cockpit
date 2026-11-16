---
name: plain-english
description: >-
  Write plain English with none of the patterns listed on Wikipedia's "Signs of AI
  writing" page. Use for anything written, public or private: PR titles and bodies,
  issue bodies, review comments, commit messages, ADRs, docs, code comments, chat
  messages and replies. Triggers: writing or editing any prose, "write the
  PR/issue/comment", "rewrite this so it doesn't sound like AI", "plain English",
  "de-AI this".
---

# Plain English

Every pattern on Wikipedia's "Signs of AI writing" page is banned here (see Source
at the end). The page is about Wikipedia edits, so each pattern is mapped to the
places we write: PRs, issues, review threads, commits, ADRs, docs and chat.
Wikipedia-only mechanics (draft review, infoboxes, wikitext) map to their nearest
equivalent.

The page's main finding: a model drifts toward the most common phrasing, so a
specific fact turns into a generic, inflated one. "Added a retry on HTTP 429"
becomes "significantly improved the resilience of the integration". Almost every
rule below is the same fix: write the specific fact, at its real size.

## Content

### Don't inflate significance

Banned: stands as, serves as, is a testament, is a reminder, a crucial / pivotal /
vital / significant / key role or moment, underscores its importance, highlights its
significance, reflects broader, symbolizing its ongoing / enduring / lasting,
contributing to the, setting the stage for, marking the, shaping the, represents a
shift, marks a shift, key turning point, evolving landscape, focal point, indelible
mark, deeply rooted, "this distinction matters because", has shaped discussions /
debate about, raises questions about, maintains an active / strong presence.

Also banned: tying a small change to a big trend; admitting something is minor and
then praising it anyway ("Although a small change, it plays a meaningful part in
…"); tacking on the wider system nobody asked about.

Instead: say what the thing does and stop. If it has a real effect, state the effect
with its number.

### Don't end a sentence with an -ing verdict

Banned: a sentence ending in ", highlighting / underscoring / emphasizing / ensuring
/ reflecting / symbolizing / contributing to / cultivating / fostering / encompassing
/ enhancing / showcasing …". Also "valuable insights", "aligns with", "resonates
with".

Bad: "The job now reads only new files, ensuring efficient and reliable processing."
Good: "The job now reads only files added since the last run."

Instead: if the effect is real, give it its own sentence and its evidence. If not,
delete the clause.

### Say the real relationship

Banned: associated with, in connection with, connected to, in association with, "has
been associated with", "began their career as" or "ventured into X as" when you mean
"was".

Instead: name the relation. Owns, calls, reads, writes, replaced, was the CEO of.

### Say what the source says, not where it appeared

Banned: piling up where something was covered instead of what it says ("featured
in", "cited in", "profiled in", "documented across major outlets", "independent
coverage", "trade publications"); "the documentation identifies / lists / describes
Y as", "is listed in", "was identified by"; attributing to a source something it
doesn't say.

Instead: state the fact, then link the exact section. "Python dicts keep insertion
order ([docs](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict))."

### No vague crowds

Banned: "experts argue", "observers have cited", "industry reports", "some critics",
"researchers and practitioners", "several sources" when you have one, "widely seen
as", "such as" in front of a list that is in fact complete.

Instead: name who said it and link it, or say it's your own view ("I think").

### No "challenges and future" formula

Banned: "Despite its X, Y faces several challenges …", "Despite these challenges …",
sections called "Challenges", "Future Outlook", "Future Directions", "Challenges and
Legacy", and a closing line that turns vaguely hopeful ("continues to thrive",
"positions it as", "could improve … in the coming years").

Instead: list the open problems as facts, each with what would fix it. No closing
cheer. Naming a problem is fine; the formula is not.

### No "X and Y" filler headings

Banned: "Awards and recognition", "Recognition", "Impact and legacy" and similar.

Instead: a heading says what is in the section.

### Don't open by defining your own title

Banned: "<Title> refers to …", "<Title> is a collection of …" as the first sentence
of a doc, ADR or README.

Instead: open with the fact the reader came for.

### No hedging to fill a gap

Banned: "While specific details are limited / scarce …", "not widely available /
documented / disclosed", "in the provided / available sources", "in the search
results", "based on available information", "it likely …" right after admitting you
don't know, "[claim] should be treated as X rather than Y", "does not by itself
establish …", "should not be confused with …" (unless a reader really would confuse
the two), "keeps details private".

Instead: say what you checked and stop. "I didn't find this in the vendor docs (not
verified)." One label, no padding, no guess dressed up as context.

### No teaching asides

Banned: "It's important to note", "It is crucial to differentiate", "It's important
to remember", "It's worth noting", "may vary".

Instead: state the thing. If the reader needs it, it belongs in the sentence itself.

### No AI meta-talk

Banned: "As of my last knowledge update", "Up to my last training update", "As an AI
language model", "As a large language model", "I cannot … but I can …", "I'm sorry,
but".

Instead: if a fact may be stale, check it or mark it "(not verified)".

### No summary endings

Banned: "In summary", "In conclusion", "Overall", a "Conclusion" section, and a last
sentence that restates the paragraph.

Instead: end when the last new fact is written.

## Words

### The overused-word list

Don't use these in their figurative or filler sense. The page groups them by model
era, and they come in clusters: where one shows up, others follow.

- Additionally (starting a sentence), align with, boasts (meaning "has"), bolstered,
  crucial, deep dive, delve, emphasizing, enduring, enhance, fostering, garner,
  highlight (verb), interplay, intricate, intricacies, key (adjective), landscape
  (abstract), meticulous, meticulously, pivotal, robust, showcase, showcasing,
  tapestry (abstract), testament, underscore (verb), valuable, vibrant.
- Newer (mid-2026 on): challenge, clearer, dependable, echoed, foster, leverage,
  matters, multifaceted, practical, prioritize, quietly, steady, universally.
- Grok-style: causal, empirical, correlate, used to sound scientific.

The literal or technical sense is fine: primary key, the `_` character, an empirical
test, a measured correlation. The page is explicit that the ban is on these words,
not their synonyms, so swapping "crucial" for "critical" fixes nothing. Rewrite the
sentence instead.

### Promotional words

Banned: boasts a, vibrant, rich, profound, enhancing, showcasing, exemplifies,
commitment to, natural beauty, nestled, in the heart of, groundbreaking, renowned,
featuring, diverse array, seamlessly, "offers a fascinating glimpse".

### Use is, are, has

| Don't write | Write |
|---|---|
| serves as, stands as, functions as, operates as, acts as | is |
| represents a …, marks a … | is, or say what changed |
| boasts, features, offers, maintains | has |
| holds the distinction of being | is the first, is the only |

### Plain verbs over stiff ones

wrote, not authored. moved, not relocated. used, not utilized. tried, not attempted.
died, not passed away.

## Sentence shapes

### No fake contrast

Banned: "not only X but also Y", "it's not just X, it's Y", "it's not X, it's Y",
"X isn't A, it's B", "no X, no Y, just Z", "Y rather than X". Example: "It's not a
cache, it's a contract." These set up a mistake nobody made so the sentence can
correct it.

Instead: say what the thing is. Compare only against an option someone actually
proposed, in two plain sentences: "We use an index on `created_at`. A full table
scan reads every row on each query."

### No padded triplets

Banned: three adjectives or three short phrases by reflex ("fast, reliable, and
scalable").

Instead: list as many items as there are. Two is fine. Four is fine.

### Same thing, same word

Banned: rotating names for one thing to avoid repeating a word ("the job … the
pipeline … the workload … the process").

Instead: pick one term and keep it.

### Punctuation and rhythm

- At most one em dash per piece, and never a spaced " — " for punch. Use a comma,
  parentheses or a colon.
- Don't overcorrect into nothing but commas and periods, with every sentence 8 to 20
  words. The page now lists that as a tell too. Mix sentence lengths; semicolons and
  parentheses are fine where they fit.
- Straight quotes and apostrophes (" '), not curly ones (“ ” ’).

## Talking to the reader

### No chatbot service lines

Banned: "Certainly!", "Of course!", "You're absolutely right!", "Great question",
"Great catch", "I hope this helps", "Let me know if …", "Would you like me to …", "Is
there anything else …", "Here is a …", "a more detailed breakdown", "check / verify
/ confirm … before publishing".

Instead: give the answer. If you need a decision, ask one direct question.

### No notes to the user left inside the artifact

Banned: "Here's the PR description:" at the top of a PR body; "In this section, we
will discuss …"; "This will include …"; "Delete this section before submission";
HTML comments with drafting instructions; "Feel free to customize"; advice on how to
post the text.

Instead: the artifact holds only what its reader needs.

### No clerk voice in review threads

Banned: "I have carefully reviewed …", "I apologize for any inconvenience this may
have caused", "If there are specific issues, please point them out and I will
address them", asking the reviewer to spell out exactly what to change, calling a
concern "speculation" or demanding "concrete evidence" to brush it off, "let's focus
on improving the content instead", insisting the text reflects your own effort.

Instead: answer the point. Say what you changed, or why you didn't, in a sentence or
two.

### No self-certification

Banned: telling the reviewer the work is right instead of showing it. "This PR is
safe and follows all repo conventions", "fully compliant with the style guide", "all
claims verified", "neutral and well-sourced". This is the model echoing its own
instructions back as content.

Instead: show the evidence (the command and its output) or say nothing.

### No headed essays in comments

Banned: a reply split into titled sections (Markdown headings, bold labels, or
"Conclusion:") when it answers one point.

Instead: a few plain sentences. Use structure only for truly separate cases.

## Commit messages, PR titles, change summaries

These are Wikipedia's "edit summaries". Say what changed in the code or data, not
how good the change is.

Banned:

- Quality claims: improved clarity / flow / readability, refined, enhanced,
  enriched, streamlined, ensured that … adheres to, in compliance with, "for
  consistency", "better structure".
- Reporting what you didn't do: "while preserving existing behavior", "retained",
  "avoided", "aimed to", "kept X intact".
- Stressing that it's backed: "added sourced / verified content", "with independent
  sources", "added tests and docs" with no word on what the tests check.
- "Addressed reviewer feedback" or "per review" as the whole message.
- Listing every parameter or file name when the diff already shows them.
- Bold, emoji, bullets or a "Concise summary:" label in a commit subject.

Good: "Retry HTTP 429 responses up to 3 times".
Bad: "Refactored retry logic for improved resilience and maintainability while
preserving existing behavior".

## Formatting

- Headings in sentence case, not Title Case.
- No H1 title at the top of a comment or PR body; the venue already shows one.
  Never several H1s.
- No bold scattered through prose. Bold at most one phrase the reader must not miss.
- No inline-header lists ("- **Thing:** explanation" on every line). Write
  sentences, or a table when the data really is tabular.
- No emoji in headings or list items.
- Use the markup the venue renders. Don't wrap a whole message in a code block,
  don't put `---` dividers between paragraphs of a comment, and don't use asterisks
  where Markdown doesn't render.

## References, links, placeholders

- Never invent a link, issue or PR number, ADR number, doc section, file path, CLI
  flag, label or function. Check it exists before citing it. The page lists made-up
  policy shortcuts as a tell; a made-up ADR section is the same thing.
- Cite the exact place: a doc section anchor, a file and line. The page's example of
  a useless citation is a book with no page number; a link to a docs home page is
  the same thing.
- Don't cite a source for a claim it doesn't make.
- Strip tracking params from URLs: `utm_source=chatgpt.com`, `utm_source=openai`,
  `utm_source=copilot.com`, `referrer=grok.com`.
- No leaked citation tokens: `contentReference[oaicite:1]`, `↩` footnote arrows.
- Every listed reference is used in the text, and every reference used is defined.
- No placeholders left in: `[Your Name]`, `[Describe …]`, `INSERT_SOURCE_URL_30`,
  `PASTE_URL_HERE`, `SOURCE_PUBLISHER`, `example.com/source1`, `2025-XX-XX`, "(add if
  available)".
- Dates are the real date of the check, never copied from an older run.
- Don't tick a PR checklist box, add a label or write "Tested: yes" for work that
  wasn't done. Wikipedia's version is a draft that arrives with its review template
  already filled in.
- Nothing cut off mid-sentence.

## Don't overcorrect

The page also lists signs that do NOT point to AI writing. Leave these alone:

- Perfect grammar.
- A mix of casual and formal tone.
- A single transition word ("However", "Also").
- Plain "is", "has", "there is".
- Firm statements where they are true: "is the only", "was the first".
- Qualifiers that carry real doubt: "perhaps", "tends to".

## Last pass before sending

1. Each sentence carries a fact the reader can check or act on. Cut the ones that
   only say something is important.
2. Search the draft for the banned phrases above, especially ", ensuring", ",
   highlighting", "not just", "rather than", "In summary", "It's important to note".
3. Every link, number, ID and name was checked, not recalled.
4. No placeholders, no tracking params, no notes to yourself or the user.
5. It reads like a message you'd send a teammate.

## Source

[Wikipedia:Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing),
revision 1385986997 (2026-11-16), licensed
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Written
independently: the banned word and phrase lists repeat the page's short "words to
watch", while every rule, example sentence and explanation here is original.
