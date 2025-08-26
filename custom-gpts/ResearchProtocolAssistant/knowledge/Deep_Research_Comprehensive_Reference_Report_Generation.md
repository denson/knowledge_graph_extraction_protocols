# Deep Research Comprehensive Reference Report Generation (Ontology-Graph Enhanced)

## Input Expectations
- **Documents:** One or more scientific papers (PDF or extracted Markdown text). Each document should be treated as a primary source to analyze.  
- **Knowledge Graphs:** For each document, a corresponding JSON-LD knowledge graph extracted from that document by the SIO Ontology Extractor (via the Ontology Extractor Protocol). Each KG must have been generated from exactly one document, and must follow the extractor’s schema (nodes with `@id`, `rdfs:label`, `schema:category`, `text_span`, `provenance`, etc.).  
- **Multi-Document Input:** Optionally, multiple (document + KG) pairs, each derived from a single source document. The protocol integrates insights across all provided sources (see **Multi-Document Integration** below).  

## Knowledge Graph Analysis
- **Entity and Process Inventory:** Examine the KG to identify its core elements. List the major *entities* and *processes* (e.g. samples, instruments, measurements, analytical steps) present. Note each node’s `rdfs:label` and type. Count the total number of nodes and note dominant classes (e.g. number of `sio:Process` vs `sosa:Sensor` nodes).  
- **Dominant Relationships:** Identify central relations. For example, which processes have many participants or outputs. Summarize key relation types (e.g. `sio:hasParticipant`, `sio:hasInput/output`, `sio:dependsOn`). Use the KG’s `text_span` fields as evidence: quote or paraphrase them and cite the KG’s `provenance.source` DOI to ground each claim.  
- **Domain Categories:** Highlight each node’s `schema:category` value. For instance, nodes tagged “core” align with SIO, “sosa” with SOSA/SSN, or “obo” with OBO vocabularies. Summarize how many nodes fall under each category (core vs sosa vs obo, etc.).  
- **Ambiguities:** Detect any pairs of nodes linked by `sio:is_ambiguous`. For each such pair, explain why the term is ambiguous in context. Include the `text_span` evidence from the KG that triggered the ambiguity and cite its provenance.  
- **Provenance Evidence:** Whenever you describe a KG node or relation in the report, cite the *source* from its `provenance` block as the authority (typically a DOI or URI). For example, if you refer to a process identified in the KG, include the reference to its provenance DOI. Use the `text_span` from the KG to quote key evidence.

## Graph Summary and Ontology Mapping
- **Graph Overview:** Provide a concise summary of the graph’s content. Report the total counts of entities and relations (e.g. “The KG contains *X* entities and *Y* relations”). Highlight dominant types (e.g. “Most nodes are instances of *sio:Process* (Z examples) and *sosa:Sensor* (W examples)”). Describe the main processes or workflows the graph captures.  
- **High-Level Insights:** Write a 2–6 sentence narrative that encapsulates the essence of the KG, similar to the extractor’s `natural_language_summary`. Mention key domains involved, main entities, and how they relate.  
- **Ontology Mapping (multi-KG):** If multiple documents/KGs are given, suggest how their graphs could be merged or mapped onto a unified schema. Identify common classes or properties across graphs. Sketch a combined ontology structure or conceptual map: group related entities into clusters (e.g. “Sample/Material cluster”, “Instrument/Device cluster”, “Observation/Measurement cluster”).  
- **Conceptual Clusters:** Based on the KG content, recommend conceptual groupings. For example, cluster by semantic category (“All sensors and instruments together”, “All analytical processes together”), by function (inputs vs outputs), or by domain (biological vs physical). Highlight any natural partitions suggested by the KG’s labels or relationships.  

## Multi-Document Integration
- **Cross-Document Synthesis:** When multiple document+KG pairs are provided, integrate findings across them. You may organize the report by theme or by document, but always indicate the origin. For each insight (e.g., a key entity or claim), note which paper or graph it came from, and cite each source separately.  
- **Overlap and Commonalities:** Compare the graphs to find overlapping entities or relations. If the same concept (label/IRI) appears in more than one KG, note it as a shared element. Highlight these overlaps and use them to reinforce recurring themes.  
- **Divergences and Contrasts:** Point out any divergent claims or unique elements. If one paper’s KG contains a concept that another does not, or if they disagree (e.g. different attribute values, or one calls a process “analysis” while another calls it “measurement”), note the discrepancy.  
- **Aggregate Insights:** Where appropriate, produce combined insights. For instance, if each document has a small KG, you might merge their counts or lists (e.g. “Across all sources, we identified **N** unique processes and **M** unique instruments.”). Use bullet lists or tables to compare side-by-side (e.g. Entity types vs Document).  
- **Clear Referencing:** Throughout, maintain clarity about provenance. For each fact or figure, indicate the specific source (document and/or its KG) it comes from, ensuring each source is distinctly cited. Do not assume a merged knowledge graph; treat each (document + KG) pair independently.

## Citation and Synthesis Guidelines
- **Synthesis:** The report must synthesize information from both the text and the ontology graph. Do not treat the KG as standalone output; instead, use it to support and enrich the narrative drawn from the paper’s content.  
- **Citing Graph Content:** When citing graph-derived information, use the `provenance.source` DOI as the citation.  
- **Evidence from KG:** You may quote the `text_span` strings from the KG entries as evidence. Include them in quotation marks and cite the provenance DOI.  
- **Formatting:** Follow the original Deep Research style: structured Markdown with clear headings, short paragraphs (3–5 sentences), and bullet lists for enumerations.  
- **Tone and Detail:** Maintain a thorough, scholarly tone. Explain concepts clearly, define any specialized terms, and connect ideas logically. Be comprehensive: background, key terms, methods/results (from the KG and text), comparisons, examples, limitations, etc.  
- **Citation Conventions:** Always use the `【source†Lx-Ly】` format for citations.  
- **Grounding Claims:** Every factual claim or insight should be grounded in either the text or the KG’s provenance.

---

