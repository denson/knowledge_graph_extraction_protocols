# Deep Research -- SIO JSON-LD Ontology Extractor (Single Document)

**Goal:** From the provided document text, extract an ontology-wide
graph (entities, relations, design patterns) with **broad coverage over
precision**. Output **only** a single JSON-LD object (no prose
before/after).

**Backbone:** SIO. Add domain vocabularies (e.g., SOSA/SSN, OBO) as
**subgraphs**.

## Hard Requirements

1.  **One output:** a single valid **JSON-LD** object.
2.  **Unified graph:** use one `@graph` for all nodes.
3.  **Node typing:** every node is an **instance** (`rdf:Resource`) with
    `@type` set to SIO/other classes as appropriate.
4.  **Resolvable IRIs:** every `@id` is an HTTP(S) IRI. For each novel
    entity (e.g., a dataset or model name), fabricate a deterministic
    IRI using the knowledge base
    `https://knowledgecrystal.com/kg/rs-foundation-models/`, e.g.\
    `https://knowledgecrystal.com/kg/rs-foundation-models/{doc_slug}/{slugified_label_or_hash}`
    (Use this base consistently for the run; stable per document.) If an
    entity corresponds to a term in an existing ontology (e.g., SIO,
    SOSA/SSN, OBO on Ontobee), use that ontology's official IRI (via
    prefix `sio:`, `sosa:`, `ssn:`, `obo:`) instead of fabricating a new
    one.
5.  **Context:** preload the vocabularies below (SIO, SOSA, SSN, OBO,
    etc.) as JSON-LD context prefixes; add more if needed. Use fully
    qualified IRIs (or prefix terms) for common properties in the output
    rather than custom aliases. For example, use `rdfs:label` and
    `rdfs:comment` (not a `"label"` or `"comment"` alias), and use
    `@id`/`@type` keys directly (instead of defining `"id"`/`"type"` in
    the context).
6.  **Domain subgraphs:** assign each node a domain category to indicate
    its vocabulary context. For example, use the property
    `schema:category` with value `"core"` for SIO-aligned nodes
    (default), or `"sosa"`, `"obo"`, etc. for nodes under SOSA or OBO
    domains.
7.  **Disambiguation:** try to disambiguate. If ambiguity remains,
    **create separate nodes** and link them with `sio:is_ambiguous`.
8.  **Text evidence:** include `text_span` (short quote or span) for
    each node **and** for each relation assertion (see `assertions`).
9.  **Provenance:** include `provenance` per node **and** per assertion:
    `{ "source": "<doi|uri>", "timestamp": "<ISO8601>", "model": "<model_name@version>" }`.
10. **Inference hooks:** allow optional OWL axioms (subclass,
    restrictions) under `axioms`.
11. **No free text outside fields.** If uncertain about a value, omit
    it.

## JSON-LD Skeleton (schema)

    {
      "@context": {
        "rdf":  "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
        "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
        "owl":  "http://www.w3.org/2002/07/owl#",
        "xsd":  "http://www.w3.org/2001/XMLSchema#",
        "prov": "http://www.w3.org/ns/prov#",
        "sio":  "http://semanticscience.org/resource/",
        "sosa": "http://www.w3.org/ns/sosa/",
        "ssn":  "http://www.w3.org/ns/ssn/",
        "obo":  "http://purl.obolibrary.org/obo/",
        "schema": "http://schema.org/"
      },
      "@graph": [],
      "natural_language_summary": ""
    }

### Node shape (put these objects inside `@graph`)

    {
      "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/<local_id>",
      "@type": ["sio:Process"],
      "rdfs:label": "Spectral analysis",
      "rdfs:comment": "Short description if the text implies one.",
      "schema:category": "core",
      "text_span": "…exact or lightly trimmed quote underpinning this node…",
      "equivalents": [
        "http://external.org/term/123"   // optional owl:sameAs during conversion
      ],
      "properties": {
        // optional simple attributes; use IRIs as keys if possible
        "sio:hasAttribute": ["https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/attr-1"]
      },
      "relations": [
        {
          "predicate": "sio:hasParticipant",
          "object": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/instrument-1",
          "text_span": "…quote showing this relation…",
          "provenance": {
            "source": "doi:10.xxxx/yyy",
            "timestamp": "2025-08-21T00:00:00Z",
            "model": "GPT-5-Thinking@2025-08"
          }
        }
      ],
      "provenance": {
        "source": "doi:10.xxxx/yyy",
        "timestamp": "2025-08-21T00:00:00Z",
        "model": "GPT-5-Thinking@2025-08"
      }
    }

### OWL axioms block (optional; include once at the **root** as `axioms`)

    {
      "axioms": {
        "classes": [
          {
            "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/SpectralAnalysis",
            "@type": ["owl:Class", "sio:Process"],
            "subClassOf": [
              "sio:AnalyticalProcess"
            ],
            "restrictions": [
              {
                "onProperty": "sio:hasParticipant",
                "someValuesFrom": "sio:Instrument"
              }
            ]
          }
        ],
        "properties": [
          {
            "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/hasOperatingWavelength",
            "@type": ["owl:ObjectProperty"],
            "subPropertyOf": "sio:hasAttribute",
            "domain": "sosa:Sensor",
            "range": "obo:UO_0000016"   // example unit/quantity
          }
        ]
      }
    }

## Extraction Logic (apply in this order)

