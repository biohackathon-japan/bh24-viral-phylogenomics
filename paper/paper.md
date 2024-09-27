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
    affiliation: 3, 4
  - name: Tomoya Tanjo
    orcid: 0000-0002-4421-9659
    affiliation: 5, 6
  - name: Yui Asano
    affiliation: 7
affiliations:
  - name: Kyoto University, Institute for Chemical Research, Japan
    index: 1
  - name: Genome Analytics Japan Inc., Japan
    index: 2
  - name: Department of Genetics, University of Cambridge, Cambridge CB2 3EH, United Kingdom
    index: 3
  - name: Database Center for Life Science (DBCLS), University of Tokyo Kashiwa-no-ha Campus Station Satellite 6F. 178-4-4 Wakashiba, Kashiwa-shi, Chiba, Japan
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
biohackathon_url: https://2024.biohackathon.org/
biohackathon_location: Fukushima, Japan 2024
group: Viral Phylogenomics
git_url: https://github.com/biohackathon-japan/bh24-viral-phylogenomics
authors_short: Neches, Ishii, Micklem, Tanjo, Asano
---


# Introduction

[IMG/VR v4](https://doi.org/10.1093/nar/gkac1037) [@usesDataFrom:imgvr2022] is the fourth version of IMG/VR (Integrated Microbial Genomes/Virus), a large collection of viral sequences systematically identified from genomes, metagenomes, and metatranscriptomes, presented with their associated functional annotations, metadata, and taxonomic classification reflecting the latest standards at the time of publication. IMG/VR v4 contains over 15 million virus genomes and genome fragments, representing a 6-fold expansion relative to the previous release. IMG/VR provides 8.7 million viral operational taxonomic units (vOTUs), including 231,408 with at least one high-quality representative. Viruses were detected using [geNomad](https://doi.org/10.1038/s41587-023-01953-y), [@usesMethodIn:camargo2023identification] and genome quality was estimated using [CheckV](https://doi.org/10.1038/s41587-020-00774-7). [@usesMethodIn:nayfach2021checkv]

[InterMine](http://intermine.org) [smith2012intermine; kalderimis2014intermine] is an open source (LGPL 2.1) data warehouse system used for biological databases that provides a customizable, user-friendly [web interface](http://intermine.org/im-docs/docs/get-started/tutorial/index/), web service APIs and clients in multiple programming languages (Python, Perl, Ruby, Java, JavaScript, and R) that supports generalized queries and pre-made template queries. Paired with [BlueGenes](https://github.com/intermine/bluegenes), the new user interface for InterMine that runs as its own service utilizing the InterMine web service API, users can upload and iteratively analyze lists of biological entities (e.g. genes, proteins) and generate high-quality visualization through configurable widgets and graphs. Databases available within the InterMine ecosystem can be found in the [InterMine Registry](https://registry.intermine.org/), and include [HumanMine](https://www.humanmine.org), [FlyMine](https://www.flymine.org/flymine), [TargetMine](https://targetmine.mizuguchilab.org/bluegenes), [MouseMine](https://www.mousemine.org/mousemine/begin.do), [WormBase/WormMine](http://intermine.wormbase.org/tools/wormmine/begin.do), and [PhytoMine](https://phytozome-next.jgi.doe.gov/phytomine/begin.do).

As part of the BioHackathon Japan 2024, we report the development of `vrmine`, an experimental InterMine providing the IMG/VR v4 data and metadata. 

# Results

While there are several documented ways to install InterMine, we recommend using a managed, containerized procedure using [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/install/). Docker Compose defines and runs multi-container Docker applications, managing an application stack (multiple containers with their associated storage volumes and network interfaces) in a single YAML configuration file. This approach reflects the way InterMine would be deployed in production. InterMine itself uses [Gradle](https://gradle.org) for build automation, and so installing, configuring and deploying InterMine consists mostly of orchestrating Gradle by means of Docker.

In addition to InterMine itself, BlueGenes provides a modern, modular user interface for InterMine. BlueGenes is built with [Clojure](https://clojure.org/), a dynamic, general-purpose dialect of Lisp that runs on the Java Virtual Machine (JVM), and uses [Leiningen](https://leiningen.org/) for build automation and dependency management.

## Requirements

Before proceeding, please verify that you have Docker and Docker Compose installed correctly. These instructions were developed using Docker version 27.2.0, build 3ab4256. You may verify your version like so :

```console=
docker --version 
```

To make sure Docker is installed correctly, you may test it by running the `hello-world` image.

```console=
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## The shortest path

InterMine has a lot of moving parts, and so a minimal configuration is useful to get the dependencies sorted out and the software up and running. In this example, we'll reuse some of components from the [biotestmine](https://github.com/intermine/biotestmine) example, but we'll make our own minimal instance.

### Step 1 : Clone repositories

First, clone [intermine-scripts](https://github.com/intermine/intermine-scripts.git) repository from GitHub, which contains scripts for initializing, managing, migrating and deploying InterMine instances. We will be using `make_mine`. 

```console=
git clone https://github.com/intermine/intermine-scripts.git
```

Next, clone the [docker-intermine-gradle](https://github.com/intermine/docker-intermine-gradle) repository, which contains a collection of Docker image definitions that contain the major components of InterMine, [Apache Tomcat](https://tomcat.apache.org/), [Apache Solr](https://solr.apache.org/), and [PostgreSQL](https://www.postgresql.org/) :

```console=
git clone https://github.com/intermine/docker-intermine-gradle.git
```

### Step 2 : Create an InterMine

Using the `make_mine` script from [intermine-scripts](https://github.com/intermine/intermine-scripts.git), make a template for your mine. This will create the necessary configuration files for your InterMine instance.

```console=
intermine-scripts/make_mine YourMine
```

At this point, we recommend that you put your InterMine under source control.

```console=
git remote add origin git@github.com:YOURNAME/YOURMINE.git
git add .
git commit -m 'Initial commit'
git push origin main
```

### Step 3 : Stage data

FIXME : What's in `all_uvigs.4.*.gff3`? There seems to be a disconnect between this step and the ones before, with the name changing from YourMine to virus.

```console=
mkdir docker-intermine-gradle/data/mine/data/virus/genome/gff
cp all_uvigs.4.*.gff3 docker-intermine-gradle/data/mine/data/virus/genome/gff
```

Next, create `project.xml`

FIXME : This step needs to follow the same names as the above steps (YourMine, vs. virus)

FIXME : What lines were edited, and why? This should probably be a diff

```xml
<project type="bio">
  <property name="target.model" value="genomic"/>
  <property name="common.os.prefix" value="common"/>
  <property name="intermine.properties.file" value="virusmine.properties"/>
  <property name="default.intermine.properties.file" location="../default.intermine.integrate.properties"/>

  <sources>
    <source name="malaria-gff" type="malaria-gff">
      <property name="gff3.taxonId" value="36329"/>
      <property name="gff3.seqDataSourceName" value="PlasmoDB"/>
      <property name="gff3.dataSourceName" value="PlasmoDB"/>
      <property name="gff3.seqClsName" value="Chromosome"/>
      <property name="gff3.dataSetTitle" value="PlasmoDB P. falciparum genome"/>
      <property name="src.data.dir" location="DATA_DIR/virus/genome/gff"/>
    </source>
    <source name="malaria-chromosome-fasta" type="fasta" dump="true">
      <property name="fasta.className" value="org.intermine.model.bio.Chromosome"/>
      <property name="fasta.dataSourceName" value="PlasmoDB"/>
      <property name="fasta.dataSetTitle" value="PlasmoDB chromosome sequence"/>
      <property name="fasta.taxonId" value="36329"/>
      <property name="fasta.includes" value="VR*.fasta"/>
      <property name="src.data.dir" location="DATA_DIR/virus/genome/fasta"/>
    </source>
    <source name="my-data-source" type="delimited">
      <property name="delimited.dataSourceName" value="TSV Source Name"/>
      <property name="delimited.dataSetTitle" value="TSV Data Set"/>
      <property name="delimited.licence" value="http//usemydatalicence.com"/>
      <property name="delimited.hasHeader" value="true"/>
      <property name="delimited.columns" value="Gene.primaryIdentifier, Organism.taxonId, null,Protein.primaryIdentifier,Protein.primaryAccession"/>
      <property name="delimited.includes" value="test.tsv"/>
      <property name="src.data.dir" location="DATA_DIR/virus/tsv"/>
    </source>

  </sources>

  <post-processing>
    <post-process name="create-references"/>
    <post-process name="transfer-sequences" dump="true"/>

    <!-- this runs the postprocessing steps in all sources -->
    <post-process name="do-sources"/>

    <post-process name="summarise-objectstore"/>
    <post-process name="create-autocomplete-index"/>
    <post-process name="create-search-index"/>
  </post-processing>
</project>
```

### Step 4 : Launch InterMine

Now that everything has been configured, we can use docker compose to bring up the containers that host the components of Intermine :

```console=
docker compose -f local.docker-compose.yml up --build --force-recreate
```

Finally, you can access your new Intermine by pointing a web browser at this URL : 

```url
http://localhost:9999/yourmine
```

FIXME : add screenshot?
FIXME : how do we see that data is loaded using the web interface?
## Loading IMG/VR data into Intermine

To make this actually useful, we made a preliminary effort to load IMG/VR into Intermine. IMG/VR is provided as a pair of FASTA files, one containing viral contigs, and the other containing predicted viral proteins, and a pair of TSV files containing the sequence metadata and predicted hosts.

```console=
mkdir work
cd work
git clone https://github.com/intermine/intermine-scripts.git
intermine-scripts/make_mine YOURMINE
cd YOURMINE
git init .
git branch
git branch -M main
```

Create repository for YOURMINE. Here, we show creation of a bare repository.

```console=
git remote add origin git@github.com:YOURNAME/YOURMINE.git
git add .
git commit -m 'Initial commit'
```

Download the properties file from biotestmine to use as a template.

```console=
mkdir data
wget -O data/YOURMINE.properties https://raw.githubusercontent.com/intermine/biotestmine/master/data/biotestmine.properties
```

Next we will need to edit the properties file. There are several changes to make, so please refer to this diff :

```diff
$ diff -u data/YOURMINE.properties data/sndmine.properties

 # Access to the postgres database to build into and access from the webapp
 db.production.datasource.serverName=localhost
-db.production.datasource.databaseName=biotestmine
+db.production.datasource.databaseName=PSQL_DB_NAME
 db.production.datasource.user=PSQL_USER
 db.production.datasource.password=PSQL_PWD

@@ -21,7 +21,7 @@

 # common target items database
 db.common-tgt-items.datasource.serverName=localhost
-db.common-tgt-items.datasource.databaseName=items-biotestmine
+db.common-tgt-items.datasource.databaseName=items-YOURMINE
 db.common-tgt-items.datasource.user=PSQL_USER
 db.common-tgt-items.datasource.password=PSQL_PWD

@@ -29,7 +29,7 @@
 # userprofile database - used by the webapp to store logins, query history,
 # saved bags, templates and tags.
 db.userprofile-production.datasource.serverName=localhost
-db.userprofile-production.datasource.databaseName=userprofile-biotestmine
+db.userprofile-production.datasource.databaseName=userprofile-YOURMINE
 db.userprofile-production.datasource.user=PSQL_USER
 db.userprofile-production.datasource.password=PSQL_PWD

@@ -44,7 +44,7 @@

 # location of tomcat server and path of webapp - e.g. access http://localhost:8080/malariamine
 webapp.deploy.url=http://localhost:8080
-webapp.path=biotestmine
+webapp.path=YOURMINE

 # tomcat username and password needed to deploy webapp
 webapp.manager=TOMCAT_USER
@@ -63,12 +63,12 @@
 # details for sending login e-mails
 mail.host=localhost
 mail.from=account@my_mail_host
-mail.subject=Welcome to MalariaMine
-mail.text=You have successfully created an account on BioTestMine
+mail.subject=Welcome to YOURMINE
+mail.text=You have successfully created an account on YOURMINE

 # text that appears in the header and elsewhere
-project.title=BioTestMine
-project.subTitle=An example of InterMine.bio with data from <i>Plasmodium falciparum</i>
+project.title=YOURMINE
+project.subTitle=An example of InterMine.bio with data from <i>YOURMINE</i>
 project.releaseVersion=tutorial

 # various URLs use this as the prefix
```

Save, commit and push the properties file to GitHub.

```console=
git add data/YOURMINE.properties
git commit -m 'Update properties file'
git push origin main
```

The container that we will use does not have bash, and so we need to update the gradlew script in the YOURMINE repo accordingly.

```console=
wget -O gradlew https://raw.githubusercontent.com/manabuishii/dragonmine/main/gradlew
git add gradlew
git commit -m 'Update gradlew'
git push origin main
```

Next, create `data/project.xml` and create a symbolic link `./project.xml`. We are not sure why there are two of these files, so we link them together to avoid having conflicting information between them.

```
wget -O data/project.xml https://raw.githubusercontent.com/YOURNAME/YOURMINE/master/data/project.xml

rm project.xml
ln -s data/project.xml .
```

FIXME : diff for lines to edit in data/project.xml

FIXME: write about prepare data. but it appear in later maybe.

Next, clone docker-intermine-gradle

```
cd ~/work
git clone git@github.com:intermine/docker-intermine-gradle.git
```

We are going to load some real data into this instance, so we need to increase the memory allocation accordingly.

FIXME : This should be a diff on `intermine_builder/intermine_builder.Dockerfile`

ENV MEM_OPTS="-Xmx1g -Xms500m" -> ENV MEM_OPTS="-Xmx16g -Xms500m"

FIXME : this should be a diff on `local.docker-compose.yml`

```
# # - ./data/mine/data:/home/intermine/intermine/data


# these lines (3 times appear) comment out `user: ${UID:-1000}:${GID:-1000}`

# edit this line
# - ./data/mine/[PUT_YOUR_MINE_NAME_HERE]:/home/intermine/intermine/[PUT_YOUR_MINE_NAME_HERE]
# into
# - ./data/mine/sndmine:/home/intermine/intermine/SndMine
# and delete next line
# - ./data/mine/biotestmine:/home/intermine/intermine/biotestmine


# - MINE_NAME=${MINE_NAME:-biotestmine}
# - MINE_NAME=${MINE_NAME:-sndmine}

# - MINE_REPO_URL=${MINE_REPO_URL:-}
# change it to your repository , next line is example
# - MINE_REPO_URL=${MINE_REPO_URL:-https://github.com/manabuishii/sndmine.git}


# this is not needed ?
# - IM_REPO_URL=${IM_REPO_URL:-}
# - IM_REPO_URL=${IM_REPO_URL:-https://github.com/manabuishii/intermine.git}

# change memory
# - MEM_OPTS=${MEM_OPTS:-"-Xmx2g -Xms1g"}
# - MEM_OPTS=${MEM_OPTS:-"-Xmx16g -Xms1g"}

```

It is extremely important to clear the staged data when you rebuild the Intermine! The directory `docker-intermine-gradle/data/mine/YOURMINE` is cloned by `intermine_builder/build.sh`. If this directory exists, `intermine_builder` will start `tomcat(cargoDeploy)`. Changes to your Intermine configuration or data will have no effect.

```console=
rm -rf ~/work/docker-intermine-gradle/data/mine/YOURMINE
```

Alternatively, if you prefer to always rebuild everything, including the database, set `FORCE_MINE_BUILD`  in `local.docker-compose.yml` of `intermine_builder` 's `environment` section. Note that although `docker-compose` has the option `--force-recreate`, but it seems to only apply to the container itself.


## TASKs for later

- [ ] Does proejct.xml require into two directory ?
- [ ] Force Rebuild option work for `docker compose`


# Discussion

## Challenges

The InterMine project is historically rooted in the science of model organisms, such as *H. sapiens*, *M. musculus*, *D. melanogaster*, and *C. elegans*. Consequently, many of the provided data models in InterMine are built to support the biological features of organisms belonging to these phylogenetic groups and data products developed by their respective research communities. Not only do the genomes of microorganisms and viruses exhibit different biological features, but they are obtained, sequenced and analyzed using different techniques, tools and statistical methods, often in the service of different research aims and scientific contexts. To accommodate viral MAGs (metagenomic assembled genomes), it is necessary to add some additional classes to InterMine's data model, including members of the [Sequence Ontology](http://www.sequenceontology.org) that were previously ommitted:

- OTU (operational taxonomic unit)
- Scaffold - seems to be synonym of this in Sequence Ontology: http://sequenceontology.org/browser/current_release/term/SO:0000148
- Host

 ?? Some of these SO terms? http://www.sequenceontology.org/browser/current_release/term/SO:0000149
  

Furthermore, some existing classes need to be extended or modified :

- Genome chemistry : DNA or RNA
- Genome topology : Linear, Circular, Endogenized

Fortunately, because InterMine's data model is represented as an extendable schema, it is possible to implement these extensions without modification of its source code, with the changes automatically reflected in the database structure and user interface. 

## Future objectives

We hope to add predicted genes, gene annotations with [PFAM](https://www.ebi.ac.uk/interpro/) [@usesDataFrom:paysan2023interpro], [NCVOG viral ortholog groups](https://github.com/faylward/ncldv_markersearch) [@usesDataFrom:moniruzzaman2020dynamic], VOGDB viral ortholog group database [@usesDataFrom:trgovec2024vogdb], [GO](https://www.geneontology.org/) [@usesDataFrom:ashburner2000gene; @usesDataFrom:consortium2023gene] and [KEGG](https://www.genome.jp/kegg/pathway.html) [@usesDataFrom:kanehisa2016kegg] to vrMine.

# Acknowledgements

The authors would like to thank Jerven Bolleman in particular for asking the gently pointed questions that ultimately set the direction for this project, Ankur Kumar for his help and support, Toshiaki Katayama and Micheal Crusoe for bringing us together, Yui Asano for joining us to make a logo for the project on extremely short notice, and the DBCLS staff for hosting, organizing and supporting the BioHackathon 2024 event that made this collaboration possible.

## References
