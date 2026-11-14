# somatico-ep01
Amostras Extras da Análise Somática

# chr19
```bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr19.fa.gz
```

# chr13
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr13.fa.gz


# criar o hg19.fa
zcat chr13.fa.gz chr19.fa.gz | sed -e "s/chr//g" > hg19.fa

# criar o .dict
./gatk-4.2.2.0/gatk CreateSequenceDictionary -R hg19.fa -O hg19.dict

# criar os intervalos
./gatk-4.2.2.0/gatk ScatterIntervalsByNs -R hg19.fa -O hg19.interval_list -OT ACGT

# verificar o SM: da amostra normal
samtools view -H tumor_wp191.bam | grep RG | cut -f5 | sed -e "s/SM://g"
samtools view -H tumor_wp018.bam | grep RG | cut -f6 | sed -e "s/SM://g"

# normais
#output: WP191
#output: WP018

# rodar mutect2
./gatk-4.2.2.0/gatk Mutect2 \
	-R hg19.fa \
	-I tumor_wp190.bam \
	-I tumor_wp191.bam \
	-normal WP191 \
	--germline-resource af-only-gnomad-chr13-chr19.vcf.gz \
	-O wp190.somatic.vcf.gz \
	-L hg19.interval_list

# calcular contamination
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp190.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp190.table

./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp191.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp191.table

./gatk-4.2.2.0/gatk CalculateContamination \
	-I tumor_wp190.table \
	-matched tumor_wp191.table \
	-O wp190.contamination.table

# criar filtered
./gatk-4.2.2.0/gatk FilterMutectCalls \
	-R hg19.fa \
	-V wp190.somatic.vcf.gz \
	--contamination-table wp190.contamination.table \
	-O wp190.filtered.vcf.gz


    # ---------------- pareado WP017

    # rodar mutect2
./gatk-4.2.2.0/gatk Mutect2 \
	-R hg19.fa \
	-I tumor_wp017.bam \
	-I tumor_wp018.bam \
	-normal WP018 \
	--germline-resource af-only-gnomad-chr13-chr19.vcf.gz \
	-O wp017.somatic.vcf.gz \
	-L hg19.interval_list

# calcular contamination
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp017.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp017.table

./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I tumor_wp018.bam \
	-V af-only-gnomad-chr13-chr19.vcf.gz \
	-L hg19.interval_list \
	-O tumor_wp018.table

./gatk-4.2.2.0/gatk CalculateContamination \
	-I tumor_wp017.table \
	-matched tumor_wp018.table \
	-O wp017.contamination.table

# criar filtered
./gatk-4.2.2.0/gatk FilterMutectCalls \
	-R hg19.fa \
	-V wp017.somatic.vcf.gz \
	--contamination-table wp017.contamination.table \
	-O wp017.filtered.vcf.gz
