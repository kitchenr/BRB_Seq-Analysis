# BRB_Seq-Analysis
Analysis for Spike in and Full Run data from BRB-Seq on the WashU HTCF  

## Spike In
### Overview of Spike In Data
**Goal of Spike-In run:** Assess data quality and determine appropriate read depth for the full sequencing run.  

**Input:** BRB-Seq FASTQs (R1, R2) and the Alithea BRB-Seq Excel metadata template.  

**Output:** MultiQC HTML report used to calculate repooling/read numbers for the full run.  

## Project Directory Structure
Create a new project directory with the following structure:  

PROJECT_NAME/  
├── data/  
│ ├── R1  
│ ├── R2  
│ ├── I1  
│ ├── I2  
├── metadata/  
│ ├── Mapping_File.txt (to be made and uploaded)  
│ └── Parameters_mouse.txt (or human paramaters file)  
├── scripts/  
│ ├── BRB_Seq_Muegge_241112_TwoStage_noMulti.sh  
│ └── BRB_MultiQC_SpikeIn_240116.sh  
└── spike_in_analysis/  

## Step 1: Create the Mapping File
After Spike In you get a file named:   {Project_Name}_Alithea_BRBSEQ_Pipeline_V5B_Primerset_Template.xlsx  
The Excel sheet contains your barcodes, well locations and sample names.  
```
a. Open the Excel template.  
b. Navigate to the Mapping_File tab.  
c. Populate it by copying the following columns from the ReverseTranscription tab:  
        i. SampleName  
        ii. Barcode  
d. The Group column is optional at this stage. If known, add real group information; otherwise, use placeholders.  
e. Save the mapping file as a tab-delimited text file and upload to your metadata/ folder on the HTCF.  
        i. An example mapping file is posted here.  
f. Remove End-of-Line Characters (\r or \n)  
        i. sed 's/\r$//' input.txt > output.txt  
g. In the metadata folder you should have your mapping file and the parameters file.
```

## Step 2: Spike-In Processing Pipeline: BRB_Seq_Muegge_241112_TwoStage_noMulti.sh  
In your Scripts folder you should have: BRB_MultiQC_SpikeIn_240116.sh and BRB_Seq_Muegge_241112_TwoStage_noMulti.sh.  

We will use BRB_Seq_Muegge_241112_TwoStage_noMulti.sh first.   
Run Type: Spike-in   Output: QC report  

### What this Script Does
For each sample in the mapping file this script: 
```
1. Demultiplexes with cutadapt  
2. Performs pre-alignment QC with FastQC  
3. Removes sequencing adapters and polyA tails with cutadapt  
4. Aligns reads to the reference genome using STAR  
5. Performs post-alignment QC using:  
        a. FastQC  
        b. RSeQC
        c. Qualimap  
6. Assigns features using Subread FeatureCounts
```
### Input for BRB_Seq_Muegge_241112_TwoStage_noMulti.sh  
```
1. Sample <> Barcode mapping file with paths to Read1 and Read2 fastq files  
        a. This is the metadata file you previously generated and uploaded  
2. Parameters file with paths to genome annotation files and STAR  
        a. This is the Parameters_mouse.txt (or human paramaters file)
```
### Usage of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh
**Usage Template**  
sbatch BRB_Seq_Muegge_241112_TwoStage_noMulti.sh PROJECT_DIRECTORY MAPPING_FILE PARAMETERS_FILE  
  
**Usage Examples**   
sbatch --array=1-8%8 scripts/BRB_Seq_Muegge_241112_TwoStage_noMulti.sh spike_in_analysis/ metadata/spikein_map.txt metadata/parameters.csv  
  
sbatch --array=1-48%20 scripts/SpikeIn/BRB_Seq_Muegge_241112_TwoStage_noMulti.sh spike_in_analysis_test/ metadata/Test.txt metadata/parameters_mouse.txt  
  
**Notes**  
1. Change arrary 1-8 to number of your samples.   
2. %8 controls how many runs occur at the same time.  

## Step 3: Spike-In Processing Pipeline: BRB_MultiQC_SpikeIn_240116.sh
We will now use BRB_MultiQC_SpikeIn_240116.sh.   
Run Type: Spike-in   Output: QC report   
### What this Script Does
This script contains wrapper scripts which:  
```
1. Aggregates feature count outputs across all samples from the output of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh into a single matrix 
2. Generates a consolidated MultiQC HTML report
```

### Input for BRB_MultiQC_SpikeIn_240116.sh  
Output of of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh

### Usage of BRB_MultiQC_SpikeIn_240116.sh
**Usage Template**  
sbatch BRB_MultiQC_SpikeIn_240116.sh PROJECT_DIR LIBRARY_NAME  
  
**Usage Example**  
sbatch scripts/SpikeIn/BRB_MultiQC_SpikeIn_240116.sh spike_in_analysis/ Test  

## Step 4: Determine Read Depth for Full Run
Step 3 will result in a MultiQC html where you can look at data quality.  
```
1. Open the generated MultiQC HTML report
2. Copy the table into Evenness_Repooling_Template_2025_1017.xlsx
3. Use this spreadsheet to calculate recommended read depth
4. Send these values to Brian to discuss need for repooling
```

