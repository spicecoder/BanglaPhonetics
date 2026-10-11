# Bangla Phonetic Keyboard (Browser IME)

A single-file, zero-dependency Bengali (Bangla) phonetic input method that runs entirely
in the browser. Type roman letters, watch them convert to Bangla **live**, and see the
exact Unicode code points being produced as you type.

**Why this exists:** phonetic Bangla typing is already solved (Avro, OpenBangla, Gboard).
What those tools don't show you is *what Unicode your keystrokes produce*. This keyboard
makes the invisible visible — it is a demonstrator and teaching tool as much as a keyboard.

## Features

- **Live transliteration** — the current word re-renders on every keystroke.
- **Live code-point display** — e.g. typing `ny` shows `U+09A8 U+09CD U+09AF` and the fused glyph ন্য.
- **Caret-safe editing** — click or arrow anywhere. Backspace edits the *roman buffer* of the
  live word (pops the last typed letter), not raw output characters.
- **Automatic conjuncts** — consecutive consonants fuse via the virama (hasanta) with no special key.
- **Inherent-vowel release** — the apostrophe `'` releases a consonant so the next consonant
  does not fuse (this is how কথা, জন্য, নমস্কার are typed).
- **On-screen keyboard** — physical rows plus special keys (দির্ঘ ই, ঞ, য়, ড়, ঢ়, ঙ, ং, ঃ, ঁ, hasanta, release).
- **Practice word strip** — click a Bangla word to hear it spoken and see the roman keys to
  type; the chip turns green when the word appears in your text.
- **Speech button** — hear the current word via the browser's speech synthesis (a Bengali
  voice is used when the device provides one).
- **No build, no dependencies, no network** — one HTML file, works offline, view-source readable.

## Run it

Open `bengali-phonetic-keyboard.html` in any modern browser. That's it.
To publish on GitHub Pages, rename it to `index.html` and enable Pages on the repo.

## How it works

The engine is a two-stage pipeline evaluated synchronously on every keystroke:

1. **Tokenise (longest match).** The roman buffer is scanned against a rules table;
   multi-letter codes (`kh`, `sh`, `bh`, `OU`, `rri`…) are tried before their single-letter
   prefixes, so `th` is থ and never ত+হ.
2. **Combine (state machine).** A single flag `pend` tracks whether the last output was a
   consonant "awaiting a vowel":
   - vowel after consonant → attach the **matra** (vowel sign) instead of the independent vowel
   - consonant after consonant → insert **U+09CD (hasanta)** first, then the consonant
   - apostrophe → clear `pend` (the consonant keeps its inherent vowel; nothing is emitted)
   - sign (ং ঃ ঁ) → attach directly, no hasanta

The rendered document is a sequence of **segments** — plain-text segments and *live word*
segments (each holding its roman buffer). This is what makes editing caret-safe:
backspace inside a live word pops its last roman letter and re-renders; once a word is
committed with Space it becomes plain text and is edited code-point by code-point.
Deleting part of a committed word's output is one-way (the roman buffer is gone) — the
remainder stays as text.

**Rendering is the font's job.** The keyboard only emits code-point sequences such as
`ন U+09A8 + ্ U+09CD + য U+09AF`; the OS text shaper (HarfBuzz / Uniscribe / CoreText)
decides whether to draw the fused conjunct ন্য or a visible hasanta. A font missing the
ন্য glyph will show ন্‌য — that is a font limitation, not a keyboard bug.

## Typing guide

### The one rule that matters

> **Two consonants in a row fuse.** Type `'` between them when they must not.

| You want | Type | Because |
|---|---|---|
| ন্য (fused) | `ny` | স + ্ + য — engine inserts the hasanta |
| জন্য (not fused) | `j'ny` | `'` releases জ; then ন্য fuses |
| কথা | `k'thA` | `'` releases ক; থ gets the া |
| নমস্কার | `n'm'skAr` | ন and ম keep inherent vowels; স্ক fuses |
| শুভেচ্ছা | `shubheccha` | `cch` → চ + ্ + ছ |

### Vowels

Independent forms appear at the start of a word or after the release `'`; after a consonant
the **matra** (vowel sign) form is attached automatically.

