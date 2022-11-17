---
title: "Australian corpus"
subtitle: "Australian obesity corpus preparation, analysis and comparison with the UK"
date: 2021-11-21
categories: "Analysis"
image: ../fig/2111_corpus.jpg
---

<p align="center">
<img src="https://daryavanichkina.com/fig/2111_corpus.jpg" width="500" />
</p>

Photo by <a href="https://unsplash.com/@ugur?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Ugur Akdemir</a> on <a href="https://unsplash.com/s/photos/corpus?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  

## Client

- [Prof Monika Bednarek](https://www.sydney.edu.au/arts/about/our-people/academic-staff/monika-bednarek.html), Faculty of Arts & Social Sciences, University of Sydney

## Purpose

- The client prepared a corpus of media articles that focused on obesity by manually downloading articles from LexisNexis on Windows. 
- The client then reached out to SIH, as multiple character encoding issues were encountered as a result of the manual download, so the first aim was to generate a "clean" corpus that could be used for data analysis.
- The second aim was to compare usage of specific language features between Australian tabloids and broadsheets, and across publications, to see whether journalists were adhering to the Obesity Collective's guidelines on reporting.
- The final aim was to compare these features between Australian and UK publications.
## Approach

- Resolved encoding issues in the 27 000+ articles from 12 Australian newspapers.
- Implemented near-duplicate detection, visualisation and removal using minhash and locality-sensitive hashing ("reimplemented Turnitin in Python") to ensure subsequent statistical analyses were not invalid due to duplicates.
- Used topic modelling to create sub-corpora for further analysis.
- Translated analytics spreadsheets used by researchers to a standalone Python script, automating and scaling one-vs-all and one-vs-one sub-corpus analysis.
- Provided comprehensive client-interpretable documentation.
- Generated parametric and non-parametric statistical analyses within the Australian corpus and between the Australian/UK corpora, including fixed and mixed effects linear modelling, in line with expectations for such analyses in statistical linguistics.
- Code from the above became part of a larger national project in computational linguistics.

## Key tools

- *Corpus cleaning/topic modelling: Python: pandas, chardet, spacy, seaborn, gensim, datasketch; Git + GitHub; Jira; fdupes*.
- *Corpus comparison and modelling: R: tidyverse, coin; Git + GitHub; Jira*.

## Outcomes

- Multiple variants of the clean corpus were prepared for use by the client, who carried out their own investigation using a series of GUI tools.
- The corpus manual has been published [here](https://github.com/Sydney-Informatics-Hub/obesitycorpus).
- The Australian corpus comparison has been submitted for publication [code here](https://sydney-informatics-hub.github.io/PIPE-3034-obesity2/).
- The Australian vs UK corpus comparison has been submitted for publication as a book chapter [code here](https://github.com/Sydney-Informatics-Hub/PIPE-3034-obesity2-bookchapter).
- The code for carrying out topic modelling, duplicate/near-duplicate detection, spreadsheet generalisation, parametric and non-parametric analyses has been generalised and incorporated into the Australian Text Analytics Platform (ATAP), a multi-centre, [ARDC-funded initiative](https://ardc.edu.au/project/australian-text-analytics-platform/) to provide linguists with GUI interfaces for carrying out common tasks.