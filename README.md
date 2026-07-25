# sqlite-memory

**Give your local LLM persistent long-term memory — in one Python file. No vector DB, no server, no framework.**

![sqlite-memory: recalling a fact in a brand-new session](assets/demo.svg)

LLMs forget everything the moment a message scrolls out of the context window,
and everything again when the process restarts. `sqlite-memory` is the smallest
possible fix: embed each fact or chat turn with a local [Ollama](https://ollama.com)
model, store it in a single SQLite file, and semantically `recall()` the relevant
bits before the model answers. The memory is just a `.db` file, so it survives
restarts and is trivial to back up or move.

## Why another memory library?

Projects like Mem0, Zep, Letta and Cognee are powerful but heavy: servers,
vector databases, cloud accounts, many dependencies. `sqlite-memory` trades
features for radical simplicity:

- **One file, standard library only** — `sqlite3`, `urllib`, `json`, `math`.
- **Local & private** — nothing leaves your machine.
- **No infrastructure** — no daemon, no vector DB; the index is a portable `.db`.
- **Readable in one sitting** — easy to audit and fork.

## Quickstart

```bash
ollama pull nomic-embed-text
ollama pull qwen2.5:7b
python example_chat.py     # tell it facts, quit, run again, ask about them
```

### It really persists across restarts (real run)

```text
# session 1 — brand-new memory.db
you> My cat is named Ada and my favorite language is Rust.
bot> Nice to meet you! A cat named Ada and you enjoy Rust — I'll keep that in mind.

# session 2 — a SEPARATE process, same memory.db
sqlite-memory chat — 2 memories on disk.
you> What is my cat's name?
bot> Your cat's name is Ada! 😊
```

> In session 2 the model was never told the name — it was recalled from the
> `.db` file via semantic search and injected into the prompt.

## API

```python
from sqlite_memory import Memory

mem = Memory("memory.db")
mem.remember("The user ships firmware for ESP32 boards", role="fact", tags="work")
mem.recall("what does the user work on?", k=5)   # semantic search -> list of hits
mem.recent(10)                                    # last N memories
mem.forget(tag="work")                            # or forget(id=..) / forget(before_ts=..)
```

## How it works

1. `remember(text)` embeds the text via Ollama's `/api/embeddings` and stores
   `(ts, role, text, tags, embedding)` in SQLite.
2. `recall(query)` embeds the query and ranks every stored memory by cosine
   similarity (computed in Python), returning the top-k.
3. In a chat loop you `recall()` before generating and inject the hits into the
   system prompt, then `remember()` the new turns.

## Configuration (env vars)

| Variable | Default |
|---|---|
| `OLLAMA_URL` | `http://127.0.0.1:11434` |
| `EMBED_MODEL` | `nomic-embed-text` |
| `CHAT_MODEL` (example only) | `qwen2.5:7b` |

## Limits (honest)

- Linear cosine scan — great up to tens of thousands of memories; add an ANN
  index beyond that.
- No automatic **consolidation or selective forgetting** (the hard, unsolved part
  of agent memory): `forget()` is manual/rule-based. Contributions welcome.
- Recall quality is only as good as the embedding model.

## License

MIT

---

## See also
Part of a small collection of **local-first AI** and **ESP32 / maker** tools:

- [sqlite-rag](https://github.com/CapitanaIcoachai/sqlite-rag) — minimal RAG — embeddings + cosine in SQLite, no vector DB
- [ollama-doctor](https://github.com/CapitanaIcoachai/ollama-doctor) — find out why Ollama is slow (CPU offload / VRAM)
- [local-voice-edge](https://github.com/CapitanaIcoachai/local-voice-edge) — ESP32 voice assistant + local STT→LLM→TTS server
- [axs15231b-landscape-lvgl](https://github.com/CapitanaIcoachai/axs15231b-landscape-lvgl) — 3.5" AXS15231B QSPI panel in landscape with LVGL
- [guition-esp32p4-lvgl9](https://github.com/CapitanaIcoachai/guition-esp32p4-lvgl9) — Guition 7" ESP32-P4 + LVGL 9 baseline
- [orcaslicer-cli-cookbook](https://github.com/CapitanaIcoachai/orcaslicer-cli-cookbook) — OrcaSlicer from the command line + fixes

⭐ If this saved you time, a star helps others find it.

## Responsible use & EU AI Act · Uso responsabile · Uso responsable · Usage responsable

**EN —** `sqlite-memory` is a free, open-source **developer tool/library** (MIT), **not** an end-user AI system, and it does **not** bundle any AI model — you connect your own (e.g. your local LLM via Ollama). Outputs from any model may be inaccurate or biased; verify important results (not legal, medical or financial advice). Third-party components (e.g. your local LLM via Ollama) keep their own licenses and usage policies. Under the **EU AI Act (Reg. (EU) 2024/1689)**, transparency and other obligations apply to the *product/deployer* built with this tool, not to this library in isolation; as a free & open-source component it falls under the Act's open-source provisions.

**IT —** `sqlite-memory` è uno **strumento/libreria per sviluppatori** libero e open source (MIT), **non** un sistema di IA per l'utente finale, e **non** include alcun modello di IA (usi il tuo). Gli output dei modelli possono essere errati o distorti: verifica i risultati importanti (non è consulenza legale, medica o finanziaria). I componenti di terzi mantengono le proprie licenze. Ai sensi dell'**AI Act (Reg. UE 2024/1689)**, gli obblighi di trasparenza ricadono sul *prodotto/deployer* costruito con questo strumento, non sulla libreria in sé; come componente libero e open source rientra nelle relative esenzioni.

**ES —** `sqlite-memory` es una **herramienta/biblioteca para desarrolladores** libre y de código abierto (MIT), **no** un sistema de IA para el usuario final, y **no** incluye ningún modelo de IA (conectas el tuyo). Las salidas de los modelos pueden ser inexactas o sesgadas: verifica los resultados importantes (no es asesoramiento legal, médico ni financiero). Los componentes de terceros conservan sus licencias. Según el **Reglamento de IA (UE 2024/1689)**, las obligaciones de transparencia recaen en el *producto/implementador*, no en la biblioteca en sí; como componente libre y de código abierto se acoge a las disposiciones open source.

**FR —** `sqlite-memory` est un **outil/bibliothèque pour développeurs** libre et open source (MIT), **pas** un système d'IA destiné à l'utilisateur final, et n'**inclut aucun** modèle d'IA (vous connectez le vôtre). Les sorties des modèles peuvent être inexactes ou biaisées : vérifiez les résultats importants (ce n'est pas un conseil juridique, médical ou financier). Les composants tiers conservent leurs licences. Selon le **Règlement IA (UE 2024/1689)**, les obligations de transparence incombent au *produit/déployeur*, pas à la bibliothèque elle-même ; en tant que composant libre et open source, il relève des dispositions open source.

### Free course: AI Basics for Developers

Article 4 of the EU AI Act requires anyone building or using AI professionally to have documented
AI competence — no certificate is legally required, just a record. We built a free, voluntary
25-minute course that gives you that record, with a certificate of attendance:
https://developcontroller.com/corso/
No sign-up, no tracking, available in EN / IT / ES / FR.
