---
layout: store
store: millenniumdb
title: MillenniumDB
description: Profile of MillenniumDB 1.0.0 — IMFD's research graph engine for RDF and property graphs. The project says it is not production ready.
---

## Overview

MillenniumDB is a graph database from the **Millennium Institute for Foundational Research on Data (IMFD)**. The licence is GPL-2.0. There is no paid edition. The README says the project is still in active development and is not production ready.

The same engine imports three kinds of database, and each kind has its own query language. An RDF database is queried with SPARQL. A quad-model database, a property graph with one edge label and directed edges, is queried with MQL, a Cypher-like language. A GQL database follows the GQL property-graph model, including undirected edges and zero or more edge labels. The README calls the GQL implementation early and still missing a lot.

The program prints **MillenniumDB v1.0.0**. The annotated tag `v1.0.0` (17 August 2026) carries the message "MillenniumDB Release". That tag is what the Docker workflow publishes. The repository has no GitHub Releases. The default branch `dev` was one commit ahead of the tag on 30 September 2026, a small bug fix, and that tree still prints v1.0.0. This profile follows `dev` at `978cf2b` and the project wiki. The repository had 267 stars on 16 October 2026.

## Architecture and scale

`mdb import` builds a database directory from a file. `mdb server` then serves that directory as one process. The default query port is 1234 and the bundled browser uses 4321. Both are configurable. The wiki and the source describe no second process, no replica, and no partition of the data.

The buffer manager keeps one write transaction and can roll pages back. Comments in the string manager say rollback of new strings is unfinished. The wiki does not describe transactions, isolation, or durability.

No product page states a triple count. Federation tests in the tree call `https://wikidata.imfd.cl/wikidata/sparql`. That URL is a research endpoint, not a managed MillenniumDB service, and this survey does not treat it as a capacity figure.

## Standards

- **SPARQL 1.1.** The README says support is fairly complete and points at the wiki status page. That page's unsupported list is updates other than `INSERT DATA` and `DELETE DATA`, named graphs, `FROM`, `GRAPH`, regular-expression flags other than `i`, and property paths with a negated property set. The executor throws for `FROM` and `GRAPH`. SELECT, ASK, CONSTRUCT and DESCRIBE have executors in the source.
- **Deviations the project documents.** Alternative property paths are not rewritten to UNION. Language tags are case-sensitive in joins and index scans, and case-insensitive in expressions. Numeric literals keep the value and not the original lexical form, so `STR` of a double can print `1.0E0`. `"0"^^xsd:boolean` is stored as `false`. Regular expressions are ECMAScript, not Perl. The path `?s :P* ?o` returns subjects of `:P`, not every subject or object in the graph.
- **Federation.** `SERVICE` is implemented, including `SILENT` and a variable IRI. The client posts the pattern over `http` or `https` and asks for SPARQL JSON. A CSV response is rejected. TLS certificates are not validated. The status page does not mention federation, and `SERVICE` is not on its unsupported list.
- **SPARQL Update.** The update grammar accepts `INSERT DATA`, `DELETE DATA`, and `CREATE <name> INDEX`. `DELETE WHERE`, `LOAD`, `CLEAR`, `DROP`, `CREATE`, `ADD` and `MOVE` are commented out.
- **Graph Store Protocol.** The HTTP server routes `/sparql` and `/update`. There is no graph-store path. Named graphs are unsupported.
- **RDF-star and RDF 1.2.** Neither appears in the wiki or in the source tree surveyed here.
- **Import syntax.** The wiki says RDF import accepts Turtle and N-Triples, and that RDF/XML is partial. The import help also lists `n3` and `rdf`. Dump writes N-Triples or Turtle. JSON-LD is not an import format.
- **GeoSPARQL, SHACL and ShEx.** `geo:` is a built-in prefix. No GeoSPARQL function, SHACL shape, or ShEx shape appears in the source.

## Reasoning

No RDFS, OWL, or rule engine appears in the wiki or the source. `rdfs:` and `owl:` are built-in prefixes only.

## Search and AI

Full-text search is in the engine and not on the wiki. `CREATE TEXT INDEX` names a predicate and can set normalization (`identity` or `nfkdCasefold`) and tokenization (`identity`, `wsSplitPunc`, `wsRmPunc`, `wsKeepPunc`). A query calls `TEXT_SEARCH` with the index name and the string, and an optional mode `prefix` (the default) or `match`. The index name `*` searches every text index.

Tensors are documented. A literal `"[1.0, 1.5, 3.2]"^^mdbtype:tensorFloat` is a one-dimensional float tensor. `mdbfn:cosineDistance` compares two of them. `CREATE HNSW INDEX` builds an approximate nearest-neighbor index on a predicate. The wiki posts that statement to `/update` because the browser blocks updates. `HNSW_TOP_K` reads the index. The source also accepts `HNSW_SCAN`. Dropping an index is deleting its directory while the server is stopped. The wiki's further-reading links, `/doc/datatypes/tensor.md` and `/doc/indexes/hnsw.md`, are not files in the repository.

