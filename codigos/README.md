# 1. Programas para ensamblaje: Nanopore
```r
# Se recomienda crear un ambiente nuevo dentro de conda, para cada programa instalado.

# Ver calidad de datos con nanoplot
## Instalar Nanoplot
conda install -c conda-forge -c bioconda nanoplot

# Filtrado de datos con nanofilt
## Insatalar Nanofilt
conda install -c bioconda nanofilt

# Ensamblaje de novo con flye
## Insatalar Flye
conda install -c bioconda flye

# Polishing de datos con minimap2 y racon.
## Instalar Minimap2
conda install -c bioconda minimap2

## Instalar Racon
conda install -c bioconda racon

# Requerimientos de MEDAKA (Pyabpoa, bcftools, samtools (v1.11), minimap2)
## Instalación
pip install pyabpoa
sudo apt install bcftools
conda install -c bioconda samtools==1.11

# Pulido final de la secuancia con medaka
## Instalar medaka
conda install -c conda-forge –c bioconda medaka

```
# 2. Basecalling
```r
# 

```
# 3. Ensamblaje Nanopore
```r
# Crear directorio "lecturas_nanopore".
mkdir lecturas_nanopore ;
cd_lecturas_nanopore ;

# Pegar los archivos fastq.gz que salieron del secuenciador.
# Inspeccionar las longitudes de los reads.
zcat SRR17110067.fastq.gz | grep -n "length" | cut -f2 -d'=' | sort -r -n | uniq | head -n 20
zcat SRR17110070.fastq.gz | grep -n "length" | cut -f2 -d'=' | sort -r -n | uniq | head -n 20

# Ver calidad de datos: nanoplot
NanoPlot -t 5 -o SRR17110067_QC --fastq SRR17110067.fastq.gz
NanoPlot -t 5 -o SRR17110070_QC --fastq SRR17110070.fastq.gz

# Filtrar datos: nanofilt
gunzip -c SRR17110067.fastq.gz | NanoFilt --logfile nanofilt.log -l 500 -q 10 | gzip > SRR17110067.trim.fastq.gz ;
gunzip -c SRR17110070.fastq.gz | NanoFilt --logfile nanofilt.log -l 500 -q 10 | gzip > SRR17110070.trim.fastq.gz ;
ls -lh ;

# Ensamblado de lecturas: flye
flye -o SRR17110067.genoma --nano-raw SRR17110067.trim.fastq.gz --threads 5 ;
flye -o SRR17110070.genoma --nano-raw SRR17110070.trim.fastq.gz --threads 5 ;
ls -lh ;

# Polishing: minimap2 + racon
minimap2 -x ava-ont -t 5 SRR17110067.genoma/assembly.fasta SRR17110067.trim.fastq.gz > overlaps1.paf ;
racon -t 5 SRR17110067.trim.fastq.gz overlaps1.paf SRR17110067.genoma/assembly.fasta > SRR17110067.racon1.fasta ;

minimap2 -x ava-ont -t 5 SRR17110070.genoma/assembly.fasta SRR17110070.trim.fastq.gz > overlaps2.paf ;
racon -t 5 SRR17110070.trim.fastq.gz overlaps2.paf SRR17110070.genoma/assembly.fasta > SRR17110070.racon1.fasta ;

minimap2 -x ava-ont -t 5 SRR17110067.racon1.fasta SRR17110067.trim.fastq.gz > overlaps3.paf ;
racon -t 5 SRR17110067.trim.fastq.gz overlaps3.paf SRR17110067.racon1.fasta > SRR17110067.racon2.fasta ;

minimap2 -x ava-ont -t 5 SRR17110070.racon1.fasta SRR17110070.trim.fastq.gz > overlaps4.paf ;
racon -t 5 SRR17110070.trim.fastq.gz overlaps4.paf SRR17110070.racon1.fasta > SRR17110070.racon2.fasta ;

# Generar secuencias consenso: medaka
medaka_consensus -i SRR17110070.trim.fastq.gz -d SRR17110070.racon2.fasta -o medaka_SRR17110070 -t 5 ;
medaka_consensus -i SRR17110067.trim.fastq.gz -d SRR17110067.racon2.fasta -o medaka_SRR17110067 -t 5 ;

# Evaluar calidad del emsamblado: QUAST
quast.py -o quast_SRR17110067 -m 0 medaka_SRR17110067/consensus.fasta
quast.py -o quast_SRR17110070 -m 0 medaka_SRR17110070/consensus.fasta
```
