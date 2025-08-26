Deep Research -- SIO JSON-LD Ontology Extractor (Single Document)

Goal: From the provided document text, extract an ontology-wide graph (entities, relations, design patterns) with broad coverage over precision. Output only a single JSON-LD object (no prose before/after).

Backbone: SIO. Add domain vocabularies (e.g., SOSA/SSN, OBO) as subgraphs.

Hard Requirements

One input: exactly one input document (PDF or extracted Markdown) per call.

One output: a single valid JSON-LD object corresponding to the input document.

Unified graph: include exactly one @graph containing all nodes.

Node typing: every node is an instance (rdf:Resource) with @type set to SIO/other classes as appropriate (multiple @type entries allowed if needed).

Resolvable IRIs: every @id is an HTTP(S) IRI. For each novel entity (e.g., a dataset or model name), fabricate a deterministic IRI using the knowledge base https://knowledgecrystal.com/kg/rs-foundation-models/. For example:
https://knowledgecrystal.com/kg/rs-foundation-models/{doc_slug}/{slugified_label}.
Use a consistent slug function (e.g., lower-case ASCII, hyphenated, with an appended 8-character hash for uniqueness) for both the document identifier and each label. This ensures IRIs are unique (collision policy: if two entities have the exact same label, suffix one with a short hash). Use the same base for all entities from a given document (stable per document). If an entity corresponds to a term in an existing ontology (SIO, SOSA/SSN, OBO, etc.), use that ontology's official IRI (via prefix sio:, sosa:, ssn:, obo:) instead of fabricating a new one. For reproducibility, include the document’s base IRI once in the output (e.g., as "base_uri" in the root metadata).

Context: preload the vocabularies below (SIO, SOSA, SSN, OBO, etc.) as JSON-LD context prefixes; add more if needed. Use fully qualified IRIs (or predefined prefixes) for common properties in the output rather than custom aliases. For example, use rdfs:label and rdfs:comment (not a "label" or "comment" alias), and use @id/@type keys directly (instead of defining "id"/"type" in the context).

Domain subgraphs: assign each node a domain category to indicate its vocabulary context. For example, use the property schema:category with value "core" for SIO-aligned nodes (default), or "sosa", "obo", etc. for nodes under SOSA, OBO, or other domain-specific vocabularies.

Unique labels and categories: Each node must have exactly one rdfs:label (primary name) and exactly one schema:category (domain tag). Do not provide multiple labels or multiple categories for a single node. (Nodes may have multiple @type values if they belong to several classes.) This constraint ensures consistent naming and simplifies downstream normalization.

Disambiguation: try to resolve ambiguity in context. If ambiguity remains, create separate nodes and link them with sio:is_ambiguous. Use two directional links if appropriate (each direction as a separate relation entry). Each ambiguity link should have its own text_span evidence and provenance (like any relation).

Text evidence: include a text_span (short quote or span) for each node and for each relation assertion. This is a direct snippet from the source document that supports the existence of the node or the relationship. For precise anchoring, you may also provide optional location fields: e.g., page number and char_start/char_end character offsets of that span in the source text. Additionally, an optional quote_hash (SHA-256 hash of the exact text span) can be included to uniquely identify and deduplicate evidence spans.

Provenance: include a provenance object for each node and each relation assertion. Format: { "source": "<doi|uri>", "timestamp": "<ISO8601>", "model": "<model_name@version>" }.

Inference hooks: allow optional OWL axioms (subclasses, restrictions, property definitions) under designated blocks in the output. Organize these into two sections at the root: axioms_local (axioms explicitly asserted by the document’s content) and axioms_inferred (additional generalizations or schema inferred by the extractor). This separation preserves original assertions vs. extractor-introduced schema.

No free text outside fields: do not include narrative text outside of designated fields (e.g., rdfs:comment or natural_language_summary). If uncertain about a value or relation, omit it rather than guessing.

Confidence (optional): If the extraction system supports uncertainty, you may include a numeric confidence score with any node or relation. Use a normalized scale (e.g., 0.0 to 1.0). Only attach a confidence value when the element is grounded in the text but considered uncertain or probabilistic. (If an element is a pure guess with no textual support, it should be omitted entirely rather than assigned a low confidence.)

