---
layout: store
store: oxigraph
title: Oxigraph
description: Profile of Oxigraph 0.5.11 — a Rust RDF store on RocksDB, with SPARQL 1.1, JSON-LD 1.1, and preliminary RDF 1.2 draft support.
---

## Overview

Oxigraph is an RDF graph database and toolkit written in Rust by Thomas Tanon. Release **0.5.11** (2 September 2026) is the current stable release. The licence is MIT or Apache 2.0, at your option. There is no paid edition. The README calls the project a work in heavy development, and the architecture notes call it a hobby project that aims to stay simple.

The same repository ships a Rust library, Python bindings (`pyoxigraph`), JavaScript bindings compiled to WebAssembly, and the `oxigraph` command-line tool. The command-line tool is also an HTTP server. The maintainer offers paid work for extra features. That work is not a published edition.

## Architecture and scale

Two stores share the query engine.

- **RocksDB.** Used when the store is given a directory. The architecture notes, written toward version 0.4, describe an LSM-tree database with indexes for the default graph and for named graphs. Terms shorter than 16 bytes are stored inline. Longer strings are stored under a 128-bit hash, and those strings are kept even after the term that used them is deleted.
- **Memory.** Used when no directory is set. Python offers this store and the RocksDB store. The JavaScript package is the in-memory store only. The command-line server is documented with a storage directory.
- **Transactions.** The store exposes repeatable read: a reader sees committed data, and that view stays stable for the whole query or update. `start_transaction` commits with `commit` and rolls back if the transaction is dropped. In memory, one write transaction runs at a time, which the notes treat as serialisable. On RocksDB, release 0.5 writes through `WriteBatchWithIndex`. The changelog says some write-write conflicts are no longer detected, for example one transaction dropping a graph while another adds a triple to it. Bulk load is atomic unless you pass `--non-atomic`.
- **Queries.** Evaluation is a Volcano iterator plan, and each query runs on one thread. The README says query evaluation has not been optimized yet.
- **One machine.** The architecture notes say a distributed Oxigraph is not a goal. A read-only secondary mode existed in the 0.3 line and was removed in 0.4.
- **Scale.** No current capacity figure is published. The bench directory compares Oxigraph 0.2 and 0.3 on a Berlin SPARQL Benchmark run of about 35 million triples. That run is not a figure for 0.5.11.

## Standards

- **SPARQL 1.1.** The README says Oxigraph implements SPARQL Query, Update and Federated Query, and that these implementations are nearly fully conformant with the 1.1 recommendations. The HTTP server speaks the SPARQL 1.1 Protocol at `/query` and `/update`, and both operations at `/sparql` since 0.5.8. The Graph Store HTTP Protocol is `/store`. A GET on the server returns a SPARQL service description. Empty graphs are recorded, so `CREATE` and `DROP` apply to them. The SPARQL wiki lists differences that are still on that page: some `SERVICE` calls whose endpoint is a variable are rejected, and numeric, boolean, calendar and duration literals are normalized to a native encoding. The same page still describes experimental SPARQL-star. Release 0.5 removed that feature, so that section is older than the release surveyed here.
- **RDF 1.2 and SPARQL 1.2.** The project README calls this preliminary support for the 1.2 drafts. It lives behind the `rdf-12` Cargo feature: triple terms, directional language tags, the `VERSION` declaration, and RDF/XML 1.2 syntax. In 0.5.11 that feature is on by default in the command-line tool, the Python package and the JavaScript package, and off by default in the Rust crate. Release 0.5 dropped the RDF-star feature. Opening a 0.4 database migrates it so that most existing queries keep working, and N-Triples and Turtle serialization changed. The 0.5 changelog is the current account of that change.
- **JSON-LD.** JSON-LD 1.0 arrived in 0.4.10, in the library, the bindings and the HTTP server. JSON-LD 1.1 has been implemented and enabled by default since 0.5.5. A parser option still selects 1.0.
- **Canonicalization.** RDF Dataset Canonicalization 1.0 is available on the in-memory graph and dataset types since 0.5.4.
- **GeoSPARQL.** The `spargeo` crate calls itself a partial GeoSPARQL 1.1 implementation. It covers the Simple Features, Egenhofer and RCC8 topology functions, and a subset of the geometry and metric functions, for WKT and GeoJSON under the CRS84 reference system. Query rewrite (`geo:sfWithin` as a predicate) and the RDFS entailment extension are absent. The crate is on by default in the command-line tool, Python and JavaScript, and off by default in the Rust crate. No conformance test report is published.
- **SHACL and ShEx.** Neither is in 0.5.11. Both are an open feature request from 2020. The maintainer pointed that request at another Rust project in October 2025.

