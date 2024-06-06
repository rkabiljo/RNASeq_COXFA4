# RNASeq_COXFA4

## Files are here on RDS

## Run FastQC on one to see if there is adapter content
```
cd /home/skgtrk2/Scratch/mito/RNASeq
qsub runFastqc.sh
```
the actual command inside the script:
```
fastqc /home/skgtrk2/Micol/PRJ24080/FASTQ/C2_S8_R2_001.fastq.gz -o fastqcMicol
```
I ran it one one sample and saw that there is Illumina adaptor, so I decided to trim it

## TrimGalore
trimming will also run FastQC, look at the log files
```
cd /home/skgtrk2/Scratch/mito/RNASeq
#submit one by one, like this - with prefix
qsub runTrimGalore.sh C5_S9
```
the command inside the script
```
trim_galore --paired --illumina --output_dir qual_ad_micol --quality 20 --length 25 --fastqc $in_r1 $in_r2
```
#output directory is qual_ad_micol and that's where trimmed fastqcs are - these are now our starting files for further analysis

## Align with Star

```
qsub alignStar.sh C5_S9
```
the command from the script:
```
fastq_r1=qual_ad_micol/${prefix}_R1_001_val_1.fq.gz
fastq_r2=qual_ad_micol/${prefix}_R2_001_val_2.fq.gz
#01_mus2_mit_S3_R1_001_val_1.fq.gz

STAR --runThreadN 6 \
     --genomeDir Ref/genome \
     --readFilesIn  $fastq_r1 $fastq_r2 \
     --sjdbGTFfile Ref/Homo_sapiens.GRCh38.111.gtf \
     --sjdbOverhang 149 \
     --outFileNamePrefix $prefix \
     --outSAMtype BAM SortedByCoordinate \
     --quantMode GeneCounts \
     --readFilesCommand zcat
```
this will produce the bam files all in the current directory. I moved them to star_micol directory. Then I indexed them like this:
```
for bam_file in *.bam; do
samtools index "$bam_file"
 done
```
## Counts with HTSeq

## Run Majiq with STAR output

Prepare the settings file
```
 #cat settings_micol.ini 
[info]
bamdirs=/home/skgtrk2/Scratch/mito/RNASeq/star_micol
sjdirs=/sj
genome=mm10
strandness=None
[experiments]
case=EVELINA_S6Aligned.sortedByCoord.out,FRANCE_S5Aligned.sortedByCoord.out,BRISTOL_S3Aligned.sortedByCoord.out,TURKEY_S4Aligned.sortedByCoord.out,ROB2_S2Aligned.sortedByCoord.out,ROB1_S1Aligned.sortedByCoord.out
control=LB_S10Aligned.sortedByCoord.out,SIA7_S11Aligned.sortedByCoord.out,C5_S9Aligned.sortedByCoord.out,C1_S7Aligned.sortedByCoord.out,C2_S8Aligned.sortedByCoord.out
```
#  Run
```
#build
majiq build -c settings_micol.ini /home/skgtrk2/Scratch/mito/RNASeq/Ref/Homo_sapiens.GRCh38.111.NDUFA4_NDUFA4L2.gff3 -o out_micol -j 1

#delta psi is to contrast groups
majiq deltapsi -o deltapsiOut -n case control -grp1 out_micol/EVELINA_S6Aligned.sortedByCoord.out.majiq out_micol/FRANCE_S5Aligned.sortedByCoord.out.majiq out_micol/BRISTOL_S3Aligned.sortedByCoord.out.majiq out_micol/TURKEY_S4Aligned.sortedByCoord.out.majiq out_micol/ROB2_S2Aligned.sortedByCoord.out.majiq out_micol/ROB1_S1Aligned.sortedByCoord.out.majiq -grp2 out_micol/LB_S10Aligned.sortedByCoord.out.majiq out_micol/SIA7_S11Aligned.sortedByCoord.out.majiq out_micol/C5_S9Aligned.sortedByCoord.out.majiq out_micol/C1_S7Aligned.sortedByCoord.out.majiq out_micol/C2_S8Aligned.sortedByCoord.out.majiq
```
### To view voila results in a local browser
```
ssh -v -L localhost:8080:localhost:35400 skgtrk2@kathleen.rc.ucl.ac.uk
```
Then prepare the environment in the same was as when I ran build and deltapsi
```
```
Run voila
```
voila view -p 35400 out_micol/splicegraph.sql deltapsiOut/case-control.deltapsi.voila
```
what for this putput:
```
2024-06-06 08:52:13,696 (PID:20499) - INFO - Command: /lustre/home/skgtrk2/majiq/bin/voila view -p 35400 out_micol/splicegraph.sql deltapsiOut/case-control.deltapsi.voila
2024-06-06 08:52:13,696 (PID:20499) - INFO - Voila v2.5.6.dev1+g8423f68
2024-06-06 08:52:22,421 (PID:20499) - INFO - ╔═══════════════════════════════════════════════════════════════╗
2024-06-06 08:52:22,421 (PID:20499) - INFO - ╠╡ ACADEMIC License applied                                     ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ║  Name: Official Majiq Academic-only License                   ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ║  File: majiq_license_academic_official.lic                    ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ║  Expiration Date: Never                                       ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ║                                                               ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ╠╡ The academic license is for non-commercial purposes by       ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ╠╡ individuals at an academic or not for profit institution.    ║
2024-06-06 08:52:22,421 (PID:20499) - INFO - ╚═══════════════════════════════════════════════════════════════╝
2024-06-06 08:52:22,421 (PID:20499) - INFO - config file: /tmp/tmpsvdvx788
2024-06-06 08:52:22,622 (PID:20499) - INFO - Creating index: /lustre/home/skgtrk2/majiq/deltapsiOut/case-control.deltapsi.voila
Indexing LSV IDs: 0 / 5
2024-06-06 08:52:27,028 (PID:20499) - INFO - Writing index: /lustre/home/skgtrk2/majiq/deltapsiOut/case-control.deltapsi.voila
Serving on http://localhost:35400

```
Then in a safari, on a laptop open http://localhost:8080, and it opens the voila results page



```

## Run Salmon directly from TrimGalore output
