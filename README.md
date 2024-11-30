# BIO312Final
This is my final repository for final term paper for the Bionformatics course taught by Professor Joshua Rest.
This repository will walk you throught all the steps I took to support the basis of my paper.

## Introduction 

Below is the table of contents.
1. BLAST Proteome Database
2. Filtering out HSP to my Query Sequence
3. Alignment to Query Sequence
4. Constructing Gene Tree with Best Alignment
5. Cosntructing Species Tree
6. Reconciling Gene Tree within Speces Tree

Clone my final repository
```
git clone https://github.com/s1-spoon/bio312final
```
move into clone folder
```
cd ~/bio312final
```
# 1. BLAST Proteome Database
### 11 chordate species studied in my study
Below are the organisms speices name and the taxonomic ID
| Organism                        | Taxonomic ID |
|---------------------------------|--------------|
| *Carcharodon carcharias*        | 13397        |
| *Chelonia mydas*                | 8469         |
| *Danio rerio*                   | 7955         |
| *Equus caballus*                | 9796         |
| *Felis catus*                   | 9685         |
| *Gallus gallus*                 | 9031         |
| *Gasterosteus aculeatus*        | 69293        |
| *Homo sapiens*                  | 9606         |
| *Salmo salar*                   | 8030         |
| *Sphaerodactylus townsendi*     | 933632       |
| *Xenopus laevis*                | 8355         |

### Create a BLAST database with single-isoform proteomes for BLAST to use for sorting and studying the 11 subject species
1. RefSeq select was used to take into account gene experience filter out human proteins, since human genome is very large and contains a lot of information
2. Longested isoform for other species 

### The following steps can only be done once.
Extract, uncompress, and concatenate the proteome files from ~bio312/lab03-s1-spoon.
Copyright and credits for creating the proteome file belongs to Professor Joshua Rest, 2024.
```
cd ~/lab03-$MYGIT

gunzip proteomes/*.gz

cat  proteomes/*.faa > allprotein.fas
```
build BLAST database with concenated proteomes and create a working directory
```
makeblastdb -in allprotein.fas -dbtype prot

mkdir ~/lab03-$MYGIT/FIG
cd ~/lab03-$MYGIT/FIG
```
Download query my protein, FIG4, and perform a BLAST search using query protein
# 2. Filtering out HSP to my Query Sequence
### Perform BLAST search to find potential homologs with my query protein: FIG4, accession number: NP_055660.1
```
ncbi-acc-download -F fasta -m protein "NP_055660.1"
blastp -db ../allprotein.fas -query NP_055660.1.fa -outfmt 0 -max_hsps 1 -out FIG.blastp.typical.out
```
Request tabular output for BLAST search
```
blastp -db ../allprotein.fas -query NP_055660.1.fa  -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out FIG.blastp.detail.out
```
### Filter BLAST output for high-scoring putative homologs
set e-vallue to be less than 1e-30
```
awk '{if ($6< 1e-30)print $1 }' FIG.blastp.detail.out > FIG.blastp.detail.filtered.out
```
## RESULTS of MSA performed using BLAST 
Count the number of total hits after the HSP has been filtered out
```
wc -l FIG.blastp.detail.filtered.out
```
### 42 FIG.blastp.detail.filtered.out
to find how many paralogs are found in each species
```
grep -o -E "^[A-Z]\.[a-z]+" FIG.blastp.detail.filtered.out  | sort | uniq -c
```
### number of parlogs in each species
| Species      |   Count   |
|--------------|-----------|
| C.carcharias |     2      |
| C.mydas      |     4      |
| D.rerio      |     5      |
| E.caballus   |     3      |
| F.catus      |     4      |
| G.aculeatus  |     4      |
| G.gallus     |     4      |
| H.sapiens    |     4      |
| S.salar      |     7      |
| S.townsendi  |     3      |
| X.laevis     |     2      |

## The original directory has been copied and placed in this repository using the following code
```
mkdir ~/bio312final/FIG
cp -r ~/lab03-$MYGIT/FIG ~/bio312final/FIG
```
### You can view this by using the following code
```
cd ~/bio312-user/bio312final/FIG/FIG
```
# 3. Alignment to Query Sequence
Use [seqkit](https://bioinf.shenwei.me/seqkit/) to grab filtered out sequences.

# for me to save stuff
cd ~/bio312final
history > bio312final.commandhistory.txt 
find . -size +5M | sed 's|^\./||g' | cat >> .gitignore; awk '!NF || !seen[$0]++' .gitignore
git add .
git commit -a -m "Adding all new data files I generated in AWS to the repository."
git pull --no-edit
git push 