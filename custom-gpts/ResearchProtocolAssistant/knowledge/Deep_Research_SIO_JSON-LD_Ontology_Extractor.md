# Deep Research — SIO JSON-LD Ontology Extractor (Single Document)

## Goal
From the provided document text, extract an ontology-wide graph (entities, relations, design patterns) with **broad coverage over precision**.  
Output only a single **JSON-LD object** (no prose before/after).

**Backbone:** SIO. Add domain vocabularies (e.g., SOSA/SSN, OBO) as subgraphs.

---

## Hard Requirements

- **One input:** exactly one input document (PDF or extracted Markdown) per call.  
- **One output:** a single valid JSON-LD object corresponding to the input document.  
- **Unified graph:** include exactly one `@graph` containing all nodes.  
- **Node typing:** every node is an instance (`rdf:Resource`) with `@type` set to SIO/other classes as appropriate (multiple `@type` entries allowed).  
- **Resolvable IRIs:**  
  - Every `@id` is an HTTP(S) IRI.  
  - For novel entities, fabricate deterministic IRIs using:  
    ```
    https://knowledgecrystal.com/kg/rs-foundation-models/{doc_slug}/{slugified_label}
    ```  
  - Slug function: lowercase ASCII, hyphenated, append 8-character hash for uniqueness.  
  - Use ontology IRIs (SIO, SOSA/SSN, OBO) when available instead of fabricating.  
  - Include `"base_uri"` once at root metadata.  

- **Context:** preload SIO, SOSA, SSN, OBO, etc. as JSON-LD context prefixes. Use fully qualified IRIs (`rdfs:label`, `rdfs:comment`) not aliases.  

- **Domain subgraphs:**  
  - `schema:category = "core"` for SIO nodes (default).  
  - `"sosa"`, `"obo"`, etc. for domain-specific vocabularies.  

- **Unique labels & categories:**  
  - One `rdfs:label` per node.  
  - One `schema:category` per node.  
  - Multiple `@type` allowed.  

- **Disambiguation:**  
  - If unresolved, create separate nodes.  
  - Link with `sio:is_ambiguous` in both directions, with evidence & provenan_
