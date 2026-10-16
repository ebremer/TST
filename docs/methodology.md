---
layout: page
title: Methodology
eyebrow: About the track
lede: What counts as a "Yes", where the facts come from, and what this survey does not (yet) tell you.
description: How stores are selected and how each comparison cell is assigned.
---

## What the track is

The RDF Triple Store Tracker is a running survey of databases that store RDF and answer SPARQL. It is organised in **rounds**. Each round takes a set of stores, records the same facts about every one of them, and publishes the result as a comparison matrix and one profile per store. Later rounds add stores and, eventually, hands-on measurements.

The published survey is a **documentation survey**: every fact comes from the steward's own documentation, release notes, source repository or pricing pages. Nothing in it is benchmarked.

## How the stores were chosen

The first five stores were picked using four criteria, applied in order:

1. **Popularity.** Drawn from the top of the [DB-Engines RDF store ranking](https://db-engines.com/en/ranking/rdf+store) in early 2026.
2. **RDF first.** SPARQL and RDF are a primary interface, not a secondary model bolted onto a document or relational database. This is why MarkLogic, which leads the DB-Engines category by score, was left out of the first five. It is surveyed below, with that limit stated.
3. **Maintained.** At least one release in the last twelve months.
4. **Different shapes.** The set should cover the main ways people run a triple store:
   - an open-source embedded store and server (Apache Jena TDB2)
   - a hybrid open-source/commercial server (Virtuoso)
   - commercial servers with free tiers (GraphDB, Stardog)
   - a fully managed cloud service (Amazon Neptune)

Two more were then added to cover cases the first five miss:

- **QLever**, for very large datasets on a single machine. It is now the engine behind the UniProt endpoint and the chosen successor to Blazegraph for the Wikidata Query Service.
- **RDF-HDT**, for read-only data shipped as compressed, self-indexed files.

HDT is a format with several engines rather than a single product. Each of its matrix cells names the component that provides the capability, and the profile marks which components are actively maintained.

**AllegroGraph** was then taken from the queue. It is a long-running commercial store with a free edition, a recent release, and a shape the first seven do not cover: partitioned writes through FedShard, alongside query-time reasoning.

**Eclipse RDF4J** and **Blazegraph** were taken from the queue after that. RDF4J is the Java framework and embedded store that several other engines build on, and release 6.1 includes RDF 1.2 triple terms. Blazegraph is the engine that ran the Wikidata Query Service. Its last stable release is 2.1.5 (March 2019) and the repository is archived, so it does not meet the maintenance criterion above. It is included because deployments still run it, and because its replication and scale-out designs are part of the public record.

**Oxigraph** was taken from the queue after that. It is a Rust store and RDF toolkit, embeddable from Rust, Python and JavaScript or run as a small SPARQL server. Release 0.5.11 (2 September 2026) is current, and the command-line build turns on preliminary RDF 1.2 draft support by default.

**sparq** was added from its public repository, [sparq-org/sparq](https://github.com/sparq-org/sparq). It is an experimental Rust triplestore and SPARQL engine, MIT-licensed, with release 0.1.6 (13 October 2026). The README calls the API unstable and pre-1.0. The default workspace build includes materialised RDFS, OWL 2 RL and Notation3.

**RDFox** was taken from the queue after that. It is Oxford Semantic Technologies' in-memory store and Datalog reasoner, now part of Samsung. Release 7.7 (29 September 2026) materialises an extended rule language, including OWL 2 RL and SWRL, and maintains the closure as the data changes. A time-limited licence key is required to start it.

**MarkLogic Server** was taken from the queue after that, despite the RDF-first criterion above. It is a multi-model document database from Progress Software. Triples are an index over documents, and it leads the DB-Engines RDF ranking by score. Release 12.1.0 (10 September 2026) is the current innovation line. The free developer licence includes every feature, including clustering and high availability, up to 1 TB, for development only.

**MillenniumDB** was taken from the queue after that. It is a research graph engine from the Millennium Institute for Foundational Research on Data (IMFD). An RDF database is queried with SPARQL, and two property-graph models are queried with MQL or an early GQL. The README says the project is still in active development and is not production ready. The last commit on the default branch is 30 September 2026, so it meets the maintenance rule. The limit to state is the README's warning that it is not production ready. Tag 1.0.0 (17 August 2026) is the version the program prints.

Stores that missed the cut are listed on the [stores page]({{ '/stores/' | relative_url }}).

## How a cell is assigned

Every capability in the matrix gets one level for the **generally available** product:

| Level | Meaning |
|---|---|
| <span class="lvl lvl-full">Yes</span> | Documented and supported in the free or base product, including official modules that ship with it. |
| <span class="lvl lvl-partial">Partial</span> | Works with notable gaps, needs a separate add-on, or covers only part of the standard. The cell note says which. |
| <span class="lvl lvl-paid">Paid tier</span> | Exists, but only in a commercial or enterprise edition. |
| <span class="lvl lvl-preview">Preview</span> | Experimental, beta, or behind a feature flag. |
| <span class="lvl lvl-none">No</span> | Not supported. |
| <span class="lvl lvl-unknown">Unverified</span> | Not confirmed from primary sources yet. |

A few rules keep the levels consistent:

- **The free edition is the baseline.** If a feature needs a paid licence, it is marked *Paid tier* even if the paid edition does it well. Amazon Neptune has no free edition, so it is judged on the service as sold.
- **Standards conformance is taken from documentation, not tested.** "Yes" for SPARQL 1.1 means the steward says it implements the specification, not that it passes every W3C test. Known deviations go in the note.
- **RDF-star and RDF 1.2 are kept apart.** RDF-star (the 2021 community group report) and RDF 1.2 "triple terms" (the W3C working group's revision) differ in their details, so a store can support one without the other.
- **Reasoning is described by strategy as well as level.** Forward-chaining materialisation (computed at load time) and query-time rewriting have very different costs, so the matrix records which one each store uses.

## Sources and freshness

Each store profile ends with the list of URLs its facts came from. The date on every profile and in the footer is when those facts were last checked. Versions, licences and pricing change often. Confirm anything important against the vendor before relying on it.

Scale figures, such as "N billion triples", are the steward's own published claims. They have not been reproduced here.

GitHub star counts were read from the public GitHub API on the checked date. The number is the stargazer count of the repository named in the cell. A dash means the database engine itself is not published on GitHub; the note then gives the count for a public client or workbench repository when one is linked from the profile. Counts change often, and they are not a measure of quality.

## Known limitations

- **No performance data.** Load and query speed depend heavily on data shape, hardware and configuration. Comparing vendor-published numbers would be misleading, so the track does not.
- **Feature lists flatten nuance.** A single "Yes" for OWL hides which profile, how complete, and at what cost. The profiles carry the detail.
- **Documentation can be wrong or stale.** Corrections are welcome as GitHub issues and pull requests.