1.  **Scan & harvest (breadth-first):** list all candidate entities
    (instruments, sensors, samples, datasets, algorithms, workflows,
    analytes, materials, parameters, roles, outcomes). Create nodes for
    each with `label`, `type` guess (SIO first), `text_span`,
    `provenance`.
2.  **Domain typing:**
3.  If sensing/observation → prefer `sosa:Sensor`, `sosa:Observation`,
    `sosa:ObservableProperty`, and mark those nodes with subgraph
    category `"sosa"`.
4.  Otherwise, for generic cases, use SIO-aligned types (subgraph
    `"core"` by default).
5.  Biomedical terms (if explicit) may use OBO IDs (subgraph `"obo"`).
6.  **Relations:** extract process--participant, input/output,
    produces/consumes, measured-of, part-of, depends-on, same-as,
    broader/narrower. Add a `relations[]` entry per assertion **with its
    own** `text_span` + `provenance`.
7.  **Disambiguate:** prefer context to choose one domain; **if still
    unclear**, create separate nodes and connect them both ways using
    `sio:is_ambiguous`.
8.  **Design patterns:** if the doc defines workflows, roles,
    constraints, add **axioms** (class, subclass, property restrictions)
    in `axioms`.
9.  **Summary:** populate `natural_language_summary` (2--6 sentences)
    describing the main entities, domains involved, and key relations.
    No citations here.

## Output Examples (concise)

### (A) Core SIO node + relation

    {
      "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/spectral-analysis",
      "@type": ["sio:Process"],
      "rdfs:label": "Spectral analysis",
      "schema:category": "core",
      "text_span": "We performed spectral analysis of the acquired signals...",
      "relations": [
        {
          "predicate": "sio:hasParticipant",
          "object": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/ms-qtof-1",
          "text_span": "…using a QTOF mass spectrometer…",
          "provenance": {
            "source": "doi:10.1111/example.2024.001",
            "timestamp": "2025-08-21T00:00:00Z",
            "model": "GPT-5-Thinking@2025-08"
          }
        }
      ],
      "provenance": {
        "source": "doi:10.1111/example.2024.001",
        "timestamp": "2025-08-21T00:00:00Z",
        "model": "GPT-5-Thinking@2025-08"
      }
    }

### (B) SOSA sensor node

    {
      "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/hsi-camera-1",
      "@type": ["sosa:Sensor"],
      "rdfs:label": "Pushbroom hyperspectral camera",
      "schema:category": "sosa",
      "text_span": "…a pushbroom HSI camera capturing 400–1000 nm…",
      "properties": {
        "sio:hasAttribute": ["https://knowledgecrystal.com/kg/rs-foundation-models/paper123/attr-wavelength-range"]
      },
      "provenance": {
        "source": "doi:10.1111/example.2024.001",
        "timestamp": "2025-08-21T00:00:00Z",
        "model": "GPT-5-Thinking@2025-08"
      }
    }

### (C) Ambiguity link

    {
      "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/spectral-analysis-ms",
      "@type": ["sio:Process"],
      "rdfs:label": "Spectral analysis (mass spectrometry)",
      "schema:category": "core",
      "text_span": "…peak picking and deconvolution…",
      "relations": [
        {
          "predicate": "sio:is_ambiguous",
          "object": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/spectral-analysis-hsi",
          "text_span": "…the term spectral analysis is used across modalities…",
          "provenance": {
            "source": "doi:10.1111/example.2024.001",
            "timestamp": "2025-08-21T00:00:00Z",
            "model": "GPT-5-Thinking@2025-08"
          }
        }
      ],
      "provenance": {
        "source": "doi:10.1111/example.2024.001",
        "timestamp": "2025-08-21T00:00:00Z",
        "model": "GPT-5-Thinking@2025-08"
      }
    }

### (D) OWL axiom example (root-level `axioms`)

    {
      "axioms": {
        "classes": [
          {
            "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/HSISpectralAnalysis",
            "@type": ["owl:Class", "sio:Process"],
            "subClassOf": ["sio:AnalyticalProcess"],
            "restrictions": [
              { "onProperty": "sio:hasParticipant", "someValuesFrom": "sosa:Sensor" }
            ]
          }
        ]
      }
    }

### (E) Root wrapper with summary (final object shape)

    {
      "@context": { /* as above */ },
      "@graph": [ /* nodes A–C etc. */ ],
      "axioms": { /* optional */ },
      "natural_language_summary": "The document describes spectral analysis workflows across HSI and MS. Key instruments include a pushbroom HSI camera and a QTOF mass spectrometer. Cross-modal use of 'spectral analysis' remains partially ambiguous; nodes are linked via sio:is_ambiguous. Relations capture participants and attributes; optional axioms constrain participants for HSI workflows."
    }

## Validation Checklist (model must self-check before returning)

-   JSON parses; `@context` present; `@graph` is an array.
-   Every node has: `@id`, `@type` (≥1), `rdfs:label`,
    `schema:category`, `text_span`, `provenance`.
-   All `relations[*].object` IRIs point to nodes present in `@graph`
    (or a well-known external IRI).
-   If ambiguity exists → `sio:is_ambiguous` edges exist in **both
    directions** or at least one direction recorded with clear
    `text_span`.
-   If SOSA terms used → `"schema:category": "sosa"` and types drawn
    from `sosa:`/`ssn:`.
-   No narrative text outside `natural_language_summary`.
-   Prefer SIO types for generic objects/processes/attributes; only
    bring in external vocab when needed.
