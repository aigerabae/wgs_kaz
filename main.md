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

# similarly i added plink2 in the same directory. i also installed admixture in another folder like this
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
cat kaz_est_hgdp_metadata.tsv | awk '{print $1 "\t" $1}' > ethnic.txt 
plink/plink --bfile estonian3 --keep ethnic.txt --make-bed --out estonian4
```

Merging HGDP dataset with estonian datasets:
```bash
plink/plink --file HGDP --make-bed --out hgdp
comm -12 <(awk '{print $2}' estonian4.bim | sort) <(awk '{print $2}' hgdp.bim | sort) > common_snps.txt
cat estonian4.bim | awk '{print $2"\t" $1}' > dictionary_chr
cat estonian4.bim | cut -f 2,4 > dictionary_pos
plink/plink --bfile hgdp --extract common_snps.txt --make-bed --out hgdp1
plink/plink2 --bfile hgdp1 --update-chr dictionary_chr --update-map dictionary_pos --sort-vars --make-pgen --out hgdp2
plink/plink2 --pfile hgdp2 --make-bed --out hgdp3
plink/plink --bfile estonian4 --bmerge hgdp3.bed hgdp3.bim hgdp3.fam --make-bed --out ref1
plink/plink --bfile hgdp3 --exclude ref1-merge.missnp --biallelic-only strict --make-bed --out hgdp4
plink/plink --bfile estonian4 --exclude ref1-merge.missnp --biallelic-only strict --make-bed --out estonian5
plink/plink --bfile estonian5 --bmerge hgdp4.bed hgdp4.bim hgdp4.fam --make-bed --out ref2
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
awk '{print $1, $4, $4, 1}' ref2.bim > ref2_positions.txt

comm -12 <(sort kaz_positions.txt) <(sort ref2_positions.txt) > common_positions.txt

plink/plink2 --bfile ref2 --extract range common_positions.txt --make-bed --out ref3
plink/plink2 --bfile kaz1 --extract range common_positions.txt --make-bed --out kaz2

join -j 1 <(awk '{print $1":"$4, $2}' ref3.bim | sort -k1,1) <(awk '{print $1":"$4, $2}' kaz2.bim | sort -k1,1) | awk '{print $2, $3}' > update_ids.txt
awk '!seen[$2]++' update_ids.txt > update_ids2.txt
plink/plink2 --bfile kaz2 --update-name update_ids2.txt 1 2 --make-bed --out kaz3
```

problem - estonian5 and kaz1 have different lengths! - solved by using join
problem - update ids has non unique ids! - trying to solve using a different vcf command; didn't help. i removed duplicates and it worked!

Merging kaz data with estonian+hgdp:
```bash
plink/plink --bfile ref3 \
      --bmerge kaz3 \
      --make-bed \
      --out kaz_ref

plink/plink --bfile ref3 --exclude kaz_ref-merge.missnp --make-bed --out ref4
plink/plink --bfile kaz3 --exclude kaz_ref-merge.missnp --make-bed --out kaz4

plink/plink --bfile ref4 \
      --bmerge kaz4 \
      --make-bed \
      --out kaz_ref1

plink/plink --bfile kaz_ref1 --geno 0.02 --make-bed --out kaz_ref2
plink/plink --bfile kaz_ref2 --mind 0.02 --make-bed --out kaz_ref3
plink/plink --bfile kaz_ref3 --maf 0.001 --make-bed --out kaz_ref4

plink/plink --bfile kaz_ref4 --keep ethnic.txt --make-bed --out kaz_ref5
```

```bash
cat kaz4.fam | cut -d " " -f 1 > kaz_samples.txt

#pca
plink/plink2 --bfile kaz_ref5 --pca 10 --out all_pca
awk '{print $1}' kaz_ref5.fam | grep -Fwf - kaz_est_hgdp_metadata.tsv > ethnic2.txt
python plot_eigenvec.py all_pca.eigenvec ethnic2.txt

# admixture
plink/plink --bfile kaz_ref5 --indep-pairwise 1000 150 0.4 --out pruned_data
plink/plink --bfile kaz_ref5 --extract pruned_data.prune.in --make-bed --out kaz_ref6
awk '{print $1}' kaz_ref6.fam | grep -Fwf - ethnic2.txt > ethnic3.txt
nohup bash -c 'for K in 5 8 12; do admixture/admixture --cv kaz_ref6.bed -j32 $K | tee log${K}.out; done' > nohup.out 2>&1 &

cat ethnic3.txt | awk '{print $2"\t" $1}'  > ethnic3.ind
perl admixture/AncestryPainter.pl -i ethnic3.ind -q ./kaz_ref6.5.Q -t Kazakh -o Kazakh_5 -f png
perl admixture/AncestryPainter.pl -i ethnic3.ind -q ./kaz_ref6.8.Q -t Kazakh -o Kazakh_8 -f png
perl admixture/AncestryPainter.pl -i ethnic3.ind -q ./kaz_ref6.12.Q -t Kazakh -o Kazakh_12 -f png
```

Getting MAFs for pharmacogenes:
```bash
awk -F'\t' '{print $21, $449}' ../kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt > columns_21_449.txt
```

Search for these in columns_21_449.txt:
```bash
grep -w rs2108622 columns_21_449.txt
grep -w rs3745274 columns_21_449.txt
grep -w rs3745274 columns_21_449.txt
grep -w rs4148323 columns_21_449.txt
grep -w rs2070959 columns_21_449.txt
grep -w rs4988235 columns_21_449.txt
grep -w rs1573496 columns_21_449.txt
grep -w rs671 columns_21_449.txt
grep -w rs4148323 columns_21_449.txt
grep -w rs2056900 columns_21_449.txt
grep -w rs2076740 columns_21_449.txt
grep -w rs189261858 columns_21_449.txt
```

Searching for these same SNPs in my GWAS dataset in local computer/biostar/gwas/redo_july/annotation/
```bash
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs2108622
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs3745274
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs3745274
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs4148323
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs2070959
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs4988235
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs1573496
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs671
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs4148323
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs2056900
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs2076740
cat autosomal_224_ext_for_annovar.vcf | cut -f 1,2,3,4,5,234 | grep -w rs189261858
```

How to kill admixture process:
```bash
kill -9 `pgrep admixture`
```

Idea: use the same ref datasets as successful admixture plots such as at https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0135820#sec013
