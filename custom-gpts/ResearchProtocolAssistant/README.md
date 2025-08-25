# Research Protocol Assistant

Purpose
- Helps design, critique, and iterate research protocols.
- Produces stepwise procedures, checklists, risk/assumption logs, and JSON-LD/SIO-aligned artifacts when applicable.

How to deploy in ChatGPT (Custom GPT)
1) Copy `prompt.md` into the GPT's Instructions.
2) Upload files in `knowledge/` (JSON-LD graphs, protocol specs, templates).
3) Add action(s) from `actions/` (OpenAI Custom GPTs → Actions → Import schema).
4) Save and test using `evals/smoke-tests.md`.

Repository hygiene
- Edit `prompt.md`; commit with concise messages.
- Version via tags (e.g., `RPA-v0.1.0`).
- Keep sensitive data out of the repo.

Suggested knowledge to upload from this repo
- `Deep_Research_SIO_JSON-LD_Ontology_Extractor.md`
- Selected graphs from `graphs/` relevant to target domains.


