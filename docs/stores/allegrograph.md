---
layout: store
store: allegrograph
title: AllegroGraph
description: Profile of Franz AllegroGraph — commercial RDF store with RDFS++ and OWL 2 RL reasoning, FedShard partitioning, multi-master replication, and the GraphTalker agent.
---

## Overview

AllegroGraph is a commercial RDF store from **Franz Inc.** of Lafayette, California. Franz's product page dates version 1.0 to 2004, and the server is still written in Allegro Common Lisp. Release 9.1.0 (15 October 2026) is the current version. From this release the only build Franz ships is the SMP build, so each server process can use every core on the machine.

The engine is closed source. Examples, Docker build files, and the Java, Python and other client libraries are on GitHub under `franzinc`.

## Editions

Franz publishes three editions, and the difference it documents is how many triples you may store:

| Edition | Triple cap | Licence key |
|---|---|---|
| Free | Under 5 million | No |
| Developer | Under 50 million | Yes |
| Enterprise | Unlimited | Yes |

The Docker guide says the same thing in one line: running without a licence works, and the store is limited to 5 million triples. Developer and Enterprise prices are not published. An Enterprise evaluation is arranged through Franz sales.

The FedShard tutorial builds its sample data to stay under the free cap, so partitioning is part of the binary you download, not a separate Enterprise package. The replication guides likewise describe the feature without an edition gate.

Hosted options:

- **AllegroGraph.cloud** has a free hosted server. The download page lists hosted Enterprise as coming soon.
- Free and Enterprise images are sold on the AWS and Azure marketplaces. Those are customer-run virtual machines, in the same sense as the other marketplace listings in this survey.

## Architecture and scale

- **Storage.** AllegroGraph is a disk-based quad store. Triples go into a set of triple indexes, and strings can also be indexed for freetext search. A repository can hold triples that are not legal RDF (a literal as subject, for example). SPARQL applies only to the RDF subset.
- **Transactions.** The server implements ACID with snapshot isolation. A transaction sees the store as it was at the last commit or rollback. Commit returns only after the transaction log write has finished. There is no triple locking, so a concurrent transaction can delete a triple another transaction is reading. Two transactions that disagree about repository metadata (a datatype mapping, a freetext index) are resolved by letting the first commit win and rejecting the second.
- **Security.** Triple attributes and security filters restrict which triples a user or role can see, down to individual statements. This is separate from ordinary catalog and repository permissions.
- **Replication.** Two mechanisms are documented. Classic replication copies a primary to transactionally consistent replicas. Multi-master replication (MMR) keeps repositories in separate data centres in sync, with a shared cluster UUID as of 9.1.0. A second version of the vector store, added in 9.1, puts embeddings in the string table so an MMR cluster can share them.
- **FedShard.** For data that will not fit one repository, triples are partitioned by a key (and an optional secondary key) into shards that can sit on different servers. A query is rewritten and run in parallel on every shard. Each shard can see one or more shared knowledge-base repositories, and it cannot see the other shards' instance data. The partition therefore has to match the questions the application asks: Franz's own example puts every triple about one patient on the same shard, and the medical ontology in the shared knowledge base. Shards can be defined while the server is running, and a shard that grows too large can be split. The older distributed-repository system, configured through `agcluster.cfg`, was removed in 9.0. Moving an old distributed store to FedShard means exporting before the upgrade and loading again after it.
- **Scale.** Franz's product page says the disk store scales to billions of triples. The trillion-triple figure is a 2011 load-and-query demonstration (1,009,690,381,946 triples), still cited by Franz as a milestone. It has not been reproduced here.
- **Deployment.** Native packages are x86-64 Linux (RPM and a tar archive). macOS and Windows are supported through Docker, and Windows also through WSL. The official image is `franzinc/agraph` on Docker Hub. A separate image and a Kubernetes guide cover MMR clusters. Each open repository needs 422 MiB of shared memory, more if you build vector stores. As of 9.1 each HTTP worker is an operating-system process, and `HTTPMaxWorkers` caps how many start.

## Standards

