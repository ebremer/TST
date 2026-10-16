---
layout: store
store: stardog
title: Stardog
description: Profile of Stardog — commercial knowledge graph platform with query-time reasoning, data virtualization, editions, clustering, standards coverage and caveats.
---

## Overview

Stardog is a commercial knowledge-graph platform from **Stardog Union**, a privately held company in Arlington, Virginia. It pairs a SPARQL database with two things most of the other stores in this round lack:

- **Query-time reasoning:** inferences are worked out when a query runs rather than stored in advance.
- **Data virtualisation:** relational, NoSQL and file sources can be queried as if they were RDF graphs, without copying them in.

In 2026 the company rebranded the product as the "Stardog Semantic AI Platform". Co-founder Kendall Clark handed the CEO role to Craig Harper in September 2025.

The server is closed source. Client libraries, examples, Helm charts and Terraform starters are open source on GitHub.

## Editions

- **Stardog Cloud Free.** A shared, hosted instance capped at **1 million edges** and three databases, with community support only.
- **Enterprise.** Can be self-managed, run as a dedicated Stardog Cloud instance, or bought through the AWS and Azure marketplaces. It adds:
  - the high-availability cluster
  - caching
  - managed backups
  - LDAP
  - the full set of data-source connectors

  Prices are not public.

Whether there is a free licence for self-hosted use is unclear. The pricing page describes a renewable one-year free licence. In 2025, however, Stardog staff said on the community forum that free on-premises licences are not being issued, and the free EULA limits use to evaluation. If you need to run Stardog locally, confirm terms with the vendor.

## Architecture and scale

- **Storage.** The "Mastiff" storage engine is built on RocksDB (an LSM-tree store) with multi-version concurrency control.
- **Transactions.** Transactions are ACID. Snapshot isolation is the default, and serialisable isolation is available. You can choose how write conflicts are resolved: last commit wins, or abort on conflict. An optional transaction log enables point-in-time restore.
- **Cluster (Enterprise).** The cluster is coordinated by **ZooKeeper** and needs at least two Stardog nodes and three ZooKeeper nodes. Every node holds a full copy of the data and the cluster is strongly consistent, so adding nodes scales reads and availability, not write throughput or capacity. Rolling upgrades are not supported. Version 12.2 added geo-replicas.
- **Scale.** All of the following are vendor-reported:
  - Wikidata (about 16.7 billion triples) on a single node
  - bulk loading at roughly 500,000 to 1 million triples per second
  - a 24-billion-triple customer deployment
  - a "trillion-edge" demonstration, achieved by federating Stardog with external SQL warehouses through virtual graphs rather than storing the edges
- **Deployment.** Docker is the recommended way to run the server; ARM images arrived in 12.2. Zip, apt and yum packages, a Helm chart for Kubernetes, the cloud marketplaces and the managed Stardog Cloud are also available. Stardog 12 needs **Java 21**.

## Standards

- **SPARQL 1.1.** Query, Update, Protocol and the Graph Store HTTP Protocol are supported. Federated `SERVICE` works, including with variable endpoints.
- **Edge properties.** Stardog uses RDF-star / SPARQL-star `<< >>` syntax, plus a syntax of its own. The feature has limits: no nesting, and it must be enabled when a database is created. The documentation calls it "not recommended for new Knowledge Graph projects" because of known performance problems. RDF 1.2 / SPARQL 1.2 triple terms are not implemented.
- **Formats.** N-Triples, Turtle, TriG, N-Quads, RDF/XML and JSON-LD, with gzip, bzip2 and zip input.
- **SHACL.** SHACL Core and SHACL-SPARQL are supported. Validation runs through a `VALIDATE` query or an optional transactional "guard mode". SHACL Advanced Features and JavaScript extensions are not supported. ShEx support was not found in the documentation.
- **GeoSPARQL** and WGS84 queries are supported, enabled per database.
- **Extensions beyond the standards:**
  - a `PATHS` query form for path finding beyond standard property paths
  - GraphQL queries over RDF
  - a BI server that speaks the MySQL wire protocol for SQL and BI tools

## Reasoning

Stardog reasons **at query time** by rewriting queries, not by materialising inferred triples. Changing the schema or the rules therefore takes effect immediately, with no reload. The trade-off is that the cost is paid on every query instead.

- **Profiles.** The reasoner (called "Blackout") supports RDFS, OWL 2 QL, RL and EL, and an "SL" profile that adds user-defined, SWRL-style rules on top of those.
- **Datalog.** A newer Datalog reasoner, "Stride", is in beta.
- **Explanations.** `reasoning explain` shows why a given inference holds, which helps when debugging an ontology.

## Virtual graphs and integration

