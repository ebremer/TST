---
layout: store
store: sparq
title: sparq
description: Profile of sparq 0.1.6 — an experimental Rust triplestore and SPARQL engine with materialised reasoning, still pre-1.0.
---

## Overview

sparq is an RDF triplestore and SPARQL engine written in Rust. Release **0.1.6** (13 October 2026) is the current GitHub release. The licence is MIT. There is no paid edition. The README status line calls it an experimental research engine whose API is unstable and pre-1.0. Conformance against the SPARQL, SHACL and inference suites is tracked by CI ratchets that only move upward.

The same repository ships a Rust library, the `sparq-cli` and `sparq-server` binaries, Python bindings and a JavaScript/WebAssembly build. A bare `cargo build` compiles the workspace default members, which include the engine, the CLI, the server, `sparq-reason`, `sparq-shacl`, `sparq-text` and `sparq-jsonld`. Federation, GeoSPARQL, vectors, natural-language query and the zero-knowledge and multi-party-computation crates sit outside that default set.

The v0.1.6 README still tells readers to build from source until the registry publications succeed. On 16 October 2026, crates.io had no `sparq-cli` crate and PyPI had no `sparq-rdf` release. The npm package `@sparq-org/sparq` existed, and its `latest` tag was 0.1.5. The GitHub release itself is published, with CLI binaries attached.

## Architecture and scale

The README describes a dictionary-encoded store. The HTTP server holds the graph in memory unless you pass `--persist DIR`. With that flag, each committed update is written to a write-ahead log and fsync'd before the success response, and a restart replays the log. The default server drops those updates on exit. SPARQL Update requests are applied as one commit: a failing operation returns 400 and publishes nothing.

The architecture document is a design blueprint, not a measurement of this release. It aims at a read-optimised, six-permutation index that could hold a full Wikidata, about 20 billion triples, on one machine. The benchmark figures in that document are other engines' published numbers, labelled as numbers to beat. sparq's own README says performance numbers are kept off the docs and live on a separate dashboard. This survey does not copy either set.

The Helm chart pins `replicaCount` to 1. Its comments say `sparq-server` has one writable store per pod and that the store is unreplicated. The blueprint's mutable, lock-free write store is described as deferred.

## Standards

- **SPARQL 1.1.** The README says sparq runs SPARQL 1.1 and 1.2 query, and SPARQL 1.1 Update. `sparq-server` serves the SPARQL 1.1 Protocol at `/sparql` and the Graph Store HTTP Protocol for read and write. The release attaches `served-conformance-v0.1.6.json`: 60 pass, 3 documented divergences, 0 fail, across 63 checks. The protocol lane is 21 pass and 2 divergences (a missing or `*/*` Accept defaults to JSON, and ASK in CSV). The service-description and graph-store lane is 39 pass and 1 divergence: an absent graph reads back as an empty 200. An empty named graph is not recorded as existing outside an in-flight update.
- **Federation.** `SERVICE` is behind the `service` Cargo feature, off in the default build. A query that uses it then fails at execution. Built with the feature, the server still denies every host until that host is on an egress allowlist.
- **RDF 1.2.** The README says the store keeps and queries RDF 1.2 triple terms. The query skill says version handling is not full SPARQL 1.2 conformance. The v0.1.6 tree has no RDF-star mode. Quoted-triple filenames in the bench tree are star-shaped join queries.
- **JSON-LD.** `sparq-jsonld` is a default workspace member and implements a JSON-LD 1.1 document pipeline (expand, flatten, compact, frame, from RDF). The HTTP server negotiates JSON-LD by default. The skill says command-line framing is still planned, and that the server and CLI writers are a separate RDF-first path, not yet the native document pipeline. RDF/XML is a separate opt-in feature on `sparq-core`, off by default.
- **GeoSPARQL.** `sparq-geo` is opt-in and is not a default member. It parses WKT and GML Simple Features literals, evaluates `geof:` functions inside SPARQL, and can build an R-tree. Coordinates default to longitude-latitude CRS84; an EPSG:4326 literal is axis-swapped on parse. The README does not name an OGC conformance class.
- **SHACL.** `sparq-shacl` is a default member. Its skill says it covers SHACL Core, SHACL-SPARQL constraints and custom SPARQL constraint components. A shapes graph that declares `sh:entailment` is an unsupported-regime failure. SHACL Advanced Features rules, and the HTTP `/shacl/validate` route, need extra server features. ShEx does not appear in the v0.1.6 tree.
- **Canonicalization and HDT.** RDF Dataset Canonicalization 1.0 is a listed feature (`sparq-canon`, a default member). `sparq-hdt` loads HDT archives and is also a default member.

