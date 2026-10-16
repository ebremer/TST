# RDF Triple Store Tracker

An open, evidence-based survey of RDF triple stores, published as a GitHub Pages site from [`docs/`](docs/).

**Site:** https://ebremer.github.io/TST/

October 2026 documentation survey of fifteen stores:

| Store | Profile |
|---|---|
| Apache Jena TDB2 | [docs/stores/jena-tdb2.md](docs/stores/jena-tdb2.md) |
| QLever | [docs/stores/qlever.md](docs/stores/qlever.md) |
| RDF-HDT | [docs/stores/hdt.md](docs/stores/hdt.md) |
| OpenLink Virtuoso | [docs/stores/virtuoso.md](docs/stores/virtuoso.md) |
| GraphDB (Graphwise) | [docs/stores/graphdb.md](docs/stores/graphdb.md) |
| Stardog | [docs/stores/stardog.md](docs/stores/stardog.md) |
| Amazon Neptune | [docs/stores/neptune.md](docs/stores/neptune.md) |
| AllegroGraph | [docs/stores/allegrograph.md](docs/stores/allegrograph.md) |
| Blazegraph | [docs/stores/blazegraph.md](docs/stores/blazegraph.md) |
| Eclipse RDF4J | [docs/stores/rdf4j.md](docs/stores/rdf4j.md) |
| Oxigraph | [docs/stores/oxigraph.md](docs/stores/oxigraph.md) |
| sparq | [docs/stores/sparq.md](docs/stores/sparq.md) |
| RDFox | [docs/stores/rdfox.md](docs/stores/rdfox.md) |
| MarkLogic Server | [docs/stores/marklogic.md](docs/stores/marklogic.md) |
| MillenniumDB | [docs/stores/millenniumdb.md](docs/stores/millenniumdb.md) |

## How the site is organised

The site is a plain Jekyll site with no external theme, so GitHub Pages builds it as-is.

| Path | What it holds |
|---|---|
| `docs/_data/stores.yml` | Every store's facts and its level for each capability. The matrix, cards and capability summaries are generated from it. |
| `docs/_data/features.yml` | The capabilities being compared, grouped. Add a row here and give every store a value for it. |
| `docs/_data/levels.yml` | Support levels (Yes, Partial, Paid tier, Preview, No, Unverified). |
| `docs/_data/candidates.yml` | Stores queued for future rounds. |
| `docs/stores/*.md` | One narrative profile per store, with sources. |
| `docs/comparison.html` | The comparison matrix. |
| `docs/methodology.md` | Selection criteria and how levels are assigned. |

### Adding a store

1. Add an entry to `docs/_data/stores.yml`, filling in every key from `features.yml`.
2. Create `docs/stores/<id>.md` with `layout: store` and `store: <id>` in its front matter.
3. List your sources at the bottom of the profile.

## Publishing

In the repository's **Settings → Pages**, choose **Deploy from a branch**, branch `main`, folder `/docs`.

## Previewing locally

You need Ruby and Bundler:

```sh
cd docs
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000/TST/.

## License

MIT. See [LICENSE](LICENSE).
