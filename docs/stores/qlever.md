---
layout: store
store: qlever
title: QLever
description: Profile of QLever, the open-source C++ SPARQL engine from the University of Freiburg built for very large datasets on one machine and chosen to replace Blazegraph behind the Wikidata Query Service.
---

## Overview

QLever is an open-source SPARQL engine from the Chair of Algorithms and Data Structures at the **University of Freiburg** (Hannah Bast's group). It was first described in a 2017 paper. It is written in C++ and licensed under Apache 2.0.

QLever was designed for one job: loading and querying **very large datasets on a single commodity server**. The project's documentation talks about hundreds of billions of triples, and its public demos include a dataset of more than a trillion.

Two 2025–2026 developments moved QLever from research engine to production infrastructure:

- **Full SPARQL 1.1.** QLever has been compliant with SPARQL 1.1 since **June 2025**, including Update and the Graph Store Protocol. A public dashboard runs the W3C test suite on every commit.
- **Wikidata.** In **June 2026**, the Wikimedia Foundation, Wikimedia Deutschland and the QLever team announced that QLever will replace Blazegraph as the backend of the **Wikidata Query Service**. The target is to have all traffic migrated by mid-2027. The **UniProt** public endpoint already switched to QLever, from Virtuoso, in June 2025.

## Architecture and scale

- **Index.** QLever builds a read-optimised index from RDF files using external memory. It stores compressed permutations of the triples, with the vocabulary held on disk. Building the index is a separate step from serving queries, driven by the `qlever` command-line tool.
- **Updates.** SPARQL Update writes go to a separate set of "delta" triples rather than into the built index. They are **not kept across restarts unless you set `PERSIST_UPDATES`**. A rebuild step folds them into a fresh index. Wikimedia's evaluation noted that performance degrades when real-time updates run for long periods, and the QLever team has recommended ways to manage this.
- **Topology.** QLever runs on a single node. There is no clustering or replication.
- **Scale.** Freiburg hosts public endpoints for several large datasets:
  - Wikidata (about 20 billion triples)
  - Wikimedia Commons
  - OpenStreetMap
  - UniProt
  - PubChem
  - DBLP

  The project also demonstrates a dataset of more than a trillion triples on one machine.
- **Performance.** The QLever documentation links to three comparisons:
  - a DBLP benchmark against Oxigraph, Jena, Stardog, GraphDB, Blazegraph and Virtuoso
  - an independent Wikidata benchmark sponsored by Wikimedia CH
  - the Sparqloscope benchmark

  Wikimedia's own testing found that QLever outperformed Blazegraph on every key indicator. The track has not reproduced these results.

## Standards

- **SPARQL 1.1:**
  - Query, Update, the Graph Store HTTP Protocol, named graphs and federated `SERVICE` are all supported.
  - Conformance is tracked publicly against the W3C test suite.
  - Update and Graph Store writes need an access token.
- **RDF-star / RDF 1.2.** Not supported. Support for SPARQL 1.2 is an open issue in the project tracker.
- **GeoSPARQL.** Supported feature by feature rather than by formal conformance class:
  - `geof:` distance, measurement and envelope functions
  - the simple-features relations (`sfContains`, `sfIntersects` and so on)
  - optimised spatial joins and nearest-neighbour search
  - optional precomputation of geometry attributes

  The OpenStreetMap endpoint relies on relations precomputed by `osm2rdf`.
- **Not supported:** SHACL, ShEx and reasoning.

## Search

QLever's text search is distinctive. It builds a combined index over RDF and text, so a single "SPARQL+Text" query can find text records where words **co-occur** with entities from the graph. The text can come from literals in the data or from a separate text collection that mentions those entities.

## Tooling

- **QLever UI.** A web interface with **context-sensitive autocompletion** for SPARQL, which suggests real IRIs and predicates from the dataset as you type. It also has map views for large numbers of geometries.
- **`qlever` CLI.** A Python package (`pip install qlever`, version 0.6.0 in August 2026). It downloads data, builds indexes, and starts the server and UI, all from a declarative `Qleverfile` per dataset.
- **Installation.** The Docker image `adfreiburg/qlever` is rebuilt on every commit. There are also native packages for Debian, Ubuntu and macOS (Apple Silicon).
- **No tagged releases.** QLever ships as rolling builds rather than versioned releases.
- **Query extensions:** named subqueries, materialised views and path search.

## Strengths

- Built for very large datasets on one machine, with production endpoints for Wikidata-scale and UniProt-scale data.
- Fully open source (Apache 2.0), with full SPARQL 1.1 conformance tracked publicly on every commit.
- Selected by Wikimedia after a year-long evaluation. That is a strong signal of maturity and long-term investment.
- Unusual search and UI capabilities: SPARQL+Text, practical spatial joins, and an autocompleting query editor.
- Very active development: daily commits and a fast-growing community.

## Caveats

- Single node only, with no replication or high availability.
- Updates are a recent addition. They are not persistent unless configured, are held as delta triples, and need periodic rebuilds. Long-running update workloads are a known weak spot.
- No reasoning, SHACL, ShEx or RDF 1.2 yet.
- Rolling builds rather than versioned releases make it harder to pin and audit a deployment.
- The project is run by a university research group. There is no commercial support offering in the project's own materials.

## Sources

<ul class="sources">
<li><a href="https://github.com/ad-freiburg/qlever">ad-freiburg/qlever on GitHub</a></li>
<li><a href="https://docs.qlever.dev/">QLever documentation</a>: <a href="https://docs.qlever.dev/update/">Update</a>, <a href="https://docs.qlever.dev/geosparql/">GeoSPARQL support</a>, <a href="https://docs.qlever.dev/text-search/">Text search</a>, <a href="https://docs.qlever.dev/benchmarks/">Benchmarks</a></li>
<li><a href="https://github-wiki-see.page/m/ad-freiburg/qlever/wiki/QLever-support-for-SPARQL-1.1-Update">QLever wiki: support for SPARQL 1.1 Update</a></li>
<li><a href="https://github.com/ad-freiburg/qlever/issues/2169">Issue #2169: Support SPARQL 1.2</a></li>
<li><a href="https://qlever.dev/">qlever.dev public endpoints</a></li>
<li><a href="https://hub.docker.com/r/adfreiburg/qlever">Docker Hub: adfreiburg/qlever</a> and <a href="https://pypi.org/project/qlever/">qlever on PyPI</a></li>
<li><a href="https://www.wikidata.org/wiki/Wikidata:Wikidata_Platform_team/Newsletter_June_2026">Wikidata Platform team newsletter, June 2026</a></li>
<li><a href="https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/WDQS_backend_update/Backend_Replacement">WDQS backend update: Backend Replacement</a></li>
<li><a href="https://wikidataworkshop.github.io/2025/papers/paper3.pdf">Patel-Schneider, Benchmarking SPARQL Engines on Wikidata Queries (2025)</a></li>
<li><a href="https://sparql.uniprot.org/">UniProt SPARQL endpoint (credits QLever as its software)</a></li>
<li><a href="https://en.wikipedia.org/wiki/QLever">Wikipedia: QLever</a></li>
</ul>
