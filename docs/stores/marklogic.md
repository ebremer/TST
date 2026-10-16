---
layout: store
store: marklogic
title: MarkLogic Server
description: Profile of MarkLogic Server 12.1.0 — Progress Software's multi-model document database, with a triple index, query-time rules, and forest clustering.
---

## Overview

MarkLogic Server is a multi-model document database from **Progress Software**. RDF is not the primary model. Documents are XML or JSON, and a triple index covers managed triple documents plus triples embedded in other documents. Template Driven Extraction can project further triples out of document values. The concepts guide says SPARQL queries against stored triples run entirely against that index.

Release **12.1.0** (10 September 2026) is the current innovation line. MarkLogic 11.3 is the long-term support line. Its latest patch, 11.3.7, was released the same day. The 11.3 chapter calls 11.3 the first LTS release and describes a four-year Active phase, then a two-year Sunset phase. This profile records 12.1.0.

The engine is closed source. The free developer licence includes every feature the product page lists, including the universal index, ACID transactions, semantics, SQL and ODBC, clustering, and high availability and disaster recovery. It allows up to 1 TB, lasts six months, and can be renewed. It is for development, not production. Express and Academic licences are no longer issued. A licence that includes the Semantics option is required for SPARQL. The developer licence includes that option. The vector index documentation says the same option covers the vector index.

Progress leads the DB-Engines RDF ranking by score. That rank is why this store was queued. It is not a published triple count.

## Architecture and scale

- **Documents, then triples.** Managed triples are stored in documents whose root is `sem:triples`. Unmanaged triples are `sem:triple` elements inside ordinary XML or JSON documents. A template can index values from a document as triples without copying them into a triple document. SPARQL reads all three. SPARQL Update writes only managed triples. On a new installation from MarkLogic 9 onward, the triple index and the collection lexicon are on by default.
- **Forests.** A database is a set of forests, and each forest is served by one host. The cluster is shared-nothing: evaluator nodes run the request, and data nodes keep the forests, including journaling and recovery. The rebalancer moves documents between forests. Managed triples are documents, so they move with that partition. A query spans the forests in the database. This is a split of one database, not a full copy of the database on every host.
- **Failover and replication.** Local-disk failover keeps one or more replica forests, on other hosts, transactionally in step with the primary. Shared-disk failover mounts the same forest on a standby host. Database replication copies journal frames from a primary cluster to a replica cluster, which is the disaster-recovery path. The concepts guide treats failover as high availability inside a cluster and database replication as recovery across clusters. The developer licence includes both. Production use needs a paid licence.
- **Transactions.** Each REST request is one auto-commit transaction unless the caller opens a multi-statement transaction and later commits or rolls it back. The application guide describes multi-version concurrency control: a query reads a timestamp, and an update takes readers/writers locks as it touches documents. The developer licence page and the Progress ACID datasheet call these transactions ACID, including multi-document, multi-statement, and XA transactions. Data nodes journal forest updates.
- **Scale.** No current Progress page states a triple-count capacity. The developer licence stops at 1 TB. Enterprise capacity is not given a triple cap in the pages read for this profile.

## Standards

