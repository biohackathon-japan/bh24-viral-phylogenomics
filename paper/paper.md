---
title: 'BioHackJP24 report: An InterMine for the IMG/VR database'
title_short: 'BioHackJP24: IMG/VR InterMine'
tags:
  - virology
  - databases
  - genomics
  - ecology
  - microbiology
authors:
  - name: Russell Neches
    orcid: 0000-0002-2055-8381
    affiliation: 1
  - name: Manabu Ishii
    orcid: 0000-0002-5843-4712
    affiliation: 2
  - name: Gos Micklem
    orcid: 0000-0002-6883-6168
    affiliation: 3,4
affiliations:
  - name: Kyoto University, Institute for Chemical Research
    index: 1
  - name: Genome Analytics Japan Inc.
    index: 2
  - name: University of Cambridge
    index: 3
  - name: DBCLS
    index: 4
date: 30 September 2024
cito-bibliography: paper.bib
event: BH24JP
biohackathon_name: "BioHackathon Japan 2024"
biohackathon_url:   "https://biohackathon-europe.org/"
biohackathon_location: "Fukushima, Japan 2024"
group: Project 26
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/bh24-viral-phylogenomics
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Neches, Ishii, Micklem
---


# Introduction

As part of the BioHackathon Europe 2023, we here report...

# Formatting

This document use Markdown and you can look at [this tutorial](https://www.markdowntutorial.com/).

## Subsection level 2

Please keep sections to a maximum of only two levels.

## Tables and figures

Tables can be added in the following way, though alternatives are possible:

Table: Note that table caption is automatically numbered and should be
given before the table itself.

| Header 1 | Header 2 |
| -------- | -------- |
| item 1 | item 2 |
| item 3 | item 4 |

A figure is added with:

![Caption for BioHackrXiv logo figure](./biohackrxiv.png)

# Other main section on your manuscript level 1

Lists can be added with:

1. Item 1
2. Item 2

# Citation Typing Ontology annotation

You can use [CiTO](http://purl.org/spar/cito/2018-02-12) annotations, as explained in [this BioHackathon Europe 2021 write up](https://raw.githubusercontent.com/biohackrxiv/bhxiv-metadata/main/doc/elixir_biohackathon2021/paper.md) and [this CiTO Pilot](https://www.biomedcentral.com/collections/cito).
Using this template, you can cite an article and indicate _why_ you cite that article, for instance DisGeNET-RDF [@citesAsAuthority:Queralt2016].

The syntax in Markdown is as follows: a single intention annotation looks like
`[@usesMethodIn:Krewinkel2017]`; two or more intentions are separated
with colons, like `[@extends:discusses:Nielsen2017Scholia]`. When you cite two
different articles, you use this syntax: `[@citesAsDataSource:Ammar2022ETL; @citesAsDataSource:Arend2022BioHackEU22]`.

Possible CiTO typing annotation include:

* citesAsDataSource: when you point the reader to a source of data which may explain a claim
* usesDataFrom: when you reuse somehow (and elaborate on) the data in the cited entity
* usesMethodIn
* citesAsAuthority
* citesAsEvidence
* citesAsPotentialSolution
* citesAsRecommendedReading
* citesAsRelated
* citesAsSourceDocument
* citesForInformation
* confirms
* documents
* providesDataFor
* obtainsSupportFrom
* discusses
* extends
* agreesWith
* disagreesWith
* updates
* citation: generic citation


# Results


# Discussion

...

## Acknowledgements

...

## References
