Log in: 
```bash
ssh dell_815@10.1.131.145
```

password: dell
How to access vcf files from home: 
```bash
cd ../../media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/
```

```bash
scp ./HGDP.ped dell_815@10.1.131.145:/media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/graphs
scp ./HGDP.map dell_815@10.1.131.145:/media/dell_815/raid_14tb/adaniyarov/kz_235_hg19_dragen/graphs
```

Now need to 1) my vcf to plink; 2) keep only snps from my referent datasets; 3) run admixture and PCA
