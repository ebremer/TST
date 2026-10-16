---
layout: store
store: jena-tdb2
title: Apache Jena TDB2
description: Profile of Apache Jena TDB2, the native transactional store of the open-source Java RDF framework, embedded or served by Fuseki. Covers standards, scale, strengths and caveats.
---

## Overview

TDB2 is the native, transactional storage engine of Apache Jena, the Apache Software Foundation's Java framework for RDF and SPARQL. It works with two other Jena components:

- **ARQ**, the SPARQL engine that evaluates queries over TDB2
- **Fuseki**, the HTTP server that exposes TDB2 datasets as SPARQL endpoints

There are two ways to use TDB2:

- **Embedded:** inside a Java application, through Jena's APIs, with no server.
- **Behind Fuseki:** as a SPARQL endpoint over HTTP.

Fuseki can also serve other backends, such as in-memory datasets, or HDT files through the third-party hdt-jena module (see [RDF-HDT]({{ '/stores/hdt/' | relative_url }})). This profile covers TDB2 together with the ARQ and Fuseki capabilities that come with it.

Everything is Apache 2.0 licensed. There is no commercial edition and no feature gating.

Jena 6 is the current major line. It began with 6.0.0 in February 2026, and the latest release is 6.2.0, announced on 31 July 2026. The 6.x releases:

- require **Java 21** or later
- retired the Fuseki WAR distribution (in 6.0.0)
- moved from the older RDF-star design to RDF 1.2 / SPARQL 1.2

## Architecture and scale