## Reasoning

`sparq-reason` forward-chains a deductive closure and materialises it. The profiles in that crate are RDFS, OWL 2 RL (which includes RDFS) and Notation3. The CLI exposes the same closure. The core engine crate does not contain the reasoner; the reasoner crate is still one of the workspace default members.

The inference report committed in the 0.1.6 tree records:

- RDF Semantics entailment: 48 pass, 0 fail. The write-up says the production materializer omits finite axiomatic and reflexive triples, and the harness adds those before the check.
- OWL 2 RL, RDF-based semantics: 78 pass, 0 fail, 13 documented divergences, 36 out of scope. The divergences are described as permanent gaps in the RL/RDF rules, including conclusions the rules do not construct. The skill calls RL sound but incomplete for OWL 2 EL.
- Notation3, against the w3c/N3 community-group suite: 1,464 pass, 0 fail, 4 documented divergences. A separate community suite, notation3tests, is described as advisory and has no published baseline.
- SPARQL 1.1 entailment regimes: 47 pass, 0 fail, 23 out of scope. OWL Direct, RIF and D-entailment are the out-of-scope regimes.

OWL 2 EL class subsumption is a separate opt-in crate, `sparq-reason-el`. OWL 2 QL query rewriting is another, `sparq-reason-ql`, marked experimental. RIF Core is an opt-in front end on the Notation3 chainer.

## Search and AI

Full-text search lives in `sparq-text`, a default member because the Python crate depends on it. It builds a BM25 index over literals and rewrites `text:` predicates into SPARQL. The skill says the lean WebAssembly bundle leaves that code out.

Vector search is `sparq-vectors`, compiled only when you ask for it. You store one embedding per term and query it with cosine similarity, using an exact scan, an in-memory HNSW index, or an on-disk graph. The crate does not run an embedding model.

Natural-language query is two further opt-in crates, `sparq-introspect` and `sparq-nlq`. One mines a schema card from the graph. The other runs a loop that grounds a question, asks a model for SPARQL, validates it, executes it and retries. Both are marked `publish = false`. A live model client is a further feature; CI uses a record/replay seam. `sparq-mcp` can call this surface. None of these crates is a default member.

The zero-knowledge and multi-party-computation crates are research scaffolds. `SECURITY.md` says they do not yet provide the security property they model, and that a relying party must not depend on them.

## Tooling

`sparq-cli` loads files, runs queries, builds a memory-mapped on-disk index for data larger than RAM, and materialises a reasoning closure. It stream-ingests gzip, bzip2 and zstd dumps. It can also import CSV as RDF, by a direct mapping or by R2RML. That import is not a SQL endpoint.

`sparq-server` listens on `127.0.0.1:3030` unless remote binds are allowed. Authentication is an optional bearer token, off unless you set one. The server exposes Prometheus metrics and optional WebSocket and SSE subscriptions. It does not serve a query page of its own.

The repository includes a Tauri desktop workbench and a web frontend. The 0.1.6 release notes ship unsigned desktop installers and say to treat them as developer and test installs. The GUI README calls the current shell a foundation: the query tool and the import drawer run, and later tools are still stubs.

A Dockerfile builds `sparq-server`. Its comments say release CI pushes the image to `ghcr.io/sparq-org/sparq-server`, and the Helm values name that repository. The image sets remote access on so the container can answer, and it starts with no auth token until you pass one. The Helm chart refuses to render the server without an auth secret, and it does not build the image itself. On 16 October 2026 an anonymous pull token for that GHCR repository was refused, so this survey did not confirm a public image.

Python is a pyo3 module. The distribution name is `sparq-rdf` and the import name is `sparq`. The Python skill says a wheel may not be published yet and shows a maturin build from the repo. JavaScript uses `@sparq-org/sparq` and the RDF/JS data model. There is no Java API. `@sparq-org/solid-server` is a loopback Solid/LDP host, and its own install note says it is not a production server.

## Strengths

