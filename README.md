# BIO312Final
This is my final repository for my BIO312 Bioinformatics final repository for the Fall 2024 semester. The course is taught at Stony Brook University by Professor Joshua Rest.
The goal of this repository is to walk you throught all the steps I took to support the basis of my paper, "Finding a Model Organism for Studying Human Illnesses caused by FIG4 Mutations through Phylogenetic Analysis of the SAC-Domain Protein Gene Family".

# Introduction 
Below is the table of contents and sequential order of the steps I took to find and analyze my results.
The gene and species tree figures generated can be found in the folder titled "FIG".
1. BLAST Proteome Database
2. Filtering out HSP homologs with e-value less than 1e-30 to my Query Sequence
3. Alignment to Query Sequence
4. Constructing Species Tree
5. Constructing Gene Tree with Best Alignment
6. Reconciling Gene Tree and Species Tree
7. Constructing a Reconciled Gene Tree Superimposed into Species Tree

To work through the workflow of my repository first clone my repository and change to my directory:
```
git clone https://github.com/s1-spoon/bio312final

cd ~/bio312final
```
You should be able to see a file named "FIG". Within the FIG file, you can view the files I created for my research during BIO312 Lab. I copied these files by using the follow lines of code:
```
mkdir ~/bio312final/FIG
cp -r ~/lab03-$MYGIT/FIG ~/bio312final/FIG
cp -r ~/lab04-$MYGIT/FIG ~/bio312final/FIG
cp -r ~/lab05-$MYGIT/FIG ~/bio312final/FIG
cp -r ~/lab05-$MYGIT/species.tre.pdf ~/bio312final/FIG
cp -r ~/lab06-$MYGIT/FIG ~/bio312final/FIG
```
# 1. BLAST Proteome Database
I created a BLAST database with single-isoform proteomes for BLAST to use for sorting and studying 11 chordate subject species. The list of species accompanied by their Taxonomic ID in the chart below.

### 11 Chordate Species in My Study
| Organism                         | Taxonomic ID |
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

### Create BLAST Database by Filterring Out Desired Proteomes
To created the BLAST database, I used RefSeq to filter out BLAST search sequence results by two bases: 1) Since human gene are very large and contains of information, I needed account for gene expression and filter out human proteins, and 2) I set the parameter to filter our the Longest isoform for the non-human species. 

To repeat what I did, I used the following code to extract, uncompress, and concatenate the proteome files from ~bio312/lab03-s1-spoon. Copyright and credit for creating the proteome file belongs to Professor Joshua Rest, 2024. Note: that this step can only be performed once.
```
cd ~/lab03-$MYGIT

gunzip proteomes/*.gz

cat  proteomes/*.faa > allprotein.fas
```
I used the following code to build the BLAST database with concenated proteomes. I created a working directory containing these proteomes.
```
makeblastdb -in allprotein.fas -dbtype prot

mkdir ~/lab03-$MYGIT/FIG
cd ~/lab03-$MYGIT/FIG
```
# 2. Filtering out HSP homologs with e-value less than 1e-30 to my Query Sequence
I performed a BLAST search to find and download potential homologs with my query protein: FIG4 (accession number: NP_055660.1) to further study my gene family, SAC-Domain Proteins within my 11 chordate species. I used BLASTp Version: Protein-Protein BLAST 2.16.0+. 

