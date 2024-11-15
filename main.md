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
# didn't do yet
```

Vcf kazakh to plink:
```bash
plink/plink --vcf ../kz_235_hg19_dragen.vcf --double-id --allow-extra-chr --make-bed --out kaz
```

Getting rsIDs:
1) making rsID - location dictionary
```bash
cat kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt | head -n 200 > graphs/header_vcf.tsv
awk -F'\t' '{print $452 "\t" $453 "\t" $454 "\t" $455 "\t" $456 "\t" $21}' kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt > graphs/snp_data.txt
```

Script to check for duplicates in the dictionary:
```bash
#!/bin/bash

# Input SNP data file
SNP_DATA="snp_data.txt"

# Check for duplicate chromosome-start-end triplets
awk '{
    key = $1 ":" $2 ":" $3;  # Combine Chromosome (col 1), Start (col 2), End (col 3) as key
    if (key in seen) {
        print "Duplicate found: " key;
        duplicates = 1;
    } else {
        seen[key] = 1;
    }
}
END {
    if (!duplicates) {
        print "No duplicates found in chromosome-start-end triplets.";
    }
}' "$SNP_DATA"
```

Script to 
1) count exact matches between dictionary and bim file
2) if they are 100% make the bim file with updated rsIDs:
```bash

```

3) copying to local computer: in file: control + L -> sftp://dell_815@10.1.131.145 -> password -> copying to biostar/wgs/

Problem - vcf file doesn't have rsIDs. Solution - take them from annotated version
Now need to 2) keep only snps from my referent datasets; 3) run admixture and PCA
