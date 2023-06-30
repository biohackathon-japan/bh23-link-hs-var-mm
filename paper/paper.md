---
title: 'BioHackJP 2023 Report R1: linking human genetic variants to their mouse counterparts'
title_short: 'BioHackJP 2023 LINK-HS-VAR-MM'
tags:
  - Human Disease Model Mouse
  - Genetic Variants
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
affiliations:
  - name: Database Center for Life Science, Joint Support-Center for Data Science Research, Research Organization of Information and Systems
    index: 1
date: 30 June 2023
cito-bibliography: paper.bib
event: BH23JP
biohackathon_name: "BioHackathon Japan 2023"
biohackathon_url:   "https://2023.biohackathon.org/"
biohackathon_location: "Kagawa, Japan, 2023"
group: R1
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-jp/bh23-variant-mapping-hg-mm
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Mitsuhashi, N. \emph{et al.}
---

# Background

Experimental mice are widely used in human disease studies. To select an experimental mouse model appropriately, it is crucial to identify mouse variants with the same effect as disease-related variants in humans. Homologous human and mouse genes have been identified and shared in databases [1] [2]. At the genetic variant level, however, the explicit relationships between human variants and their mouse counterparts have yet to be included in these databases. A previous study successfully mapped human pathogenic variants mostly found in conserved coding regions to orthologous positions in the cow and pig genomes [3].

In BH2023, we aimed to map human pathogenic variants to orthologous positions in the mouse genome based on the sequence similarity between the two species. The mapping results will be provided as the links between a comprehensive human variation database TogoVar [4] and a model mouse genome database MoG+ [5].


# Outcomes

We have developed an API that returns mouse variants and strains in MoG+ as counterparts to ClinVar variants within the gene region specified by an HGNC gene symbol. For example, users can obtain a list of mouse variants linked to ClinVar variants in the human PAX4 gene by using the following link: https://sparql-support.dbcls.jp/rest/api/human_variant_to_mouse?hgnc_symbol=PAX4&clinvar=true. 

# Future work

We will evaluate the mapping accuracy by comparing annotations of human variants with those of their corresponding mouse variants. The annotations include substitution patterns of amino acids and molecular functions. 

## Acknowledgements

We would like to thank the fellow participants at BioHackathon 2023 for their collaboration and constructive advice, which greatly influenced our project. We are grateful to the organizers for providing this platform and the developers of open source language models. Special thanks to our mentors, advisors, and colleagues for their guidance and support. Without their contributions, our project in linked data standardization with LLMs in bioinformatics would not have been possible.

## References

1. Wang J, Al-Ouran R, Hu Y, et al. MARRVEL: Integration of Human and Model Organism Genetic Resources to Facilitate Functional Annotation of the Human Genome. Am J Hum Genet. 2017;100(6):843-853. doi:10.1016/j.ajhg.2017.04.010
2. HomoloGene. https://www.ncbi.nlm.nih.gov/homologene 
3. Zhao R, Talenti A, Fang L, et al. The conservation of human functional variants and their effects across livestock species. Commun Biol. 2022;5(1):1003. Published 2022 Sep 21. doi:10.1038/s42003-022-03961-1
4. Mitsuhashi N, Toyo-Oka L, Katayama T, et al. TogoVar: A comprehensive Japanese genetic variation database. Hum Genome Var. 2022;9(1):44. Published 2022 Dec 12. doi:10.1038/s41439-022-00222-9
5. Takada T, Fukuta K, Usuda D, et al. MoG+: a database of genomic variations across three mouse subspecies for biomedical research. Mamm Genome. 2022;33(1):31-43. doi:10.1007/s00335-021-09933-w

