# Run FASTQC to determine trimming parameters

Load parallel

`module load parallel/20190222/intel-19.0.5`

Set java parameters

`java -Xmx192g`

Activate conda 

`source /home/gabby297/miniconda3/etc/profile.d/conda.sh`

`conda activate fastqc`

`/home/gabby297/miniconda3/envs/fastqc/bin/fastqc`

Change working directory

`cd /work/gabby297/RNA/`

Use a text file with all your sample names for parallel and run `fastqc`

`cat sample_names_final.txt | parallel --jobs 16 "fastqc {}.fq -o /work/gabby297/RNA/fastqc_final/"`

Full bash script with no notes can be found [here()]

# Run Trimmomatic

Load Parallel

`module load parallel/20190222/intel-19.0.5`

Load Trimmomatic

`source /home/gabby297/miniconda3/etc/profile.d/conda.sh `

`conda activate trimmomatic `

`/home/gabby297/miniconda3/envs/trimmomatic/bin/trimmomatic`

Change working directory

`cd /work/gabby297/RNA/merged_fasta/`

Run Trimmomatic

`cat sample_list.txt | parallel "trimmomatic PE {}_R1_001.fastq.gz {}_R2_001.fastq.gz /work/gabby297/RNA/merged_fasta/trimmomatic/{}.1P.fq /work/gabby297/RNA/merged_fasta/trimmomatic/{}.1U.fq /work/gabby297/RNA/merged_fasta/trimmomatic/{}.2P.fq /work/gabby297/RNA/merged_fasta/trimmomatic/{}.2U.fq ILLUMINACLIP:/project/sackettl/software/Trimmomatic-0.39/adapters/NexteraPE-PE.fa:2:30:10 SLIDINGWINDOW:4:20 MINLEN:25 AVGQUAL:20"`

# Use Star to align fasta files to a reference genome

Load Star

`module load star/2.7.11b `

Define directories

`IN_DIR="/work/gabby297/RNA/merged_fasta/trimmomatic"
OUT_DIR="/work/gabby297/RNA/merged_fasta/bam_files"
GENOME="/project/sackettl/HAAM-from-OAAM-ref-genome"`

Make the ouput directory

`mkdir -p "$OUT_DIR"`

Change directory to input files' location

`cd "$IN_DIR"`

Run Star

`for r1 in *.1P.fq; do
  r2="${r1/.1P.fq/.2P.fq}"
  sample=$(basename "$r1" .1P.fq)

 ` STAR \
    --genomeDir "$GENOME" \
    --runThreadN 19 \
    --readFilesIn "$r1" "$r2" \
    --outFileNamePrefix "$OUT_DIR/${sample}_" \
    --outSAMtype BAM SortedByCoordinate \
    --outSAMstrandField intronMotif
done`

# Merge BAM Files with SAMtools

Load samtools

`source /home/gabby297/miniconda3/etc/profile.d/conda.sh`

`conda activate samtools`

`/home/gabby297/miniconda3/envs/samtools/bin/samtools`

Change directory to bam files' location
`cd /work/gabby297/RNA/merged_fasta/bam_files`

Index individual bams (good practice)
`for bam in *_Aligned.sortedByCoord.out.bam; do
  samtools index "$bam"
done`

Merge
`samtools merge -@ 19 Maui_all_samples.bam *_Aligned.sortedByCoord.out.bam`

Sort + index (merge usually preserves sort, but I always re-sort to be safe)
`samtools sort -@ 19 -o Maui_all_samples.sorted.bam Maui_all_samples.bam
samtools index Maui_all_samples.sorted.bam`

# Use merged BAM file for transcriptome assembly with Trinity

Load Trinity

`source /home/gabby297/miniconda3/etc/profile.d/conda.sh
conda activate trinity`

Run

`Trinity --genome_guided_bam /work/gabby297/RNA/merged_fasta/bam_files/Maui_all_samples.bam  \
--genome_guided_max_intron 30000 \
--SS_lib_type FR \
--jaccard_clip \
--normalize_reads \
--max_memory 180G --CPU 18 \
--output /work/gabby297/RNA/merged_fasta/trinity/`

# Use BUSCO to assess alignment

Load busco

`source /home/gabby297/miniconda3/etc/profile.d/conda.sh
conda activate busco_env`

Run using busco downloaded lineage reference

`busco -i /work/gabby297/RNA/merged_fasta/trinity/Trinity-GG.fasta -l /work/gabby297/RNA/trinity/busco_downloads/lineages/aves_odb10 -o busco_merged -m transcriptome`

## Pipeline in Progress 🏗️