- **Storage.** TDB2 maps every RDF term to an 8-byte node ID and stores triples and quads in custom B+tree indexes over memory-mapped files. Small values such as integers, decimals, dates and booleans are stored inline in the node ID, so filters on them don't need a lookup.
- **Transactions.** TDB2 is transactional only. Transactions are serialisable, with multiple concurrent readers and a single writer (MR+SW), and there is no size limit on a transaction.
- **Compaction.** TDB2 uses copy-on-write structures, so the database grows on disk as it is updated. It needs periodic **compaction**, which can run online but makes writers wait until it finishes.
- **Topology.** Jena runs on a single node. There is no built-in replication or clustering. The separate, non-ASF [RDF Delta](https://github.com/afs/rdf-delta) project can replicate Fuseki servers for high availability.
- **Scale.** Wikidata's full dump (about 16.7 billion triples, December 2021) has been loaded into TDB2 with `tdb2.xloader` in roughly 104 hours, as recorded in the Wikimedia Phabricator task and the W3C LargeTripleStores wiki.
- **Deployment.** Fuseki runs as a standalone server or as an embeddable `jena-fuseki-server` jar. The project ships a Dockerfile and compose kit but publishes **no official container image**. Community images exist.

## Standards

Jena has the broadest standards coverage of the stores in this round:

- **SPARQL 1.1.** Query, Update and Protocol, plus the Graph Store Protocol with Jena's quad extensions. Federated `SERVICE` is supported and can be disabled per server. The *Service Enhancer* module adds caching and bulk requests for `SERVICE` calls.
- **RDF 1.2 / SPARQL 1.2.** Triple terms are supported in queries, updates and result formats. Jena 6.1 added full RDF 1.2 syntax, including RDF/XML. The older RDF-star community-group mode has been archived, so `<< >>` syntax now has RDF 1.2 meaning. RDF 1.1 data behaves exactly as before.
- **Serialisations.** Turtle, TriG, N-Triples, N-Quads, RDF/XML, JSON-LD 1.1 (through Titanium), TriX, RDF/JSON, and the binary RDF Thrift and RDF Protobuf formats.
- **SHACL.** `jena-shacl` implements SHACL Core and SPARQL-based constraints, plus the SHACL Compact Syntax. It is available as a Fuseki `/shacl` operation and as a command-line tool.
- **ShEx.** `jena-shex` supports ShExC, without semantic actions or `EXTERNAL`.
- **GeoSPARQL 1.0.** All six conformance classes, backed by a JTS spatial index. It is available as a Fuseki module.

## Reasoning

Jena includes a general-purpose rule engine that can run in three modes:

- **Forward chaining** (RETE), which materialises inferred triples
- **Backward chaining**, which works out inferences at query time
- **Hybrid**, which mixes the two

On top of that engine it ships RDFS and three OWL rule sets (Micro, Mini and Full). The OWL rule sets are sound but deliberately incomplete. The reasoners work in memory and are not designed to run over large TDB2 datasets.

For datasets, a separate "Data and RDFS" wrapper applies RDFS inference at query time without materialising anything.

## Search and AI

- **Full-text search.** `jena-text` indexes literals with Apache Lucene and exposes the index through a SPARQL property function. Elasticsearch support was dropped after Jena 3.17.
- **Vector and LLM features.** The Apache codebase has none.

## Tooling

- **Fuseki UI.** A Vue 3 web interface with the YASQE editor and YASR results viewer. It can show GeoSPARQL results on a map.
- **Admin API.** An HTTP API under `/$/` for datasets, backup, compaction, statistics, tasks and metrics.
- **Security.** Apache Shiro handles authentication, and `jena-fuseki-access` adds graph-level access control.
- **Command-line tools:**
  - `riot` (parsing and conversion)
  - `arq` (queries)
  - `shacl` (validation)
  - the `tdb2.*` family: query, update, dump, backup, compact and stats
- **Bulk loaders:**
  - `tdb2.tdbloader`, with basic, sequential, phased and parallel modes
  - `tdb2.xloader`, for very large datasets. It builds a new database from scratch and runs only on Linux and other Unix systems.
- **Java APIs:**
  - the Model/Graph API
  - `RDFConnection`, for local and remote access
  - QueryBuilder
  - RDF Patch
  - a new OWL 2 API (`jena-ontapi`)
- **Extension points.** ARQ custom functions, property functions, and Fuseki Modules.

## Strengths

- Fully open source under a permissive licence, with ASF governance and a steady, roughly quarterly release cadence.
- The broadest standards coverage in the round. Jena 6 ships RDF 1.2 / SPARQL 1.2 triple terms in the stable line, with SHACL, ShEx, GeoSPARQL and many serialisations.
- It can be embedded and extended at every layer: library, query engine, server modules.
- Proven at multi-billion-triple scale on a single node.
- It doubles as a reference toolkit (`riot`, `shacl`, `arq`) used throughout the RDF ecosystem.

## Caveats

- Single node only, with one writer at a time. High availability needs the external RDF Delta project.
- TDB2 grows as it is updated and needs compaction, which blocks writes while it runs.
- The reasoners are in-memory and the OWL rule sets are incomplete. Jena is not a substitute for a materialising store like GraphDB on large ontologies.
- There is no official container image, and `xloader` does not run on Windows.
- Some documentation pages are out of date, for example pages that still mention the WAR file or Elasticsearch.

## Sources

<ul class="sources">
<li><a href="https://jena.apache.org/">Apache Jena</a> and <a href="https://jena.apache.org/download/">downloads (Java 21 requirement)</a></li>
<li><a href="https://apache.googlesource.com/jena/+/refs/heads/main/CHANGES.txt">CHANGES.txt (6.x release history)</a> and <a href="https://github.com/apache/jena/tags">GitHub tags</a></li>
<li><a href="https://jena.apache.org/documentation/fuseki2/">Fuseki documentation</a>, <a href="https://jena.apache.org/documentation/fuseki2/fuseki-docker.html">Fuseki Docker kit</a>, <a href="https://jena.apache.org/documentation/fuseki2/fuseki-server-protocol.html">Fuseki server protocol</a></li>
<li><a href="https://jena.apache.org/documentation/tdb2/">TDB2</a>, <a href="https://jena.apache.org/documentation/tdb2/tdb2_admin.html">TDB2 administration (compaction)</a>, <a href="https://jena.apache.org/documentation/tdb2/tdb2_cmds.html">TDB2 commands</a>, <a href="https://jena.apache.org/documentation/tdb/tdb-xloader.html">xloader</a></li>
<li><a href="https://jena.apache.org/documentation/tdb/architecture.html">TDB architecture</a> and <a href="https://jena.apache.org/documentation/txn/">transactions</a></li>
<li><a href="https://jena.apache.org/documentation/io/">RDF I/O</a> and <a href="https://jena.apache.org/documentation/io/json-ld-11.html">JSON-LD 1.1</a></li>
<li><a href="https://jena.apache.org/documentation/shacl/">SHACL</a> and <a href="https://jena.apache.org/documentation/shex/">ShEx</a></li>
<li><a href="https://jena.apache.org/documentation/inference/">Inference</a></li>
<li><a href="https://jena.apache.org/documentation/geosparql/">GeoSPARQL</a> and <a href="https://jena.apache.org/documentation/query/text-query.html">text search</a></li>
<li><a href="https://jena.apache.org/documentation/query/service.html">SERVICE configuration</a></li>
<li><a href="https://www.w3.org/wiki/LargeTripleStores">W3C wiki: LargeTripleStores</a> and <a href="https://phabricator.wikimedia.org/T299460">Wikimedia Phabricator T299460</a></li>
<li><a href="https://github.com/afs/rdf-delta">RDF Delta</a></li>
</ul>
