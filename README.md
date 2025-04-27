# AutoGVP scripts
Scripts for running autoGVP and the pre processing steps

# Clone repo
Autogvp — https://github.com/diskin-lab-chop/AutoGVP?tab=readme-ov-file
```bash
cd ~/OpenPedCan_Data/
git clone git@github.com:diskin-lab-chop/AutoGVP.git
```


# First, Split multi-patient VCF files into single VCF files. 
List of proband IDs were pulled from the Variant Workbench (VWB) table called `Occurences`
with the following code (Code is in the U24 Data Studio session on CAVATICA):
```python
ocr = spark.read.parquet(OCR_STUDY_PATH).where( (F.col('is_proband') == True))
chd = ocr.select('sample_id').distinct().toPandas()
pd.DataFrame(chd['sample_id'].values).to_csv('/sbgenomics/output-files/chd_probands.txt', index=False,header=False)
```


There are two bash scripts that split the multi-patient VCFs. 
~/OpenPedCan_Data/single_vcfs/**split_vcfs.sh** --> uses bcftools to split the VCFs
~/OpenPedCan_Data/single_vcfs/**run_splits.sh** --> calls split_vcfs.sh for each cohort (eg CHD, NBL, GNINT, etc.)

```
# (ignore the names of the files eg. CHD_KF_*, these are incorrect)
GNINT VCFs = ~/OpenPedCan_Data/CHD_KF_phs001846/
CHD VCFs =  ~/OpenPedCan_Data/CHD_KF_phs001138/vcf/vep_105/
NBL VCFs = ~/OpenPedCan_Data/CHD_KF_phs001436/vcf/

# Proband lists locations
chd_probands= ~/OpenPedCan_Data/single_vcfs/proband_ids/chd_probands.txt
nbl_probands= ~/OpenPedCan_Data/single_vcfs/proband_ids/nbl_probands.txt
gnint_probands= ~/OpenPedCan_Data/single_vcfs/proband_ids/gnint_probands.txt

# Output dirs
chd_out_dir= ~/OpenPedCan_Data/single_vcfs/split_vcfs/chd/
gnint_out_dir= ~/OpenPedCan_Data/single_vcfs/split_vcfs/gnint/
nbl_out_dir= ~/OpenPedCan_Data/single_vcfs/split_vcfs/nbl/
```


# 1. InterVar 
### Needs to be run using the HPC via SLURM job submission. See below for details
takes about an hour per a 2GB VCF...
Directory = /scr1/users/stearb/U24/InterVar

```bash
git clone https://github.com/WGLab/InterVar.git
# put mim2gene.txt in intervar/Intervar/intervardb/, aka GitHub repo dir.
curl https://www.omim.org/static/omim/data/mim2gene.txt > intervar/Intervar/intervardb/mim2gene.txt  
```
Copy Perl scripts from annovar to interVar directory (InterVar needs several of them.)
`cp ~/OpenPedCan_Data/autoGVP/AutoGVP-main/prereqs/annovar/*.pl ~/OpenPedCan_Data/autoGVP/AutoGVP-main/prereqs/intervar/InterVar/`

Run in the InterVar GitHub directory (this will d/l a bunch of large files the first time it's run.)
```
python Intervar.py -b hg38 -i ~/OpenPedCan_Data/CHD_KF_phs001846/809aa738-a3a2-4923-ae67-b065ba5f353e.single.vqsr.filtered.vep_105.vcf.gz --input_type=VCF -o test_VEP_interVar
```


## Submit an array job to process both CHD and NBL cohorts 

get number of files in both cohorts. Set it as a parameter in the script you will run sbatch with.
```
# https://stackoverflow.com/questions/77789177/how-to-run-a-slurm-job-array-that-iterates-over-a-number-of-files
ls /mnt/isilon/opentargets/OpenPedCan_Data/single_vcfs/split_vcfs/chd/*single.vcf.gz > chd_files_list.txt
cat chd_files_list.txt nbl_files_list.txt > chd_nbl_vcfs.txt
rm chd_files_list.txt nbl_files_list.txt 
wc -l chd_nbl_vcfs.txt     # put this in this parameter #SBATCH --array=1-1157

# and run with:
# this will launch 1157 jobs, so test before you do this
sbatch --array=1-1157 test.sh 
```

Look at the output files to check if there were errors:
```
# check if any of the interval jobs had an ‘Error:’ 
grep -rnw 'out_files/' -e 'Error:' 
```
If you do | wc -l , you can get the number of files that have an ‘Error:’



# 2. Annovar 
Directory = ~/OpenPedCan_Data/autoGVP/AutoGVP-main/prereqs/annovar 

Download ANNOVAR from website
Need to enter email on website and they will send download link.
tar -xvzf annovar.latest.tar.gz

Run Annovar:
```
perl table_annovar.pl data/test_VEP.vcf hg38 --buildver hg38 --out test_VEP --remove --protocol gnomad211_exome,gnomad211_genome --operation f,f --vcfinput
```




/mnt/isilon/dbhi_bfx/bin/R-3.1.3/bin/Rscript



No docker on HPC, use singularity to pull docker image — https://elearning.vib.be/courses/introduction-to-docker/lessons/run-and-execute-singularity-images/topic/using-docker-images-with-singularity/
singularity pull autogvp.sif docker://pgc-images.sbgenomics.com/diskin-lab/autogvp:v1.0.3

