---
layout: store
store: blazegraph
title: Blazegraph
description: Profile of Blazegraph — archived Java graph database with SPARQL 1.1, materialised RDFS+, quorum replication and a sharded scale-out mode. Last stable release 2.1.5.
---

## Overview

Blazegraph is a Java graph database from **SYSTAP, LLC**. The project describes it as supporting the Blueprints API and the RDF / SPARQL 1.1 family, and says development of the open-source platform began in 2006. Release **2.1.5** (19 March 2019) is the last stable release. A 2.1.6 release candidate was tagged on 3 February 2020 and was never published as a final release. GitHub shows the repository archived by its owner on 23 March 2026.

The project wiki says Blazegraph powers the Wikimedia Foundation's Wikidata Query Service. Wikimedia has since chosen [QLever]({{ '/stores/qlever/' | relative_url }}) to replace it, with migration due by mid-2027.

## Licensing

The GitHub repository is GPL-2.0. The About page describes a dual licence: GPLv2 and a commercial licence, with SYSTAP offering support subscriptions for both. The replication cluster and the horizontally sharded federation are described as deployment modes of that same platform, not as a separate edition.

## Architecture and scale

- **Modes.** A namespace can run as triples, as quads, or with statement identifiers (SIDs), which attach provenance to a statement without the quad indexes. The FAQ says one journal can hold many stores. One reported deployment kept 15,000 triple stores in a single journal.
- **Single machine.** The README and the 2.1.5 release notes say one machine holds up to 50 billion edges. That figure is the project's own claim.
- **Transactions.** The transaction guide says the default indexes are unisolated. Against a journal, those unisolated writes "can provide full ACID semantics", and read-only transactions with snapshot isolation are always available. Full read/write transactions, which buffer a write set and validate it at commit, are optional (`isolatableIndices=true`) and slower. There can be only one open unisolated connection on a journal. In scale-out, unisolated writes are described as shard-wise ACID.
- **Replication.** HAJournalServer keeps a quorum of services coordinated through Apache ZooKeeper. The leader accepts writes. Followers can serve reads. Writes travel as cache blocks along a pipeline, and a two-phase commit applies the root blocks only if a majority votes for the commit. The HA page says this is an ACID guarantee for the replication cluster.
- **Scale-out.** The About page also describes a horizontally sharded federation. Each data service uses a journal as a write buffer. When the journal fills, writes migrate onto read-only index segments. The IO guide says that layout is a side effect of the dynamic sharding algorithm, and that key-range scans then become sequential reads.
- **Deployment.** Distributions are an executable jar, a WAR, or a tar.gz. There is no official container image.

## Standards

- **SPARQL 1.1.** The About page lists Query, Update, basic federated query, and Service Description. The REST API is the NanoSparqlServer SPARQL endpoint. The Graph Store HTTP Protocol is not in that list.
- **Inference commands in Update.** `DISABLE ENTAILMENTS`, `ENABLE ENTAILMENTS`, `CREATE ENTAILMENTS` and `DROP ENTAILMENTS` manage materialised inferences from SPARQL Update.
- **RDF-star and RDF 1.2.** Neither is described. Statement identifiers are Blazegraph's own reification mode, not RDF-star.
- **JSON-LD.** Not documented as a load format.
- **Geospatial.** Since 2.1.0, coordinates can be stored as typed literals and queried with a custom `SERVICE geo:search` (rectangle, circle, and an optional time range). The geospatial page says GeoSPARQL might come in a future release. It did not.
- **SHACL and ShEx.** Not found in the 2.1 documentation.

## Reasoning

Inference is materialised. The bundled ruleset is RDFS plus a small amount of OWL ("RDFS+"), and the engine can be extended with rules written as conjunctive queries. Two modes are documented: compute the closure of the whole database at once, or maintain it incrementally as data changes. Incremental maintenance is described as fast for small updates and expensive for large ones, so bulk load is supposed to use the database-at-once mode. Inference is not available in quads mode out of the box, because there is no standard for which named graph the inferences should land in.

## Search

A built-in full-text index tokenises literals into a B+tree and is reached from SPARQL through magic predicates. It is off until `textIndex` is set. An external full-text service is documented separately. There is no vector index and no LLM tooling.

## Tooling

The Java API is the Sesame SAIL (the lineage that became [Eclipse RDF4J]({{ '/stores/rdf4j/' | relative_url }})) plus Blueprints. The workbench is the NanoSparqlServer UI. Python access is through a generic SPARQL client.

## Strengths

- A GPL server with SPARQL 1.1 Query, Update, federation and Service Description, plus materialised RDFS+.
- Quorum replication and a sharded scale-out design are both documented in the open-source tree.
- It is the engine the Wikidata Query Service ran on, so a large body of operational practice still refers to it.

## Caveats

- The repository is archived. 2.1.5 is seven years old, and the 2.1.6 candidate was never a release. Security fixes and SPARQL errata since 2019 are not in this tree.
- Quads mode does not do inference. GeoSPARQL, SHACL, RDF-star and RDF 1.2 are absent.
- Default writes are unisolated. Full read/write transactions are a configuration switch, and scale-out ACID is per shard.
- There is no official Docker image and no vendor-operated cloud service.

## Sources

<ul class="sources">
<li><a href="https://github.com/blazegraph/database">blazegraph/database</a> (GPL-2.0, archived 23 March 2026) and the <a href="https://github.com/blazegraph/database/releases/tag/BLAZEGRAPH_RELEASE_2_1_5">2.1.5 release</a> (19 March 2019; 50 billion edges)</li>
<li><a href="https://github.com/blazegraph/database/wiki/About_Blazegraph">About Blazegraph</a> (SPARQL 1.1 coverage, dual licence, HA and scale-out)</li>
<li><a href="https://github.com/blazegraph/database/wiki/">Project wiki home</a> (Wikidata Query Service; 50 billion statements)</li>
<li><a href="https://github.com/blazegraph/database/wiki/TxGuide">Transaction guide</a> and <a href="https://github.com/blazegraph/database/wiki/GroupCommit">group commit</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/HAJournalServer">HAJournalServer</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/PerformanceOptimization">Performance optimization</a> (scale-out journals and index segments) and <a href="https://github.com/blazegraph/database/wiki/IOOptimization">IO optimization</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/InferenceAndTruthMaintenance">Inference and truth maintenance</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/GeoSpatial">Geospatial</a> and <a href="https://github.com/blazegraph/database/wiki/FullTextSearch">full-text search</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/REST_API">REST API</a> and <a href="https://github.com/blazegraph/database/wiki/FederatedQuery">federated query</a></li>
<li><a href="https://github.com/blazegraph/database/wiki/Blazegraph_FAQ">FAQ</a> (quads and inference, many stores per journal)</li>
</ul>
