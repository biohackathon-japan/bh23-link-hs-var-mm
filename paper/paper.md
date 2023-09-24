---
title: 'BioHackJP 2023 Report R1: Mapping human genome variations to their mouse counterparts for identifying disease model mouse strains with human genome variations'
title_short: 'BioHackJP 2023 MAP-HS-VAR-MM'
tags:
  - Human Disease Model Mouse
  - Genome Variation
authors:
  - name: Nobutaka Mitsuhashi
    orcid: 0000-0003-3300-7308
    affiliation: 1
  - name: Hirokazu Chiba
    orcid: 0000-0003-4062-8903
    affiliation: 1
  - name: Yuki Moriya
    orcid: 0000-0001-8195-5893
    affiliation: 1
  - name: Toyoyuki Takada
    orcid: 0000-0001-6796-2085
    affiliation: 2
affiliations:
  - name: Database Center for Life Science, Joint Support-Center for Data Science Research, Research Organization of Information and Systems
    index: 1
  - name: Integrated Bioresource Information Division, RIKEN BioResource Research Center
    index: 2
date: 30 June 2023
cito-bibliography: paper.bib
event: BH23JP
biohackathon_name: "BioHackathon Japan 2023"
biohackathon_url:   "https://2023.biohackathon.org/"
biohackathon_location: "Kagawa, Japan, 2023"
group: R1
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-jp/bh23-link-hs-var-mm
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Mitsuhashi, N. \emph{et al.}
---

# Background

Experimental mice are widely used in human disease studies. Since the inception of mouse genetic research, hundreds of diverse strains have been established for biomedical research. In disease model mouse strains, information on genomic variations is essential for elucidating the relationship between haplotypes and disease susceptibility. To select a disease model mouse appropriately, it is crucial to identify mouse variants with the same effect as disease-causing variants in humans. Homologous human and mouse genes have been identified and shared in integrated human and model organism databases [@wang2017][@homologene]. At the genetic variant level, however, the explicit relationships between human variants and their mouse counterparts have yet to be included in these databases. A previous study successfully mapped human pathogenic variants mostly found in conserved coding regions to orthologous positions in the cow and pig genomes [@zhao2022]. In BH2023, we aimed to map human pathogenic variants to orthologous positions in the mouse genome based on the sequence similarity between the two species.

# Outcomes

Using the reference sequences of humans and mice, we compared the regions encoding proteins of homologous genes. First, we focused on nucleotide variants involved in amino acid substitutions. We have developed an API that returns mouse variants and strains in MoG+ as counterparts to ClinVar variants located within the gene region specified by an HGNC gene symbol. Target human variants can be limited to variants with ClinVar significance. The API runs on a SPARQList, which is a REST API server, and the data processing workflow is described in Table 1.

|Step|Description|Data sources, tools and APIs|
| -------- | -------- | ----- |
|1|Collect human variants located within the input gene region along with their positions in the coding sequences (CDS)  and ClinVar significance|TogoVar RDF, Ensembl Variant Effect Predictor (VEP) and ClinVar|
|2|Identify the mouse counterpart gene for the input human gene|Homologene|
|3|Obtain the CDS of both human and mouse|Ensembl API|
|4|Perform a global alignment of the human and mouse CDS|ggsearch|
|5|Locate mouse counterpart variants in the mouse CDS for each human variant collected in Step 1, based on the global alignment performed in Step 4||
|6|Convert the local position within the CDS to the position in the mouse reference genome|Ensembl API|
|7|Search for a MoG+ variant and strain name using the mouse reference genome position|MoG+ API|

Table: A step-by-step description of the workflow of the API

## Example of an API response for the human ABCA12 gene
The API returns a mouse strain “HMIv1” that has a counterpart mouse variant of a human ABCA12 gene variant C > T at chr2:14937608 (GRCh38).
```
{
    "uri": "http://identifiers.org/hco/2/GRCh38#214937608-G-A",
    "hgvsc": "ENST00000272895.12:c.7444C>T",
    "hgvsp": "ENSP00000272895.7:p.Arg2482Ter",
    "consequence": "stop_gained",
    "significance": "Pathogenic",
    "human": {
      "assembly": "GRCh38",
      "chromosome": "2",
      "position": 214937608,
      "strand": -1,
      "ref": "C"
    },
    "mouse": {
      "assembly": "GRCm39",
      "chromosome": "1",
      "position": 71286389,
      "strand": -1,
      "ref": "C",
      "strain": [
        "HMIv1"
      ]
    }
  },
```

# Future work

We will work on the evaluation and improvement of the mapping accuracy. The human-mouse gene relationships were extracted only from the HomoloGene ortholog cluster. The use of other ortholog databases [citation needed?] along with HomoloGene can improve the accuracy. Additionally, the API currently covers only one-to-one relationships between human and mouse genes. Exploring one-to-N relationships is worthwhile because biologically significant mouse homologs related to immunity and olfaction have been reported. It is also important to assess alignment tools other than ggsearch. The mapping results will be presented as the links between a comprehensive human variation database TogoVar [4] and a model mouse genome database MoG+ [5].

In the future, we will not only focus on comparisons based solely on the homology of nucleotide sequences encoding proteins in humans and mice, but also take into consideration the functionality of cis-elements. Mapping variants in non-coding (UTR and intron) and intergenic regions is worthwhile but challenging. Additionally, we will prioritize genomic variants of disease-causing genes based on literature information and the results of large-scale gene knockout projects in mice. The goal is to develop an information infrastructure for accurately selecting the most suitable mouse model strains for human disease research.

## Acknowledgements

We would like to thank the participants and organizers of BioHackathon 2023 for their constructive advice, which greatly influenced our project. Special thanks go to the members of MoG+ and TogoVar development teams for their guidance and support. Without their contributions, our project would not have been possible.

## References
