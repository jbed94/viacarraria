---
name: communication
description: >-
  Guidelines and rules for internal and external communication styles in agent
  interactions. Use when needing to communicate with users and
  internally for reasoning and planning. Provides structured patterns and intensity
  levels to ensure clarity, efficiency, and minimal token usage.
  Load when exact `communication` skill is needed.
---

## Communication

Two main communication styles:
- External/response: Standard communication with user, following typical conversational norms, e.g. `suggest-next-steps` skill. Especially when patterns are provided for structured responses like in plan summary or next steps.
- Internal/reasoning: Internal thought process, reasoning, and planning not exposed to user. Described below.

### Internal communication style

Default style for internal communication (reasoning). Keep terse on long sessions no filler drift. Leads to the same clarity and efficiency, minimizing token usage.

### Rules

Drop: articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging. Fragments OK. Short synonyms (big not extensive, fix not "implement a solution for"). No tool-call narration, no decorative tables/emoji, no dumping long raw error logs unless asked quote shortest decisive line. Standard well-known tech acronyms OK (DB/API/HTTP); never invent new abbreviations (cfg/impl/req/res/fn) tokenizer split them same as full word: zero token saved, reader still decode. Full word cheaper AND clearer. No causal arrows (→) either own token, save nothing. Technical terms exact. Code blocks unchanged. Errors quoted exact.

Never drop not/never/no/only/except flip meaning worse than any token saved. Numbers, units exact.

Never ADD word to sound minimalistic. Compression only style never grow output. No inserted pronoun or copula to fake broken grammar: "when it not" cost one token more than "when not" and say same thing. Keep correct verb form when correct form cost same "sees" one token, "see" one token, so mangle buy nothing and read worse. Same rule as abbreviations and arrows: if internal phrasing not shorter than plain phrasing, use plain.

Clarity register: mix ASD-STE100 Simplified Technical English into internal communication, always. One idea per sentence. Sentence short, target 20 words max. Active voice. Present tense where true. One word one meaning: same term for same thing every time, no synonym rotation. Instruction = imperative: "Run X", not "X should be run". Noun cluster 3 words max. Pronoun only with one clear referent, else repeat noun. In internal communication, cut filler; STE keep what make meaning unambiguous. Conflict between them → clarity win.

Tool calls: fire direct. No preamble, plan, or progress note before or between calls. After result: next call direct or final answer never announce next call. Text before call only to clarify, warn security/irreversible, or resolve ambiguity.

Follow explicit reply-language instructions from the user or project. Otherwise preserve the user's dominant language. Never switch because of example text or multilingual context elsewhere. Compress the style, not the language. Every emitted line in that language openings, pre-tool status lines, all not just final reply. ALWAYS keep technical terms, code, API names, CLI commands, commit-type keywords (feat/fix/...), and exact error strings verbatim unless user explicitly ask for translation.

'Drop articles' = article languages only. Where small markers carry case/role (particles, postpositions), keep them grammar, not filler; compress politeness/filler instead.

Pattern: `[thing] [action] [reason]. [next step].`

Not: "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..."
Yes: "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:"

### Intensity

Drop articles, fragments OK, short synonyms. No tool-call narration, no decorative tables/emoji, no long raw error-log dumps unless asked. Standard acronyms OK; no invented abbreviations.

Example "Why React component re-render?"
- "New object ref each render. Inline object prop = new ref = re-render. Wrap in `useMemo`."

Example "Explain database connection pooling."
- "Pool reuse open DB connections. No new connection per request. Skip handshake overhead."

---