- **SPARQL 1.1.** Query, Update and federated query (`SERVICE`) are supported, including JSON, CSV, TSV and XML results. The HTTP endpoint speaks the SPARQL protocol. Franz documents two gaps: there is no SPARQL 1.1 Service Description, and there is no Graph Store HTTP Protocol. Franz says SPARQL Query and Update can do what the Graph Store Protocol does. Entailment regimes are not implemented as specified; inference goes through RDFS++ or the materializer instead. Numbers and dates are stored in encoded form, so non-canonical lexical forms such as `"+1"^^xsd:integer` and `"1"^^xsd:integer` collapse, and DAWG tests that require those forms to round-trip fail.
- **Query engines.** SBQE is the default. MJQE uses less memory and is the engine Franz points at for path expressions and FedShard. Select it with `PREFIX franzOption_engine: <franz:mjqe>`.
- **RDF-star.** Support is off until you enable it on a repository, in WebView or with `agtool repos create --rdf-star-mode`. Franz says the implementation passes the community-group test suite, and also says it is experimental because the specification it follows is still the draft. Turtle, TriG, N-Triples, N-Quads and NQX carry quoted triples once the mode is on. Two limits are documented: you cannot turn the mode off while any quoted triple remains, including unreferenced "garbage" quoted triples, and quoted triples are counted in the repository size. The 9.1 RDF-star document still describes that specification as a draft that might become part of RDF 1.2, and it does not describe RDF 1.2 triple terms. The product page, separately, lists SPARQL 1.2 among the standards AllegroGraph supports. The two pages do not say the same thing, and the implementation notes are the more specific of the two.
- **Formats.** JSON-LD, N-Quads, N-Triples, RDF/XML, TriG, TriX and Turtle, plus extended N-Quads for triple attributes. JSON-LD 1.1 conformance is not stated. Rapper can convert formats the loader does not read.
- **SHACL.** `agtool shacl-validate`, a Lisp function, a REST service and SPARQL magic properties all run validation. The SHACL document covers SHACL Core and SHACL-SPARQL, including SPARQL-based constraints and custom SPARQL constraint components. Franz notes that the SPARQL components are much slower than the Core components, and that `owl:imports` is not followed when the shapes graph is built. ShEx is not mentioned in the documentation.
- **GeoSPARQL.** The 9.1 GeoSPARQL page lists WKT and GeoJSON literals, the Simple Features relations (as magic properties and as functions), the Egenhofer and RCC8 relation families, and the usual non-topological functions (`distance`, `intersection`, `envelope`, and so on). Franz describes this as much of the OGC specification, and does not name a conformance class. Geohash indexes and a `nearby` property are Franz extensions. A separate n-dimensional geospatial type system, older than the GeoSPARQL work, encodes coordinates into typed literals and is queried with its own magic properties. Temporal interval relations are another set of magic properties, not a W3C standard.

## Reasoning

AllegroGraph has two reasoners.

**RDFS++** runs at query time and does not write triples. It covers `rdf:type`, `rdfs:subClassOf`, `rdfs:subPropertyOf`, `rdfs:domain`, `rdfs:range`, `owl:sameAs`, `owl:inverseOf`, `owl:SymmetricProperty` and `owl:TransitiveProperty`. An optional restriction module adds `owl:hasValue`, `owl:someValuesFrom` and `owl:allValuesFrom`. Franz describes it as fast and predictable, and incomplete relative to a description-logic reasoner.

The **materializer** writes an OWL 2 RL subset into a graph you choose. It is aimed at stores that change rarely, or at queries that need more than RDFS++. Changing the store means running it again. Rule groups can be selected (`class`, `property`, `restriction`, `same-as`, `values-from`, or `all`). The documentation's rule table marks a number of OWL 2 RL rules as not implemented, including several consistency checks. A bulk mode is faster and skips the transaction log while it runs.

**Prolog** is a second query language, sent through the same HTTP endpoint as SPARQL (`queryLn=prolog`). Magic properties, which you can define in Lisp, are how freetext, geospatial, temporal, social-network and SHACL operations show up inside SPARQL.

## Search and AI

- **Full-text search.** Freetext indexes over chosen predicates, with boolean expressions, wildcards and phrases. A Solr integration is documented separately.
- **Vector stores.** Native vector storage arrived in 8.2. You point `agtool llm index` at strings in a repository and an embedding model. Version 2, in 9.1, stores the embeddings in the string table so MMR replicas share them. `agtool export` and `agtool load` now carry the binary vector file as well as the triples. A `llm:similarity` magic property returns the dot product of two embeddings.
- **GraphTalker.** The main feature of the 9.0 line, upgraded to 2.0 in 9.1. It is a chat interface that uses Claude to read a repository's SHACL shapes, try SPARQL, and return an answer. You can attach images, PDFs, text and CSV to a message. Per-repository notes (ontology, query patterns, conventions) are versioned context files. Saved queries and sessions live in a metagraph repository on the server GraphTalker was started from. The Anthropic endpoint URL can be overridden, which is how a proxy that holds the API key is configured. GraphTalker can also be driven from the Python client.
- **MCP.** The Python client can start an MCP server (`mcp-allegro-graph` or `mcp-agraph`).

## Tooling

- **WebView** is the admin and query UI. The older "Traditional WebView" was unlinked in 9.0 and removed in 9.1.
- **Gruff** is Franz's graph browser. A browser version is built into WebView. Desktop builds exist for macOS, Windows and Linux, and a public demo needs no install.
- **agtool** loads, exports, queries, validates SHACL, defines FedShard repositories, materializes, and manages replication and GraphTalker.
- **Clients.** Java (Sesame and Jena), Python, and the native Lisp API. The HTTP interface is the path for everything else. The download archive also points at older community clients for C#, Ruby, Perl and Scala.