| Type | Independent | Matra | Sound (approx.) |
|---|---|---|---|
| `a` / `A` | আ | া | a |
| `i` | ই | ি | i |
| `I` | ঈ | ী | ī (long i) |
| `u` | উ | ু | u |
| `U` | ঊ | ূ | ū |
| `e` | এ | ে | e |
| `o` / `O` | ও | ো | o |
| `OU` | ঔ | ৌ | ou |
| `OI` | ঐ | ৈ | oi |
| `rri` | ঋ | ৃ | ri (Sanskrit loanwords) |

### Consonants

| Type | Letter | | Type | Letter | | Type | Letter |
|---|---|---|---|---|---|---|---|
| `k` | ক | | `T` | ট | | `p` | প |
| `kh` / `q` | খ | | `Th` | ঠ | | `ph` / `f` / `F` | ফ |
| `g` | গ | | `D` | ড | | `b` | ব |
| `G` / `gh` | ঘ | | `Dh` | ঢ | | `bh` / `B` / `v` | ভ |
| `ng` | ঙ | | `N` | ণ | | `m` | ম |
| `c` | চ | | `t` | ত | | `y` | য |
| `ch` | ছ | | `th` | থ | | `Y` | য় |
| `j` | জ | | `d` | দ | | `r` | র |
| `jh` | ঝ | | `dh` | ধ | | `l` | ল |
| `NY` | ঞ | | `n` | ন | | `R` | ড় |
| | | | | | | `rh` | ঢ় |
| `s` | স | | `S` | ষ | | `sh` | শ |
| `x` | ক্ষ | | `z` | ষ | | `h` | হ |

### Signs & special keys

| Type | Produces | Notes |
|---|---|---|
| `M` | ং | anusvara — attaches directly, no hasanta |
| `H` | ঃ | visarga |
| `Q` | ঁ | chandrabindu |
| `'` | *(nothing)* | **release**: consonant keeps inherent vowel; next consonant won't fuse |
| `` ` `` | ্ | visible hasanta (vowel-killer) inserted manually |

## Our take on Bengali vowels

Bengali is an **abugida**: every consonant letter carries an inherent vowel, by default
/ɔ/ ("o"-ish). Written vowels are optional modifiers. This creates the central design
problem of any Bengali IME — when does a consonant keep its inherent vowel, and when does
it give it up?

Our answers, in order of importance:

1. **The apostrophe is a first-class citizen.** It does exactly one thing: mark that the
   preceding consonant *keeps* its inherent vowel. This mirrors the linguistics directly —
   `'` is how you spell the /ɔ/ that the abugida already implies. Without it, কথা, জন্য,
   সবার, নমস্কার are untypable in a purely positional scheme. Most "broken output" in
   phonetic typing traces back to missing this distinction.
2. **`o` means the ও-kar (ো), `a` means the া-kar.** Bengali orthography writes the /ɔ/
   sound two ways: as inherent (কথা) or explicitly (শুভো). We type the explicit one with
   `o` and the implicit one with `'` + nothing. It is the typist's job to know which the
   word uses — no IME can infer it.
3. **Digraphs for diphthongs.** `OU` → ঔ/ৌ and `OI` → ঐ/ৈ, because Bengali diphthongs are
   single vowel phonemes with dedicated letters and matras; splitting them into `o`+`u`
   would attach two matras to one consonant, which Bengali orthography forbids.
4. **ঋ gets a digraph `rri`.** A Sanskrit vowel with no roman equivalent; digraph beats
   stealing `rI` (which must remain available for র + ী, e.g. প্রীতি).
5. **Signs are not consonants.** ং ঃ ঁ attach directly with no hasanta and break the
   pending-consonant state, matching their Unicode category as spacing marks.

## Bengali Unicode quick reference

Bengali is encoded in the Unicode block **U+0980–U+09FF** (encoded since Unicode 1.1,
sharing its architecture with ISCII). Key ranges:

