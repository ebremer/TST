---
layout: store
store: hdt
title: RDF-HDT
description: Profile of RDF-HDT (Header-Dictionary-Triples), the compressed, self-indexed binary RDF format, and the libraries and servers used to query it with SPARQL.
---

## Overview

HDT (Header, Dictionary, Triples) is a **compact, binary, self-indexed serialisation of RDF**. It was published as a W3C Member Submission in March 2011 by researchers from the University of Valladolid, the University of Chile and DERI Galway. An HDT file can be queried in place, without being decompressed or loaded into a database first. That makes it both an exchange format and, in effect, a read-only triple store.

HDT is not a single product. This profile covers the format and the main ways to serve SPARQL over it:

| Component | What it does | Licence | Latest release | Activity |
|---|---|---|---|---|
| hdt-cpp | C++ library and CLI tools (`rdf2hdt`, `hdtSearch`, `hdt2rdf`) | LGPL | v1.3.3 (2019) | Commits in 2026, no new release |
| hdt-java | Java library, plus **hdt-jena**: an HDT-backed Jena graph for ARQ and Fuseki | Core LGPL-3.0, API Apache 2.0 | v3.0.10 (Sept 2023) | Dormant since 2024; built against Jena 4.3 |
| qEndpoint | HDT-based SPARQL triple store on Eclipse RDF4J, from The QA Company | GPL-3.0 (core LGPL-3.0); commercial licence available | v2.5.4 (Dec 2025) | Active |
| rdflib-hdt | Python bindings, usable as an RDFLib store | MIT | v3.3 (July 2026) | Active |
| hdt (npm) | Node.js bindings, used by Comunica and the Linked Data Fragments server | LGPL | 3.3.2 (May 2026) | Active |

## How it stores data

- **Format.** An HDT file has three parts:
  - a **Header** with metadata
  - a **Dictionary** that maps every IRI, blank node and literal to an integer ID, in compressed form
  - a compact bitmap encoding of the **Triples** over those IDs

  The file supports subject-bound triple-pattern lookups directly. A companion `.hdt.index` file (HDT-FoQ) adds the other access paths needed to answer arbitrary triple patterns.
- **Footprint.** HDT files are much smaller than text serialisations. Most of the dictionary and triples can stay in memory, and files are usually memory-mapped. The best-known example is **LOD-a-lot** (2017): 28 billion unique triples from about 650,000 datasets in a single HDT file, served with 524 GB of disk and 15.7 GB of RAM.
- **Read-only.** HDT files are immutable. Changing the data means generating a new file. qEndpoint gets around this by sending writes to a delta store, which it periodically merges into a new HDT file.
- **Generation cost.** Converting large datasets to HDT can need a lot of memory. The Java tools can generate HDT using disk instead of memory.

## Standards

What you get depends on the engine in front of the file:

- **SPARQL 1.1 Query.** Either qEndpoint (RDF4J's SPARQL engine), or Jena ARQ over an hdt-jena graph. Federated `SERVICE` comes from whichever host engine you use.
- **SPARQL Update.** Only through qEndpoint's delta store. Plain HDT files cannot be updated.
- **Triple Pattern Fragments.** The Linked Data Fragments server publishes HDT files as Triple Pattern Fragments, a lightweight interface that clients such as Comunica use to evaluate SPARQL on the client side.
- **Named graphs.** HDT 1.0 encodes triples, not quads. The hdt-cpp parser reads N-Quads, N-Triples, TriG and Turtle as input when converting files.
- **GeoSPARQL and full-text search.** Provided by qEndpoint only.
- **Not supported:** RDF-star, RDF 1.2, SHACL, ShEx and reasoning.

## Tooling

- **CLI tools:**
  - `rdf2hdt`, `hdt2rdf` and `hdtSearch` (C++)
  - equivalent Java scripts
  - the qEndpoint CLI (via Homebrew, Scoop or a zip download)
- **qEndpoint.** Ships a backend jar (Java 17), an optional web front end for writing queries, and Docker images. `qacompany/qendpoint-wikidata` can download a prebuilt Wikidata Truthy index.
- **Serving HDT from Fuseki.** hdt-java once bundled Fuseki as `hdt-fuseki`, but that module was removed in hdt-java 2.2.0. Today you configure Fuseki yourself with the hdt-jena graph. hdt-jena targets Jena 4.x, so compatibility with current Jena releases is not guaranteed.

## Strengths

- One file is the whole dataset, already indexed. Publishers can share large datasets cheaply, and consumers can query them within minutes of downloading.
- Very low hosting cost for read-only endpoints. Files are memory-mapped and need modest RAM, and because they are immutable, serving many concurrent readers is simple.
- Proven at web scale: LOD-a-lot (28 billion triples in one file), plus Wikidata HDT dumps.
- A good fit with Linked Data Fragments and Comunica for client-side SPARQL.
- Open source throughout, with bindings for C++, Java, Python and Node.js.

## Caveats

- The format is read-only. Any change means rebuilding the file, unless you run qEndpoint with its delta store.
- The ecosystem is fragmented and unevenly maintained. hdt-java has had no release since 2023 and targets Jena 4, hdt-cpp's last release was in 2019, and the bundled Fuseki server is gone.
- SPARQL performance is limited to what the host engine can do with triple-pattern lookups. There is no query optimiser designed around the HDT format.
- Triples only (no named graphs in HDT 1.0), with no RDF 1.2, reasoning or validation.
- Converting very large datasets can need a lot of memory and time.

## Sources

<ul class="sources">
<li><a href="https://www.w3.org/submissions/2011/SUBM-HDT-20110330/">W3C Member Submission: Binary RDF Representation for Publication and Exchange (HDT), 2011</a></li>
<li><a href="https://www.rdfhdt.org/what-is-hdt/">rdfhdt.org: What is HDT</a></li>
<li><a href="https://github.com/rdfhdt/hdt-cpp">rdfhdt/hdt-cpp</a> and <a href="https://github.com/rdfhdt/hdt-cpp/releases">releases</a></li>
<li><a href="https://github.com/rdfhdt/hdt-java">rdfhdt/hdt-java</a>, <a href="https://github.com/rdfhdt/hdt-java/blob/master/LICENSE">licences per module</a>, <a href="https://github.com/rdfhdt/hdt-java/releases">releases</a></li>
<li><a href="https://github.com/the-qa-company/qEndpoint">the-qa-company/qEndpoint</a> and <a href="https://github.com/the-qa-company/qEndpoint/releases">releases</a></li>
<li><a href="https://github.com/RDFLib/rdflib-hdt">RDFLib/rdflib-hdt</a></li>
<li><a href="https://www.npmjs.com/package/hdt">hdt on npm</a> and <a href="https://github.com/LinkedDataFragments/Server.js">Linked Data Fragments Server.js</a></li>
<li><a href="https://aic.ai.wu.ac.at/qadlod/lodalot/iswc2017/">LOD-a-lot: A Queryable Dump of the LOD Cloud (ISWC 2017)</a></li>
</ul>