```
ncbi-acc-download -F fasta -m protein "NP_055660.1"
blastp -db ../allprotein.fas -query NP_055660.1.fa -outfmt 0 -max_hsps 1 -out FIG.blastp.typical.out
```
I performed a multiple sequence alignment (MSA) for proteomes that matched my query protein, and filtered out the BLAST ouput for high-scoring putative homologs (HSP) with the e-value set at less than 1e-30 using the following code.
```
blastp -db ../allprotein.fas -query NP_055660.1.fa  -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out FIG.blastp.detail.out

awk '{if ($6< 1e-30)print $1 }' FIG.blastp.detail.out > FIG.blastp.detail.filtered.out
```
### MSA Analysis
I analyzed the results of the MSA and found that there were 42 total hits for all the subject species after filtering out the HSP.
This value was found using the following code to count the number of HSP hits.
```
wc -l FIG.blastp.detail.filtered.out
```
The number of paralogs for each subject species is recorded in the table below. The table was generated using the following code:
```
grep -o -E "^[A-Z]\.[a-z]+" FIG.blastp.detail.filtered.out  | sort | uniq -c
```
### Number of Parlogs in each Subject Species
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
## Global MSA
I performed a global multiple sequence alignment using Muscle (Version: muscle 5.2.linux64). To begin, I first grabbed the "pattern file" filtered out sequences from proteome file using [seqkit](https://bioinf.shenwei.me/seqkit/). I printed the file with the filtered out sequences using Rscript and named the file "FIG.homologs.fas", can be found in the FIG folder. The lines of code used are below.
```
seqkit grep --pattern-file ~/lab03-$MYGIT/FIG/FIG.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit grep -v -p "carpio" > ~/lab04-$MYGIT/FIG/FIG.homologs.fas

muscle -align ~/lab04-$MYGIT/FIG/FIG.homologs.fas -output ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas

Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas
```
## Analysis of Global MSA
### Alignment Lengths
To analyze the global MSA I conducted using Muscle, I used Alignbuddy (Version: AlignBuddy 1.4.0 (from 2021-12-05)). I found that the alignment length was 2364, alighment length with gaps removed was 26, and alignment length with invariant (completely conserved) position removed was 2363. This led to my analysis that there were 2338 columns in the alignment with gaps, and 1 column in the alignment that is invariant. The codes I used that lead to this analysis are below.

To find Alignment Length:
```
alignbuddy  -al  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas
```
To find Alignment Length with Gaps Removed:
```
alignbuddy -trm all  ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | alignbuddy  -al
```
To find Alignment Length with Invariant Positions Removed
```
alignbuddy -dinv 'ambig' ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | alignbuddy  -al
```
### Average Percent Identity using 2 Programs

I found the average percent identity among all sequences using two methods: T_coffee (Version: T-COFFEE Version_13.46.0.919e8c6b) and Alignbuddy. The results of each calculation different. T-coffee resulted in 41.78 (VAR = 568.39, STD = 23.84) percent identity of all sequences, while Alignbuddy results in 29.5956 percent identity of all sequences. The following are the codes I ran to gather these results:
```
t_coffee -other_pg seq_reformat -in ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas -output sim

alignbuddy -pi ~/lab04-$MYGIT/FIG/FIG.homologs.al.fas | awk ' (NR>2)  { for (i=2;i<=NF  ;i++){ sum+=$i;num++} }
     END{ print(100*sum/num) } '
```

# 4. Constructing Species Tree
I use Newick Utilities to constsruct a ASCII rooted species tree for my 11 species. The tree file is named "species.tre" and can be viewed as a .svg graphic or .pdf file in the FIG folder. I used the following lines of code to generate my species tree.
```
echo "((((((F.catus,E.caballus)Laurasiatheria,H.sapiens)Boreoeutheria,(S.townsendi,(C.mydas,G.gallus)Archelosauria)Sauria)Amniota,X.laevis)Tetrapoda,(D.rerio,(S.salar,G.aculeatus)Euteleosteomorpha)Clupeocephala)Euteleostomi,C.carcharias)Gnathostomata;"  > ~/lab05-$MYGIT/species.tre

nw_display ~/lab05-$MYGIT/species.tre
nw_display -s ~/lab05-$MYGIT/species.tre > ~/lab05-$MYGIT/species.tre.svg

convert ~/lab05-$MYGIT/species.tre.svg ~/lab05-$MYGIT/species.tre.pdf
```
# 5. Constructing Gene Tree with Best Alignment
### Searching for the Gene Tree with Best Alignment Using IQTree
To construct a gene tree with the best alignment, I used IQ-TREE (Version: IQ-TREE multicore version 2.3.6 for Linux x86 64-bit built Aug  1 2024) to find and generate the maximum likehood tree estimate by calculating the optimal amino acid substitution model and amino acid frequencies. At the same time, a tree search with branch length estimates was running. The brangth lengths are the bootstrap values. The optimal tree IQ-TREE found had a log likelihood of -29778.492. The model of rate heterogeneity for this tree was "Invar+FreeRate with 4 categories". I used the following lines of code to get these results:
```
iqtree -s ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas -bb 1000 -nt 2 

less ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas.iqtree
```
### Constructing a Midpoint Rooted Gene Tree from the Gene Tree with Best Alignment using Gotree
I used Gotree (v0.4.5) to construct a midpoint rooted gene tree with the tree with best alignment generated by IQtree. The unrooted gene tree to constructed into a midpoint rooted tree using the code below. I generated two versions of this tree using Newick Utilities to view it in two formats. The first is a phylogram gene tree with different branch lengths, which are the bootstrap values previosuly calculted by IQTree. This phylogram tree is named "FIG.homologsf.al.mid.treefile". The second is a cladogram gene tree with uniform branch lengths that are annotated with the bootstrap values. This cladogram tree is easier to view, and is named "FIG.homologsf.al.midCl.treefile". Both tree can be viewed can found in the FIG folder and be viewed as both .svg graphic or a .pdf file. The lines of code I used to generated these gene tree files are below.

Phylogram Gene Tree
```
gotree reroot midpoint -i ~/lab05-$MYGIT/FIG/FIG.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile

nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile  | nw_display -

nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile | nw_display -w 1000 -b 'opacity:0' -s  >  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.svg -

convert  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.svg  ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.pdf
```
Cladogram Gene Tree (Uniform Branches, easier to view)
```
nw_order -c n ~/lab05-$MYGIT/FIG/FIG.homologsf.al.mid.treefile | nw_topology - | nw_display -s  -w 1000 > ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.svg -

convert ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.svg ~/lab05-$MYGIT/FIG/FIG.homologsf.al.midCl.treefile.pdf
```
# 6. Reconciling the Gene Tree and Species Tree
### Using Notung
I performed the reconciliation of the gene tree and species tree using Notung (Version: 3 (Notung-3.0_24-beta) to estimate duplication events and loss events. I assigned the internal node names with the common ancestral organism names. The gene tree reconciliation allow me to distinguish orthologs and paralogs, and resulted in 21 total duplication event and 59 total loss events (in the rec.event.txt file). I used the following line of code to perform this analyses.
```
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s ~/lab05-$MYGIT/species.tre -g ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/FIG/
```
# 7. Constructing a Reconciled Gene Tree Superimposed into Species Tree
### Using Python and Thirdkind
I generated a RecPhyloXML object using python (Version: Python 2.7, conda environment) to view the gene-within-species tree reconciled using Thirdkind (Version: Thirdkind 3.6.8). The cost of my reconciled tree was 90.5. The Reconciled Gene Tree Superimposed into Species Tree is named "FIG.homologsf.al.mid.treefile.rec" and can be view as a .pdf or .svg graphic in the FIG folder. I used the following lines of code to generate these files.
```
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.rec.ntg --include.species

thirdkind -Iie -D 40 -f ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.rec.ntg.xml -o  ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.rec.svg

convert  -density 150 ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.rec.svg ~/lab06-$MYGIT/FIG/FIG.homologsf.al.mid.treefile.rec.pdf
```
# End Notes
This concludes my final repository and the code used to generate my results and analyses. Further interpretation of my results can be found in my term paper, submitted via Brightspace.

Copyright of all work found in this repository belongs to Joshua Rest, 2024.
All analyses in this repository regarding the SAC-domain protein and FIG4 protein was generated by Sara Poon, 2024. 