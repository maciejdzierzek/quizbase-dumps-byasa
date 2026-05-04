# QuizBase BY-SA enrichment dump

This repository publishes **QuizBase's enrichment** for trivia records derived from 6 CC-BY-SA-licensed upstream sources. It exists to satisfy § 3(b) ShareAlike obligations of CC-BY-SA-4.0 — every modification we make to BY-SA records must be made available under the same (or compatible) license.

**License:** [CC-BY-SA-4.0](LICENSE) (3.0 sources upgraded per § 3(b)(1) "later version" + § 4(b) compatibility).

**Last update:** 2026-05-04T16:52:02.344Z — 481,654 records across 9 files.

---

## What's in here

Only **our derivative work** for the 6 BY-SA sources:

| Modification | Predicate (in QuizBase DB) | Files |
|---|---|---|
| **Polish translations** of EN questions | our translator (not native) | `<source>__translations-pl.jsonl.gz` |
| **English text refinements** of upstream questions | Pipeline E rewrite | `<source>__refinements-en.jsonl.gz` |
| **Quizifications** — Q&A → multiple-choice + boolean | Pipeline C (E + C combined) | `<source>__quizified.jsonl.gz` |

**Total: 481,654 records, 9 files.** See [`manifest.json`](manifest.json) for SHA-256 + sizes per file.

---

## What's NOT in here

- ❌ **Upstream originals** — those are at the source (links below).
- ❌ **Enrichment for non-BY-SA sources** (entityq, mintaka, creak, qasc, webq) — MIT/CC-BY licensing lets us keep them under our terms, available via the QuizBase API.
- ❌ **Categorization, subcategories, tags, curated topics** — independent metadata under our license, not derivative of upstream questions.
- ❌ **Display labels, embeddings, difficulty calibration, search index, distractor validation flags** — value-add layer, only in API.

The QuizBase API is at https://quizbase.runriva.com (post-launch).

---

## Schema

All record types share these fields:

```jsonc
{
  "source": "opentdb",
  "source_id": "opentdb:abc123",          // original ID — JOIN key with upstream dataset
  "language": "pl" | "en",
  "license": "CC-BY-SA-4.0",
  "license_url": "https://creativecommons.org/licenses/by-sa/4.0/",
  "attribution": "Original: <upstream attribution>, <upstream license>. <Modification>: QuizBase, CC-BY-SA-4.0.",
  "disclaimer": "Provided AS IS, no warranties of any kind. See LICENSE."
}
```

### `<source>__translations-pl.jsonl.gz` — TranslationRecord

```jsonc
{
  ...base,
  "language": "pl",
  "type": "multiple" | "boolean" | "text_input",
  "text_pl": "Co to jest?",
  "correct_answer_pl": "Tak",
  "incorrect_answers_pl": ["Nie", "Może", "Nigdy"]
}
```

### `<source>__refinements-en.jsonl.gz` — RefinementRecord

Diff between upstream original and our refined version. `incorrect_answers_*` are `null` for `type='boolean'` or when refinement didn't touch distractors.

```jsonc
{
  ...base,
  "language": "en",
  "type": "multiple" | "boolean",
  "text_original": "what was the name of the ship that sank in 1912 ?",
  "text_refined": "What was the name of the ship that sank in 1912?",
  "correct_answer_original": "Titanic",
  "correct_answer_refined": "Titanic",
  "incorrect_answers_original": ["Lusitania", "Britannic", "Olympic"] | null,
  "incorrect_answers_refined": ["Lusitania", "Britannic", "Olympic"] | null
}
```

### `<source>__quizified.jsonl.gz` — QuizifiedRecord

For sources where upstream provided Q&A pairs (no distractors). We reformulated the question text + selected/generated distractors. Discriminated union by `type`:

