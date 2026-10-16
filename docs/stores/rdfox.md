---
layout: store
store: rdfox
title: RDFox
description: Profile of RDFox 7.7 — Oxford Semantic Technologies' in-memory RDF store, with incremental Datalog materialisation, OWL 2 RL and SWRL, and leaderless replication.
---

## Overview

RDFox is a main-memory RDF store and reasoning engine from **Oxford Semantic Technologies**, a spin-out of the University of Oxford computer science department. The company's September 2026 announcement of RDFox Mantix says Samsung acquired Oxford Semantic Technologies in 2024. The company still publishes the product under its own name. Release **7.7** (29 September 2026) is the current version. The welcome page calls RDFox a centralized store: one process holds the data, and a high-availability group is several processes holding the same data.

The engine is closed source. Creating a server requires a time-limited licence key. The published evaluation agreement allows the software for evaluation only, and the term is the expiry date printed on the key. Commercial use needs a separate licence. Prices are not published. There is no standing free edition with a published triple or core cap.

The same binary runs as an embedded library, through the Java, C and C++ APIs, or as a server with a REST endpoint and a browser console. An Oxford Semantic Technologies paper describes the Personal Data Engine on the Samsung Galaxy S25 as an Android build of RDFox.

## Architecture and scale

- **Memory.** RDFox keeps facts in RAM. The requirements page puts typical fact storage at 45 to 85 bytes per fact, and says to allow another 10 to 100 percent for querying, reasoning and compaction. It warns that performance falls sharply if the operating system pages that memory. When persistence is on, the same page asks for 40 to 60 bytes of disk per triple, including working space for compaction.
- **Identifiers.** The default data store type is `parallel-nn`. The suffixes choose 32-bit (`n`) or 64-bit (`w`) identifiers for resources and for triples. Release 7.7 adds `triple-table-vp`, a vertically partitioned triple table that stores each class as a unary table and each predicate as a binary table. It is triples only, and only for narrow resource identifiers.
- **Transactions.** The transactions page claims ACID, with serializability: any number of read-only transactions, and at most one read/write transaction. A reader sees the snapshot from when its transaction started. Durability is the part you configure. `file` and `file-sequence` persistence write committed state to the server directory. `persistence off` is a supported way to run a purely in-memory server, and that mode drops the store when the process exits.
- **One machine, then copies.** High availability uses `file-sequence` persistence and a server directory that every instance can see. The group is leaderless: any instance accepts writes, and the others apply the new version after they notice it. The documentation calls this eventually consistent. The file-system poll interval defaults to 60 seconds, and instances can also notify each other. Restoring a snapshot in the `highly-available` mode loads the new copy before taking the lock, and the requirements page says to reserve at least as much memory again for that. A shared file system has to meet the criteria in the persistence chapter. The high-availability chapter also describes nominating a single writer when an application cannot accept concurrent writers.
- **Scale.** The product page says RDFox handles billions of triples. That figure is the vendor's claim. The practical limit in the requirements page is how much RAM you give the process.

## Standards

- **SPARQL 1.1 Query.** The querying chapter says RDFox fully supports the query language except two features whose semantics it considers underspecified: the `DESCRIBE` form, and `BNODE` with a string argument. Property paths are implemented. The default `property-paths-cardinality` setting follows the specification, which keeps duplicate solutions for paths that have no `*`, `+` or `?`. An `ASK` query is rewritten as `SELECT WHERE { ... } LIMIT 1`. The endpoint is `/datastores/<name>/sparql`, and the API chapter says it implements the SPARQL 1.1 Protocol for query and update.
- **SPARQL 1.1 Update.** The same chapter says update is fully supported, including insert, delete, load, clear, create, drop, copy, move, add, and grouped updates.
- **Graph Store Protocol.** `/content` follows the Graph Store HTTP Protocol when the request sets exactly one of `default` or `graph`. `PATCH` is not implemented. The documentation says the protocol does not specify its meaning. A request without those parameters uses the RDFox Update Protocol, which can replace facts, rules and axioms, including several named graphs in one call.
- **Federation.** Release 7.5 added `SERVICE`, by POST to the remote endpoint. Variable endpoints are not supported. The endpoint IRI has to be a constant. `http-remotes` can attach authentication, and a batch size, to a named remote. A `SERVICE` block that uses an RDFox extension such as `TT` has to land on another RDFox server.
- **RDF-star and RDF 1.2.** The 7.7 documentation and release notes do not describe quoted triples, triple terms, or SPARQL 1.2.
- **Formats.** Import and export cover N-Triples, Turtle, N-Quads, TriG, OWL 2 Functional-Style Syntax, and proprietary generalized variants of the RDF syntaxes in which any position may be an IRI, a blank node or a literal. Datalog is its own format. JSON-LD and RDF/XML are not in that list.
- **SHACL.** The tuple-table chapter says RDFox supports SHACL Core. Validation is a query against a `SHACL` tuple table, which returns a validation report. It is available while answering queries, and not inside Datalog rules. A commit procedure, which is a SPARQL update run at the end of every read/write transaction, can call that tuple table and reject an update that does not conform. The chapter says the subclass, superclass and type constraints use a limited form of taxonomical reasoning.
- **GeoSPARQL.** There is no GeoSPARQL implementation. `GEODIST` takes two latitude and longitude pairs and returns metres. Lucene data sources can run a geospatial query against a Lucene index. Neither is the OGC standard.
- **ShEx.** Not mentioned in the 7.7 documentation.

