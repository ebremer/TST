---
layout: store
store: neptune
title: Amazon Neptune
description: Profile of Amazon Neptune Database as an RDF/SPARQL store — managed-only architecture, standards coverage, pricing model, strengths and caveats.
---

## Overview

Amazon Neptune is AWS's managed graph database. One service speaks two graph models: property graphs (Gremlin and openCypher) and RDF (SPARQL 1.1). This profile covers **Neptune Database** used as an RDF store.

Neptune Analytics, a separate in-memory analytics engine under the same brand, can import RDF as N-Triples, but it queries that data with openCypher. Its documentation does not describe a SPARQL endpoint.

The engine is closed source. AWS has never documented where the code came from. Neptune is widely reported to derive from Blazegraph: in late 2017, after Amazon hired key Blazegraph developers, open-source Blazegraph development wound down.

## Architecture and scale

- **Storage.** Every cluster sits on one shared cluster volume on SSD, built from 10 GB segments. It keeps six copies of the data across three Availability Zones. The volume grows automatically to **128 TiB** (64 TiB in the China and GovCloud regions).
- **Topology.** A cluster has **one writer and up to 15 read replicas**, all on the same volume. Writes do not scale horizontally, because there is no sharding. **Global Database** adds up to five read-only secondary regions, typically less than a second behind.
- **Serverless.** Serverless clusters scale between 1 and 128 Neptune Capacity Units (NCUs), at about 2 GiB of memory per NCU.
- **Transactions.** Transactions are ACID. Read-only queries see a snapshot (MVCC). Updates run at READ COMMITTED with range locks that prevent phantom reads. Under heavy concurrent writes, AWS's documentation says some transactions fail on lock conflicts and must be retried by the client.
- **Data model.** Every triple is stored as a quad. The default graph is the union of all named graphs. Triples loaded without a graph go into a Neptune-specific fallback graph.

## Standards

Neptune implements **SPARQL 1.1 Query, Update, Protocol and the Graph Store HTTP Protocol**. Federated `SERVICE` calls work, with limits: they are read-only, can only reach endpoints inside the VPC, and do not reach other regions or accounts when IAM authentication is on.

The bulk loader reads N-Triples, N-Quads, Turtle and RDF/XML from S3. SPARQL `LOAD` also accepts TriG, N3 and JSON-LD. Results come back in all the common SPARQL result and RDF formats.

Several standards are not supported:

- RDF-star and RDF 1.2 / SPARQL 1.2
- SHACL
- GeoSPARQL (Neptune's newer geospatial functions are openCypher-only)

Neptune also has known implementation quirks. `xsd:dateTime` values are normalised to UTC, which loses the original timezone, and numeric values are limited to 64 bits.

## Reasoning and validation

Neptune Database has **no built-in RDFS or OWL inference** and no SHACL engine. AWS's own OWL examples do validation and inference in application code. If your workload depends on entailment, you have to materialise the inferred triples before loading, or do it in the client.

## Search and AI

- **Full-text search** comes from Amazon OpenSearch Service, kept in sync through Neptune Streams (change data capture). It can be queried from SPARQL. Blank nodes are not replicated to the index.
- **Neptune ML** trains graph neural networks for SPARQL tasks: object classification, object regression, and link prediction.
- **Vector search and Bedrock Knowledge Bases GraphRAG** run on **Neptune Analytics**, not Neptune Database, so they are not reachable through SPARQL.

## Operations and pricing

Neptune is AWS-only. There is no self-hosted, on-premises or local developer edition. Clusters live in a VPC. Public endpoints became available in September 2025 for newer engine versions.

Costs are made up of four parts (the [pricing page](https://aws.amazon.com/neptune/pricing/) has current rates):

- **Compute:** instance-hours, or NCU-hours on Serverless.
- **Storage:** billed per GB-month on the high-water mark, so deleting data does not lower the bill until the space is reclaimed.
- **I/O:** charged per request, unless you choose the I/O-Optimized configuration, which drops I/O charges in exchange for higher instance and storage rates.
- **Backup:** included up to the size of cluster storage.

A 30-day free trial covers a small instance.

Tooling around the engine is open source under Apache 2.0:

- graph-notebook, which powers the Neptune Workbench Jupyter notebooks
- Graph Explorer, a visual browser that supports SPARQL
- neptune-export
- the GraphRAG toolkit

## Strengths

- Fully managed high availability: six-way replicated storage, up to 15 read replicas, point-in-time restore, cross-region Global Database and a Serverless option.
- Broad SPARQL 1.1 protocol coverage, including the Graph Store Protocol and federated `SERVICE`.
- RDF and property-graph workloads in the same managed service.
- Deep AWS integration: IAM (SigV4) auth, S3 bulk loading, OpenSearch, Streams, SageMaker-based Neptune ML.
- Frequent engine releases with a published support lifecycle.

## Caveats

- Proprietary and AWS-only, so there is lock-in and no way to run the same engine on a laptop or in CI.
- No reasoning, SHACL, GeoSPARQL or RDF-star / RDF 1.2.
- A single writer with no write scale-out. Clients need retry logic for lock conflicts.
- The cost model has several moving parts: high-water-mark storage, separate I/O charges, and idle instances that still bill.
- The newest AI features (vector search, GraphRAG) live in Neptune Analytics, which does not use SPARQL.

## Sources

<ul class="sources">
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/engine-releases.html">Neptune engine releases</a> and <a href="https://docs.aws.amazon.com/neptune/latest/userguide/engine-releases-1.5.0.0.html">engine 1.5.0.0 release notes</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/feature-sparql-compliance.html">SPARQL standards compliance</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/sparql-service.html">SPARQL federated queries (SERVICE)</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/sparql-graph-store-protocol.html">Graph Store HTTP Protocol</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/bulk-load-tutorial-format-rdf.html">Bulk loader RDF formats</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/limits.html">Limits</a> and <a href="https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-storage.html">storage overview</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/transactions-neptune.html">Transaction semantics</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search.html">Full-text search with OpenSearch</a></li>
<li><a href="https://docs.aws.amazon.com/neptune/latest/userguide/machine-learning-sparql-inference-queries.html">Neptune ML SPARQL inference queries</a></li>
<li><a href="https://docs.aws.amazon.com/neptune-analytics/latest/userguide/using-rdf-data.html">Neptune Analytics: using RDF data</a></li>
<li><a href="https://aws.amazon.com/neptune/pricing/">Neptune pricing</a> and <a href="https://aws.amazon.com/neptune/features/">features</a></li>
<li><a href="https://aws.amazon.com/blogs/database/model-driven-graphs-using-owl-in-amazon-neptune/">AWS blog: model-driven graphs using OWL in Neptune</a></li>
<li><a href="https://repost.aws/questions/QUsudFphfrQMu3xhJ--yX10A/is-rdf-star-supported">AWS re:Post: RDF-star support</a></li>
<li><a href="https://i-programmer.info/news/84-database/11358-amazon-neptune-graph-database.html">I Programmer: Amazon Neptune and Blazegraph (2017)</a></li>
<li><a href="https://github.com/aws/graph-notebook">aws/graph-notebook</a>, <a href="https://github.com/aws/graph-explorer">aws/graph-explorer</a>, <a href="https://github.com/awslabs/graphrag-toolkit">awslabs/graphrag-toolkit</a></li>
</ul>