The engine does not call a language model. [wukong-engine](https://github.com/MillenniumDB/wukong-engine) is a separate repository for extracting knowledge from documents.

## Tooling

The setup page targets x86-64 Linux, Windows through WSL, and macOS on Apple silicon. Intel Macs and Windows without WSL are pointed at Docker. The build needs GCC 8.1 or newer, CMake 3.12 or newer, and Boost 1.82.

The Docker page shows `docker build -t mdb .`. The release workflow pushes `imfd/millenniumdb` on a `v*.*.*` tag, for `linux/amd64` and `linux/arm64`. On 16 October 2026 the Hub tags `latest`, `1.0`, `1.0.0` and `sha-aaf65c0` were the 17 August 2026 image. The 30 September commit is not that image.

The server ships a query browser. While `mdb server` is running, the wiki opens it at `http://localhost:4321`. The image copies that browser in. [MillenniumDB-browser](https://github.com/MillenniumDB/MillenniumDB-browser) is the Java source of the interface.

The official Python driver is [millenniumdb-driver](https://pypi.org/project/millenniumdb-driver/) on PyPI, imported as `millenniumdb_driver`, under the MIT licence. It speaks WebSocket (`ws://localhost:1234` in the example) and `session.run` sends a query string. The README example is MQL, with parameter bindings. The wiki lists the same driver as a way to query an RDF database. There is an official JavaScript driver. There is no Java client library.

## Strengths

- GPL-2.0 engine for RDF, a quad-model property graph, and an early GQL model, each with its own import and query language.
- SPARQL 1.1 query and `SERVICE`, with the gaps written down by the project.
- Native float tensors, cosine distance, and an HNSW index, documented on the wiki.
- An official multi-arch image, `imfd/millenniumdb`, and a query browser bundled with the server.

## Caveats

- The README says the project is not production ready. The `dev` branch is one commit past tag 1.0.0, and the published image is the tag.
- Named graphs, `FROM`, `GRAPH`, and most of SPARQL Update are unsupported. The status page lists further deviations in language tags, numeric lexical forms, booleans, regular expressions, and property paths.
- There is no replication and no sharding. Transaction behaviour is not documented, and string rollback is marked unfinished in the source.
- The federated client does not validate TLS certificates. A CSV result from a remote endpoint is rejected.
- Full-text search is implemented and absent from the wiki. The tensor wiki links two doc files that are not in the repository.

## Sources

<ul class="sources">
<li><a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/README.md">README</a> at <code>dev</code> <code>978cf2b</code> (30 September 2026; not production ready; SPARQL 1.1, quad model, early GQL) and the <a href="https://github.com/MillenniumDB/MillenniumDB/wiki">wiki home</a></li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/tree/v1.0.0">Tag v1.0.0</a> (annotated tag "MillenniumDB Release", 17 August 2026; the program prints v1.0.0) and <a href="https://imfd.cl/">IMFD</a></li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/wiki/SPARQL-Implementation-Status">SPARQL implementation status</a> (unsupported list and documented deviations) and <a href="https://github.com/MillenniumDB/MillenniumDB/wiki/Database-models">database models</a> (Turtle, N-Triples, partial RDF/XML)</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/query/optimizer/rdf_model/binding_iter_constructor.h">FROM and GRAPH</a> throw, and <a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/query/parser/grammar/sparql/SparqlUpdateParser.g4">the update grammar</a> keeps INSERT DATA, DELETE DATA and CREATE INDEX</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/query/executor/binding_iter/sparql_service.h">SERVICE</a> (SILENT, variable IRI) and the <a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/network/sparql/service/request.cc">federated client</a> (JSON request, CSV rejected, TLS certificates not validated)</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/wiki/Creating-and-running-a-database">Creating and running a database</a> (import, server, <code>/sparql</code>, browser, Python and JavaScript drivers) and <a href="https://github.com/MillenniumDB/MillenniumDB/wiki/Setup">setup</a> (GCC, CMake, Boost 1.82, Linux, WSL, Apple silicon)</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/bin/mdb.cc">mdb help</a> (import formats ttl, nt, n3, rdf; dump nt and ttl; the binary prints v1.0.0)</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/wiki/Working-with-tensors">Working with tensors</a> (<code>mdbtype:tensorFloat</code>, <code>mdbfn:cosineDistance</code>, CREATE HNSW INDEX, <code>HNSW_TOP_K</code>)</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/query/update/sparql/text_index_options.h">Text index options</a> and <a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/src/query/executor/binding_iter/procedure/text_search_multiscan.cc">TEXT_SEARCH</a> (prefix or match). The wiki does not describe this index</li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB/wiki/Docker">Docker wiki</a> (local <code>docker build</code>) and <a href="https://hub.docker.com/r/imfd/millenniumdb">imfd/millenniumdb</a> (1.0.0 image, 17 August 2026, amd64 and arm64). The workflow is <a href="https://github.com/MillenniumDB/MillenniumDB/blob/978cf2b5514878b2430dbf74c3e57f7c5dda8af9/.github/workflows/docker-publish.yml">docker-publish.yml</a></li>
<li><a href="https://github.com/MillenniumDB/MillenniumDB-driver-python">Python driver</a> (MIT, WebSocket, MQL example) and <a href="https://github.com/MillenniumDB/wukong-engine">wukong-engine</a> (separate LLM extraction project)</li>
</ul>
