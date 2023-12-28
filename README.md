# Grain Mcirobiome (Fungal Workshop)

# Introduction
Mycobiome was profiled in rye, wheat, and rye+wheat that are used as a starting material for sourdough. Grains were grinded separately, mixed in water and filtered. Filtered water was used for the DNA extraction and subjected to mycobiome sequencing using ITS primers.

<img width="170" alt="image" src="https://github.com/vetrabindra01/YorkCollege-Grain-Mcirobiome-Fungal-Workshop/assets/97687143/9fb51be3-a978-4d4e-9a22-d58ac7634bf9">

# Primers

Region: ITS2 ([ITS as an environmental DNA barcode for fungi](https://bmcmicrobiol.biomedcentral.com/articles/10.1186/1471-2180-10-189)).

FP (position: 2024-2045 bp), ITS3f: GCATCGATGAAGAACGCAG

RP (position: 2390-2409 bp), ITS4r: TCCTCCGCTTATTGATATGC

Maximum length: 2409-2024 = 385 bp

# Commands
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

# Calssify the representative sequences

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
```
qiime feature-classifier fit-classifier-naive-bayes \
--i-reference-reads unite-ver8-seqs_99_04.02.2020.qza \
--i-reference-taxonomy unite-ver8-taxonomy_99_04.02.2020.qza \
--o-classifier unite-ver8-99-classifier-04.02.2020.qza
```
```
qiime feature-classifier fit-classifier-naive-bayes \
--i-reference-reads sh_refs_qiime_ver9_dynamic_25.07.2023.qza  \
--i-reference-taxonomy sh_taxonomy_qiime_ver9_dynamic_25.07.2023.qza \
--o-classifier classifier-sh_refs_qiime_ver9_dynamic_25.07.2023.qza
```


















