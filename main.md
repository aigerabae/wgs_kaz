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
      --make-bed \
      --out kaz
```

```bash
awk '{print $1, $4, $4, 1}' kaz.bim > kaz_positions.txt
awk '{print $1, $4, $4, 1}' estonian3.bim > estonian3_positions.txt

comm -12 <(sort kaz_positions.txt) <(sort estonian3_positions.txt) > common_positions.txt

plink/plink2 --bfile estonian3 --extract range common_positions.txt --make-bed --out estonian4
plink/plink2 --bfile kaz --extract range common_positions.txt --make-bed --out kaz1

join -j 1 <(awk '{print $1":"$4, $2}' estonian4.bim | sort -k1,1) <(awk '{print $1":"$4, $2}' kaz1.bim | sort -k1,1) | awk '{print $2, $3}' > update_ids.txt
plink/plink2 --bfile kaz1 --update-name update_ids.txt 1 2 --make-bed --out kaz2
```

problem - estonian4 and kaz1 have different lengths! - solved by using join
problem - update ids has non unique ids! - trying to solve using a different vcf command



Merging kaz data with estonian:
```bash
plink/plink --bfile estonian3 --bmerge kaz --make-bed --out kaz_est

```


Top Tip: remove indels and merge by position:
 
