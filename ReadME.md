
# Commands run on the Whole Genome Sequencing (WGS) pipeline for assembly and annotation of a bacterial genome using Unicycler, Pilon, and BAKTA.

## Run unicycler without installation

Reference: https://github.com/rrwick/Unicycler
```
git clone https://github.com/rrwick/Unicycler.git
cd Unicycler
make
```

## Assign bash variable to runner path

```
UNICYCLER_RUNNER=/home/abt191-2/Unicycler/unicycler-runner.py
```



## Run unicycler on forward and reverse trimmed reads

Input: Trimmed FASTq files (forward _1 and reverse _2)

This was generated using fastp in Google Colab: https://colab.research.google.com/drive/1oRmIzL_X8qLwaKXSraMcZJEmqjCqI9n2?usp=sharing


```
/usr/bin/time -v $UNICYCLER_RUNNER -t 32 -1 trimmed_1.fastq -2 trimmed_2.fastq -o unicycler_out &> unicycler_log.txt
```

Output: `assembly.fasta` file


Note: from here a python venv was activated (installed quast using `pip install quast`)


## Run quast

Input: fasta nucleotide (fna) file as reference and assembly.fasta file
```
 quast.py  ./assembly.fasta
```
Output: quast report

## Improve assembly using Pilon

### Index Unicycler Assembly
```
bashcd ~/project
bwa index ./unicycler_out/assembly.fasta
```

### Map Your Trimmed Reads Back to the Assembly

```
bashbwa mem \
  -t 4 \
  ./unicycler_out/assembly.fasta \
  ./trimmed_1.fastq \
  ./trimmed_2.fastq \
  > ./aligned.sam
```

### Convert, Sort, and Index the BAM File
bash# Convert SAM to BAM

```
samtools view -Sb ./aligned.sam > ./aligned.bam

```

### Sort the BAM

```
samtools sort ./aligned.bam -o ./aligned_sorted.bam
```
### Index the sorted BAM
```
samtools index ./aligned_sorted.bam
```


### Run Pilon

```
bashpilon \
  --genome ./unicycler_out/assembly.fasta \
  --frags ./aligned_sorted.bam \
  --output pilon_round1 \
  --outdir ./pilon_out \
  --changes \
  --vcf \
  --threads 4
```

### Check How Many Changes Were Made

```
cat ./pilon_out/pilon_round1.changes
```

Note: Run QUAST on the Pilon-Polished Assembly

## Run Taxonomic Classification on MiGA Web Server


## Used BAKTA Web to annotate `assembly.fasta`
Output: gff3 file