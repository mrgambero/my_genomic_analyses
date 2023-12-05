# First step cat the files


```
cd /xdisk/laubitz/schiro/Gabri_genome
mkdir concat

for file in *L001_R1_001.fastq.gz
do
name=${file%%_L001_R1_001.fastq.gz}
cat ${file} ${name}_L001_R1_001.fastq.gz ${name}_L002_R1_001.fastq.gz ${name}_L003_R1_001.fastq.gz ${name}_L004_R1_001.fastq.gz > concat/${name}_R1.fastq.gz
cat ${file} ${name}_L001_R2_001.fastq.gz ${name}_L002_R2_001.fastq.gz ${name}_L003_R2_001.fastq.gz ${name}_L004_R2_001.fastq.gz > concat/${name}_R2.fastq.gz
done
```
Now we clean it

```
#!/bin/bash -l

# --------------------------------------------------------------
### PART 1: Requests resources to run your job.
# --------------------------------------------------------------
### Optional. Set the job name
#SBATCH --job-name=trimming
### Optional. Set the output filename.
### SLURM reads %x as the job name and %j as the job ID
#SBATCH --output=%x-%j.out
### REQUIRED. Specify the PI group for this job
#SBATCH --account=laubitz
### Optional. Request email when job begins and ends
### SBATCH --mail-type=ALL
### Optional. Specify email address to use for notification
### SBATCH --mail-user=schiro@email.arizona.edu
### REQUIRED. Set the partition for your job.
#SBATCH --partition=standard
### REQUIRED. Set the number of cores that will be used for this job. 
#SBATCH --ntasks=12
### REQUIRED. Set the number of nodes
#SBATCH --nodes=1
### REQUIRED. Set the memory required for this job.
#SBATCH --mem=64gb
### REQUIRED. Specify the time required for this job, hhh:mm:ss
#SBATCH --time=05:00:00


# --------------------------------------------------------------
### PART 2: Executes bash commands to run your job
# --------------------------------------------------------------
conda activate quality_check

cd /xdisk/laubitz/schiro/Gabri_genome/concat

mkdir trimmed
mkdir unpaired 
mkdir trimmed/logs

for file in *1.f*; do
	name=${file%%R1.*}

trimmomatic PE ${name}R1.fastq.gz ${name}R2.fastq.gz \
trimmed/${name}R1_trm.fastq.gz unpaired/${name}R1_unpaired.fastq.gz \
trimmed/${name}R2_trm.fastq.gz unpaired/${name}R2_unpaired.fastq.gz \
LEADING:20 TRAILING:20 SLIDINGWINDOW:4:15 MINLEN:50  -threads 16  &>> trimmed/logs/${name}_log.txt

done


cd trimmed
mkdir fastqc
fastqc *.fa* -t 20 -o fastqc
cd fastqc
multiqc .
```