JSON-LD Skeleton (schema)
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

Node shape (put these objects inside @graph)
{
  "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/<local_id>",
  "@type": ["sio:Process"],
  "rdfs:label": "Spectral analysis",
  "rdfs:comment": "Short description if the text implies one.",
  "schema:category": "core",
  "text_span": "…exact or lightly trimmed quote underpinning this node…",
  "page": 5,                     // optional page number of source
  "char_start": 1024,            // optional character index where span starts
  "char_end": 1087,              // optional character index where span ends
  "quote_hash": "3AC4E...FFD7",  // optional SHA-256 hash of the raw text span
  "confidence": 0.92,            // optional confidence score for this node
  "equivalents": [
    "http://external.org/term/123"   // optional owl:sameAs (external reference)
  ],
  "properties": {
    // optional simple attribute assertions; use IRIs as keys if possible
    "sio:hasAttribute": ["https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/attr-1"]
  },
  "relations": [
    {
      "predicate": "sio:hasParticipant",
      "object": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/instrument-1",
      "text_span": "…quote showing this relation…",
      "confidence": 0.75,          // optional confidence for this relation
      "page": 5,                   // optional page number for this relation evidence
      "char_start": 1100,          // optional char span start for relation evidence
      "char_end": 1150,            // optional char span end for relation evidence
      "quote_hash": "98B21...C34F",// optional SHA-256 hash of relation text span
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

OWL axioms blocks (optional; include at root as needed)

If the document or extractor provides ontology schema details, include them under two top-level keys:

axioms_local – for axioms explicitly asserted by the document (e.g., definitions or restrictions stated in text).

axioms_inferred – for additional axioms or generalizations introduced by the extractor (to enrich or normalize the schema).

Both blocks follow a similar structure (collections of class and property axioms). For example:

{
  "axioms_local": {
    "classes": [
      {
        "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/<doc>/SpectralAnalysis",
        "@type": ["owl:Class", "sio:Process"],
        "subClassOf": ["sio:AnalyticalProcess"],
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
        "range": "obo:UO_0000016"   // e.g., unit/quantity class for nanometer
      }
    ]
  },
  "axioms_inferred": {
    "classes": [
      // (optional) e.g., generalized superclasses introduced by the extractor
    ],
    "properties": [
      // (optional) e.g., additional property definitions introduced by the extractor
    ]
  }
}


(In the above example, SpectralAnalysis is asserted as a subclass of an analytical process with a participant restriction – likely derived from the document. The hasOperatingWavelength property might be an extractor-added formalization of an attribute, linking sensors to a unit of measurement.)

Extraction Logic (apply in this order)

Scan & harvest (breadth-first): List all candidate entities (instruments, sensors, samples, datasets, algorithms, workflows, analytes, materials, parameters, roles, outcomes, etc.). Create a node for each with a rdfs:label, an initial @type guess (prefer SIO classes for generic entities), a supporting text_span, and a provenance record.

Domain typing: Assign domain-specific types where applicable.

If the context suggests sensing/observation, prefer SOSA/SSN terms for types (e.g., use sosa:Sensor for a device, sosa:Observation for an observation event, sosa:ObservableProperty for a measured property) and mark those nodes with schema:category: "sosa".

Otherwise, for general cases, use SIO-aligned types (with schema:category: "core" by default).

If specific biomedical or domain-specific terms are explicitly mentioned and clearly refer to known ontology concepts, you may use corresponding OBO identifiers (and set schema:category: "obo" for those nodes). For example, a known chemical or protein might use an OBO IRI.

Relations: Extract relational assertions between nodes such as process participation, inputs/outputs of processes, produced/consumed outputs, measured-of relationships, part-of hierarchies, dependency links, equivalences, and broader/narrower links. For each relation found, add an entry in the subject node’s relations array with its own predicate (relation type), object (the target node’s IRI), text_span evidence (snippet supporting that relation), and provenance. (Follow the evidence guidelines: every relation should have an evidence span. If a relation is implied but not explicitly stated, it may be better to omit it unless high confidence.)

Predicate canonicalization: Unify relation predicate names to standard ontology IRIs using a mapping. In other words, map any detected relation phrases or synonyms to canonical predicates from SIO/PROV/SOSA whenever possible. For example, "involved in" or "participant" relations should be output as sio:hasParticipant; "used as input" or "consumes" → sio:hasInput; "produces" or "yields" → sio:hasOutput; "was generated by" → prov:wasGeneratedBy. Maintain a central relation dictionary (alias-to-IRI map) to ensure consistent predicate usage and avoid drift over time. For instance:

# relation_dictionary.yaml (alias -> canonical IRI)
participant: sio:hasParticipant
involved: sio:hasParticipant
input: sio:hasInput
consumes: sio:hasInput
output: sio:hasOutput
produces: sio:hasOutput
wasGeneratedBy: prov:wasGeneratedBy
partOf: sio:hasPart
dependsOn: sio:depends_on
sameAs: owl:sameAs
broader: rdfs:subClassOf    # or skos:broader for taxonomy links
narrower: rdfs:subClassOf   # (use with care, taxonomy vs part-of)


(During post-processing, replace any relation keywords/aliases with their canonical IRIs per this dictionary.)

Disambiguate: Use context to decide if an entity could belong to multiple domains or meanings. If a single interpretation cannot be confidently chosen, represent each plausible interpretation as a separate node. Connect ambiguous nodes in both directions using sio:is_ambiguous relations (i.e., node A sio:is_ambiguous node B, and vice versa) with appropriate evidence spans showing the ambiguity in text. Provide text_span and provenance for each ambiguity link relation as well.

Design patterns: If the document explicitly defines higher-level schema (e.g., it introduces new classes, workflows, roles, or constraints), capture these in the output as OWL axioms. Use the axioms_local block for any class definitions or property restrictions that the document itself asserts. Additionally, if the extractor generalizes or infers new schema elements (like abstract classes or properties not explicitly stated), include those under axioms_inferred. (Both are optional; include them only if such patterns are identified. Each axiom should ideally trace back to a source statement or be flagged as an inference.)

Summary: Finally, populate the natural_language_summary field with 2–6 sentences of high-level summary describing the main entities identified, the domains involved, and key relationships among them. This summary should be in plain English and capture the essence of the extracted graph. Do not include any citations or JSON in the summary. It should read as a brief narrative of the document’s content in terms of the knowledge graph.

Output Examples (concise)
(A) Core SIO node + relation
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
      "page": 3,
      "char_start": 250,
      "char_end": 282,
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

(B) SOSA sensor node
{
  "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/hsi-camera-1",
  "@type": ["sosa:Sensor"],
  "rdfs:label": "Pushbroom hyperspectral camera",
  "schema:category": "sosa",
  "text_span": "…a pushbroom HSI camera capturing 400–1000 nm…",
  "page": 4,
  "char_start": 210,
  "char_end": 243,
  "properties": {
    "sio:hasAttribute": ["https://knowledgecrystal.com/kg/rs-foundation-models/paper123/attr-wavelength-range"]
  },
  "provenance": {
    "source": "doi:10.1111/example.2024.001",
    "timestamp": "2025-08-21T00:00:00Z",
    "model": "GPT-5-Thinking@2025-08"
  }
}


(The above sensor node refers to an attribute node attr-wavelength-range for the numeric wavelength range. That attribute node would include the actual values and unit, as shown in example (F) below.)

(C) Ambiguity link
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


(In this example, two "Spectral analysis" nodes exist – one for a mass spectrometry context and one for a hyperspectral imaging context – because the term was used ambiguously. They are linked via sio:is_ambiguous with evidence.)

(D) Numeric measurement attribute node
{
  "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/attr-wavelength-range",
  "@type": ["sio:Attribute"],
  "rdfs:label": "Wavelength range",
  "schema:category": "core",
  "text_span": "400–1000 nm",
  "properties": {
    "rdf:value": { "@value": 1000, "@type": "xsd:integer" },
    "unit": "obo:UO_0000016"   // unit for nanometer (OBO Unit Ontology)
  },
  "provenance": {
    "source": "doi:10.1111/example.2024.001",
    "timestamp": "2025-08-21T00:00:00Z",
    "model": "GPT-5-Thinking@2025-08"
  }
}


This attribute node represents a numeric measurement (a wavelength range) with a value and a unit. The rdf:value holds the numeric literal (here 1000 as an example endpoint of the range, typed as xsd:integer), and unit points to an ontology term for the unit (in this case, OBO’s term for “nanometer”). In practice, you might include both range endpoints or represent the range as two values; the key point is to use typed numeric literals and controlled unit IRIs (OBO or QUDT) for measurement data.

(E) OWL axiom blocks example (root-level axioms_local and axioms_inferred)
{
  "axioms_local": {
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
  },
  "axioms_inferred": {
    "properties": [
      {
        "@id": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/hasOperatingWavelength",
        "@type": ["owl:ObjectProperty"],
        "subPropertyOf": "sio:hasAttribute",
        "domain": "sosa:Sensor",
        "range": "obo:UO_0000016"
      }
    ]
  }
}


(In this example, the axioms_local block defines a class HSISpectralAnalysis as a subclass of an analytical process, with a restriction that it has a sensor participant – presumably directly asserted by the document (e.g., an experimental method definition). The axioms_inferred block defines a property hasOperatingWavelength with a domain and range – an ontology addition that the extractor introduced to capture a measurement attribute. In a real output, these blocks would be included only if such information is available.)

(F) Root wrapper with summary (final object structure)
{
  "@context": { /* as above */ },
  "base_uri": "https://knowledgecrystal.com/kg/rs-foundation-models/paper123/",  // document base IRI
  "@graph": [ /* nodes A–D, etc. */ ],
  "axioms_local": { /* optional, as needed */ },
  "axioms_inferred": { /* optional, as needed */ },
  "natural_language_summary": "The document describes spectral analysis workflows in both hyperspectral imaging (HSI) and mass spectrometry (MS) domains. Key instruments include a pushbroom HSI camera capturing 400–1000 nm spectra and a QTOF mass spectrometer. The term 'spectral analysis' is used in multiple contexts, which the extraction represents as distinct nodes linked by an ambiguity relation. Important relations such as participants and attributes are captured, and an optional set of ontology axioms defines spectral analysis as a subclass of analytical process with appropriate restrictions."
}


(This final wrapper shows how everything comes together. The @graph contains all the extracted nodes (examples A through D, etc.), and the optional axioms_local/axioms_inferred blocks appear at the root if any schema-level axioms were extracted. The natural_language_summary provides a human-readable synopsis of the knowledge graph content. The base_uri field records the base IRI used for this document, ensuring ID reproducibility.)

Validation Checklist (the model must self-check before returning)

JSON is well-formed and parses. An @context is present. @graph is an array of nodes.

Every node has: an @id, at least one @type, exactly one rdfs:label, exactly one schema:category, a text_span, and a provenance object.

All relations[*].object IRIs either point to another node present in @graph (internal link) or are well-known external IRIs (e.g., an ontology term).

If ambiguity exists in the document, sio:is_ambiguous edges are included (preferably in both directions between the ambiguous nodes, or at least one direction) with clear text_span evidence for each ambiguity link.

If SOSA/SSN terms are used for a node, then that node’s "schema:category" is "sosa" (or appropriate) and its @type is drawn from the SOSA/SSN vocabulary.

No narrative text appears outside of natural_language_summary (i.e., all information is in structured fields).

Prefer SIO types for generic objects/processes/attributes; only introduce external vocabulary terms when necessary for domain-specific concepts.

If numeric measurement data is captured, the values are typed (using xsd: data types) and units are represented with proper IRIs (e.g., OBO UO terms or QUDT units).

Any provided "confidence" fields are numeric (e.g., between 0.0 and 1.0) and reflect real uncertainty estimates (not arbitrary strings).

If present, page is an integer page number and char_start/char_end are integer indices that correspond to the text_span. An included quote_hash should be the correct SHA-256 hash of the exact text span (for auditability).