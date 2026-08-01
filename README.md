# Tesla vs. Rivian — NLP Analysis of 10-K Filings & Earnings Calls

A financial NLP study comparing how Tesla (TSLA) and Rivian (RIVN) talk about risk,
performance, and outlook — across regulatory filings and earnings calls, and across
fiscal years.

The question: **what does each company's own language reveal about how it frames its
business, and how does that framing shift year over year?**

## Data

Three text sources per company:

| Source | What was extracted |
| --- | --- |
| 10-K filings (FY2023, FY2024) | Item 1A (Risk Factors) and Item 7 (MD&A), pulled from each PDF by page range |
| Earnings call transcripts | Most recent quarterly call, split into the Management Discussion and Q&A sections |
| CEO / CFO prepared remarks | Standalone written comments, analyzed separately to compare leadership tone |

### Getting the data

To run the notebook, download the source data from [GitHub Releases](https://github.com/mauriciosutton/tsla-rivn-nlp-analysis/releases):

1. Go to the [Releases page](https://github.com/mauriciosutton/tsla-rivn-nlp-analysis/releases)
2. Download the `data-*.zip` file from the latest release
3. Extract the zip file in the repo root directory — this creates a `data/` folder
4. Run the notebook, which will read from the extracted `data/` directory

Alternatively, the notebook reads from a Google Drive directory (`BASE_DIR`); point it at your own copies of the filings and transcripts to re-run it with different sources.

## Pipeline

1. **Extract** — pull raw text from PDFs by page range (`PyPDF2`), and split transcripts
   on section markers (`MANAGEMENT DISCUSSION SECTION` / `QUESTION AND ANSWER SECTION`)
   with regex. A `visitor_text` callback drops running headers and footers by y-position.
2. **Clean** — multi-pass regex: HTML escapes and tags, markdown links, bracketed
   references, URLs, bullets, timestamps, ticker-date stamps, speaker/title labels,
   parentheticals, and whitespace collapsing. Tables were removed from the 10-K sections
   by hand before analysis.
3. **Analyze** — sentiment, readability, year-over-year similarity, and topic extraction.
4. **Compare** — line results up across company, section, and fiscal year.

## Methods

**Sentiment — two independent lexicons.** The
[Loughran-McDonald master dictionary](https://sraf.nd.edu/loughranmcdonald-master-dictionary/)
scores each document across seven finance-specific categories (positive, negative,
uncertainty, litigious, strong modal, weak modal, constraining) as a percentage of
dictionary-matched words — it exists because general-purpose sentiment lists misfire on
financial language. The Bing Liu opinion lexicon (via NLTK) runs alongside it as a
general-purpose check, producing one averaged score per document, paired with a
negation-word count as a hedging proxy.

**Readability — Gunning Fog Index.** `0.4 × (words ÷ sentences + 100 × complex words ÷ words)`,
where a word is "complex" at 3+ syllables. Transcripts get an extra cleaning pass first,
since speaker labels and timestamps would otherwise distort sentence and word counts.
Syllables are estimated with a vowel-cluster heuristic rather than a phonetic dictionary.

**Year-over-year similarity.** TF-IDF vectors per 10-K section (unigrams and bigrams,
stopwords removed), then cosine similarity between each company's 2023 and 2024 vector,
computed separately for Item 1A and Item 7. Near 1.0 means the wording barely moved.

**Topic extraction.** Top-weighted TF-IDF terms per document as a lightweight topic proxy,
filtered through a custom stopword list — NLTK stopwords plus a 10,000-word common-word
frequency list plus manually identified finance/legal boilerplate — so that generic terms
("company," "revenue") don't crowd out distinctive language. 2023 and 2024 topic sets are
then diffed into added / removed / unchanged.

## Selected findings

- **Net tone, full earnings call:** Tesla `0.0` vs. Rivian `+1.3` pts — Tesla's call was
  sentiment-neutral overall, Rivian's skewed net positive.
- **Tesla's Q&A is the only net-negative section in the entire analysis**, with 89
  negation-word hits against Rivian's 31 — markedly more hedging under analyst questioning.
- **Readability (Gunning Fog):** Tesla `12.2` vs. Rivian `14.9` — Rivian's call used
  denser language. Both sit in the "fairly difficult" range typical of executive speech.
- **Year-over-year stability:** Risk Factors (Item 1A) barely move for either company
  (cosine similarity ≥ 0.98). Item 7 (MD&A) moves more, and Rivian's moved most — only
  3 of its top 10 topics carried over from 2023 to 2024, against 8/10 for Tesla.
- **What each call was about:** Tesla's top TF-IDF keywords — `fsd`, `optimus`,
  `autonomy`, `unsupervised`, `lidar` — a product and technology story. Rivian's — `cogs`,
  `capex`, `ebitda`, `gaap`, `incremental` — a cost-discipline and path-to-profitability
  story. "Autonomy" is the one word both calls share.

Full results, including the charts, are in [`deck/`](deck/).

## Stack

`PyPDF2` · `pandas` · `scikit-learn` (`TfidfVectorizer`, `cosine_similarity`) · `NLTK`
(opinion lexicon, tokenizers, stopwords) · `spaCy` stopwords · Loughran-McDonald master
dictionary · `matplotlib` · `textacy`

## Notes

- Written to run in Google Colab against a Drive-mounted data directory. A few steps
  (writing out intermediate `.txt` files for manual table removal) were run locally in
  Jupyter on purpose, so that re-running the notebook wouldn't overwrite the hand-edited
  files.
- Oversized cell outputs — the ones that rendered full filing and transcript text — have
  been truncated in the committed notebook, both to keep it renderable on GitHub and to
  avoid republishing source transcripts. All code, results, and figures are intact.
- Prompts used while writing some of the regex and plotting code are preserved inline as
  markdown cells, as they were in the original.
