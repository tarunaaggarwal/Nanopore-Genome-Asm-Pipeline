# Genome Assembly, Annotation and Evaluation Pipeline

#### The PATH for final genome assembly and annotation files:
#### Canu
#### Genome assembly: `~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round2/sn12-canu-polish-round2.fasta`
#### Prokka annotation gff: `~/annie-rowe/analysis-results/prokka-results/ont/trim01/canu-largestContig-polished/PROKKA_06172019.gff`

#### Flye (my favorite)
#### Genome assembly: `~/annie-rowe/data-clean/ont/pilon-polished-assembly/flye/round2/sn12-flye-polish-round2.fasta`
#### Prokka annotation gff: `~/annie-rowe/analysis-results/prokka-results/ont/trim01/flye-largestContig-polished/PROKKA_05302019.gff`

> A very simple circular visual of *Thioclava electrotropha*
![Thioclava electrotropha](https://www.dropbox.com/s/5iewr5b3ftjyzzq/Thioclava.eletrotropha.draft-genome.png?raw=1)
---

## Basecalling and Binning of `fast5` file in a directory called `barcode01`. This directory is different than `trim01` which contains previously basecalled and binned samples. 

### basecall with Albacore v2.3.4
#### dir 0
```
read_fast5_basecaller.py \
-f FLO-MIN106 \
-k SQK-LSK108 \
-t 4 \
-s ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/0 \
-o fastq \
-q 100000 \
-i ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/barcode01/0
```
#### dir 1
```
read_fast5_basecaller.py \
-f FLO-MIN106 \
-k SQK-LSK108 \
-t 4 \
-s ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/1 \
-o fastq \
-q 100000 \
-i ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/barcode01/1
```
### combine the resulting fastqs into one

```
cat ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/0/workspace/pass/fastq_runid_06bbc11f247086b80d0373145c1921813bd431f3_0.fastq \
~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/0/workspace/pass/fastq_runid_ee5c2b12808a7222850d88bc213f1837d43f3dac_0.fastq \
~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/1/workspace/pass/fastq_runid_ee5c2b12808a7222850d88bc213f1837d43f3dac_0.fastq > ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/bc01.combined.fastq
```

### Our reads are already classified as barcode 01 but I still run DeepBinner v0.2.0 to double check that our reads contain barcode 01

#### dir 0
```
deepbinner classify --native ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/barcode01/0/ > ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/classifications
```
#### dir 1 
```
deepbinner classify --native ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/barcode01/1/ >> ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/classifications
```
### manually remove the second line with "read_ID	barcode_call"
```
nano +4000 classifications 
```
### bin the classified reads
```
deepbinner bin \
--classes ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/classifications \
--reads ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/albacore-v2.3.4/bc01.combined.fastq \
--out_dir ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/
```
### unzip the deepbinner binned fastq file
```
gunzip ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/barcode01.fastq.gz 
```
### create a read length text file using the custom script in the scripts folder
```
python ~/annie-row/scripts/counting_lines_in_fastq_file_TAS.py \
--input ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/barcode01.fastq \
--output ~/annie-rowe/data-raw/ont/tas-basecalled-binned-data/deepbinner-v0.2.0/barcode01.fastq.read.len.txt
```

### additionally, before assembling the albacore and deepbinner processed fastq file, count the read length for the BC01.fastq that Annie Rowe's group demultiplexed during the run
```
python ~/annie-rowe/scripts/custom-script/counting_lines_in_fastq_file_TAS.py \
--input ~/annie-rowe/data-raw/ont/previously-demultiplexed-basecalled-data/trim01/BC01.fastq \
--output ~/annie-rowe/data-raw/ont/previously-demultiplexed-basecalled-data/trim01/BC01.fastq.read.len.txt
```
### Below are the sequence distribution histograms for barcode01 and trim01. 
> Barcode 01 - total number of **5,513** sequences
![Barcode01 Sequence Distribution](https://www.dropbox.com/s/nkq1u400pl3p6k5/barcode01-seq-len-histo.png?raw=1)

> Trim 01 - total number of **55,639** sequences
![Trim 01 Sequence Distribution](https://www.dropbox.com/s/tj5o57smki681qe/trim01-seq-len-histo.png?raw=1)

### assemble with Canu v1.8
```
canu \
-p barcode01 \
-d ~/annie-rowe/data-clean/ont/canu-results/barcode01 \
genomeSize=4m \
-nanopore-raw ~/annie-rowe/data-raw/ont/tas-basecalled-data/deepbinner-v0.2.0/barcode01.fastq
```
### assemble with Flye v2.4.2
```
flye \
--nano-raw ~/annie-rowe/data-raw/ont/tas-basecalled-data/deepbinner-v0.2.0/barcode01.fastq \
--genome-size 4m \
--out-dir ~/annie-rowe/data-clean/ont/flye-results/barcode01 \
--threads 4
```

**Notes**

1. canu errored out
2. flye assembly resulted in 37 contigs, which means that these data files do not contain enough information. 
3. so I proceeded to assemble and polish the trim01 sequences with Illumina data.

---
---

## Assembling `BC01.fastq` in `trim01/` directory. 
### assemble with Canu v1.8
```
canu \
-p t.elec \
-d ~/annie-rowe/data-clean/ont/canu-results/trim01 \
genomeSize=4m \
-nanopore-raw ~/annie-rowe/data-raw/ont/previously-demultiplexed-basecalled-data/trim01/BC01.fastq
```
### Canu assembly Stats below
```
# contigs (>= 0 bp)         5
# contigs (>= 1000 bp)      5
# contigs (>= 5000 bp)      5
# contigs (>= 10000 bp)     5
# contigs (>= 25000 bp)     4
# contigs (>= 50000 bp)     3
Total length (>= 0 bp)      7357826
Total length (>= 1000 bp)   7357826
Total length (>= 5000 bp)   7357826
Total length (>= 10000 bp)  7357826
Total length (>= 25000 bp)  7342966
Total length (>= 50000 bp)  7306876
# contigs                   5
Largest contig              4265114
Total length                7357826
GC (%)                      57.28
N50                         4265114
N75                         2851038
L50                         1
L75                         2
# N's per 100 kbp           0.00
```

### assemble with Flye v2.4.2
```
flye \
--nano-raw ~/annie-rowe/data-raw/ont/previously-demultiplexed-basecalled-data/trim01/BC01.fastq \
--genome-size 4m \
--out-dir ~/annie-rowe/data-clean/ont/flye-results/trim01 \
--asm-coverage 30 \
--threads 4
```
### Flye assembly stats below
```
# contigs (>= 0 bp)         3
# contigs (>= 1000 bp)      3
# contigs (>= 5000 bp)      3
# contigs (>= 10000 bp)     3
# contigs (>= 25000 bp)     3
# contigs (>= 50000 bp)     3
Total length (>= 0 bp)      7308384
Total length (>= 1000 bp)   7308384
Total length (>= 5000 bp)   7308384
Total length (>= 10000 bp)  7308384
Total length (>= 25000 bp)  7308384
Total length (>= 50000 bp)  7308384
# contigs                   3
Largest contig              4278531
Total length                7308384
GC (%)                      57.01
N50                         4278531
N75                         2860821
L50                         1
L75                         2
# N's per 100 kbp           0.00
```
---
---

## Polishing the above two assemblies

### First round of polishing for canu assembly
### pull out the largest contig
```
samtools faidx ~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.contigs.fasta tig00000001 > ~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.largestContig.fasta
```
### index the largest contig
```
bwa index ~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.largestContig.fasta
```
### map the raw Illumina reads
```
bwa mem \
-t 8 \
~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.largestContig.fasta \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p1.fastq.gz \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p2.fastq.gz > ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.sam
```
### convert sam to bam
```
samtools view \
-bS ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.sam > ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.bam
```
### sort the bam file
```
samtools sort \
~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.bam \
-o ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.sorted.bam
```
### index the sorted bam file
```
samtools index ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.sorted.bam
```
### polish with Pilon v1.23 
```
java -Xmx16G -jar ~/pkgs/pilon/pilon-1.23.jar \
--genome ~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.largestContig.fasta \
--frags ~/annie-rowe/data-clean/ont/illumina-reads-against-canu-asm/LW_SG_SN12--68.sorted.bam \
--outdir ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1 \
--output sn12-canu-polish-round1 \
--fix all --changes
```

### repeat a second round of polishing
```
bwa index ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.fasta
```
```
bwa mem -t 8 \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.fasta \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p1.fastq.gz \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p2.fastq.gz > ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.sam
```
```
samtools view -Sb \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.sam > ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.bam
```
```
samtools sort \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.bam \
-o ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.sorted.bam
```
```
samtools index ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.sorted.bam
```

```
java -Xmx16G -jar ~/pkgs/pilon/pilon-1.23.jar \
--genome ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.fasta \
--frags ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round1/sn12-canu-polish-round1.sorted.bam \
--outdir ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round2 \
--output sn12-canu-polish-round2 \
--fix all --changes
```

### run prokka on the unpolished largest contig
```
prokka \
--outdir ~/annie-rowe/analysis-results/prokka-results/ont/trim01/canu-largestContig-unpolished \
~/annie-rowe/data-clean/ont/canu-results/trim01/t.elec.largestContig.fasta
```

### run prokka on the polished largest contig
```
prokka --outdir ~/annie-rowe/analysis-results/prokka-results/ont/trim01/canu-largestContig-polished ~/annie-rowe/data-clean/ont/pilon-polished-assembly/canu/round2/sn12-canu-polish-round2.fasta
```


### First round of polishing for flye assembly
### pull out the largest contig
```
samtools faidx ~/annie-rowe/data-clean/ont/flye-results/trim01/assembly.fasta contig_1 > ~/annie-rowe/data-clean/ont/flye-results/trim01/largestContig.fasta
```
### index the largest contig
```
bwa index ~/annie-rowe/data-clean/ont/flye-results/trim01/largestContig.fasta
```
### map the raw Illumina reads
```
bwa mem \
-t 8 \
~/annie-rowe/data-clean/ont/flye-results/trim01/largestContig.fasta \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p1.fastq.gz \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p2.fastq.gz > ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.sam
```
### convert sam to bam
```
samtools view \
-bS ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.sam > ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.bam
```
### sort the bam file
```
samtools sort \
~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.bam \
-o ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.sorted.bam
```
### index the sorted bam file
```
samtools index ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.sorted.bam
```

### polish with Pilon v1.23
```
java -Xmx16G -jar ~/pkgs/pilon/pilon-1.23.jar \
--genome ~/annie-rowe/data-clean/ont/flye-results/trim01/largestContig.fasta \
--frags ~/annie-rowe/data-clean/ont/illumina-reads-against-flye-asm/LW_SG_SN12--68.sorted.bam \
--outdir ~/annie-rowe/data-clean/ont/pilon-polished-assembly/flye/round1 \
--output sn12-polish-round1
--fix all --changes
```

### repeat a second round of polishing
```
bwa index ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.fasta
```
```
bwa mem -t 8 \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.fasta \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p1.fastq.gz \
~/annie-rowe/data-raw/illumina/LW_SG_SN12--68.raw1_p2.fastq.gz > ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.sam
```
```
samtools view -Sb \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.sam > ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.bam
```
```
samtools sort \
~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.bam \
-o ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.sorted.bam
```
```
samtools index ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.sorted.bam
```
```
java -Xmx16G -jar ~/pkgs/pilon/pilon-1.23.jar \
--genome ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.fasta \
--frags ~/annie-rowe/data-clean/ont/pilon-polished-assembly/round1/sn12-polish-round1.sorted.bam \
--outdir ~/annie-rowe/data-clean/ont/pilon-polished-assembly/flye/round2 \
--output sn12-flye-polish-round2
--fix all --changes
```
### run prokka on the unpolished largest contig
```
prokka \
--outdir ~/annie-rowe/analysis-results/prokka-results/ont/trim01/flye-largestContig-unpolished \
~/annie-rowe/data-clean/ont/flye-results/trim01/largestContig.fasta
```
### run prokka on the polished largest contig
```
prokka --outdir ~/annie-rowe/analysis-results/prokka-results/ont/trim01/flye-largestContig-polished ~/annie-rowe/data-clean/ont/pilon-polished-assembly/flye/round2/sn12-flye-polish-round2.fasta
```

### run blast against swissprot db. The basic blastp command is below
```
blastp -num_threads 4 \
-db reference_database \
-query contig.fasta \
-out contig.blast \
-outfmt "6 qseqid qlen sseqid sallseqid slen qstart qend sstart send evalue bitscore length pident nident mismatch positive gapopen gaps frames qframe sframe stitle qcovs" \
-max_target_seqs 1
```
### filter blast results for polished and unpolished assembly using the following `awk` command. Pull out cols 1, 2, 3, 5 which contain queryID, queryLen, subjectID, and subjectLen, respectively.
```
cat contig.blast | awk '{print$1"\t"$2"\t"$3"\t"$5}' - > contig.qlen.slen.blast
```

### make histograms in R to compare whether polishing reduced the number of indels and number of pseudogenes
  - [Canu assembly comparison before and after polishing.](https://github.com/tarunaaggarwal/Nanopore-Genome-Asm-Pipeline/blob/master/pdfs/Canu-Assembly-Indels-Before-and-After-Polishing.pdf)
  - [Flye assembly comparsion before and after polishing.](https://github.com/tarunaaggarwal/Nanopore-Genome-Asm-Pipeline/blob/master/pdfs/Flye-Assembly-Indels-Before-and-After-Polishing.pdf)
  - The above R markdown files are also in `~/annie-rowe/scripts/custom-script`. 
  - For more info, read [Mick Watson's blog](http://www.opiniomics.org/a-simple-test-for-uncorrected-insertions-and-deletions-indels-in-bacterial-genomes/) on checking for indels in a genome assembly.

### Below are quick numbers for polished and unpolished assemblies

| 				  | Flye Unpolished | Flye Polished  | Canu Unpolished  | Canu Polished
|-----------------|:-------------|:---------------:|---------------:|---------------:
| Bases | 4278531  | 4268982     | 4265114    | 4284365
| CDS     | 6174          | 4009      | 7470            | 4067
| tRNA      | 52         | 57             | 57            | 58
| rRNA      | 9         | 9             | 9            | 9
| Hypothetical protein      | 3852         | 1778             | 4806            | 1823
| Percent of Hypothetical protein      | 62.39         | 44.35             | 64.33            | 44.82


### I also ran Progressive Mauve to explore congruencies among Illumina Assembly (by Lizzy) and my Canu and Flye assemblies. The Mauve outputs are in `~/annie-rowe/analysis-results/prog-mauve-output`. You can check out the `pngs` folder with lots of images for comparing the different assemblies.

### Lastly, below are some resources for making a nice circular visual for your genome. Properly annotating and labeling the circular genome will require manual power.

- [SnapGene Viewer](https://www.snapgene.com/snapgene-viewer/) is free tool for importing fasta and gff files to make a visual. I am becoming a big fan of this tool. It's very user friendly and the figs are good looking!
- [Geneious](https://www.geneious.com/) is a great tool as well, but it costs money.
- [CGView](http://wishart.biology.ualberta.ca/cgview/index.html) is another tool. I haven't used it because I couldn't figure it out. 
