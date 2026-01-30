Run FASTQC to determine trimming parameters

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

Run Trimmomatic

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
