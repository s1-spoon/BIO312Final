# BIO312Final
This is my final repository for final term paper for the Bionformatics course taught by Professor Joshua Rest.
This repository will walk you throught all the steps I took to support the basis of my paper.

## Introduction ##

Below is the table of contents.
1. BLAST Proteome Database
2. Alignment to Query Sequence, NP_055660.1
3. Constructing Gene Tree with Best Alignment
4. Cosntructing Species Tree
5. Reconciling Gene Tree within Speces Tree

Clone my final repository
```
git clone https://github.com/s1-spoon/bio312final
```
move into clone folder
```
cd ~/bio312final
```
## 1. BLAST Proteome Database
### 11 chordate species studied in my study
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

The following steps can only be done once.
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
Download query my protein, FIG4
```
ncbi-acc-download -F fasta -m protein "NP_055660.1"
```

### This directory has been copied and placed in this repository using the following code
```
mkdir ~/bio312final/FIG
cp -r ~/lab03-$MYGIT/FIG ~/bio312final/FIG
```





```


### Perform BLAST search to find potential homologs with my query protein: FIG4, accession number: NP_055660.1