## Reasoning

RDFox answers queries by materialising every triple that follows from the facts and the rules, then running SPARQL on that closure. The reasoning chapter says the materialisation is maintained incrementally when facts or rules are added or deleted, so a change does not recompute the closure from scratch. Derived triples cannot be deleted as if they had been loaded. Deleting an explicit triple retracts the consequences that depended on it. An update can also ask for reasoning from scratch. The shell `explain` command prints a proof of a derived fact, down to the explicit triple it came from.

The rule language is Datalog extended with stratified negation-as-failure, stratified aggregation, bind and filter expressions, named graphs, and the built-in functions from the SPARQL chapter. `NOW`, `RAND`, `UUID` and `STRUUID` are excluded from rules because they are not determined by their arguments. A rule set is rejected when negation or aggregation participates in a cycle, when `owl:sameAs` is combined with negation or aggregation, or when a head variable is left unbound. Equality itself is off until the data store is created with it. `noUNA` treats `owl:sameAs` as equality without the unique-name assumption, which is the OWL 2 DL treatment.

OWL 2 axioms can be loaded in Functional-Style Syntax or recovered from triples. RDFox stores axioms it cannot translate. Only axioms that conform to OWL 2 RL become rules, and a mixed axiom may be translated only in part. `owl:imports` is not followed: loading an ontology does not load the ontologies it imports. Reading axioms out of RDF is off by default (`owl-in-rdf-support`). Turning it on, or running `importaxioms` against a named graph, extracts axioms and then the RL rules. A built-in rule set, imported from `<rdfox:TBoxReasoning>`, adds a partial encoding of the RDFS and OWL vocabularies. The page names transitive and reflexive `rdfs:subClassOf`, and the rule that every class is a subclass of `owl:Thing`. RDFox does not offer class subsumption as its own reasoning service. The suggested substitute is to add a fresh instance and query it, or to import that TBox rule set.

SWRL rules are loaded and stored as OWL axioms, from Functional-Style Syntax or Turtle, and then translated to RDFox rules. A documented list of SWRL built-ins has no SPARQL equivalent and is not implemented, including several duration, date and list functions. `ObjectComplementOf` is rejected unless the data store is created with `swrl-negation-as-failure` on, and even then only a complement of a class name is accepted. That option reads the complement as negation-as-failure, which is the closed-world reading, not the open-world reading in the SWRL submission.

## Search and AI

Full-text search is a data source, not an index of the RDF literals. A Lucene data source queries an Apache Lucene index. Lucene 9.6 or newer is installed separately, and RDFox starts a JVM to reach it. A Solr data source does the same for an external Solr collection. Release 7.6 added geospatial search on Lucene data sources. The console extension mechanism that used to load faceted search was removed in an earlier 7.x release.

Semantic similarity is built in. `SEMSIM` embeds two strings and returns a similarity between -1 and 1. It stopped being marked experimental in 7.7. The archive ships a default model in GGUF format, and `RDFOX_EMBEDDING_MODEL_PATH` selects another. Inputs longer than the model's context, or longer than `RDFOX_EMBEDDING_MAX_NUMBER_OF_TOKENS` when that is set, are truncated. Semantic-index tuple tables, added in 7.7, precompute embeddings for strings chosen from the store so the same search can be repeated without embedding the candidates again. A persisted semantic index is tied to the model that built it. An external Java embedder is still marked experimental. If it is configured, it takes precedence over the model file.

RDFox Mantix is a separate product. Oxford Semantic Technologies announced it on 29 September 2026, and the download page lists Mantix 1.0 as a Docker Compose project. The announcement calls it a visual companion for RDFox customers. An LLM helps phrase the question. The answer is computed by RDFox from the organisation's data, rules and ontology. The RDFox 7.7 engine itself does not call a language model.

## Tooling

The endpoint serves the RDFox Console at `/console/`, for querying and for a basic view of a data store. The executable also has a shell, a sandbox mode, and a remote shell against a running server. Bulk import is parallel, using the server's thread pool, and covers RDF, Datalog, and OWL and SWRL axioms. OpenMetrics on `GET /` and on a data store's info resource feeds Prometheus. `GET /health` returns 204.

Official images are `oxfordsemantic/rdfox` and the companion `oxfordsemantic/rdfox-init` on Docker Hub. The init image prepares a server directory, including the first role, so the long-running container can drop every capability. The 7.7 images are x86-64. Native macOS and Linux builds are also published for ARM64. Windows packages are x86-64. The features page lists Windows 10 or higher, macOS 10.14 or higher, and Ubuntu 18.04 or Amazon Linux 2 or higher. The download page raises the ARM macOS floor to macOS 11. A public Helm chart is in `OxfordSemantic/RDFox-helm`.

