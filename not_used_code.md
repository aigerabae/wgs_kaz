Getting rsIDs:
1) making rsID - location dictionary
```bash
cat kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt | head -n 200 > graphs/header_vcf.tsv
awk -F'\t' '{print $452 "\t" $453 "\t" $455 "\t" $456 "\t" $21}' kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt > graphs/snp_data.tsv
cd graphs/
```

```bash
awk '{key=$1":"$2":"$3; if (key in seen) {print "Duplicate found: " key; duplicates=1} else {seen[key]=1}} END {if (!duplicates) print "No duplicates found in chromosome-start-end triplets."}' snp_data.tsv
```

Script to 
1) count exact matches between dictionary and bim file

```bash
[[ -f kaz.bim && -f snp_data.tsv ]] && echo "Number of matching lines: $(awk 'BEGIN {FS=OFS="\t"} NR==FNR {dict[$1"\t"$2"\t"$3"\t"$4]=1; next} {if (($1"\t"$4"\t"$5"\t"$6) in dict) count++} END {print count}' snp_data.tsv kaz.bim)" || echo "Error: Required files kaz.bim or snp_data.tsv not found!"
```
Problem - only 10160 matches!
 
2) if they are 100% make the bim file with updated rsIDs:
```bash

```


3) copying to local computer: in file: control + L -> sftp://dell_815@10.1.131.145 -> password -> copying to biostar/wgs/

Problem - vcf file doesn't have rsIDs. Solution - take them from annotated version. Problem: when i make a dictionary it only has 10000 matches between chr/position/ref/alt in dictionary and plink file
Now need to 2) keep only snps from my referent datasets; 3) run admixture and PCA

Let's count hpw many rsIDs I have in total:
```bash
awk -F'\t' '$5 != "." {count++} END {print count}' snp_data.tsv

# 23573705 rsIDs and 7118940 missing. not bad. now let's extract them for our refined dictionary

awk -F'\t' '$5 != "."' snp_data.tsv > snp_data2.tsv

```
Problem:
cat kz_235_hg19_dragen.vcf | wc -l
26606334
wc -l kz_235_hg19_dragen.LATEST.annovar.hg19_multianno.header.txt 
30692645 

I made plink file with regular version while annotated contains more SNPs!