- **SPARQL 1.1 Query.** The concepts guide says MarkLogic supports SPARQL 1.1. The semantics guide says it supports the syntax and capabilities of SPARQL 1.1, and that a query against stored triples runs against the triple index. SELECT, ASK, CONSTRUCT and DESCRIBE are documented, as are property paths, aggregates, and `EXISTS`, `NOT EXISTS` and `MINUS`. `sem:sparql` runs a query from XQuery or JavaScript. `GET` and `POST` on `/v1/graphs/sparql` are the SPARQL 1.1 Protocol endpoint. A `cts` query can restrict which documents contribute triples.
- **SPARQL 1.1 Update.** `sem:sparql-update` and `POST /v1/graphs/sparql` with `application/sparql-update` run graph update and graph management. Only managed triples are affected. Triples embedded in documents, and triples projected by a template, stay as they are. Change those by updating the document, or by changing or removing the template.
- **Graph Store Protocol.** `GET`, `PUT`, `DELETE`, `POST` and `HEAD` on `/v1/graphs` implement those methods of the SPARQL 1.1 Graph Store HTTP Protocol when the request sets exactly one of `default` or `graph`. The collection lexicon has to be enabled. DELETE removes managed triples only. If a graph also holds embedded triples, those remain and the graph remains. The PUT page limits a body to 512 MB for RDF/XML and 64 MB for other formats.
- **Federation.** The MarkLogic 12 semantics guide does not document SPARQL 1.1 Federated Query or `SERVICE`.
- **RDF-star and RDF 1.2.** The 12.1 release notes and the 12 semantics guide do not describe quoted triples, triple terms, or SPARQL 1.2. `xdmp:shortestPath` is a SPARQL magic property for a path through the triple index. It is not RDF-star.
- **Formats.** `sem:rdfParse` and `sem:rdf-serialize` accept `ntriple`, `nquad`, `turtle`, `rdfxml`, `n3`, `trig`, `rdfjson` and `triplexml`. `rdfjson` is RDF/JSON. JSON-LD is not in that list.
- **GeoSPARQL.** The MarkLogic 11 geospatial notes, still current in the 11 guide, say MarkLogic implements a subset of OGC GeoSPARQL functions: `geof:sfWithin`, `geof:sfContains`, `geof:sfOverlaps`, `geof:sfCrosses`, `geof:sfEquals`, `geof:sfDisjoint`, `geof:sfIntersects` and `geof:sfTouches`. The published signatures take `cts:region` values, which are MarkLogic regions, not a GeoSPARQL geometry literal vocabulary. No conformance class is stated. Document geospatial search (points, polygons, GeoJSON, GML, KML) is a separate index and is not that standard.
- **SHACL and ShEx.** Neither appears in the MarkLogic 12 semantics guide.

## Reasoning

Inference is backward-chaining and runs at query time. MarkLogic does not materialise a closure. The inference chapter ships three rulesets, each as an optimized file and a fuller file, under the install `Config` directory: RDFS, RDFS-Plus, and OWL Horst. It describes `rdfs.rules` as approximating the full RDFS ruleset. A database can name default rulesets. They are off unless configured, and when several are on they add together. A query can name rulesets of its own, or ignore the defaults.

Custom rules use the MarkLogic ruleset language. A rule is close to a SPARQL `CONSTRUCT`. The `WHERE` side is limited to triple patterns, joins and filters. The path operators `/` and `^` are allowed. `|`, `?`, `*` and `+` are not allowed in rules. Rulesets live in the Schemas database or in `Config`. Inference has a per-query memory limit, 100 MB by default, and raises `INFFULL` past it. The chapter does not describe OWL 2 DL, OWL 2 RL, or SWRL. It mentions materialising by rewriting with property paths, or by writing the derived triples yourself, as something the application can do. The product does not do that automatically.

## Search and AI

The universal index covers the words in documents. A SPARQL query can call `cts:contains`, or take a `cts:query`, so full-text and graph constraints run together. That search hits the documents that hold the triples.

MarkLogic 12 adds a vector library and an approximate nearest-neighbor index. Vectors are values in JSON or XML, or columns filled by Template Driven Extraction. The Optic API mixes a vector query with full-text, structured, graph and geospatial operators. Cosine similarity is the documented score. Release 12.1 adds `vec.precision` and `vec.trunc`, and a precision property on a TDE vector column. The vector index documentation says it requires the Semantics option, which the developer licence includes. There is no SPARQL function that searches an RDF literal embedding index. New databases exclude `//vec:vector` and `//_vector` from the universal index so embeddings are not word-indexed.

MarkLogic 12 is described as compatible with the Progress MarkLogic MCP Server. That server is a separate process. It forwards a retrieve request to the MarkLogic Retrieval API, which can run full-text and vector search. The 12.1 engine does not call a language model.

## Tooling

The Admin Interface listens on port 8001. A new install requests the developer licence from that UI. Query Console listens on port 8000 and has a SPARQL query type. App servers also use 8000, 8001 and 8002 out of the box.

Bulk load of RDF is MarkLogic Content Pump with `-input_file_type RDF`, or `sem:rdf-load` and `sem:rdf-insert`. Content Pump 12.1.0 matches server 12.1.0. Older Content Pump builds fail against 12.1.0 because user-supplied code on the REST and XDBC evaluation paths, including SPARQL, SQL, Optic and GraphQL, now runs isolated from request, response and session state.