| Range | Contents |
|---|---|
| U+0981 | ঁ chandrabindu (nasalisation) |
| U+0982–U+0983 | ং anusvara, ঃ visarga |
| U+0985–U+0994 | independent vowels অ আ ই ঈ উ ঊ ঋ এ ঐ ও ঔ |
| U+0995–U+09B9 | consonants ক … হ (with ঙ ঞ ণ ন in the retroflex/dental slots) |
| U+09BC | ় nukta |
| U+09BE–U+09CC | vowel signs (matras) া ি ী ু ূ ৃ ে ৈ ো ৌ |
| U+09CD | ্ hasanta / virama — the vowel-killer and conjunct joiner |
| U+09D7 | ৗ (au length mark, used in ৌ) |
| U+09DC–U+09DD | ড় ঢড় (ৰ-series with nukta, precomposed) |
| U+09DF | য় (yo-phala variant / antastha ya) |
| U+09E6–U+09EF | digits ০–৯ |
| U+09CE | ৎ khanda-ta (final t, precomposed) |
| U+09E0–U+09E1 | ৠ ৡ vocalic letters (Sanskrit) |
| U+09F1–U+09F3 | ৰ (ra), ৲ ৳ (historic/rupee signs) |
| U+09BD | ঽ avagraha |

**Anatomy of a conjunct — ন্য:**

```
ন  U+09A8  consonant "na" (with inherent vowel)
্  U+09CD  virama: kills the inherent vowel of the preceding consonant
য  U+09AF  consonant "ya"
→ rendered as the single conjunct glyph ন্য (ya-phala)
```

The hasanta is the heart of the script: consonant + hasanta + consonant is *the* way
Bengali conjuncts are encoded. There is no precomposed "ন্য" code point (only a few
conjuncts like ড় ঢ় ৎ have dedicated characters). Joining behaviour can be tuned with
U+200D (zero-width joiner, force conjunct) and U+200C (zero-width non-joiner, force
visible hasanta). Whether a given sequence renders fused depends on the font's glyph
inventory — the encoding is always the same.

References: *The Unicode Standard*, Chapter 12 (South Asian Scripts); the Unicode Indic
FAQ; Unicode Bengali code charts.

## Practice mode — learning to write with it

The strip under the editor holds seventeen common words, from মা to নমস্কার. Click one to
**hear it spoken** and see the roman keys to type. When the word appears in your text the
chip turns green and you hear a spoken "খুব ভালো". The 🔊 button repeats the current word —
useful for checking whether what you typed *sounds* like what you meant.

For someone who speaks Bangla but doesn't yet write it, the loop is:
**hear → type → see → hear again.** Speaking skill is converted into writing skill one word
at a time, and every word in the strip is typeable with the scheme above (each was verified
against the engine — note how many use the `'` release: `n'dI` → নদী, `b'i` → বই,
`j'l` → জল, `bAMlA` → বাংলা).

Speech quality depends on the device: Android Chrome and Windows generally ship a Bengali
(bn-BD / bn-IN) voice; other platforms may fall back to a default voice with a foreign
accent or stay silent. The typing features work regardless.

**Pronunciation overrides.** TTS engines sometimes read a word flat or with the wrong
vowel. The script has a small `SPEAK_AS` table that respells a word the way it should be
*spoken*: জন্য is spoken as "জন্ নো" — the inserted space forces a syllable beat, putting
depth on the "jon-" part — at a slightly slower rate. Add your own entries there in the
same form (Bangla orthographic → Bangla spoken).

## Limitations

- Typing a letter with the caret mid-word **appends at the end of that word** (standard
  IME behaviour; the roman buffer is edited at word granularity).
- Deleting part of a *committed* word is one-way: the roman buffer is discarded.
- The ruleset is deliberately compact (Avro-compatible in spirit, not in completeness).
- No candidate window, no dictionary, no word prediction — by design.
- The caret is forced to the logical end of edited words.

## Changelog

- **1.1** — practice word strip + speech button; `SPEAK_AS` pronunciation overrides.
  Fixed a crash on typing (`fullText` vs `full` name mismatch): keystrokes updated the
  document but the re-render never reached the textarea.
- **1.0** — caret-safe segment document model, live code points, on-screen keyboard.

## License

MIT. Use it, embed it, teach with it.
# BanglaPhonetics