## Reasoning

There is no inference engine. In a 2023 discussion the maintainer said reasoning is not built in, and that a SPARQL update can be used to materialise a closure such as transitive `rdfs:subClassOf` if the application runs that update itself. The GeoSPARQL notes list the RDFS entailment extension as out of scope for the same reason.

Python can register custom SPARQL functions. That is an extension point for one function, not a rule language.

## Search and AI

Full-text search is an open feature request, filed in 2020 and still open, with milestone 1.0. Nothing in the 0.5.11 tree describes a vector index or an LLM agent.

## Tooling

`oxigraph serve --location DIR` listens on `localhost:7878` and serves a [YASGUI](https://yasgui.triply.cc) query page. `oxigraph load` bulk-loads a file, in parallel, as one atomic load unless `--non-atomic` is set. The same binary converts, dumps, queries and updates from the shell. A systemd unit example ships in the command-line README.

The published container image is `ghcr.io/oxigraph/oxigraph`. Pre-built binaries are attached to the GitHub release, and the command-line tool is also on PyPI and conda-forge.

Python is a real client: `pyoxigraph` embeds the store, in memory or on disk, and speaks SPARQL itself. JavaScript embeds the in-memory store and uses the RDF/JS data model. There is no Java API. `oxrdflib`, in a separate repository, is an RDFLib adapter on top of the Python bindings.

## Strengths

- One permissively licensed codebase for a Rust library, Python, JavaScript and a SPARQL server, with an official container image.
- SPARQL 1.1 Query, Update, Protocol and the Graph Store Protocol, plus JSON-LD 1.1.
- Preliminary RDF 1.2 draft support in the released command-line, Python and JavaScript builds, with the Rust crate left on the 1.1 path unless you opt in.

## Caveats

- The README says query evaluation has not been optimized, and each query runs on one thread. No current capacity figure is published.
- RocksDB transactions can miss some concurrent write conflicts. Strings are not reclaimed when a term is deleted.
- No reasoning, SHACL, ShEx, full-text search, replication or sharding. The JavaScript build is memory only.
- The SPARQL wiki still describes experimental SPARQL-star. Release 0.5 removed that feature. The architecture page still aims its answers at version 0.4.

## Sources

<ul class="sources">
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/README.md">README at v0.5.11</a> (specs, licence, preliminary RDF 1.2) and the <a href="https://github.com/oxigraph/oxigraph/releases/tag/v0.5.11">0.5.11 release</a> (2 September 2026)</li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/CHANGELOG.md">Changelog at v0.5.11</a> (RDF 1.2 feature, JSON-LD 1.1, transactions, GeoSPARQL, bulk load)</li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/cli/README.md">CLI README</a> (protocol, Graph Store Protocol, YASGUI, Docker image <code>ghcr.io/oxigraph/oxigraph</code>) and <a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/cli/Cargo.toml">cli/Cargo.toml</a> (default features <code>rdf-12</code> and <code>geosparql</code>)</li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/lib/oxigraph/Cargo.toml">Rust crate features</a> (<code>rdf-12</code> off by default), <a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/python/Cargo.toml">Python features</a> and <a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/js/Cargo.toml">JavaScript features</a></li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/lib/oxigraph/README.md">Library README</a> and <a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/python/README.md">pyoxigraph README</a></li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/js/README.md">JavaScript README</a> (in-memory store)</li>
<li><a href="https://github.com/oxigraph/oxigraph/wiki/Architecture">Architecture wiki</a> (RocksDB, single-machine goal, repeatable read; written toward v0.4) and the <a href="https://github.com/oxigraph/oxigraph/wiki/SPARQL">SPARQL wiki</a> (documented SPARQL 1.1 differences; its SPARQL-star section predates 0.5)</li>
<li><a href="https://github.com/oxigraph/oxigraph/blob/v0.5.11/lib/spargeo/README.md">spargeo README</a> (partial GeoSPARQL 1.1)</li>
<li><a href="https://github.com/oxigraph/oxigraph/discussions/401">Discussion 401</a> (no built-in inference), <a href="https://github.com/oxigraph/oxigraph/issues/55">issue 55</a> (SHACL and ShEx) and <a href="https://github.com/oxigraph/oxigraph/issues/48">issue 48</a> (full-text search)</li>
<li><a href="https://github.com/oxigraph/oxigraph">oxigraph/oxigraph</a> (1,995 stars on 16 October 2026; MIT OR Apache-2.0)</li>
</ul>
