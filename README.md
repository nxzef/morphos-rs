# morphos-rs

**A zero-copy, morphologically-aware tokenization engine for Indic languages — built in Rust.**

---

## The Problem

Large Language Models (LLMs) and their tokenizers are built on Western linguistic patterns.
When processing morphologically rich and agglutinative languages like Sanskrit, Malayalam, Tamil, Kannada, and Telugu, standard tokenizers shatter words into meaningless byte fragments.

```
Input: "രാമോഽഗച്ഛത്" (Sanskrit: "Rama went")

Standard BPE tokenizer (GPT/Claude):
  [രാ] [മോ] [ഽഗ] [ച്] [ഛ] [ത്]  →  6 tokens, zero meaning

morphos-rs:
  [രാമൻ:NOUN:NOM] [ഗമ്:VERB:PAST]  →  2 tokens, full meaning
```

This causes what we call the **Indic Tax**:

- **3x–5x higher token costs** for Indian enterprises using LLM APIs
- **Degraded semantic understanding** — the model never sees meaningful units
- **Massive memory overhead** during inference at scale

---

## The Solution

`morphos-rs` is a low-level, high-performance morphological tokenization engine that understands the grammar of Indic languages — not just their bytes.

Instead of frequency-based merging (BPE), it uses:

- **Linguistic compilation** — Sanskrit/Indic grammar rules (Sandhi, Samasa, inflection) encoded as Finite State Transducers (FSTs)
- **Zero-copy architecture** — Rust's ownership model enables near-zero heap allocation; all token output is `&str` slices into the original input buffer
- **Morpheme-aware splitting** — produces lemma + grammatical features, not arbitrary substrings
- **MCP integration** — plugs directly into the Model Context Protocol as a preprocessing layer for any LLM

---

## Language Coverage

| Language   | Script      | Status         |
|------------|-------------|----------------|
| Sanskrit   | Devanagari  | 🔧 In Progress |
| Malayalam  | Malayalam   | 🔧 In Progress |
| Tamil      | Tamil       | 📋 Planned     |
| Kannada    | Kannada     | 📋 Planned     |
| Telugu     | Telugu      | 📋 Planned     |

---

## Architecture

```
RAW UTF-8 INPUT
      │
      ▼
┌─────────────────────┐
│  STAGE 1            │
│  Script Normalizer  │  → NFC normalization, script detection
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 2            │
│  Grapheme Segmenter │  → Unicode grapheme cluster iteration
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 3            │
│  Sandhi Splitter    │  → FST-based junction detection and reversal
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 4            │
│  Morphological      │  → Suffix stripping + trie lexicon lookup
│  Analyzer           │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 5            │
│  Ambiguity Resolver │  → Probabilistic scoring over parse candidates
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 6            │
│  Token Emitter      │  → Zero-copy MorphToken Iterator
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  STAGE 7            │
│  Output Interfaces  │  → MCP server · Python bindings · JSON API
└─────────────────────┘
```

---

## Token Output Format

Every token produced by `morphos-rs` carries full morphological information:

```rust
MorphToken {
    lemma: &str, // root form — zero-copy slice into input
    pos: PartOfSpeech, // Noun, Verb, Adjective ...
    case: Option<Case>, // Nominative, Accusative, Locative ...
    number: Option<Number>, // Singular, Dual, Plural
    tense: Option<Tense>, // Present, Past, Future ...
    gender: Option<Gender>, // Masculine, Feminine, Neuter
    confidence: f32, // parse confidence score
}
```

---

## Design Principles

**Zero-copy.**
Token output is `&str` slices pointing directly into the input buffer.
No heap allocation per token. Suitable for high-concurrency inference pipelines.

**Memory safe.**
Written in Rust. No null pointers, no buffer overflows, no use-after-free.
The borrow checker enforces these guarantees at compile time.

**Linguistically grounded.**
Sandhi rules are not heuristics. They are formal FST encodings of Paninian grammar —
the most mathematically rigorous linguistic description ever written.

**Embeddable.**
Pure logic, no runtime dependencies, no UI.
Compiles to a native library, WebAssembly, or Python extension.

---

## The Competitive Moat

This project exists at the intersection of three rare capabilities:

```
Systems Engineering          Computational Linguistics
(Rust, zero-copy,       ×   (Paninian grammar, morphology,
 memory safety,              FST theory, Sandhi rules)
 high throughput)
         ×
   Indic Language Domain Knowledge
   (Sanskrit BA, Dravidian script structure,
    Unicode Brahmic block architecture)
```

No existing tool combines all three.
Gérard Huet's Sanskrit Heritage Engine (INRIA) is the closest predecessor —
written in OCaml, academic scope, no production NLP integration.
`morphos-rs` is its successor for the LLM era.

---

## Benchmarks

*Coming in v0.2 — comparing morphos-rs against tiktoken (BPE) and SentencePiece
on Sanskrit and Malayalam corpora from the Digital Corpus of Sanskrit (DCS).*

Metrics reported:
- Token fertility (tokens per word — lower is better)
- Lemmatization accuracy against DCS gold standard (F1)
- Throughput (tokens per second)
- API cost reduction (% token savings on real enterprise workloads)

---

## Roadmap

```
v0.1  Script normalizer + grapheme segmenter       ← current
v0.2  Sandhi FST (Visarga and vowel Sandhi)
v0.3  Sanskrit nominal morphological analyzer
v0.4  Sanskrit verbal morphological analyzer
v0.5  Malayalam agglutination splitter
v0.6  Ambiguity resolver (probabilistic)
v0.7  Python bindings (PyO3 + maturin)
v0.8  MCP server integration
v0.9  Tamil, Kannada, Telugu support
v1.0  Production release + benchmark paper
```

---

## Background and Motivation

Sanskrit and Dravidian languages are among the most morphologically complex languages on earth.
Sanskrit's grammar, described by Pāṇini in the Aṣṭādhyāyī (~500 BCE), is the most complete
formal linguistic description ever written — and it maps directly to finite-state computation.

560 million people speak Dravidian languages.
Their languages are currently second-class citizens in AI infrastructure.
`morphos-rs` exists to fix that — one morpheme at a time.

---

## Academic Context

This project is the engineering foundation of an ongoing research program in
computational morphology for Indic languages.
A technical paper describing the FST architecture and benchmark results
is in preparation for submission to the
*Sanskrit Computational Linguistics Symposium (SCLS)* and *ACL Findings*.

Primary academic influences:
- Gérard Huet — *Lexicon-directed Segmentation and Tagging of Sanskrit* (INRIA)
- Oliver Hellwig — Digital Corpus of Sanskrit (DCS)
- Kimmo Koskenniemi — *Two-Level Morphology* (1983)

---

## License

MIT — free to use, modify, and build on.
If this engine powers your research or product, a citation or mention is appreciated.

---

## Author

Built by a systems engineer and Sanskrit scholar working at the intersection of
low-level Rust programming and Indic computational linguistics.

*MA Linguistics — Kerala University*
*Research focus: Morphologically-aware tokenization for Indic language LLM infrastructure*