- One MIT codebase for a Rust library, a CLI, an HTTP server, Python and JavaScript, plus a Dockerfile and a Helm chart.
- SPARQL 1.1 Query, Update, Protocol and the Graph Store Protocol, with a machine-readable served-surface report attached to the 0.1.6 release.
- Materialised RDFS, OWL 2 RL and Notation3 in a crate the default workspace build includes, with a published inference report.
- SHACL Core in the same default build, and opt-in full-text, GeoSPARQL, vector and natural-language query crates in the same tree.

## Caveats

- The project calls the engine experimental and the API unstable. Registry installs for 0.1.6 were still incomplete on the check date: crates.io and PyPI returned no package, and the npm `latest` tag was 0.1.5.
- `SERVICE` is off unless you rebuild with that feature, and then every host is denied until you allow it. SPARQL 1.2 triple terms are implemented, and the query skill says that path is not full 1.2 conformance.
- The default HTTP server keeps data in memory. Durable mode fsyncs a WAL, and the server docs still list durable `CLEAR` / `DROP` of an existing named graph as deferred. There is no replication and no sharding.
- The desktop workbench bundles are unsigned, and the GUI README still marks later tools as stubs. Zero-knowledge proofs and multi-party computation are research scaffolds with no security guarantee.
- No published sparq capacity figure is included here. The Wikidata target in the architecture blueprint is a design goal.

## Sources

<ul class="sources">
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/README.md">README at v0.1.6</a> (experimental status, SPARQL 1.1 and 1.2, feature list, registry-install caveat, MIT) and the <a href="https://github.com/sparq-org/sparq/releases/tag/v0.1.6">0.1.6 release</a> (13 October 2026; CLI binaries and unsigned GUI bundles)</li>
<li><a href="https://github.com/sparq-org/sparq/releases/download/v0.1.6/served-conformance-v0.1.6.json">Served-surface conformance report</a> (protocol 21/2/0, graph store 39/1/0)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/Cargo.toml">Workspace Cargo.toml</a> (version 0.1.6, MSRV 1.88, default members)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/sparql-query/SKILL.md">SPARQL query skill</a> (1.2 conformance limit, triple terms) and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/http-server/SKILL.md">HTTP server skill</a> (protocol, graph store, atomic update, <code>--persist</code> WAL, <code>SERVICE</code> default-deny)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/inference/SKILL.md">Inference skill</a> and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/inference-conformance-report.md">inference conformance report</a> (RDFS, OWL 2 RL, Notation3, SPARQL entailment regimes)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/shacl-validation/SKILL.md">SHACL skill</a>, <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/jsonld/SKILL.md">JSON-LD skill</a>, <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/geosparql/SKILL.md">GeoSPARQL skill</a> and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/data-formats/SKILL.md">data-formats skill</a> (RDF/XML opt-in, HDT)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/full-text-search/SKILL.md">Full-text skill</a>, <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/vector-search/SKILL.md">vector skill</a> and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/genai-retrieval/SKILL.md">GenAI retrieval skill</a></li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/cli/SKILL.md">CLI skill</a> (memory-mapped build, streamed ingest, CSV and R2RML import) and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/python/SKILL.md">Python skill</a> (<code>sparq-rdf</code>, import <code>sparq</code>, maturin)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/Dockerfile">Dockerfile</a> and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/deploy/helm/sparq/values.yaml">Helm values</a> (<code>ghcr.io/sparq-org/sparq-server</code>, one replica; anonymous GHCR pull refused on 16 October 2026) and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/skills/helm-deploy/SKILL.md">Helm skill</a></li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/gui/README.md">GUI README</a> (workbench shell, stub tools) and <a href="https://github.com/sparq-org/sparq/blob/v0.1.6/SECURITY.md">SECURITY.md</a> (research scaffolds)</li>
<li><a href="https://github.com/sparq-org/sparq/blob/v0.1.6/research/ARCHITECTURE.md">Architecture blueprint</a> (one-machine Wikidata target; competitor figures are targets, not sparq results)</li>
<li><a href="https://github.com/sparq-org/sparq">sparq-org/sparq</a> (13 stars on 16 October 2026; MIT). crates.io <code>sparq-cli</code> and PyPI <code>sparq-rdf</code> returned 404 that day; npm <code>@sparq-org/sparq</code> latest was 0.1.5</li>
</ul>
