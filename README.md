# Ngordnet — NGram & WordNet Explorer

A Java-based web application for exploring how English words change over time and how words are related through the WordNet semantic hierarchy.

Built as part of **UC Berkeley CS 61B Project 2A/2B**, this project combines historical language data, time-series analysis, directed graphs, and browser-based query handling.

## Overview

Ngordnet provides an interactive interface for querying two complementary datasets:

* **Google NGram data** — historical word frequency and popularity over time.
* **WordNet** — semantic relationships between words, including synsets and hyponyms.

The application exposes these capabilities through a browser interface where users can enter words, year ranges, and ranking parameters.

## Screenshots

### Hyponyms

A basic hyponym query returns the WordNet-related terms associated with a word.

<img width="731" height="726" alt="hyponymskblank" src="https://github.com/user-attachments/assets/ab5ebbcd-9bd4-4d6c-8000-94af697a3aac" />

### Ranked Hyponyms

With `k = 5`, the application combines WordNet relationships with historical NGram frequency.

<img width="722" height="729" alt="hyponyms5" src="https://github.com/user-attachments/assets/18751905-c672-4e94-adea-3efacae56f2b" />


### Common Ancestors

The application can also expose graph relationships through common-ancestor queries.

<img width="722" height="728" alt="commonancestorskblank" src="https://github.com/user-attachments/assets/d0ee350f-8a51-40f4-9af2-a775c3e32a9b" />

### Ranked Common Ancestors

The interface supports the `k` parameter for graph-based ranked queries as well.

<img width="722" height="728" alt="commonancestorsk5" src="https://github.com/user-attachments/assets/a648cdda-5314-4170-84de-f18d9aa095f1"/>

## Measurable Result

The completed application demonstrates:

* Historical queries across a **21-year range** (`2000–2020`)
* WordNet hyponym traversal
* `k = 5` ranked queries
* Browser-based interaction through a local Java web server
* Integration of **two independent datasets**: NGram and WordNet
* In-memory data structures designed to avoid reparsing the datasets for every query


| Metric                                 | Result          |
| ----------------------------           | --------        |
| NGram and WordNet datasets load time   | `49393.9939 ms` | 
| Average hyponym query                  | `13.5321 ms`    |
| Average common ancestor query          | `50.56724 ms`   |
| Average hyponym query (`k=5`)          | `20.7950 ms`    |
| Average common ancestor query (`k=5`)  | `138.75708 ms`  |

## What I Learned

This project strengthened my understanding of:

* Directed graphs and graph traversal
* Hash-based and tree-based data structures
* Time-series data modeling
* Parsing large structured datasets
* Algorithmic complexity and performance
* Separation of data, logic, and HTTP request handling
* Designing helper classes instead of placing all logic in request handlers
* Integrating multiple datasets into a single application

## Tech Stack

* **Java**
* **JUnit**
* **HTML / JavaScript**
* **Jetty-based local web server**
* **Google NGram dataset**
* **WordNet dataset**
* **Git**

## Project Context

This project is based on UC Berkeley CS 61B's Ngordnet assignments:

* Project 2A — NGrams
* Project 2B — WordNet

The implementation extends the supplied browser interface with data-processing and graph functionality.

---------------------------------------------------------------------------------------------------------------------------------------------------

# Following the 61B Technical Specification

## Features

### Historical word analysis

Query the historical popularity of one or more words over a selected year range.

Example:

* Words: `sun`
* Start year: `2000`
* End year: `2020`

The application returns the corresponding historical data for the requested word.

### History text queries

The `History (Text)` handler converts NGram data into a readable time-series representation.

Example output:

```text
sun: {2000=..., 2001=..., ..., 2020=...}
```

### WordNet hyponyms

The `Hyponyms` handler searches the WordNet hierarchy and returns words that represent more specific concepts beneath a queried word.

For example, a query for `sun` returns related WordNet terms such as:

```text
[sun, sunburst, sunlight, sunshine]
```

### Ranked hyponyms

The `k` parameter limits the output to the most frequently occurring hyponyms within the requested time range.

Example:

```text
words = sun
start = 2000
end = 2020
k = 5
```

This combines WordNet graph traversal with NGram frequency data to determine which candidate hyponyms are most relevant historically.

### Browser-based interface

The project runs a local Java web server and exposes the query functionality through an HTML interface.

Supported operations include:

* History (Text)
* Hyponyms
* Hyponym ranking with `k`
* Common ancestor queries

## Architecture

```text
                         Browser UI
                             │
                             ▼
                    NgordnetServer / Handlers
                             │
             ┌───────────────┼────────────────┐
             │               │                │
             ▼               ▼                ▼
        History         Hyponyms        Common Ancestors
        Handler          Handler             Handler
             │               │                │
             ▼               ▼                ▼
         NGramMap        WordNet          Graph Traversal
             │               │
             ▼               ▼
        TimeSeries      Directed Graph
             │               │
             └───────────────┼────────────────┘
                             ▼
                       Data Files
                    ┌────────┴────────┐
                    │                 │
                 NGrams            WordNet
```

### Core components

**`TimeSeries`**

Represents year-to-value mappings and provides operations needed for historical analysis.

**`NGramMap`**

Loads the NGram datasets and provides efficient historical frequency and weighted-popularity queries.

**`HistoryTextHandler`**

Converts NGram history results into the textual format expected by the browser.

**`HistoryHandler`**

Generates the visual history response for the browser.

**`Graph`**

Represents the directed WordNet relationship structure.

**`WordNet`**

Parses synsets and hyponym relationships and provides word-to-synset and graph-based lookup operations.

**`HyponymsHandler`**

Processes browser queries, performs WordNet traversal, removes duplicates, sorts results, and optionally ranks candidates using NGram frequency data.

## Technical Approach

### NGram processing

Rather than repeatedly scanning the source files for every browser request, the data is parsed during initialization and represented using in-memory data structures.

This makes subsequent queries substantially cheaper because the application can perform direct lookups and time-series operations rather than reparsing the dataset.

### WordNet graph

WordNet is modeled as a directed graph:

```text
Hypernym
   │
   ▼
Hyponym
   │
   ▼
More specific hyponym
```

Each synset corresponds to a graph node, while entries in the hyponym data define directed edges between synsets.

A graph traversal identifies all reachable synsets for a requested word.

The resulting synsets are converted back into words, deduplicated, and sorted.

### Combining WordNet and NGram data

The most interesting part of the project is the `k` query.

The application first determines which words are valid hyponyms using the WordNet graph. It then uses NGram frequency data to rank those candidates over the requested time period.

Conceptually:

```text
User query
    │
    ▼
Find WordNet synsets
    │
    ▼
Traverse hyponym graph
    │
    ▼
Collect candidate words
    │
    ▼
Calculate NGram frequency
    │
    ▼
Select top k
    │
    ▼
Alphabetically sorted result
```

This combines two different data-processing problems: semantic graph traversal and historical frequency analysis.
