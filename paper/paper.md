---
title: "BioHackJP24 report: An InterMine for the IMG/VR database"
title_short: "BioHackJP24: IMG/VR InterMine"
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
  - name: Tomoya Tanjo
    orcid: 0000-0002-4421-9659
    affiliatnion: 5,6
  - name: Yui Asano
    Affiliation: 7
affiliations:
  - name: Kyoto University, Institute for Chemical Research, Japan
    index: 1
  - name: Genome Analytics Japan Inc., Japan
    index: 2
  - name: University of Cambridge, England
    index: 3
  - name: Database Center for Life Science, Japan
    index: 4
  - name: Bioinformation and DDBJ Center, Japan
    index: 5
  - name: BioData Science Initiative, Japan
    index: 6
  - name: Swallow Design Studio Inc., Japan
    index: 7
date: 31st August 2024
cito-bibliography: paper.bib
event: BH24JP
biohackathon_name: BioHackathon Japan 2024
biohackathon_url: https://biohackathon-europe.org/
biohackathon_location: Fukushima, Japan 2024
group: Viral Phylogenomics
git_url: https://github.com/biohackathon-japan/bh24-viral-phylogenomics
authors_short: Neches, Ishii, Micklem, Tanjo, Asano
---


# Introduction

[IMG/VR v4](https://doi.org/10.1093/nar/gkac1037) [@usesDataFrom:imgvr2022] is the fourth version of IMG/VR (Integrated Microbial Genomes/Virus), a large collection of viral sequences systematically identified from genomes, metagenomes, and metatranscriptomes, presented with their associated functional annotations, metadata, and taxonomic classification reflecting the latest standards at the time of publication. IMG/VR v4 contains over 15 million virus genomes and genome fragments, representing a 6-fold expansion relative to the previous release. IMG/VR provides 8.7 million viral operational taxonomic units (vOTUs), including 231,408 with at least one high-quality representative. Viruses were detected using [geNomad](https://doi.org/10.1038/s41587-023-01953-y), [@usesMethodIn:camargo2023identification] and genome quality was estimated using [CheckV](https://doi.org/10.1038/s41587-020-00774-7). [@usesMethodIn:nayfach2021checkv]

[InterMine](http://intermine.org) [smith2012intermine; kalderimis2014intermine] is an open source data warehouse system used for biological databases that provides a customizable, user-friendly [web interface](http://intermine.org/im-docs/docs/get-started/tutorial/index/), web service APIs and clients in multiple programming languages (Python, Perl, Ruby, Java, JavaScript, and R) that supports generalized queries and pre-made template queries licensed under the LGPL 2.1. Paired with [BlueGenes](https://github.com/intermine/bluegenes), the new user interface for InterMine that runs as its own service utilizing the InterMine web service API, users can upload and analyze lists of biological entities (e.g. genes, proteins) and generate high-quality visualization through configurable widgets and graphs. Databases available within the InterMine ecosystem can be found in the [InterMine Registry](https://registry.intermine.org/), and include [FlyMine](https://www.flymine.org/flymine), [HumanMine](https://www.humanmine.org), [MouseMine](https://www.mousemine.org/mousemine/begin.do), [WormBase/WormMine](http://intermine.wormbase.org/tools/wormmine/begin.do), and [PhytoMine](https://phytozome-next.jgi.doe.gov/phytomine/begin.do).

As part of the BioHackathon Japan 2024, we report the development `vrmine`, an experimental InterMine of providing the IMG/VR v4 data and metadata. 

# Results

While there are several documented ways to install InterMine, we recommend using a managed, containerized procedure using [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/install/). Docker Compose defines and runs multi-container Docker applications, managing an application stack (multiple containers with their associated storage volumes and network interfaces) in a single YAML configuration file. This approach reflects the way InterMine would be deployed in production. InterMine itself uses [Gradle](https://gradle.org) for build automation, and so installing, configuring and deploying InterMine consists mostly of orchestrating Gradle by means of Docker.

In addition to InterMine itself, BlueGenes provides a modern, module user interface for InterMine. BlueGenes is built with [Clojure](https://clojure.org/), a dynamic, general-purpose dialect of Lisp that runs on the Java Virtual Machine (JVM), and uses [Leiningen](https://leiningen.org/) for build automation and dependency management.

## The shortest path




# Discussion

## Challenges

The InterMine project is historically rooted the science of model organisms, such as *H. sapens*, *M. musculus*, *D. melanogaster*, and *C. elegans*. Consequently, many of the provided data models in InterMine are built to support the biological features of organisms belonging to these phylogenetic groups and data products developed by their respective research communities. Not only do the genomes of microorganisms and viruses exhibit different biological features, but they are obtained, sequenced and analyzed using different techniques, tools and statistical methods, often in the service of different research aims and scientific contexts. To accommodate viral MAGs (metagenomic assembled genomes), it is necessary to add some additional classes to InterMine's data model :

- OTU (operational taxonomic unit)
- Scaffold
- Host

Furthermore, some existing classes need to be extended or modified :

- Genome chemistry : DNA or RNA
- Genome topology : Linear, Circular, Endogenized

Fortunately, because InterMine's data model is represented as an extendable schema, it is possible to implement these extensions without modification of its source code.

## Future objectives

Predicted genes, gene annotations with [PFAM](https://www.ebi.ac.uk/interpro/) [@usesDataFrom:paysan2023interpro], [NCVOG viral ortholog groups](https://github.com/faylward/ncldv_markersearch) [@usesDataFrom:moniruzzaman2020dynamic], VOGDB viral ortholog group database [@usesDataFrom:trgovec2024vogdb], [GO](https://www.geneontology.org/) [@usesDataFrom:ashburner2000gene; @usesDataFrom:consortium2023gene] and [KEGG](https://www.genome.jp/kegg/pathway.html) [@usesDataFrom:kanehisa2016kegg].

## Acknowledgements

The authors would like to thank Jerven Bolleman in particular for asking the gently pointed questions that ultimately set the direction for this project, Ankur Kumar for his help and support, Toshiaki Katayama and Micheal Crusoe for bringing us together, Yui Asano for joining us to make a logo for the project on extremely short notice, and the DBCLS staff for hosting, organizing supporting the event that made this collaboration possible.
## References
