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
  --input-path laneless \
  --input-format CasavaOneEightLanelessPerSampleDirFmt \
  --output-path demux-paired-end.qza
```







