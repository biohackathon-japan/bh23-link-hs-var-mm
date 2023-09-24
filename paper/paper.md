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

Experimental mice are widely used in human disease studies. Since the inception of mouse genetic research, hundreds of diverse strains have been established for biomedical research. In disease model mouse strains, information on genomic variations is essential for elucidating the relationship between haplotypes and disease susceptibility. To select a disease model mouse appropriately, it is crucial to identify mouse variants with the same effect as disease-causing variants in humans. Homologous human and mouse genes have been identified and shared in integrated human and model organism databases [@wang2017] [@homologene2023]. At the genetic variant level, however, the explicit relationships between human variants and their mouse counterparts have yet to be included in these databases. A previous study successfully mapped human pathogenic variants mostly found in conserved coding regions to orthologous positions in the cow and pig genomes [@zhao2022]. In BH2023, we aimed to map human pathogenic variants to orthologous positions in the mouse genome based on the sequence similarity between the two species.

# Outcomes

Using the reference sequences of humans and mice, we compared the regions encoding proteins of homologous genes. First, we focused on nucleotide variants involved in amino acid substitutions. We have developed an API that returns mouse variants and strains in the MoG+ database [@takada2022] as counterparts to ClinVar [@landrum2020] variants located within the gene region specified by an HGNC gene identifier or symbol. Target human variants can be limited to variants with ClinVar significance. An example of the API response is shown in Figure 1.

```
  {
    "uri": "http://identifiers.org/hco/1/GRCh38#94080658-C-T",
    "hgvsc": "ENST00000370225.4:c.919G>A",
    "hgvsp": "ENSP00000359245.3:p.Gly307Ser",
    "consequence": "missense_variant",
    "significance": "Uncertain significance",
    "human": {
      "assembly": "GRCh38",
      "chromosome": "1",
      "position": 94080658,
      "strand": -1,
      "ref": "G"
    },
    "mouse": {
      "assembly": "GRCm39",
      "chromosome": "3",
      "position": 121877561,
      "strand": 1,
      "ref": "G",
      "strain": [
        "SPRET/EiJ"
      ]
    }
```
Figure: Example of an API response for the human ABCA4 gene
The API returns a mouse strain “SPRET/EiJ” that has a counterpart variant of a human ABCA4 gene variant C > T at chr1:94080658 (GRCh38).

The API runs on a SPARQList [@sparqlist2023], which is a REST API server, and the data processing workflow is described in Table 1.

|Step|Description|Data sources, tools and APIs|
| -- | -------- | ----- |
|1|Collect human variants located within the input gene region including their positions in an Ensembl transcript sequence and ClinVar significance.|TogoVar RDF [@mitsuhashi2022], Ensembl Variant Effect Predictor (VEP) [@mclaren2016a] and ClinVar [@landrum2020]|
|2|Identify the mouse counterpart gene for the input human gene.|Homologene [@homologene2023]|
|3|Obtain the coding sequences (CDS) of the human and mouse gene identified in Step 2.|Ensembl API [@ensembl2023]|
|4|Perform a global alignment of the human and mouse CDS.|ggsearch [@ggsearch2023]|
|5|Locate mouse counterpart variants in the mouse CDS for each human variant collected in Step 1, based on the global alignment performed in Step 4.||
|6|Convert the local position within the CDS to the position in the mouse reference genome.|Ensembl API [@ensembl2023]|
|7|Search for a variant and strain name in the MoG+ database  using the mouse reference genome position.|MoG+ API [@takada2022]|

Table: Data processing workflow steps of the API
The table explains how data was processed and which data sources, tools, and APIs were used at each stage in the API. Between these steps, TogoID [@ikeda2022]  was used for conversion between Ensembl transcript IDs and NCBI gene IDs. The workflow written in Markdown format is available at https://github.com/biohackathon-japan/bh23-map-hs-var-mm/blob/main/sparqlet/human_variant_to_mouse.md.

# Future work

We will work on improving the precision of variant mapping. Currently, gene relationships between humans and mice are derived exclusively from the HomoloGene ortholog cluster [@homologene2023]. However, supplementing HomoloGene with other ortholog databases could potentially enhance accuracy. Our current system covers only one-to-one gene relationships between humans and mice. Nevertheless, it is valuable to explore one-to-many relationships, especially in understanding essential mouse homologs related to immunity and olfaction. Additionally, it is crucial to evaluate alignment tools beyond ggsearch. The outcomes of our variant mapping will be presented as links connecting the comprehensive human variation database, TogoVar [@mitsuhashi2022], and the model mouse genome database, MoG+ [@takada2022].

In the future, we will not only focus on comparisons based solely on the homology of nucleotide sequences encoding proteins in humans and mice, but also take into consideration the functionality of cis-elements. Mapping variants in non-coding (UTR and intron) and intergenic regions is worthwhile but challenging. Additionally, we will prioritize genomic variants of disease-causing genes based on literature information and the results of large-scale gene knockout projects in mice. The goal is to develop an information infrastructure for accurately selecting the most suitable mouse model strains for human disease research.

## Acknowledgements

We would like to thank the participants and organizers of BioHackathon 2023 for their constructive advice, which greatly influenced our project. Special thanks go to the members of MoG+ and TogoVar development teams for their guidance and support. Without their contributions, our project would not have been possible.

## References
