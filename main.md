Log in: 
```bash
ssh dell_815@10.1.131.145
```

password: dell
How to access vcf files from home: 
```bash
cd ../../media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/
```

Downloading HGDP data from local computer on to server:
*from ~/biostar/gwas/redo_july/working_with_ref_data/hgdp/
```bash
scp ./HGDP.ped dell_815@10.1.131.145:/media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/graphs
scp ./HGDP.map dell_815@10.1.131.145:/media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/graphs
```

Installing PLINK:
```bash
mkdir plink
cd plink
wget https://s3.amazonaws.com/plink1-assets/plink_linux_x86_64_20241022.zip
unzip plink_linux_x86_64_20241022.zip
chmod +x plink
```

Estonian dataset:
```bash
wget https://evolbio.ut.ee/turkic/turkic.fam
wget https://evolbio.ut.ee/turkic/turkic.bim
wget https://evolbio.ut.ee/turkic/turkic.bed
wget https://evolbio.ut.ee/caucasus/caucasus_paper_data_dbSNP-b131_pos-b37_1KG_strand.bim
wget https://evolbio.ut.ee/caucasus/caucasus_paper_data_dbSNP-b131_pos-b37_1KG_strand.fam
wget https://evolbio.ut.ee/caucasus/caucasus_paper_data_dbSNP-b131_pos-b37_1KG_strand.bed
wget https://evolbio.ut.ee/sakha/sakha_paper_data_dbSNP-b131_pos-b37_1KG_strand.bim
wget https://evolbio.ut.ee/sakha/sakha_paper_data_dbSNP-b131_pos-b37_1KG_strand.fam
wget https://evolbio.ut.ee/sakha/sakha_paper_data_dbSNP-b131_pos-b37_1KG_strand.bed
wget https://evolbio.ut.ee/jew/jew_paper_data_dbSNP-b131_pos-b37_1KG_strand.bim
wget https://evolbio.ut.ee/jew/jew_paper_data_dbSNP-b131_pos-b37_1KG_strand.fam
wget https://evolbio.ut.ee/jew/jew_paper_data_dbSNP-b131_pos-b37_1KG_strand.bed

plink/plink --bfile caucasus_paper_data_dbSNP-b131_pos-b37_1KG_strand --bmerge jew_paper_data_dbSNP-b131_pos-b37_1KG_strand --make-bed --out ./estonian1
plink/plink --bfile estonian1 --bmerge sakha_paper_data_dbSNP-b131_pos-b37_1KG_strand --make-bed --out estonian2
plink/plink --bfile estonian2 --bmerge turkic --make-bed --out estonian3
```

Merging HGDP dataset with estonian datasets:
```bash

```

Vcf kazakh to plink:
```bash
plink/plink2 --vcf ../kz_235_hg19_dragen.vcf \
      --double-id \
      --allow-extra-chr \
      --set-missing-var-ids @:#_hg19_\$r_\$a \
      --impute-sex \
      --split-par hg19 \
      --max-alleles 2 \
      --new-id-max-allele-len 50 missing \
      --make-bed \
      --out kaz
```

```bash
awk '$2 == "." {print $1, $4}' kaz.bim > long_allele_variants.txt
plink/plink2 --bfile kaz \
      --exclude long_allele_variants.txt \
      --make-bed \
      --out kaz1


awk '{print $1, $4, $4, 1}' kaz1.bim > kaz_positions.txt
awk '{print $1, $4, $4, 1}' estonian3.bim > estonian3_positions.txt

comm -12 <(sort kaz_positions.txt) <(sort estonian3_positions.txt) > common_positions.txt

plink/plink2 --bfile estonian3 --extract range common_positions.txt --make-bed --out estonian4
plink/plink2 --bfile kaz1 --extract range common_positions.txt --make-bed --out kaz2

join -j 1 <(awk '{print $1":"$4, $2}' estonian4.bim | sort -k1,1) <(awk '{print $1":"$4, $2}' kaz2.bim | sort -k1,1) | awk '{print $2, $3}' > update_ids.txt
awk '!seen[$2]++' update_ids.txt > update_ids2.txt
plink/plink2 --bfile kaz2 --update-name update_ids2.txt 1 2 --make-bed --out kaz3
```

problem - estonian4 and kaz1 have different lengths! - solved by using join
problem - update ids has non unique ids! - trying to solve using a different vcf command; didn't help. i removed duplicates and it worked!



Merging kaz data with estonian:
```bash
plink/plink --bfile estonian4 \
      --bmerge kaz3 \
      --make-bed \
      --out kaz_est

plink/plink --bfile estonian4 --exclude kaz_est-merge.missnp --make-bed --out estonian5
plink/plink --bfile kaz3 --exclude kaz_est-merge.missnp --make-bed --out kaz4

plink/plink --bfile estonian5 \
      --bmerge kaz4 \
      --make-bed \
      --out kaz_est

plink/plink --bfile kaz_est --geno 0.02 --make-bed --out kaz_est1
plink/plink --bfile kaz_est1 --mind 0.02 --make-bed --out kaz_est2
plink/plink --bfile kaz_est2 --maf 0.001 --make-bed --out kaz_est3

cat kaz4.fam | cut -d " " -f 1 > kaz_samples.txt

#pca
plink/plink2 --bfile kaz_est3 --pca 10 --out all_pca

Need to make ethnic_final.tsv! and get plot_eigenvec.py on the server
#python plot_eigenvec.py all_pca.eigenvec ethnic_final.tsv

# admixture
plink/plink --bfile kaz_est3 --indep-pairwise 1000 150 0.4 --out pruned_data
plink/plink --bfile kaz_est3 --extract pruned_data.prune.in --make-bed --out kaz_est4
for K in 8; do admixture/admixture --cv kaz_est4.bed -j8 $K | tee log${K}.out; done

#awk '{print $1}' kaz_est4.fam | grep -Fwf - ethnic2.txt > ethnic3.txt

```


Top Tip: remove indels and merge by position:
 