```jsonc
// type='multiple'
{
  ...base,
  "language": "en",
  "type": "multiple",
  "text_original": "when did the us enter world war 1",   // raw upstream query
  "text_quizified": "When did the United States enter World War I?",
  "correct_answer": "1917-04-06",
  "incorrect_answers": ["1914-07-28", "1916-04-06", "1918-11-11"]
}

// type='boolean'
{
  ...base,
  "language": "en",
  "type": "boolean",
  "text_original": "did mario lopez win dancing with the stars",
  "text_quizified": "Did Mario Lopez win Dancing with the Stars?",
  "correct_answer": "True" | "False",
  "incorrect_answers": []
}
```

---

## How to use

```bash
# Download manifest + a file
curl -fsSL https://github.com/maciejdzierzek/quizbase-dumps-byasa/raw/main/manifest.json | jq
curl -fsSLO https://github.com/maciejdzierzek/quizbase-dumps-byasa/releases/latest/download/opentdb__translations-pl.jsonl.gz

# Read records
gunzip -c opentdb__translations-pl.jsonl.gz | head -1 | jq

# Verify integrity
shasum -a 256 opentdb__translations-pl.jsonl.gz
# Compare against manifest.json: .sources[].files[] | select(.name == "...") | .sha256
```

To rejoin with upstream originals, use the `source` + `source_id` fields. For example, `opentdb:abc123` matches the same record at https://opentdb.com.

---

## Sources

| Source | Author | Upstream URL | Upstream license | What we add |
|---|---|---|---|---|
| **opentdb** | Open Trivia Database (PixelTail Games) | https://opentdb.com | CC-BY-SA-4.0 | Polish translations |
| **opentriviaqa** | uberspot | https://github.com/uberspot/OpenTriviaQA | CC-BY-SA-4.0 | Polish translations + EN text refinements |
| **arc** | Allen Institute for AI | https://allenai.org/data/arc | CC-BY-SA-4.0 | Polish translations |
| **kqa-pro** | Cao et al. 2022 | https://github.com/shijx12/KQAPro_Baselines | CC-BY-SA-4.0 | Polish translations + EN text refinements |
| **nq-open** | Lee et al. 2019, Google Research | https://github.com/google-research-datasets/natural-questions | CC-BY-SA-3.0 → 4.0 | Polish translations + EN quizifications (Q&A → multiple-choice) |
| **mkqa** | Apple ML Research | https://github.com/apple/ml-mkqa | CC-BY-SA-3.0 → 4.0 | EN quizifications (multiple + boolean) |

**MKQA Polish translations are NOT in this dump** — Apple's upstream MKQA is multilingual and includes native Polish (`translator='native'`), so we only re-import Apple's PL records without modification. Apple already distributes those under CC-BY-SA-3.0.

---

## Removal requests

If you are an upstream rights holder and want a record removed:

> Email **maciej.dzierzek@gmail.com** with the `source` + `source_id` of the affected record(s).

We will publish a new dump version with the record(s) excluded within 14 days. GitHub Releases are immutable per-version (assets can be deleted, but the version's history remains), so the latest published version always reflects current removals. Full DMCA procedure will be at https://quizbase.runriva.com/data post-launch.

---

## License

This dump is licensed under [Creative Commons Attribution-ShareAlike 4.0 International (CC-BY-SA-4.0)](LICENSE). When you redistribute or build on this data, you must:

1. Give appropriate credit (per-record `attribution` field).
2. Link to the license.
3. Indicate modifications.
4. License your derivative work under CC-BY-SA-4.0 or a [compatible license](https://creativecommons.org/share-your-work/licensing-considerations/compatible-licenses/).

The 3.0 → 4.0 upgrade for `nq-open` and `mkqa` records is permitted under § 3(b)(1) of CC-BY-SA-3.0 ("later version" clause) + § 4(b) of CC-BY-SA-4.0 (compatibility).

---

## Contact

- License questions, attribution corrections, translation issues, removal requests: **maciej.dzierzek@gmail.com**
- Bug in dump format / regeneration cadence: open a GitHub issue
- Found this useful? https://quizbase.runriva.com (post-launch)

---

_Generated by [QuizBase](https://quizbase.runriva.com) — multilingual trivia API. Plan: [PREP/plans/17-1.6-data-byasa-dump.md](https://github.com/maciejdzierzek/quizbase) (in main repo)._