## Strengths

- FedShard partitions writes by a key and runs the query on every shard, with ontologies kept in shared knowledge bases.
- Two reasoning modes on one server: predictable query-time RDFS++, and an OWL 2 RL materializer for data that sits still.
- Geospatial and temporal queries are first-class, both through GeoSPARQL functions and through Franz's own n-dimensional and interval indexes.
- GraphTalker, native vectors and an MCP server are in the current release, on top of a store that already has SHACL.
- A free edition with no time limit, and the same binary the paid editions use. The FedShard tutorial is written to fit inside the free cap.
- Multi-master replication across data centres, including a vector-store layout meant to replicate.

## Caveats

- Proprietary, with no public price list. The free edition holds 5 million triples, which is enough to learn the API and not enough for a production graph.
- The server runs natively on x86-64 Linux. Other operating systems go through Docker, WSL, a marketplace VM, or AllegroGraph.cloud. Hosted Enterprise is listed as not yet available.
- RDF-star is opt-in and still labelled experimental in the implementation notes. The product page lists SPARQL 1.2, and those same notes do not describe RDF 1.2 triple terms. Enabling RDF-star is effectively one-way once quoted triples exist.
- There is no Graph Store HTTP Protocol. SPARQL 1.1 entailment regimes are not implemented. OWL coverage is a subset, and the materializer does not implement every OWL 2 RL rule.
- FedShard only works when each query can be answered from one shard plus the shared knowledge bases. A query that has to join instance data across shards is the wrong shape for it.
- The trillion-triple result is a 2011 vendor demonstration on specialised hardware. Treat it as a historical claim.

## Sources

<ul class="sources">
<li><a href="https://franz.com/agraph/support/documentation/release-notes.html">AllegroGraph 9.1.0 release notes</a> (documentation updated 13 October 2026) and the <a href="https://franz.com/ftp/pri/acl/ag/">download archive</a> (9.1.0 dated 2026-10-15)</li>
<li><a href="https://franz.com/agraph/downloads/">Download page</a>: packages, Docker, AllegroGraph.cloud, AWS and Azure Marketplace</li>
<li><a href="https://franz.com/agraph/support/documentation/docker.html">Docker and AllegroGraph</a> (5 million triples without a licence; <code>franzinc/agraph</code>)</li>
<li><a href="https://franz.com/agraph/ag4start.lhtml">Customer portal</a> (licence key required above 5 million triples)</li>
<li><a href="https://franz.com/agraph/support/documentation/agraph-introduction.html">Introduction</a>: ACID, FedShard, RDFS++ predicates, load formats, freetext, replication</li>
<li><a href="https://franz.com/agraph/support/documentation/sparql-overview.html">SPARQL overview</a> and <a href="https://franz.com/agraph/support/documentation/sparql-1.1-release-notes.html">SPARQL implementation notes</a></li>
<li><a href="https://franz.com/agraph/support/documentation/rdf-star.html">RDF-star support</a></li>
<li><a href="https://franz.com/agraph/support/documentation/shacl.html">SHACL</a></li>
<li><a href="https://franz.com/agraph/support/documentation/geosparql-doc.html">GeoSPARQL support</a> and <a href="https://franz.com/agraph/support/documentation/geospatial-nd.html">n-dimensional geospatial</a></li>
<li><a href="https://franz.com/agraph/support/documentation/materializer.html">Materialized reasoner</a> (OWL 2 RL subset)</li>
<li><a href="https://franz.com/agraph/support/documentation/dynamic-cluster-setup.html">FedShard setup</a> and <a href="https://franz.com/agraph/support/documentation/dynamic-cluster-tutorial.html">tutorial</a></li>
<li><a href="https://franz.com/agraph/support/documentation/replication.html">Replication</a></li>
<li><a href="https://franz.com/agraph/support/documentation/graphtalker.html">GraphTalker</a> and <a href="https://franz.com/agraph/support/documentation/vector-dbs-and-rag.html">vector databases and RAG</a></li>
<li><a href="https://franz.com/agraph/support/documentation/python/changes.html">Python client release history</a> (GraphTalker client, MCP server) and <a href="https://github.com/franzinc">github.com/franzinc</a></li>
<li><a href="https://allegrograph.com/press_room/franzs-allegrograph-sets-new-record-1-trillion-rdf-triples/">Franz, 16 August 2011: 1 trillion RDF triples</a></li>
<li><a href="https://allegrograph.com/products/allegrograph/">AllegroGraph product page</a> (billions of triples; version 1.0 in 2004; the page also lists SPARQL 1.2)</li>
</ul>
