---
layout: store
store: rdf4j
title: Eclipse RDF4J
description: Profile of Eclipse RDF4J 6.1 — Java RDF framework with memory, native and LMDB stores, SPARQL 1.2, SHACL, and RDF 1.2 triple terms.
---

## Overview

Eclipse RDF4J is a Java framework for RDF from the **Eclipse Foundation**, and the successor to OpenRDF Sesame. It parses and writes RDF, stores it, reasons over it, and queries it. The same Repository API talks to a local store or to a remote SPARQL endpoint. Release **6.1.0** (12 September 2026) is the version the download page calls the latest stable release. It requires Java 25.

The licence is the Eclipse Distribution License, a BSD 3-Clause licence. There is no paid edition. An optional Elasticsearch backend pulls in the Elastic License (or the SSPL); the store documentation tells you to read those terms before turning it on.

RDF4J is also the access layer inside other products. [GraphDB]({{ '/stores/graphdb/' | relative_url }}) is built on it, and [Blazegraph]({{ '/stores/blazegraph/' | relative_url }}) spoke the older Sesame API that RDF4J continues.

## Architecture and scale

The about page lists two core databases. A third, LMDB, has been in the tree since 4.0 and is still marked experimental.

- **Memory Store.** Transactional main memory, with an optional directory so the contents survive a restart. Documented for datasets under 100,000 triples.
- **Native Store.** On-disk B-trees. The repository guide calls it a fit for up to about 100 million triples, "and probably even more" when the operating system caches the files. The 6.0 notes say a write-ahead log recovers RDF 1.2 values across a restart. Index choice is a string such as `spoc,posc`.
- **LMDB Store.** Memory-mapped storage. The LMDB page gives a footprint of about 120–130 bytes per quad with three indexes, so on the order of 120–130 GB per billion quads, varying with the size of IRIs and literals. Release 6.1.0 asks everyone to upgrade because a bug could lose data in LMDB. Sketch-based join planning exists and is off unless you enable it.
- **Elasticsearch Store.** Experimental. The highest isolation it documents is READ COMMITTED, and only when every other transaction uses that level too. There is no write-ahead log, so a lost connection during commit can leave a partial write.
- **Transactions.** Each call on a connection commits on its own unless you open an explicit transaction. The connection API groups modifications so that none of them are visible until commit. SHACL validation can run at SERIALIZABLE isolation on snapshot transactions.
- **Federation.** FedX spreads a query across local repositories and remote endpoints. In 6.0, grouped source selection is on by default: one grouped `SELECT` per endpoint instead of an `ASK` per pattern.
- **Deployment.** Embed the library, or run RDF4J Server and Workbench in Tomcat or Jetty. The source tree includes a Docker Compose kit that builds an image. The documentation does not point at an image on a public registry. There is no replication and no sharding of a single store.

## Standards

- **SPARQL.** The about page says RDF4J fully supports SPARQL 1.1 Query and Update. The REST API says the server implements the SPARQL 1.1 Protocol and, per repository, the Graph Store HTTP Protocol at `/rdf-graphs`.
- **RDF 1.2 and SPARQL 1.2.** These arrived in 6.0 and stayed in 6.1, which is a bug-fix release. Triple terms are model values in object position. MemoryStore, NativeStore and LMDBStore persist them, along with literals that carry both a language tag and a base direction. Turtle, TriG, N-Triples, N-Quads, RDF/XML and RDF/JSON were updated. The 6.0 notes say the work includes the emerging W3C conformance suites, and that FedX has initial support for querying triple terms. `LATERAL` is implemented as well, and the notes say it is not part of SPARQL 1.2.
- **RDF-star.** 6.0 replaced the RDF-star type names, parser names and result formats with the RDF 1.2 triple-term APIs. The older community-group mode is not a separate switch in this release.
- **JSON-LD.** The about page lists JSON-LD among the mainstream RDF formats RDF4J reads and writes. JSON-LD 1.1 is not claimed on its own.
- **GeoSPARQL.** An optional module, `rdf4j-queryalgebra-geosparql`, adds what the page calls partial GeoSPARQL support on top of any repository, using Spatial4J and JTS. No conformance class is named. The Lucene sail can index `geo:asWKT` so those queries do less work.
- **SHACL.** ShaclSail validates during `commit()`. It plans the check from the transaction's changes when it can, and it has a bulk mode for stores that do not fit in memory. Parallel validation is available and the SHACL page warns that it can deadlock on the Native Store.
- **ShEx.** Not found in the documentation.

