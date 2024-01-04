# Grain Mcirobiome (Fungal Workshop)

# Introduction
Mycobiome was profiled in rye, wheat, and rye+wheat that are used as a starting material for sourdough. Grains were grinded separately, mixed in water and filtered. Filtered water was used for the DNA extraction and subjected to mycobiome sequencing using ITS primers.

<img width="170" alt="image" src="https://github.com/vetrabindra01/YorkCollege-Grain-Mcirobiome-Fungal-Workshop/assets/97687143/9fb51be3-a978-4d4e-9a22-d58ac7634bf9">

# Primers

Region: ITS2 ([ITS as an environmental DNA barcode for fungi](https://bmcmicrobiol.biomedcentral.com/articles/10.1186/1471-2180-10-189)).

FP (position: 2024-2045 bp), ITS3f: GCATCGATGAAGAACGCAG

RP (position: 2390-2409 bp), ITS4r: TCCTCCGCTTATTGATATGC

Maximum length: 2409-2024 = 385 bp

# Core diversity analysis
The mycobiome analysis was adapted from the following tutorials.
1. [Fungal ITS analysis tutorial](https://forum.qiime2.org/t/fungal-its-analysis-tutorial/7351).
2. [Processing ITS sequences with QIIME2 and DADA2](https://john-quensen.com/tutorials/processing-its-sequences-with-qiime2-and-dada2/).


1) Activate qiime2 using conda.
```
conda activate qiime2-amplicon-2023.9
```
2) Import using input format Laneless.
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path ITSReads \
  --input-format CasavaOneEightLanelessPerSampleDirFmt \
  --output-path demux-paired-end.qza
```
3) Trim adapters.
```
qiime cutadapt trim-paired \
  --i-demultiplexed-sequences demux-paried-end.qza \
  --p-front-f GCATCGATGAAGAACGCAG \
  --p-front-r TCCTCCGCTTATTGATATGC \
  --o-trimmed-sequences trimmed-demux-paried-end.qza \
  --p-discard-untrimmed --p-match-read-wildcards    
```
4) copy demultiplexed file.
```
cp trimmed-demux-paried-end.qza demux.qza 
```
5) Generate a summary of demultiplexing results.
```
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
```
6) View demux file
```
qiime tools view demux.qzv
```
7) Sequence quality control and feature table construction using DADA2.
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left-f 0 \
  --p-trim-left-r 0 \
  --p-trunc-len-f 280 \
  --p-trunc-len-r 180 \
  --o-representative-sequences rep-seqs-dada2.qza \
  --o-table table-dada2.qza \
  --o-denoising-stats stats-dada2.qza
```
8) Visualize dada2 stats.
```
qiime metadata tabulate \
  --m-input-file stats-dada2.qza \
  --o-visualization stats-dada2.qzv
```
View stats-dada2.qzv file and download metadata tsv file. Rename it as sample-metadata.tsv. Add a description column in the last column.

9) Rename files for easy to use commands.
```
mv rep-seqs-dada2.qza rep-seqs.qza
mv table-dada2.qza table.qza
```

10) Summarize feature table.
```
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv
```
11) Visualize representative sequences.
```
qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv
```
12) Make tree files for phylogenetic analysis.
```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```
13) Core diversity analysis
```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 99000 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir diversity-core-metrics-phylogenetic
```

# Buliding feature classifier for fungal ITS

Downlaod the [Taxonomy files](https://unite.ut.ee/repository.php).
1) Import the UNITE reference sequences into QIIME2.
```
qiime tools import \
  --input-path sh_refs_qiime_ver9_dynamic_25.07.2023.fasta \
  --output-path sh_refs_qiime_ver9_dynamic_25.07.2023.qza \
  --type 'FeatureData[Sequence]'
```
2) Import the taxonomy file
```
qiime tools import \
--type 'FeatureData[Taxonomy]' \
--input-path sh_taxonomy_qiime_ver9_dynamic_25.07.2023.txt \
--output-path sh_taxonomy_qiime_ver9_dynamic_25.07.2023.qza  \
--input-format HeaderlessTSVTaxonomyFormat
```
3) Train the classifier

   
## Classification of fungal ITS sequences
Quoted from QIIME2 platform "In our experience, fungal ITS classifiers trained on the UNITE reference database do NOT benefit from extracting/trimming reads to primer sites. We recommend training UNITE classifiers on the full reference sequences. Furthermore, we recommend the “developer” sequences (located within the QIIME-compatible release download) because the standard versions of the sequences have already been trimmed to the ITS region (excluding portions of flanking rRNA genes that may be present in amplicons generated with standard ITS primers)"[Link](https://docs.qiime2.org/2023.9/tutorials/feature-classifier/). This step takes more than few hours on HPC.

```
qiime feature-classifier fit-classifier-naive-bayes \
--i-reference-reads sh_refs_qiime_ver9_dynamic_25.07.2023.qza \
--i-reference-taxonomy sh_taxonomy_qiime_ver9_dynamic_25.07.2023.qza \
--o-classifier classifier-sh_refs_qiime_ver9_dynamic_25.07.2023.qza
```
Here is the link to download the [fungal classifier](https://drive.google.com/drive/u/0/folders/1iZeHSnP7WtKxq8jWv3OQDRB9tF908HME?ths=true).

## Taxonomic assignment
1) Assign taxonomy to rep.seqs (classify features). Relax! This step will take sometime maybe 15-20 mins.
```
qiime feature-classifier classify-sklearn \
  --i-classifier classifier-sh_refs_qiime_ver9_dynamic_25.07.2023.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza
```

2) Visualize taxonomy.
```
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```
3) Make bar plots.
```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```

4) Collapse taxonomy at genus level.
```
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 7 \
  --o-collapsed-table table-level7.qza
```
5) Export qza file.
```
qiime tools export --input-path table-level7.qza --output-path exported-Table-level-7

cd exported-Table-level-7 
```

6) Convert .txt file back to biom file.
```
biom normalize-table -r -i feature-table.biom -o rel-abun-feature-table.biom

biom convert -i rel-abun-feature-table.biom -o rel-abun-feature-table.txt --to-tsv
```

Taxonomic assignment at order level.

<img width="669" alt="Screenshot 2024-01-04 at 4 53 58 PM" src="https://github.com/vetrabindra01/YorkCollege-Grain-Mcirobiome-Fungal-Workshop/assets/97687143/b53f0157-78ee-4e72-a463-792059059ff7">



Taxonomic assignment at order level from [SeqCenter](https://www.seqcenter.com/).
<img width="629" alt="Screenshot 2024-01-04 at 4 54 25 PM" src="https://github.com/vetrabindra01/YorkCollege-Grain-Mcirobiome-Fungal-Workshop/assets/97687143/4273cecb-d959-49bc-b938-3956cf8e3f25">