## Full Run
### Overview of Full Run Data
**Basics of BRB-Seq Full run:** Get low-depth, but fast sequencing that provides an accurate snapshot of the highly expressed RNA in a dataset.  

**Input:** BRB-Seq FASTQs (R1, R2) from GTAC and the Alithea BRB-Seq Excel metadata template.  

**Output:** Raw and De-duplicated counts tables.  

## Project Directory Structure
Create a new project directory/same as Spike-in with the following structure:  

PROJECT_NAME/  
├── data/  
│ ├── R1  
│ ├── R2    
├── metadata/  
│ ├── Mapping_File.txt (to be made and uploaded) 
│ ├── Abbreviated_Mapping_File.txt (columns 1 and 3 of Mapping_File.txt)  
│ └── Parameters_mouse.txt (or human paramaters file)  
├── scripts/  
│ ├── BRB_Seq_Muegge_241112_TwoStage_noMulti.sh  
│ ├── BRB_MultiQC_FullRun_240116.sh    
│ └── Multiplex_BRB_Align_Counts_240125.sh      
└── full_run_analysis/  

## Step 1: Create the Mapping File
After the Full Run you get a file named:   {Project_Name}_Alithea_BRBSEQ_Pipeline_V5B_Primerset_Template.xlsx  
The Excel sheet contains your barcodes, well locations and sample names.  
```
a. Open the Excel template.  
b. Navigate to the Mapping_File tab.  
c. Populate it by copying the following columns from the ReverseTranscription tab:  
        i. SampleName  
        ii. Barcode  
d. The Group column is optional at this stage. If known, add real group information; otherwise, use placeholders.  
e. Save the mapping file as a tab-delimited text file and upload to your metadata/ folder on the HTCF.  
        i. An example mapping file is posted here.  
f. Remove End-of-Line Characters (\r or \n)  
        i. sed 's/\r$//' input.txt > output.txt  
g. In the metadata folder you should have your mapping file and the parameters file.
```
## Step 2: Full Run Processing Pipeline: BRB_Seq_Muegge_241112_TwoStage_noMulti.sh   
We will use BRB_Seq_Muegge_241112_TwoStage_noMulti.sh first.     

### What this Script Does
For each sample in the mapping file this script: 
```
1. Demultiplexes with cutadapt  
2. Performs pre-alignment QC with FastQC  
3. Removes sequencing adapters and polyA tails with cutadapt  
4. Aligns reads to the reference genome using STAR  
5. Performs post-alignment QC using:  
        a. FastQC  
        b. RSeQC
        c. Qualimap  
6. Assigns features using Subread FeatureCounts
```
### Input for BRB_Seq_Muegge_241112_TwoStage_noMulti.sh  
```
1. Sample <> Barcode mapping file with paths to Read1 and Read2 fastq files  
        a. This is the metadata file you previously generated and uploaded  
2. Parameters file with paths to genome annotation files and STAR  
        a. This is the Parameters_mouse.txt (or human paramaters file)
```
### Usage of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh
**Usage Template**  
sbatch BRB_Seq_Muegge_241112_TwoStage_noMulti.sh PROJECT_DIRECTORY MAPPING_FILE PARAMETERS_FILE  
  
**Usage Examples**   
sbatch --array=1-8%8 scripts/BRB_Seq_Muegge_241112_TwoStage_noMulti.sh full_run_analysis/ metadata/Mapping_File.txt metadata/parameters_mouse.txt    
  
sbatch --array=1-48%20 scripts/FullRun/BRB_Seq_Muegge_241112_TwoStage_noMulti.sh full_run_analysis/ metadata/Mapping_File.txt metadata/parameters_mouse.txt  
  
**Notes**  
1. Change arrary 1-8 to number of your samples.   
2. %8 controls how many runs occur at the same time.
 
## Step 3: Full Run Processing Pipeline: BRB_MultiQC_FullRun_240116.sh
We will now use BRB_MultiQC_FullRun_240116.sh.    
### What this Script Does
This script contains wrapper scripts which:  
```
1. Aggregates feature count outputs across all samples from the output of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh into a single matrix 
2. Generates a consolidated MultiQC HTML report to check quality and ensure successful demultiplexing and pre-alignment
```

### Input for BRB_MultiQC_FullRun_240116.sh  
Output of of BRB_Seq_Muegge_241112_TwoStage_noMulti.sh

### Usage of BRB_MultiQC_SpikeIn_240116.sh
**Usage Template**  
sbatch BRB_MultiQC_FullRun_240116.sh PROJECT_DIR LIBRARY_NAME  
  
**Usage Example**  
sbatch scripts/BRB_MultiQC_FullRun_240116.sh full_run_analysis/ Test  

## Step 4: Full Run Processing Pipeline: Multiplex_BRB_Align_Counts_240125.sh

# Useful Commands on HTCF

## Checking Job Status
squeue -u yourWUSTLkey