The Java Client API, documented for MarkLogic 12, exposes `SPARQLQueryManager` (SELECT, ASK, CONSTRUCT, DESCRIBE, and SPARQL Update) and `GraphManager`. The support matrix pairs server 12.1.0 with Java client 8.2.0. The Node.js client has `graphs.sparql` and rulesets. The official Python client, `marklogic-python-client` on PyPI, speaks the REST API. Its `rows.query` method submits Optic, SQL, SPARQL and GraphQL against row views. The user guide's SPARQL example queries a view, not the triple index. Triple-store SPARQL is a POST to `/v1/graphs/sparql` through the same client. The Python repository is the client, not the engine. It showed 9 stars on 16 October 2026.

Docker is a supported platform. The `marklogic/marklogic-docker` repository publishes `progressofficial/marklogic-db`. The image can start uninitialized. Using the server still needs a developer or enterprise licence, which the Admin Interface can request. Customer-run cloud includes AWS Graviton on Amazon Linux 2023 and Red Hat Enterprise Linux, added in 12.1. The supported-platform table lists Windows Server 2019 and 2022, Windows 10 x64 for development, Red Hat Enterprise Linux 8 and 9 (x64) and 9 (aarch64), and Amazon Linux 2023 on x64 and Graviton.

Progress also sells managed hosting of MarkLogic Server, described on the features page as managed cloud hosting with the same features as the software, and on the Managed Services page as day-to-day operations run by Progress. Older pages still describe MarkLogic Data Hub Service as a fully managed service on AWS and Azure. A customer-operated cluster in a cloud account is not that service.

SQL and ODBC read Template Driven Extraction views and Optic plans. Release 12.1 adds inline Optic views, so a query can name documents without a template installed first. This is a lens over documents MarkLogic already holds. It is not a virtualization layer in front of an external relational database.

## Strengths

- One database for documents, full-text, SPARQL and SQL, with the triple index on by default for new installations.
- Query-time RDFS, RDFS-Plus, OWL Horst and custom rules, without a materialised closure to maintain.
- Forests partition a database across hosts. Forest failover and database replication ship in the same product, including under the developer licence.

## Caveats

- RDF is an index over documents. SPARQL Update and the Graph Store writes do not change triples embedded in documents or projected by a template.
- The 12 semantics guide does not document `SERVICE`, RDF-star, RDF 1.2, JSON-LD, SHACL or ShEx. The GeoSPARQL functions are a subset, and they take MarkLogic regions.
- Reasoning is query-time only, on an approximate RDFS ruleset or on OWL Horst, with a default 100 MB inference budget. OWL 2 RL and SWRL are not the shipped profiles.
- The developer licence is not a production licence. It caps the deployment at 1 TB and expires after six months unless renewed. No Progress page read for this profile states an enterprise triple-count limit.
- Vector search and the MCP server sit beside the triple index. Embeddings are document and template values. The MCP server is a separate product.

## Sources

