
# 1000 genomes exome VCF processing
3202 individual exomes, split by chromosome, so there are 23 multi-vcf chromosome files (1-22, X)

#### Download FASTA for decomposing VCFs (split multi-allelic sites)

in references
$ mkdir -p ~/references; cd ~/references
$ 
$ wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz
$ gunzip hg38.fa.gz

see what format chromosomes are in, make sure they match the format of the ones in your VCFs
$ grep "^>" hg38.fa | head

$ 


#### Decompose


#### Run VEP


#### Run echtvar for gnomad (allele frequency) annotation


#### Split with vcf-split tool
make sure to include header and output in bcf format
