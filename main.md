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
plink/plink2 --vcf ../kz_235_hg19_dragen.vcf --double-id --allow-extra-chr --set-missing-var-ids @:#_hg19 --split-par hg19 --make-bed --out kaz
```

Merging kaz data with estonian:
```bash
plink/plink --bfile estonian3 --bmerge kaz --make-bed --out kaz_est

```


Top Tip: remove indels and merge by position:
 
