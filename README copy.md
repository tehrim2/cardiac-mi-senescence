# Senescence Driven Cardiac Fibrosis After Myocardial Infarction

## Anti-Fibrotic Gene Therapy Target Discovery in Human Cardiac Fibroblasts

## Overview

This project investigates cardiac fibroblast senescence following myocardial
infarction (MI), with the goal of identifying candidate gene therapy targets
for anti-fibrotic intervention. Recent literature describes fibroblast
senescence after MI as time dependent, transient senescence in the days
immediately following injury appears reparative, limiting excessive scarring,
while persistent senescence drives chronic, SASP-mediated fibrosis. The
molecular switch point between these two states is not yet well defined at
single-cell resolution in human tissue. This project reanalyzes a public
human single-nucleus RNA-seq dataset to characterize senescence signal
within the cardiac fibroblast population and identify genes distinguishing
high senescence from low senescence fibroblasts.

## Dataset

Kuppe et al. 2022, *Nature*. Spatial multi-omic map of human myocardial
infarction. DOI: 10.1038/s41586-022-05060-x

Single-nucleus RNA-seq data (191,795 nuclei, 23 patients, five injury zones:
control, remote, border, ischemic, and fibrotic) accessed via CZ CELLxGENE
Discover.

## Repository Structure

```
cardiac-mi-senescence/
├── data/
│   ├── raw/            (not tracked; original CELLxGENE download)
│   └── processed/      (not tracked; intermediate .rds objects)
├── notebooks/
│   ├── 01_setup_and_literature.Rmd
│   ├── 02_qc_preprocessing.Rmd
│   ├── 03_clustering_annotation_scoring.Rmd
│   └── 04_differential_expression_target_discovery.Rmd
├── results/
│   ├── figures/
│   └── tables/
├── renv.lock
├── .gitignore
└── README.md
```

## Methods Summary

**Quality control.** Nuclei with a doublet score of 0.3 or higher were
excluded as an explicit, documented reproducibility choice, applied on top
of the mitochondrial, gene count, and UMI filtering already performed by
the original authors. This left 46,453 fibroblast nuclei.

**Ambient RNA correction.** DecontX was used to correct for ambient RNA
contamination arising from necrotic tissue in the ischemic zone, a
limitation acknowledged in the original publication. The ischemic zone remained 
transcriptionally distinct from other zones after this correction, indicating a 
genuine biological difference rather than a technical artifact.

**Clustering and annotation.** Batch correction was performed with Harmony
(donor as the correction variable), followed by Leiden clustering. Marker
gene analysis identified three clusters as non-fibroblast populations
(immune cells, proliferating cells, and cardiomyocyte-contaminated cells)
based on canonical marker expression across independent gene panels, and
these were excluded, leaving a final, verified population of 39,256
fibroblasts. An independent doublet detection pass (scDblFinder) run on the
full multi-cell-type dataset was cross-referenced against this population
as an additional validation step.

**Senescence scoring.** The SenMayo gene set (Saul et al. 2022; 116 of 124
genes present in this dataset) was scored using two independent methods,
AddModuleScore and AUCell, which showed strong agreement (Spearman
rho = 0.87).

**Differential expression.** Fibroblasts were split into high and low
senescence groups using a consensus approach requiring agreement between
both scoring methods (top and bottom third of each distribution). Pseudobulk
aggregation by patient and group, followed by a paired DESeq2 design
(`~ donor_id + group`), compared 20 matched patient pairs.

**Pathway enrichment and target prioritization.** Significant genes were
tested against MSigDB Hallmark and Reactome gene sets. Candidate targets
were ranked by statistical strength and convergence across independently
enriched pathways, then cross-referenced against the Open Targets Platform
and DGIdb for existing drug and clinical development evidence.

## Key Findings

- 2,729 genes significantly differ between high and low senescence
  fibroblasts (adjusted p < 0.1)
- Strongest enriched pathways: TNF-alpha signaling via NF-kB, epithelial to
  mesenchymal transition, extracellular matrix organization, and a
  coordinated VEGF/IGF signaling axis
- Wnt/beta-catenin signaling as a pathway was not enriched, despite CTNNB1
  being the single strongest individual differentially expressed gene
- A final list of 17 candidate genes was produced, split into an
  established tier (7 genes with existing clinical-stage drug development,
  including TNC, MMP2, and VEGFA) and an exploratory tier (6 genes with
  strong statistical and pathway support but no current therapeutic
  development, including CCN1, IGFBP4, and ACVR1B)
- CCN1's identification independently corroborates a published mechanism
  in which this SASP protein has been reported to influence fibroblast
  senescence following MI

## Tools

R, Seurat, Harmony, DecontX (celda), scDblFinder, AUCell, DESeq2,
clusterProfiler, msigdbr, org.Hs.eg.db, ggplot2, ggrepel. Open Targets
Platform and DGIdb GraphQL APIs for druggability assessment.

## Reproducing This Analysis

1. Clone this repository
2. Restore the R environment: `renv::restore()`
3. Download the Kuppe et al. 2022 snRNA-seq data from CZ CELLxGENE Discover
   into `data/raw/`
4. Run the notebooks in order, 01 through 04

## Limitations and Future Directions

This analysis is scoped to the cardiac fibroblast lineage specifically,
given its direct mechanistic role in fibrosis. Cardiomyocytes, endothelial
cells, and macrophages are all independently documented to undergo
senescence in the post-MI heart, and understanding cross-talk between these
populations and senescent fibroblasts, via cell-cell communication analysis, is 
a natural next step for the project.
Ambient RNA correction was performed using DecontX rather than SoupX or
CellBender, since the raw, unfiltered droplet matrices required by those
tools were not available through this dataset's public distribution.
