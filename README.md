# BIO312Final
This is my final repository for final term paper for the Bionformatics course taught by Professor Joshua Rest.
This repository will walk you throught all the steps I took to support the basis of my paper.
In order for you to be able to view the files I created for my analysis, the original directory has been copied and placed in this repository using the following codes
```
mkdir ~/bio312final/FIG
cp -r ~/lab03-$MYGIT/FIG ~/bio312final/FIG

cp -r ~/lab04-$MYGIT/FIG ~/bio312final/FIG

cp -r ~/lab05-$MYGIT/FIG ~/bio312final/FIG
```

## Introduction 

Below is the table of contents.
1. BLAST Proteome Database
2. Filtering out HSP to my Query Sequence
3. Alignment to Query Sequence
4. Constructing Species Tree
5. Constructing Gene Tree with Best Alignment
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
## Version: Protein-Protein BLAST 2.16.0+
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
set e-value to be less than 1e-30
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

# 3. Alignment to Query Sequence
Use [seqkit](https://bioinf.shenwei.me/seqkit/) to grab "pattern file" filtered out sequences from proteome file and naming the file "FIG.homologs.fas"
```
seqkit grep --pattern-file ~/lab03-$MYGIT/FIG/FIG.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit grep -v -p "carpio" > ~/lab04-$MYGIT/FIG/FIG.homologs.fas
```
### Results: 42 patterns loaded from file
Use MUSCLE to perform a global nultiple sequence alignment.
## Muscle Version: muscle 5.2.linux64
```
muscle -align ~/lab04-$MYGIT/FIG/FIG.homologs.fas -output ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas
```
Use Rscript to print MSA.
```
Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas
```

## Alignbuddy Version: AlignBuddy 1.4.0 (2021-12-05)
To find Alignment length
```
alignbuddy  -al  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas
```
### RESULTS: Alignment length = 2364
To find Alignment length w gaps removed
```
alignbuddy -trm all  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | alignbuddy  -al
```
### RESULTS: Alignment length w Gaps Removed: 26
To find Alignment length with invariant (completely conserved) positions removed
```
` alignbuddy -dinv 'ambig' ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | alignbuddy  -al`
```
### RESULTS: Alignment length with invariant (completely conserved) positions removed = 2363
### This lead to my analysis that there were 2338 columns in the alignment with gaps, and 1 column in the alignment that is invariant.

To find the average percent identity among all sequences, I used T_coffee and Alignbuddy.

## T-Coffee Version: T-COFFEE Version_13.46.0.919e8c6b
```
t_coffee -other_pg seq_reformat -in ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas -output sim
```
## Alignbuddy Version: AlignBuddy 1.4.0 (2021-12-05)
```
alignbuddy -pi ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | awk ' (NR>2)  { for (i=2;i<=NF  ;i++){ sum+=$i;num++} }
     END{ print(100*sum/num) } '
```
### RESULT: Average percent identity among all sequences
## T-Coffee: 41.78 (VAR = 568.39, STD = 23.84)
## Alignbuddy: 29.5956

# 4. Constructing Species Tree
I use Newick Utilities to constsruct a species tree.
## Newick Utilities Version: 
I constructed a ASCII rooted species tree for my 11 species, and generated a graphic (.svg file)
```
echo "((((((F.catus,E.caballus)Laurasiatheria,H.sapiens)Boreoeutheria,(S.townsendi,(C.mydas,G.gallus)Archelosauria)Sauria)Amniota,X.laevis)Tetrapoda,(D.rerio,(S.salar,G.aculeatus)Euteleosteomorpha)Clupeocephala)Euteleostomi,C.carcharias)Gnathostomata;"  > ~/lab05-$MYGIT/species.tre

nw_display ~/lab05-$MYGIT/species.tre
nw_display -s ~/lab05-$MYGIT/species.tre > ~/lab05-$MYGIT/species.tre.svg
```
I converted the .svg to .pdf
```
convert ~/lab05-$MYGIT/species.tre.svg ~/lab05-$MYGIT/species.tre.pdf
```
# 5. Constructing Gene Tree with Best Alignment
I used IQ-TREE to find the maximum likehood tree estimate. 
## IQTree Version: IQ-TREE multicore version 2.3.6 for Linux x86 64-bit built Aug  1 2024
(Developed by Bui Quang Minh, Nguyen Lam Tung, Olga Chernomor, Heiko Schmidt,
Dominik Schrempf, Michael Woodhams, Ly Trong Nhan, Thomas Wong)

I calculated the the optimal amino acid substitution model and amino acid frequencies. At the same time a tree search and estimate tree branch lengths, which are the bootstrap values.
```
iqtree -s ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas -bb 1000 -nt 2 
```
## RESULT: log likelihood of the optimal tree that IQ-TREE found: -29778.492

To find the rate heterogeneity multiplier to estimate the tree, I used the following code:
```
less ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas.iqtree
```
## RESULT: Model of rate heterogeneity: Invar+FreeRate with 4 categories 

I used gotree to construct a midpoint rooted gene tree. The unrooted gene tree to constructed into a midpoint rooted tree using the code below.
```
gotree reroot midpoint -i ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile
```
The rooted tree is a phylogram gene tree using Newick Utilities graphic (.svg), which gets converted into a .pdf file using the codes below. The different branch lengths are the bootstrap values previously calculated.
```
nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile  | nw_display -

nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile | nw_display -w 1000 -b 'opacity:0' -s  >  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.svg -

convert  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.svg  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.pdf
```

The phylogram is converted to a cladogram gene tree using Newick Utilities graphic (.svg), which gets converted into a .pdf file using the codes below. The cladogram is easier to view than the phylogram because the length of the branches are uniform.
```
nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile | nw_topology - | nw_display -s  -w 1000 > ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.svg -

convert ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.svg ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.pdf
```


# for me to save stuff
cd ~/bio312final
history > bio312final.commandhistory.txt 
find . -size +5M | sed 's|^\./||g' | cat >> .gitignore; awk '!NF || !seen[$0]++' .gitignore
git add .
git commit -a -m "Adding all new data files I generated in AWS to the repository."
git pull --no-edit
git push 