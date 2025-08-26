From Papers to Knowledge Graphs: A Deep Research Workflow

The literature overload: Every researcher knows the feeling of drowning in papers. There are thousands of new articles every year in each field, each one packed with details. It’s nearly impossible to read and remember all of them, and even harder to see how ideas connect across studies. We all waste too much time plowing through text to find key concepts and evidence.

Turning papers into graphs: Imagine if instead of walls of text, each paper gave you a structured map of its ideas. That’s the promise of the Deep Research JSON-LD Ontology Extractor. Given one paper, it automatically produces a knowledge graph – basically a network of the paper’s main entities and processes, all linked together. This graph is encoded in JSON-LD (a standard format for linked data), using known ontologies like SIO (Semanticscience Integrated Ontology) and domain vocabularies (e.g. SOSA/SSN for sensors, OBO for biology). In short, each important thing in the paper becomes a node, and each relationship between things becomes an edge.

Entities (nodes): the core items mentioned in the paper, such as samples, instruments, techniques, or outcomes. Each node gets a clear label, a type (using an ontology class), and a globally unique identifier (an HTTP IRI) to avoid confusion.

Relationships (edges): the interactions or associations between entities, like “Instrument X measures Sample Y” or “Process A depends on Model B”. These are captured with standard predicates (e.g. sio:hasParticipant, sio:hasInput) so that the meaning is precise.

Provenance and evidence: crucially, every node and edge carries a snippet of text from the paper (text_span) that justifies it, along with source information (the paper’s DOI or URI, timestamp, and model version). In practice, this means each fact in the graph is directly linked back to its evidence in the paper.

Ontology categories: each node is tagged by category (like “core”, “sosa”, or “obo”) to indicate which vocabulary it came from. This helps group related items (for example, all sensors might be in “sosa”, all sample materials in an “obo” subclass) and keeps the graph organized.

This structured output means the paper’s content is turned into a self-contained graph: exactly one graph per paper, with all nodes in a single JSON-LD object. You end up with a data file where you can programmatically query or visualize the key concepts and their interconnections, rather than wading through paragraphs.

What’s inside the graph

What does the inside of one of these knowledge graphs look like, in human terms? Think of it as a labeled network where each point is a research concept and each link is an explicit relationship. For example, if a paper discusses “spectral analysis” of a sample, there might be a node labeled “Spectral analysis” of type sio:Process. That node could have edges connecting it to a “Sample” node and a “Sensor” node via relationships like “has participant” or “is measured by”. Each connection (and each node) also carries the exact quote from the text that supports it. It might look like a little data record:

A node for Spectral analysis might include:

rdfs:label: “Spectral analysis”

@type: sio:Process

text_span: “We performed spectral analysis on the sample” (a short quote from the paper)

provenance: { source: DOI, timestamp, model } to say exactly which paper and system produced this node.

A relation (edge) like hasParticipant could link “Spectral analysis” to a “Sample” node. This relation entry would also include a text_span quote (“Sample X was used in spectral analysis”) and the same provenance.

In summary, the graph holds entities (nodes), relationships, and provenance. Each node has a unique ID and category, each edge has a defined predicate, and everything is backed by text. This means the knowledge isn’t floating in free text – it’s structured and traceable.

Building a report from the graph

Having the graph is great, but how do we use it? The next step in the workflow is the report protocol, which walks through the graph and writes a narrative overview of the paper (or papers). In practice, the report generation does something like this:

Inventory key elements: First, we list out the main entities and processes found in the graph. For example, we might say “This paper’s graph contains 50 nodes, including 10 instruments, 20 samples, and 15 processes”. These counts and types come straight from the graph data.

Highlight main relations with evidence: Next, we identify the dominant relationships. For each, we quote the exact text_span from the graph and cite its provenance DOI. For instance, we might write “Process ‘Spectral analysis’ has participant ‘Sample A’ (based on the text “…used Spectral analysis on Sample A…”)”. In other words, each statement in the report is linked to a quote and a paper.

Summarize the graph: We then give an overview summary. This might start, “In summary, the KG contains X entities and Y relations, mainly of types SIO:Process (Z instances) and SOSA:Sensor (W instances). The graph highlights that the study’s core workflow involves measuring samples with various instruments and analyzing results.”

Narrative high-level insight: Finally, we write a concise paragraph capturing the essence of the graph. This is like translating the map back into words: “Overall, this study focuses on [key domain] with [main entities] connected by [key processes].” These few sentences use the terms from the graph (again citing sources) and serve as the take-home message.

If we have multiple papers, the report protocol goes further: it will compare graphs. It can point out which entities or methods occur in more than one paper (reinforcing a theme), and which are unique or even contradictory (showing differences). Throughout, every claim in the report is grounded by the graph’s provenance: we always cite the original DOI and quote from the text. The result is a readable, structured summary that weaves together evidence from one or many sources.

Big wins for researchers

Why go through all this trouble? The benefits make it worth trying:

Faster understanding: Instead of reading a paper end to end, you can quickly inspect its graph to see the core concepts and how they fit together. It’s like scanning an outline or a mind-map of the paper, so you grasp the big picture in minutes.

Evidence-grounded insights: Every fact in the report is tied to a snippet of the original text and a citation. You get confident summaries that you can easily trace back to the source. This builds trust in the synthesis, because nothing is stated without proof.

Cross-paper synthesis: When dealing with many papers, the graphs let you merge and compare information at the data level. You can find common entities (e.g. the same instrument used by multiple groups) or spot conflicts in one place. This would be extremely tedious manually. The workflow automates the hard part of aligning and contrasting different studies.

Reusability and scalability: Because the graphs are in a standard format (JSON-LD with ontologies), they can be queried or plugged into other tools. Over time, your research notes become a knowledge base rather than a stack of PDFs. New papers can be added to the graph and instantly connected to existing knowledge.

Imagine no longer missing key details buried in text, or reinventing the wheel because you missed a related method in another paper. Instead, think of having a constantly updating concept map of your research field, built from the literature itself.

Conclusion: Try it out

The Deep Research + JSON-LD extractor workflow represents a new way to handle the literature flood. By turning prose into a structured graph, and then building reports from that graph, it shifts us from manual note-taking to machine-aided synthesis. As fellow researchers, we encourage you to give it a try: feed it a few of your recent papers and see what kind of knowledge graph it produces. You might be surprised how much faster you can summarize and connect ideas. The goal is clear – to make research reading smarter, not harder. The tools are emerging now, and by adopting this workflow, we can spend more time on insight and innovation, and less time buried in documents. The literature might be vast, but with structure on our side, we can navigate it together.