<ul class="sources">
<li><a href="https://developer.marklogic.com/free-developer/">Developer licence</a> (all features, including semantics, clustering, HA/DR and ACID; 1 TB; 6 months; development only; Express and Academic withdrawn) and the <a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-12/page/topics/intro.html">semantics introduction</a> (SPARQL needs the Semantics option)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-whats-new-12/page/topics/release-notes.html">MarkLogic 12 release notes</a> (12.1.0 on 10 September 2026; Graviton; inline views; vector precision; MCP Server compatibility) and <a href="https://docs.progress.com/bundle/marklogic-server-whats-new-11/page/topics/release-notes.html">MarkLogic 11 release notes</a> (11.3.7 on the same day)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-whats-new-11/page/topics/new-features-in-marklogic-11-3.html">MarkLogic 11.3 LTS</a> (four-year Active phase, then two years of Sunset) and <a href="https://docs.progress.com/bundle/marklogic-server-install-12/page/topics/requirements-and-database-compatibility/supported-platforms.html">supported platforms</a> (Windows, RHEL, Amazon Linux 2023, Graviton, Docker)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-understand-concepts-12/page/topics/search.html">Searching</a> (SPARQL 1.1 against the triple index; TDE) and <a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-12/page/topics/semantic-searches.html">semantic queries</a> (SELECT, ASK, CONSTRUCT, property paths, EXISTS, NOT EXISTS, MINUS, cts restrictions)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-12/page/topics/indexes.html">Triple index</a> (on by default from MarkLogic 9; collection lexicon) and <a href="https://docs.marklogic.com/sem.rdfParse">sem:rdfParse</a> (ntriple, nquad, turtle, rdfxml, n3, trig, rdfjson, triplexml)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-11/page/topics/sparql-update.html">SPARQL Update</a> (managed triples only) and <a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-10/page/topics/embedded.html">unmanaged triples</a> (embedded triples are not updated by SPARQL Update)</li>
<li><a href="https://docs.marklogic.com/REST/GET/v1/graphs">GET</a>, <a href="https://docs.marklogic.com/REST/PUT/v1/graphs">PUT</a>, <a href="https://docs.marklogic.com/REST/DELETE/v1/graphs">DELETE</a>, <a href="https://docs.marklogic.com/REST/POST/v1/graphs">POST</a> and <a href="https://docs.marklogic.com/REST/HEAD/v1/graphs">HEAD /v1/graphs</a> (Graph Store HTTP Protocol when <code>default</code> or <code>graph</code> is set; managed triples; lexicon; PUT size limits)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-develop-with-semantic-graphs-12/page/topics/inferencing.html">Inference</a> (query-time RDFS, RDFS-Plus, OWL Horst, custom rules, 100 MB default) and <a href="https://docs.progress.com/bundle/marklogic-server-whats-new-11/page/topics/new-features-in-marklogic-11/optic-geospatial.html">Optic geospatial</a> (GeoSPARQL <code>geof</code> subset; <code>cts:region</code> arguments)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-configure-scal-avail-perf-12/page/topics/clustering.html">Clustering</a> (forests, shared-nothing, e-nodes and d-nodes) and <a href="https://docs.progress.com/bundle/marklogic-server-configure-scal-avail-perf-12/page/topics/failover.html">failover</a> (local-disk and shared-disk)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-configure-database-replication-12/page/topics/dbrep_intro.html">Database replication</a> and <a href="https://docs.progress.com/bundle/marklogic-server-understand-concepts-12/page/topics/backup-replication.html">high availability and disaster recovery</a></li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-develop-server-side-apps-12/page/topics/transactions.html">Transactions</a> (multi-statement transactions, MVCC) and the <a href="https://www.progress.com/resources/papers/marklogic-acid-transactions">ACID datasheet</a> (multi-document, multi-statement, XA)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-get-started-optic-12/page/topics/building-vector-queries.html">Vector queries</a> (ANN index, cosine, Semantics option) and <a href="https://docs.progress.com/bundle/marklogic-server-whats-new-12/page/topics/release-notes.html">12.1 release notes</a> (MCP Server as a compatible companion)</li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-develop-with-java-12/page/topics/semantics.html">Java Client API semantics</a> (<code>SPARQLQueryManager</code>, <code>GraphManager</code>) and <a href="https://marklogic.github.io/marklogic-python-client/rows.html">Python client rows guide</a> (SPARQL against views)</li>
<li><a href="https://developer.marklogic.com/products/mlcp/">Content Pump</a> (RDF bulk load; 12.1.0) and <a href="https://github.com/marklogic/marklogic-docker">marklogic-docker</a> (<code>progressofficial/marklogic-db</code>)</li>
<li><a href="https://www.progress.com/marklogic/services/what-we-offer/managed-services">Managed Services</a>, <a href="https://www.progress.com/marklogic/server/features">server features</a> (managed hosting) and <a href="https://developer.marklogic.com/concept/data-hub-service/">Data Hub Service</a></li>
<li><a href="https://docs.progress.com/bundle/marklogic-server-whats-new-12/page/topics/known-incompatibilities-with-previous-versions/between-marklogic-12.1.0-and-marklogic-12.0.3.html">12.1 incompatibilities</a> (isolated evaluation of SPARQL, SQL, Optic and GraphQL; Content Pump must match)</li>
</ul>