## Reasoning

`SchemaCachingRDFSInferencer` is a forward-chaining RDF Schema layer that stacks on the memory or native store. The repository guide says the overhead grows with the store, and that large datasets should use something else.

`CustomGraphQueryInferencer` takes one SPARQL `CONSTRUCT` rule and a matcher query that decides which inferences to retract.

`SpinSail` forward-chains SPARQL Inferencing Notation rules. The SPIN page says the sail is not actively maintained, has performance and scalability problems, and was never designed for transactions: an update can fail to add new inferences, or can leave old ones behind. For validation it points you to SHACL.

No OWL reasoner is documented.

## Search and AI

LuceneSail indexes literals and answers a `search:matches` pattern, including a score and a snippet. An Elasticsearch sail is the other maintained index. The Solr sail was removed in 6.0 after being deprecated in 5.3. Nothing in the documentation describes a vector index or an LLM agent.

## Tooling

The Java API is the product: Rio for parsing, Sail for storage, Repository for applications. The Console creates local databases (including a native store with RDFS) and can connect to a server. Workbench is the browser UI for managing repositories and running queries. Python access is a generic SPARQL client against the server.

## Strengths

- RDF 1.2 triple terms and SPARQL 1.2 evaluation in a stable release, in the memory, native and LMDB stores.
- SPARQL 1.1 Query, Update, Protocol and the Graph Store Protocol, plus SHACL checked on commit.
- A BSD-licensed library you embed, with a server and workbench in the same project. Other stores in this survey already sit on this API.

## Caveats

- The native store's documented range is about 100 million triples. The about page says the core databases are meant for small to medium data, and points at third-party stores for larger ones.
- LMDB is still annotated experimental, and 6.1.0 exists because an LMDB bug could lose data.
- The RDFS inferencer gets expensive as the data grows. The SPIN sail is explicitly unmaintained.
- No replication, no sharding, and no container image you can pull. 6.0 is a breaking move to Java 25 and Jakarta servlets.
- GeoSPARQL is a partial algebra, with no conformance class stated.

## Sources

<ul class="sources">
<li><a href="https://rdf4j.org/download/">Download page</a> (6.1.0, latest stable, Java 25) and the <a href="https://rdf4j.org/news/2026/09/12/rdf4j-6.1.0-released/">6.1.0 announcement</a> (12 September 2026)</li>
<li><a href="https://rdf4j.org/release-notes/6.1.0/">6.1.0 release notes</a> (LMDB data-loss fix) and <a href="https://rdf4j.org/release-notes/6.0.0/">6.0.0 release notes</a> (RDF 1.2, SPARQL 1.2, Jakarta, Docker)</li>
<li><a href="https://rdf4j.org/about/">About RDF4J</a> (stores, SPARQL 1.1, JSON-LD, EDL)</li>
<li><a href="https://rdf4j.org/documentation/programming/repository/">Repository API</a> (memory and native stores, transactions, RDFS and custom rules)</li>
<li><a href="https://rdf4j.org/documentation/programming/lmdb-store/">LMDB store</a></li>
<li><a href="https://rdf4j.org/documentation/reference/rest-api/">REST API</a> (SPARQL 1.1 Protocol and Graph Store HTTP Protocol)</li>
<li><a href="https://rdf4j.org/documentation/programming/shacl/">SHACL</a> and <a href="https://rdf4j.org/documentation/programming/spin/">SPIN</a></li>
<li><a href="https://rdf4j.org/documentation/programming/geosparql/">GeoSPARQL</a> and <a href="https://rdf4j.org/documentation/programming/lucene/">Lucene sail</a></li>
<li><a href="https://rdf4j.org/documentation/tools/server-workbench/">Server and Workbench</a>, including the Docker Compose kit</li>
<li><a href="https://github.com/eclipse-rdf4j/rdf4j">eclipse-rdf4j/rdf4j</a> (BSD 3-Clause)</li>
</ul>