Virtual graphs map external sources into RDF using Stardog Mapping Syntax (SMS/SMS2) or R2RML. The mapped data can be queried in place or imported. Supported sources include:

- relational databases
- CSV and JSON files
- MongoDB, Cassandra and Elasticsearch
- Databricks, SingleStore and Teradata

This is Stardog's most distinctive capability among the stores in this round. It is the basis for its "query data where it lives" positioning.

## Search and AI

- **Full-text search.** Lucene-based (`textMatch`).
- **Voicebox.** An LLM question-answering layer that generates SPARQL from natural language. It runs on Stardog Cloud, or on-premises through Launchpad.
- **MCP and LangChain.** There is an MCP server for Stardog Cloud and LangChain integration packages.
- **Machine learning.** The older built-in machine-learning features are deprecated as of 12.2.
- **Other.** Entity resolution and graph analytics are included.

## Tooling

- **Web apps:**
  - Stardog Studio, a web IDE for queries and modelling
  - Explorer, for browsing the graph
  - Designer, for visual modelling and mapping
- **CLI:** `stardog` and `stardog-admin`.
- **Clients:**
  - Java (SNARL and HTTP)
  - Python: pystardog (Apache 2.0)
  - JavaScript: stardog.js
  - .NET, Spring and others, as described in the documentation

## Strengths

- Mature data virtualisation over SQL, NoSQL and file sources, combined with SPARQL federation.
- Query-time OWL 2 and rules reasoning with explanations. Schema changes need no rematerialisation.
- Broad standards and access coverage: SPARQL 1.1 including the Graph Store Protocol, SHACL Core and SHACL-SPARQL, GeoSPARQL, plus GraphQL and SQL/BI access.
- Strong vendor-reported single-node scale and load throughput.
- Active development: yearly major releases, roughly quarterly minor releases and frequent patches.

## Caveats

- Proprietary, with no public pricing. The terms for a free self-hosted licence are unclear, and the vendor's own pages contradict each other.
- The Cloud Free tier (1M edges, three databases) is too small for realistic evaluation at scale.
- No RDF 1.2 / SPARQL 1.2. The existing edge-property (RDF-star) support is discouraged for new projects.
- The cluster replicates every node in full, depends on ZooKeeper and does not support rolling upgrades.
- Headline performance figures come from the vendor or from studies it commissioned. Leadership and positioning have shifted recently toward enterprise AI.

## Sources

<ul class="sources">
<li><a href="https://docs.stardog.com/release-notes/stardog-platform">Stardog Platform release notes</a></li>
<li><a href="https://docs.stardog.com/install-stardog/">Installing Stardog</a> and <a href="https://docs.stardog.com/install-stardog/client/">client install (Java 21)</a></li>
<li><a href="https://www.stardog.com/pricing/">Pricing</a>, <a href="https://www.stardog.com/get-started/">Get started (Cloud Free limits)</a>, <a href="https://www.stardog.com/legal/stardog-free/">Stardog Free EULA</a></li>
<li><a href="https://community.stardog.com/t/how-can-one-obtain-the-license-for-the-free-version/5373">Community forum: obtaining a free licence (2025)</a></li>
<li><a href="https://www.stardog.com/blog/mastiff-storage-engine">Mastiff storage engine</a></li>
<li><a href="https://docs.stardog.com/operating-stardog/database-administration/transactions">Transactions</a></li>
<li><a href="https://docs.stardog.com/high-availability-cluster/">High-availability cluster</a></li>
<li><a href="https://docs.stardog.com/query-stardog/edge-properties">Edge properties (RDF-star)</a> and <a href="https://docs.stardog.com/query-stardog/geospatial-queries">geospatial queries</a></li>
<li><a href="https://docs.stardog.com/data-quality-constraints">Data quality constraints (SHACL)</a></li>
<li><a href="https://docs.stardog.com/inference-engine/">Inference engine</a></li>
<li><a href="https://docs.stardog.com/virtual-graphs/">Virtual graphs</a></li>
<li><a href="https://docs.stardog.com/voicebox/">Voicebox</a></li>
<li><a href="https://www.stardog.com/blog/query-knowledge-graphs-with-billions-of-triples-in-less-time-than-it-takes-to-read-this-headline-about-performance-benchmarks/">Stardog blog: billion-triple performance benchmarks</a> and <a href="https://www.stardog.com/blog/trillion-edge-knowledge-graph/">trillion-edge knowledge graph</a></li>
<li><a href="https://www.stardog.com/news/">Stardog news</a></li>
<li><a href="https://github.com/stardog-union">github.com/stardog-union</a> and <a href="https://pypi.org/project/pystardog/">pystardog on PyPI</a></li>
</ul>
