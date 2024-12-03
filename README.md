
**SUMF Gene Family Phylogenetic Analysis Pipeline**
This repository contains the workflow and scripts used to analyze the phylogenetic relationships within the SUMF gene family.

**Prerequisites:**
Ensure the following tools and dependencies are installed:

NCBI E-utilities (ncbi-acc-download)
BLAST+ suite
Standard Unix tools (awk, grep, sed)
MUSCLE (alignment tool)
IQ-TREE (phylogenetic analysis)
seqkit
Newick Utilities (nw_display, nw_order, nw_reroot)
gotree (tree manipulation)
LaTeX (PDF generation)
Notung (gene tree/species tree reconciliation)
thirdkind (visualization)
RPS-BLAST
Pfam database
R (with ggtree and drawProteins libraries)
Python 2.7
Workflow:

**1. Sequence Collection and BLAST Analysis**
a. Sequence Retrieval:
Download SUMF protein sequences in FASTA format:
ncbi-acc-download -F fasta -m protein "<SUMF_ACCESSIONS>"
Output: SUMF.fasta

b. BLAST Search:
Perform a BLAST search against a custom protein database:
blastp -db ../allprotein.fas -query SUMF.fasta -outfmt 0 -max_hsps 1 -out SUMF.blastp.out
Output: SUMF.blastp.out

c. Filtering and Analysis:
Filter BLAST results for significant hits (E-value < 1e-5):
awk '{if ($6< 1e-5)print $1 }' SUMF.blastp.detail.out > SUMF.blastp.filtered.out
Count total hits: wc -l SUMF.blastp.filtered.out
Output: SUMF.blastp.filtered.out

**2. Multiple Sequence Alignment**
a. Generate Alignment:
Use MUSCLE to align sequences:
muscle -align SUMF.fas -output SUMF.aligned.fas
Output: SUMF.aligned.fas

**3.Phylogenetic Tree Construction and Visualization**
a. Phylogenetic Tree Construction:
Build a maximum likelihood tree with 1,000 bootstraps:
iqtree -s SUMF.aligned.fas -bb 1000 -nt 2
Outputs:

SUMF.treefile
SUMF.treefile.log
SUMF.treefile.iqtree
b. Midpoint Rooting:
Root the tree at the midpoint:
gotree reroot midpoint -i SUMF.treefile -o SUMF.mid.treefile
Output: SUMF.mid.treefile

c. Visualization:
Generate an SVG and PDF visualization:
nw_display -w 1000 -b 'opacity:0' -s SUMF.mid.treefile > SUMF.tree.svg
convert SUMF.tree.svg SUMF.tree.pdf
Outputs: SUMF.tree.svg, SUMF.tree.pdf

**4. Phylogenetic Reconciliation Analysis**
a. Prepare Files and Directory:
Create a working directory and copy necessary files:
mkdir ~/lab06-$MYGIT/SUMF
cp ~/lab05-$MYGIT/mygenefamily/mygenefamily.homologs.al.mid.treefile ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile

b. Reconciliation:
Perform reconciliation using Notung:
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar
-s ~/lab05-$MYGIT/species.tre
-g ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile
--reconcile --speciestag prefix --savepng --events
--outputdir ~/lab06-$MYGIT/SUMF/

c. Extract Species Tree:
Visualize the reconciled species tree:
grep NOTUNG-SPECIES-TREE ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.ntg |
sed -e "s/^&&NOTUNG-SPECIES-TREE//" -e "s//;/" | nw_display -

d. Convert to RecPhyloXML:
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py
-g ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.ntg
--include.species

e. Generate reconciliation visualization with thirdkind:
thirdkind -Iie -D 40 -f ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.ntg.xml
-o ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.svg
convert -density 150 ~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.svg
~/lab06-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile.rec.0.pdf
Outputs:

SUMF.homologsf.al.fas.treefile.rec.0.svg
SUMF.homologsf.al.fas.treefile.rec.0.pdf
**5. Protein Domain Analysis (with RPS-BLAST)**
a. Prepare Sequence File:
sed 's/*//' ~/lab04-$MYGIT/SUMF/SUMF.homologs.fas > ~/lab08-$MYGIT/SUMF/SUMF.homologs.fas
Output: SUMF.homologs.fas

b. Domain Search:
rpsblast -query ~/lab08-$MYGIT/SUMF/SUMF.homologs.fas
-db ~/data/Pfam/Pfam
-out ~/lab08-$MYGIT/SUMF/SUMF.rps-blast.out
-outfmt "6 qseqid qlen qstart qend evalue stitle"
-evalue .0000000001
Output: SUMF.rps-blast.out

c. Tree and Domain Visualization:
Rscript --vanilla ~/lab08-$MYGIT/plotTreeAndDomains.r
~/lab08-$MYGIT/SUMF/SUMF.homologsf.al.fas.treefile
~/lab08-$MYGIT/SUMF/SUMF.rps-blast.out
~/lab08-$MYGIT/SUMF/SUMF.tree.rps.pdf
Output: SUMF.tree.rps.pdf