The in-process APIs are Java (`JRDFox.jar` plus the native library), C and C++. The C and C++ APIs left experimental status in 7.4. The 7.7 documentation does not describe a Python API. Other languages use the REST endpoint.

## Strengths

- Incremental materialisation of a large Datalog fragment, with OWL 2 RL and SWRL compiled into the same rule engine, and a proof for each derived triple.
- Queries read the closure. They do not re-evaluate the rules.
- One binary for an embedded library and a server, with leaderless replication, official container images, and semantic-similarity indexes in 7.7.

## Caveats

- The working set has to fit in RAM. The "billions of triples" figure is the vendor's, and the requirements page says paging the process will hurt.
- Replication is eventual consistency over a shared directory. Any replica can write. There is no sharding, and there is no quorum commit.
- SPARQL 1.1 Query omits `DESCRIBE` and one-argument `BNODE`. `SERVICE` cannot take a variable endpoint. JSON-LD, RDF-star, RDF 1.2, GeoSPARQL and ShEx are absent from the 7.7 documentation.
- OWL reasoning is the RL fragment, on a best-effort translation, and `owl:imports` is not followed. Axioms written as RDF triples are ignored until OWL-in-RDF support or `importaxioms` is turned on.
- A licence key is required to start. The evaluation agreement is not a standing free edition, and the commercial price is not published.

## Sources

<ul class="sources">
<li><a href="https://docs.oxfordsemantic.tech/welcome-to-rdfox.html">Welcome</a> (centralized main-memory store, SPARQL 1.1, Datalog, OWL 2, SWRL) and <a href="https://docs.oxfordsemantic.tech/features-and-requirements.html">features and requirements</a> (APIs, operating systems, memory and disk, licence key, embedding models, Lucene)</li>
<li><a href="https://docs.oxfordsemantic.tech/release-notes.html">Release notes</a>: 7.7 on 29 September 2026 (semantic indexes, <code>SEMSIM</code> no longer experimental), 7.6 on 9 June 2026 (<code>GEODIST</code>, Lucene geospatial search), 7.5 on 4 December 2025 (federated query)</li>
<li><a href="https://docs.oxfordsemantic.tech/querying.html">Querying</a> (SPARQL 1.1 exceptions, update, <code>SERVICE</code> restrictions, <code>GEODIST</code>, <code>SEMSIM</code>)</li>
<li><a href="https://docs.oxfordsemantic.tech/apis.html">APIs</a> (SPARQL protocol, Graph Store Protocol, no <code>PATCH</code>) and <a href="https://docs.oxfordsemantic.tech/import-and-export.html">import and export</a> (formats, parallel import)</li>
<li><a href="https://docs.oxfordsemantic.tech/reasoning.html">Reasoning</a> (incremental materialisation, Datalog extensions, OWL 2 RL, <code>owl:imports</code>, TBox rules, SWRL, explanations)</li>
<li><a href="https://docs.oxfordsemantic.tech/tuple-tables.html">Tuple tables</a> (SHACL Core, vertical partitioning) and <a href="https://docs.oxfordsemantic.tech/transactions.html">transactions</a> (serializability, commit procedures)</li>
<li><a href="https://docs.oxfordsemantic.tech/data-sources.html">Data sources</a> (CSV, PostgreSQL, ODBC, SQLite, Solr, Lucene)</li>
<li><a href="https://docs.oxfordsemantic.tech/high-availability.html">High availability</a> and <a href="https://docs.oxfordsemantic.tech/persistence.html">persistence</a> (<code>file-sequence</code>, eventual consistency)</li>
<li><a href="https://docs.oxfordsemantic.tech/docker.html">Docker images</a> (<code>oxfordsemantic/rdfox</code>, <code>oxfordsemantic/rdfox-init</code>, x86-64 tagging)</li>
<li><a href="https://www.oxfordsemantic.tech/download">Download page</a> (RDFox 7.7, Mantix 1.0, platform floors) and the <a href="https://www.oxfordsemantic.tech/rdfox-evaluation-license">evaluation licence</a> (evaluation only; term is the key expiry)</li>
<li><a href="https://www.oxfordsemantic.tech/rdfox">Product page</a> (billions of triples; SPARQL 1.1, Datalog, SWRL, OWL 2 RL, SHACL) and the <a href="https://www.oxfordsemantic.tech/blog/oxford-semantic-technologies-launches-rdfox-mantix">Mantix announcement</a> (29 September 2026; Samsung acquisition in 2024)</li>
<li><a href="https://ceur-ws.org/Vol-4085/paper5.pdf">Semantic Technology in Your Pocket</a> (Crocker, Horrocks and Nenov; Galaxy S25 Personal Data Engine as an Android build of RDFox)</li>
<li><a href="https://github.com/OxfordSemantic/RDFox-helm">OxfordSemantic/RDFox-helm</a> (public Helm chart). The engine is not published on GitHub.</li>
</ul>
