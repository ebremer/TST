---
layout: store
store: graphdb
title: GraphDB
description: Profile of Graphwise (formerly Ontotext) GraphDB — RDF4J-based store with forward-chaining reasoning, editions, clustering, standards coverage and caveats.
---

## Overview

GraphDB is a commercial RDF store built on Eclipse RDF4J and known best for its **materialising reasoner**. Ontotext first developed it in Sofia. In 2024 Ontotext merged with Semantic Web Company to form **Graphwise**, which now sells it as *Graphwise GraphDB*. In August 2026 Oakley Capital agreed to take a majority stake in Graphwise. The documentation still lives at `graphdb.ontotext.com`, and the GitHub organisation is still named `Ontotext-AD`.

The engine is closed source. The GraphDB Workbench web UI is open source (Apache 2.0).

## Editions

There are two editions. The former Standard Edition has been discontinued but is still supported for existing customers.

| | Free | Enterprise |
|---|---|---|
| Cores / concurrent queries | 1 core, 2 concurrent queries | Licensed per core; one concurrent query per core |
| Repositories | Up to 5 | Unlimited |
| Size ceiling | ~2 billion RDF nodes (32-bit entity IDs) | 40-bit entity IDs |
| High-availability cluster | — | Yes |
| Elasticsearch / OpenSearch / Solr / Kafka connectors | — | Yes |
| LDAP, Kerberos, OpenID/OAuth, fine-grained access control | — | Yes |

Since version 11.0 (April 2025), the Free edition no longer ships with a built-in licence. You have to request a free licence from Graphwise. Enterprise prices are not public. Enterprise is also sold through the AWS and Azure marketplaces.

## Architecture and scale

- **Storage.** GraphDB uses a native, file-based store. An entity pool maps RDF terms to IDs, and statement indexes cover PSO and POS orders. Context and predicate-list indexes are optional.
- **Transactions.** There is one write transaction at a time per repository. Reads are not blocked. Inferred statements are materialised at commit.
- **Cluster (Enterprise).** The cluster uses the **Raft** consensus protocol: one leader, followers, and quorum writes, with reads load-balanced across nodes. Every node holds a full replica, so the cluster scales reads and availability but not dataset size. Multi-region and secondary clusters are supported.
- **Scale.** Graphwise's official sizing guide goes up to **30 billion statements**, with a 128 GB heap and about 2.9 TB of disk. GraphDB 10.1 has an audited LDBC SNB Interactive result at scale factor 30.
- **Deployment.** You can run GraphDB from a desktop installer (Windows, macOS, Linux), a standalone server distribution, or official Docker images (amd64 and arm64). There is also an official Helm chart for Kubernetes and Terraform modules for AWS and Azure. It needs Java 21. The desktop installer bundles its own runtime.

## Standards

- **SPARQL 1.1.** Query, Update, Protocol and the Graph Store HTTP Protocol are all supported. Federation works through `SERVICE` and through internal federation across repositories. The documentation lists known deviations. One example: `DELETE` without a `GRAPH` clause removes matching triples from *all* graphs.
- **RDF-star and SPARQL-star.** Supported since version 9.2, using the community-group model of embedded triples. The 11.x documentation does not mention RDF 1.2 or SPARQL 1.2 triple terms.
- **Formats.** Turtle, TriG, N3, N-Triples, N-Quads, JSON-LD (and newline-delimited JSON-LD), RDF/JSON, RDF/XML, TriX and RDF4J binary. The Turtle and TriG parsers also accept the star syntax.
- **SHACL.** Validation runs at transaction commit, using RDF4J's SHACL engine. It supports SHACL-SPARQL constraints and DASH extensions, and you can also validate in bulk through the REST API. A few property-path forms are not supported in shapes. **ShEx** is not supported.
- **GeoSPARQL 1.0** through a plugin.

## Reasoning

Reasoning is GraphDB's signature feature. It uses **forward-chaining materialisation with incremental retraction**: when a statement is deleted, any inferences that depended on it are removed without recomputing everything.

The built-in rulesets are:

- RDFS
- RDFS-Plus (the default)
- OWL-Horst
- OWL-Max
- OWL 2 RL
- OWL 2 QL (the documentation notes this one is not fully conformant)

You can also write custom rulesets in GraphDB's `.pie` rule language.

`owl:sameAs` gets special handling. Each set of equivalent resources is stored as one representative node rather than being expanded into every combination of triples.

## Search and AI

