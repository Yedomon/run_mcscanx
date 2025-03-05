
# **MCScanX Pipeline Using DIAMOND for Protein Comparisons**


by Ange Zoclanclounon X:@angeomics


## Toy data to play with available [here](https://github.com/wyp1125/MCScanX/tree/master/data)

## **Overview**
This tutorial demonstrates how to use **DIAMOND** instead of BLAST for protein sequence comparisons and then analyze synteny using **MCScanX**. The example uses two species: *Arabidopsis thaliana* (at) and *Perilla frutescens* (pc), but the pipeline can be extended to other species.

## **Act 1: Install MCScanX**
A great tutorial on installing MCScanX is available [here](https://www.youtube.com/watch?v=KMlj8CGnB2c).
It is quite simple. Do


```
git clone https://github.com/wyp1125/MCScanX.git
cd MCScanX
make
```
---

## **Act 2: Conducting a BLAST Search Using DIAMOND**

### **Step 1: Activate the DIAMOND Environment**
```bash
source activate diamond_env
```

### **Step 2: Run All-Versus-All Pairwise Comparisons**
#### **(1) Self-BLAST for *Perilla* (pc vs. pc)**
```bash
diamond makedb --in perilla_v1.0_protein_without_point.fasta -p 64 -d perilla_v1.0_protein_without_point

diamond blastp -d perilla_v1.0_protein_without_point -q perilla_v1.0_protein_without_point.fasta \
  -p 64 --evalue 0.00001 --out pc_vs_pc_diamond.cleaned.csv --outfmt 6 &> log.run.diamond.cleaned.pc_pc &
```

#### **(2) *Perilla* vs. *Arabidopsis* (pc vs. at)**
```bash
diamond makedb --in Athaliana_447_Araport11.protein.cleaned.fa -p 64 -d Athaliana_447_Araport11.protein.cleaned

diamond blastp -d Athaliana_447_Araport11.protein.cleaned -q perilla_v1.0_protein_without_point.fasta \
  -p 64 --evalue 0.00001 --out pc_vs_ara_diamond.cleaned.csv --outfmt 6 &> log.run.diamond.cleaned.pc_ara &
```

#### **(3) *Arabidopsis* vs. *Perilla* (at vs. pc)**
```bash
diamond blastp -d perilla_v1.0_protein_without_point -q Athaliana_447_Araport11.protein.cleaned.fa \
  -p 64 --evalue 0.00001 --out ara_vs_pc_diamond.cleaned.csv --outfmt 6 &> log.run.diamond.cleaned.ara_pc &
```

#### **(4) Self-BLAST for *Arabidopsis* (at vs. at)**
```bash
diamond blastp -d Athaliana_447_Araport11.protein.cleaned -q Athaliana_447_Araport11.protein.cleaned.fa \
  -p 64 --evalue 0.00001 --out ara_vs_ara_diamond.cleaned.csv --outfmt 6 &> log.run.diamond.cleaned.ara_ara &
```

### **Step 3: Concatenate Results**
```bash
cat pc_vs_pc_diamond.cleaned.csv pc_vs_ara_diamond.cleaned.csv \
    ara_vs_pc_diamond.cleaned.csv ara_vs_ara_diamond.cleaned.csv > pc_at.blast
```

---

## **Act 3: Preparing GFF Files for MCScanX**
Modify the GFF3 files of *Perilla* and *Arabidopsis* by keeping only four essential columns:
- **Chromosome ID**
- **Gene ID**
- **Start Position**
- **End Position**

Example format (**at_pc.gff**):
```
at5	at05t0507200	25150045	25150392
at2	at02t0774100	33540257	33541838
at5	at05t0480600	23720056	23723583
at6	at06t0299300	11200887	11202509
at7	at07t0587100	24548577	24551667
pc6	pc06t0299300	11200887	11202509
pc7	pc07t0587100	24548577	24551667
pc5	pc05t0526700	26286838	26287599
```

---

## **Act 4: Running MCScanX**
1. Create a directory for the analysis and move the concatenated BLAST file into it:
   ```bash
   mkdir pc_at_dir
   mv pc_at.blast pc_at_dir/
   ```

2. Run MCScanX:
   ```bash
   ./MCScanX pc_at_dir/pc_at
   ```

This command generates the **collinearity file**, which provides insights into syntenic regions between the genomes.

---


## Act 5: Visualise your synteny plot at https://synvisio.github.io/#/upload by uploading the MCScanX Collinearity and gff files.

## **Conclusion**
This pipeline allows efficient genome synteny analysis using **DIAMOND** for protein comparisons instead of traditional BLAST, speeding up processing while maintaining accuracy. The results from **MCScanX** can be used to identify gene duplications, syntenic blocks, and evolutionary relationships between *Arabidopsis* and *Perilla* (or any other species pairs).

Happy analyzing!

