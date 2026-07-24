---
layout: project
title: "GraphRAG for Books"
subtitle: "Spoiler-safe character & plot tracking for novels, powered by local LLMs"
date: 2026-07-23
tech_stack: [Python, FastAPI, Neo4j, Qdrant, MariaDB, vLLM, Docker]
github: https://github.com/hanusspi/BookAI.git
demo: null
image: '/img/graphrag-books.png'
featured: true
---

## Overview

A system that reads an EPUB/PDF novel and builds a spoiler-safe knowledge
graph of its characters, relationships, and plot, entirely with locally
hosted LLMs, no external API calls. Every information is locked to the chapter it
was discovered in, so a reader mid-book can look up "who is this character?"
or ask a RAG question and only ever see information valid up to their
current reading position. 

The architecture splits the work by hardware lifecycle and current personal availability: a GPU machine, which is normally used as main desktop does
the expensive one-time extraction (parsing, character/entity extraction,
summarization, embedding) and then can be used normally again; a lightweight always-on server, my N100 based homelab,
holds the permanent graph/vector/relational stores and serves the reading
UI, admin tools, and Q&A, so the GPU box only needs to be on while
ingesting a new book or is asking LLM dependent questions, not while anyone is reading the commonly available information.

## Key Features

- Two-machine, two-lifecycle architecture: a GPU worker for
  ingestion, an always-on homelab server for reading/querying
- Fully local/offline LLM pipeline (vLLM + Qwen2.5, OpenAI-compatible API,
  schema-constrained JSON generation)
- Perfect data safty as no data leaves the machine
- Hybrid retrieval: Neo4j for relationship/graph queries, Qdrant for
  semantic search, MariaDB for structured chapter/character rows
- Spoiler-safe by construction: every character state and chapter summary
  is versioned per chapter (`as_of_chapter`), not just a single mutable record
- Automatic non-story content detection (front/back matter, previews of
  other books, acknowledgments) so real-world names don't litter the
  character graph
- Entity resolution tuned against real score distributions and leans on user input under uncertainty
- Admin correction tooling: merge misidentified duplicate characters, edit
  chapter summaries, adjust relationships, bulk-prune low-relevance side
  characters, review queue for low-confidence extractions
- Resumable, checkpointed ingestion, a crashed or interrupted run picks up
  from the last completed chapter, not from scratch
- Highly asyncronys API, using FastAPI

## Technical Implementation

- **Ingestion pipeline** (per book): parse EPUB/PDF → detect chapter
  boundaries (regex heuristics + an LLM tiebreaker for ambiguous cases) →
  chunk chapter text → **Tier 1** extraction (schema-constrained LLM calls
  pull characters/entities/events per chunk) → merge & deduplicate mentions
  into a persistent character registry via embedding similarity → **Tier 2**
  synthesis (one LLM call per chapter for a spoiler-safe summary, plus
  diff-based character-state regeneration — only characters actually touched
  in a chapter get a new state, so cost scales with what changed, not with
  book length) → **Tier 3** embedding (semantic vectors for chunks, chapter
  summaries, and character entities) → push everything to the homelab's
  MariaDB rows, Neo4j nodes/edges, and Qdrant collections.
- **Structured generation**: every LLM call is constrained to a Pydantic
  schema via vLLM's JSON-schema response format, then re-validated
  server-side — the constraint engine is never trusted blindly, since
  guided decoding can still produce schema-valid-but-referentially-invalid
  output (e.g. a relationship pointing at a character id that doesn't exist).
- **Reading UI**: a chapter slider drives every query's `as_of_chapter`
  cutoff — character panels, the relationship graph, and RAG answers all
  filter through this predicate so nothing from unread chapters is ever
  visible.
- **Resumability**: a local per-book SQLite checkpoint tracks each chapter's
  pipeline phase, so a killed/restarted ingestion job resumes from the last
  completed step instead of reprocessing (and re-billing GPU time for)
  everything from the start.

## Results & Learnings

- **VRAM is the real constraint, not model choice.** Running a 14B model
  instead of 7B for richer character descriptions meant trading away context
  window (16GB card, single GPU) — required careful, empirical tuning of
  `gpu-memory-utilization` and `max-model-len` rather than trusting vLLM's
  defaults or its own suggested fixes, which twice produced a config that
  still wouldn't boot.
- **Two distinct context-overflow bugs, only found by running real books
  end-to-end**: a Tier-2 prompt that embedded a book's *entire* growing
  character roster into every call (fine for a short book, fatal on a
  denser one), and a truncation-retry loop that would double its token
  budget past the server's hard context limit rather than respecting it.
  Neither showed up in unit tests with synthetic short books — both needed
  a real novel's real density to surface.
- **Calibration beats intuition for entity resolution.** The initial
  merge/no-merge similarity thresholds were reasonable-sounding guesses;
  scoring thousands of real candidate pairs and hand-inspecting the
  boundary cases moved them substantially and caught systematic false
  merges (e.g. two side characters with a coincidentally high name-embedding
  similarity) that intuition alone missed.
- **LLM extraction is never good enough to run unsupervised.** Duplicate
  character identities, mislabeled roles, and non-story content (author's
  acknowledgments read as if they were part of the plot) all happen even
  with a bigger model — which is why the admin correction tooling (merge,
  edit, prune, review queue) ended up being as important as the extraction
  pipeline itself, not an afterthought bolted on later.