- **Full-text search.** Lucene is available in every edition. The Enterprise edition adds connectors that keep Elasticsearch, OpenSearch or Solr in sync with the graph. The Solr connector is deprecated and scheduled for removal in version 12.
- **Vector and similarity search.** Semantic similarity indexes are built in. Vector search runs through the Elasticsearch and OpenSearch connectors, added in 11.2.
- **LLM features:**
  - *Talk to Your Graph*, an LLM agent that works with OpenAI, Azure OpenAI, Gemini/Vertex AI and Amazon Bedrock
  - GPT functions you can call from SPARQL
  - A ChatGPT Retrieval connector
  - An MCP server, added in 11.1

## Tooling

- **Workbench** is the web UI for administration, SPARQL editing, visual graph exploration and, since 11.4, map views of query results.
- **APIs.** GraphDB offers the full RDF4J Java API, plus a REST API for administration, the cluster and security.
- **GraphQL** has been supported natively since 11.0. An extended GraphQL module is sold as a paid add-on.
- **Virtualisation and SQL.** Since 9.5, Ontop-based virtual repositories have exposed relational databases as SPARQL through R2RML or OBDA mappings. GraphDB's own data can also be queried from SQL tools through JDBC-backed SQL views.
- **ImportRDF** is the offline bulk loader, with `load` and `preload` modes. It cannot run against a live cluster, and `preload` skips inference.
- **Clients.** Official JavaScript client (graphdb.js). Python access through RDFLib's `GraphDBClient`.
- **Security and monitoring.** Per-repository encryption at rest (11.4) and Prometheus metrics.

## Strengths

- The most complete materialising reasoner in this round: several standard rulesets, custom rules, incremental retraction and `owl:sameAs` compression.
- Built on RDF4J, so it is API-compatible with the wider RDF4J ecosystem and covers SPARQL 1.1 broadly.
- Raft-based high-availability cluster, including multi-region.
- The most developed search and AI integrations of the stores surveyed: external search connectors, vector search, LLM agents, an MCP server and GraphQL.
- A steady release cadence of roughly quarterly minor releases.

## Caveats

- The engine is closed source. The Free edition now needs a requested licence and is limited to one core and two concurrent queries, which rules it out for most production use.
- The cluster, enterprise authentication and external connectors are Enterprise-only. The cluster replicates data rather than sharding it.
- There is a single writer per repository, and the fast offline loader cannot be used with a running cluster.
- RDF-star follows the older community-group model, RDF 1.2 is not yet documented, and GeoSPARQL support is version 1.0 only.
- Ownership has changed recently (the Graphwise merger, then Oakley Capital's investment). Watch for roadmap or licensing changes.

## Sources

<ul class="sources">
<li><a href="https://graphwise.ai/components/graphdb/">Graphwise: GraphDB product page</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/release-notes.html">GraphDB 11.5 release notes</a> and <a href="https://graphdb.ontotext.com/documentation/11.0/release-notes.html">11.0 release notes</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/licensing.html">Licensing and editions</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/requirements.html">Requirements and sizing</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/storage.html">Storage</a> and <a href="https://graphdb.ontotext.com/documentation/11.5/cluster-basics.html">cluster basics</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/sparql-compliance.html">SPARQL compliance</a> and <a href="https://graphdb.ontotext.com/documentation/11.5/rdf-sparql-star.html">RDF-star / SPARQL-star</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/rdf-formats.html">RDF formats</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/shacl-validation.html">SHACL validation</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/reasoning.html">Reasoning</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/geosparql-support.html">GeoSPARQL support</a></li>
<li><a href="https://graphdb.ontotext.com/documentation/11.5/loading-data-using-importrdf.html">Loading data with ImportRDF</a></li>
<li><a href="https://github.com/Ontotext-AD/graphdb-workbench">Ontotext-AD/graphdb-workbench</a>, <a href="https://github.com/Ontotext-AD/graphdb-helm">graphdb-helm</a>, <a href="https://github.com/Ontotext-AD/graphdb.js">graphdb.js</a></li>
<li><a href="https://hub.docker.com/r/ontotext/graphdb/tags">Docker Hub: ontotext/graphdb</a></li>
<li><a href="https://www.prnewswire.co.uk/news-releases/graphdb-9-5-offers-data-virtualization-from-tables-to-graphs-and-back-811309393.html">GraphDB 9.5 announcement: data virtualisation and JDBC access (2020)</a></li>
<li><a href="https://ldbcouncil.org/docs/audits/snb/LDBC_SNB_I_20230131_SF30_graphdb-executive_summary.pdf">LDBC SNB Interactive audit, GraphDB SF30 (2023)</a></li>
<li><a href="https://tech.eu/2026/08/19/oakley-capital-takes-majority-stake-in-graphwise-in-one-of-bulgarias-largest-software-exits/">tech.eu: Oakley Capital takes majority stake in Graphwise (Aug 2026)</a></li>
</ul>
