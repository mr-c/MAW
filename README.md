[![License](https://img.shields.io/badge/License-MIT%202.0-blue.svg)](https://opensource.org/licenses/MIt)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-blue.svg)](https://GitHub.com/zmahnoor14/MAW/graphs/commit-activity)

# Metabolome Annotation Workflow
 - (tested on Linux and MAC OS)

This repository hosts Metabolome Annotation Workflow (MAW). The workflow has been developed using the LCMS-2 dataset from a marine diatom _Skeletonema marinoi_. The workflow takes .mzML format data files as an input in R and performs spectral database dereplication using R Package [Spectra](https://rformassspectrometry.github.io/Spectra/) and compound database dereplication using [SIRIUS](https://bio.informatik.uni-jena.de/software/sirius/) and [MetFrag](https://ipb-halle.github.io/MetFrag/projects/metfragcl/) (with KEGG and PubChem). The results are savd as .csv files and are post processed in Python using [RDKit](https://www.rdkit.org/) and [PubChemPy](https://pubchempy.readthedocs.io/en/latest/).The classification of the tentative candidates from the input data are classified using [CANOPUS]() and [ClassyFire](http://classyfire.wishartlab.com/), with a python client [pybatchclassyfire](https://gitlab.unige.ch/Pierre-Marie.Allard/pybatchclassyfire/-/tree/master) for ClassyFire.

## Install MAW with Docker container
Install Docker on your MAC OS with (https://www.docker.com/get-started/) and for Linux with (https://docs.docker.com/engine/install/ubuntu/). <br>
To create a docker image, following files are required:
1. Dockerfile
2. maw.yml
3. install_packages.R <br>
These files will create a docker image on your local system with the following command:
```shell
# build the image
docker build -t maw .
# run MAW in a jupyter notebook; change /workdir to your working directory
docker run -v /workdir:/workdir -i -t -p 8888:8888 maw /bin/bash -c "jupyter notebook --notebook-dir=/workdir --ip='*' --port=8888 --no-browser --allow-root"
```
If you want to run the workflow in a docker container, then use the following command
```shell
docker run -i -t maw /bin/bash
#either run R or python3 within the container shell
```
If you want to use the working directory on your system rather than the docker container, then follow this command:
```shell
#create a directory named within the docker container
mkdir /mnt
#quit the session and type this command in your system shell
#check if you docker directory is your pwd and then run:
docker run -v $(pwd):/mnt -i -t maw /bin/bash
#either run R or python3 within the container shell
```
You are ready to use the workflow on a docker container on your system

## Install MAW with Conda Environment
To create a conda environment, install conda from (https://www.anaconda.com/products/distribution). Once installed, type:
```shell
conda init # add to bashrc
conda 
```
To install the packages for MAW, using the maw.yml environment file which is created using conda, follow the command:
```
conda env create -f maw.yml
activate mawRpy # which is the name of the environment in the maw.yml file
```
Currently, MsBackendMsp requires R 4.2, but to avoid the need to install R 4.2 which is unavailable on conda currently), use Docker image. The following code is only possible if you have an installation folder for MsBackednMsp saved (as it was previously available to doenaload with 4.1.2). Follow the following instructions on the terminal to make the package available in your current R installation if you have a folder installed.
```shell
conda activate myenv
echo $CONDA_PREFIX
# you will receive a path, where you can keep the MsBackendMsp folder
```
There are some packages that are not available via any channel on conda OR there are errors downloading these packags with conda. To install such R packages, run the install_packages.R file within the mawRpy environment. For Python, install following packages using ```pip3 install``` command.
```
pip3 install rdkit-pypi pubchempy requests_cache pybatchclassyfire
```
  
### Install SIRIUS 
If the conda environment installation method is used, please install the latest SIRIUS version with <https://bio.informatik.uni-jena.de/software/sirius/> for Linux or MAC OS.
1. Installation with Linux
  ```shell
  echo $PATH # check which is already added to PATH variable
  sudo emacs /etc/environment
  PATH="usr/s_cost/sirius-gui/bin"
  source /etc/environment
  ```
2. Installation with MAC
  To use this command line, add the following path to either .bash_profile or .zprofile. You can find these files easily using FileZilla <https://filezilla-project.org/>. open your user name folder. Open your .bash_profile, add this to the $PATH variable.
  ```shell
  PATH="/usr/s_cost/sirius.app/Contents/MacOS/:${PATH}"
  export PATH
  ```
The path may be different, so check your Sirius installation folder to get the correct path name.

## Input files and Directories

An input directory (/input_dir) should have the following files.
1. All LCMS-2 spectra .mzML files
2. SIRIUS installation zip folder (only if you install via conda environment; no need to have this with docker image installation)
3. MetFrag jar file which can be downloaded from <https://github.com/ipb-halle/MetFragRelaunched/releases>.
4. MetFrag_AdductTypes.csv can be downloaded from <https://github.com/schymane/ReSOLUTION/blob/master/inst/extdata/MetFrag_AdductTypes.csv>
5. (optional) /QC folder containing all (MS1) QC .mzML files 
6. (optional) Suspect List in .csv format (important column - "SMILES")


## Functions to store online Spectral Databases to your R environment

## Functions to use a Suspect List(in-house library)
A suspect list of compounds can be used within the workflow to provide confidence to the predictions. This library is matched against results from Spectral and Compound Databases. MAW provides two functions to generate the input suspect list compounds for SIRIUS and MetFrag. The only important information in the SuspectList.csv file should be a column with SMILES. 
1. SIRIUS requires a folder with many .tpt files which contain fragmentation tree for each SMILES. To generate suspect list input for SIRIUS, use the function ```slist_sirius```. This function generates a result folder /input_dir/SL_Frag.
```
hello
```
2. MetFrag requires a txt file with InChIKeys. This can be obtained with the function ```sl_metfrag```. 
```
hello
```
### Tutorial of Workflow

1. Load Dependencies:
```R
library(Spectra)
library(MsBackendMgf)
library(MsBackendHmdb)
library(MsCoreUtils)
library(MsBackendMsp)
library(readr)
library(dplyr)
library(rvest)
library(stringr)
library(xml2)
```

2. Define the input directory. Make sure that you have input LCMS-2 spectra files in .mzML format in the input directory.
```R
input_dir <- "usr/s_cost/"
```
             
3. Load the open spectral libraries. This function will take a lot of computational resources. However, to skip this function, you can download the current versions of these databases from <write the link here and ask where can you provide datasets>. The databases are stored in the same input directory and in .rda format, as an R object.
```R
# db argument can take "all", "gnps", "mbank" and "hmdb". Default (and recommended) is "all".
download_specDB(input_dir, db = "all")
```

4. Load the database .rda objects to the current R session.
```R
load(file = paste(input_dir,"gnps.rda", sep = ""))
load(file = paste(input_dir,"hmdb.rda", sep = ""))
load(file = paste(input_dir,"mbank.rda", sep = ""))
```

5. Create a table that lists all the input .mzML files and the result directories with the same name as the input file. The function ```ms2_rfilename``` gives an id to the file as well. 
```R
input_table <- data.frame(ms2_rfilename(input_dir))
```

6. (optional) If you are using QC files, follow these steps as well. QC samples are used to normalize the signals across all samples. So, generally, QC files contain all the signals from MS1 files with higher m/z resolution. These files also contain the isotopic peaks and can be used for formula identification in SIRIUS. 
    1. If you have files with positive and negative modes in one file, follow the first section of the code. It takes all files in the QC folder with a certain pattern (choose a pattern from your file names, could be even .mzML as this is the format of all files in the QC folder) and divides the pos and neg modes from each file and generate different mode .mzML files (01pos.mzmL and 01neg.mzML). These files are read by "CAMERA", which is loaded within the function. the outputs are several CSV files with CAMERA results, which are merged into one CSV file for each model.
    2. If you have files with positive and negative modes in separate files, follow the second section of code. It takes one file in the QC folder with either pos or neg mode. These files are read by "CAMERA", which is loaded within the function.          
```R
# first section
cam_funcMode(path = paste(input_dir, "QC", sep =""), pattern = "common")
merge_qc(path = paste(input_dir, "QC", sep =""))
```

```R
# second section
cam_func(path = "QC/", f = "01pos.mzML", mode = "pos")
cam_func(path = "QC/", f = "02neg.mzML", mode = "neg")
```

7. (optional, follow if followed 6.) add the QC files to your input files. This can be done simply by adding one more column to the input_table and adding the pos files against pos mode LCMS2 data and neg files against neg mode LCMS2 files. An example is given below:             
```R
for (i in 1:nrow(input_table)){
    if (grepl("pos", input_table[i, "mzml_files"], fixed=TRUE)){
        input_table[i, "qcCAM_csv"] <- "./QC/Combined_Camera_pos.csv"
    }
    if (grepl("neg", input_table[i, "mzml_files"], fixed=TRUE)){
        input_table[i, "qcCAM_csv"] <- "./QC/Combined_Camera_neg.csv"
    }
}
```
  
8. After the previous steps, we have all the inputs, their directories and optionally the QC csv files. Next is to initiate the workflow, (assuming we want to process only one LCMS2 .mzML file). Use the spec_Processing function to read and pre-process the MS2 spectra. The output is processed spectra and a list of precursor m/z(s) present in the .mzML file. Give the file path as an argument.                                     
```R
spec_pr <- spec_Processing(as.character(input_table[1, "mzml_files"]))
# Extract spectra
sps_all <- spec_pr[[1]]
# Extract precursor m/z
pre_mz<- spec_pr[[2]]
```
  
9. Using the above-processed spectra, the workflow is ready to perform spectral database deprelication. In this workflow, "GNPS", "HMDB" and "MassBank" or all of these databases can be used. This function can be performed for all precursor m/z(s) in a loop. The function ```spec_dereplication``` takes one precursor m/z at a time, the result directory path, either from the input_table or as a whole written path e.g: "usr/s_cost/file_pos_01". The file_id is also taken from the input_table, as was generated by the ms2_rfilename function, but is customizable as any other id given as a string e.g: "IDfile_pos_01". ppmx is the ppm value used by the function to match the two spectra which have their m/z values at most 15 ppm apart to be considered a match. This results in a directory called spectral_dereplication and contains results in CSV files for each database.
```R
for (i in pre_mz){
    df_derep <- spec_dereplication(i, db = "all", result_dir = input_table[1, "ResultFileNames"],
                file_id = input_table[1, "File_id"], input_dir, ppmx = 15)
}
```
                
10. In order to perform dereplication using compound databases, the Workflow uses SIRIUS and MetFrag. For these tools, we need MS2 fragmentation peak lists. The next function ```ms2_peaks``` extracts MS2 fragmentation spectra and stores the peaks for each precursor m/z in "/insilico/peakfiles_ms2/Peaks_01.txt". It takes the processed spectra from ```spec_Processing``` and results in a directory path.              
```R
spec_pr2 <- ms2_peaks(spec_pr, input_table[i, "ResultFileNames"])
```
  
11. For Formula identification in SIRIUS, a fragmentation tree generates a tree score and isotopic peaks generate an isotopic score, both of which form the Sirius score. In order to utilize the isotopic score, isotopic peak annotation is required. For this purpose, the CAMERA results generated using CAMERA, are utilized here to extract the isotopic peaks for each precursor m/z and store them in "/insilico/peakfiles_ms2/Peaks_01.txt". The function ```ms1_peaks``` takes results from ms2_peaks, the associated QC CSV file, the result directory and whether the QC file is being used or not. If there is not QC CSV file, enter NA and QC = FALSE.
  
```R
ms1p <- ms1_peaks(spec_pr2, input_table[i, "qcCAM_csv"], input_table[i, "ResultFileNames"], QC = TRUE)
```
        
        
11. To create SIRIUS input files, use ```sirius_param``` function, which takes the output table from ```ms1_peaks``` and the result directory. If a suspect list is used, then SL = TRUE. 
        
```R
sirius_param_files <- sirius_param(ms1p, result_dir = input_table[1, 'ResultFileNames'], SL = TRUE)
```
  
  
12. Run SIRIUS. Give the output of sirius_param as an argument to this function, which contains a table of all the input and output paths for SIRIUS. Keep QC = TRUE, if QC files were used. If a suspect list is used, then SL = TRUE. Give path to the fragmentation tree directory. The suspect list fragmentation trees are calculated with a python defined function ```slist_SIRIUS``` and are stored in a directory named ScostSLS/ within the input directory. Candidates are the number of molecular formulas considered for further SIRIUS calculations. If SL = FALSE, keep the SL_path = NA
```R
run_sirius(files=sirius_param_files, ppm_max = 5, ppm_max_ms2 = 15, QC = TRUE, SL = TRUE, 
  SL_path = paste(input_dir, 'ScostSLS/', sep = ""), candidates = 30)
```
  
12. To run MetFrag, SIRIUS results are preprocessed to extract the best score Molecular formula and its adduct. The Adduct annotation is important to run Metfrag. In this function, give the result directory and whether a suspect list was used or not.
```R
sirius_pproc <- sirius_postprocess(input_table[i, "ResultFileNames"], SL = TRUE)
```
                
13. Generate MetFrag parameter files. Give the Sirius processed results as input. add the path of the MetFrag_AdductTypes.csv. Add the path of the suspect list InChIKeys generated by python function ```slist_metfrag``` for Metfrag.
```R
met_param <- metfrag_param(sirius_pproc, result_dir = input_table[1, "ResultFileNames"],input_dir, 
  adducts = paste(input_dir, "MetFrag_AdductTypes.csv", sep = ""), 
  sl_mtfrag = paste(input_dir, "sl_metfrag.txt", sep = ""), SL = TRUE)
```
  
14. Run MetFrag.
```R
for (files in met_param){
    system(paste("java -jar",  paste(input_dir, "MetFrag2.4.5-CL.jar", sep = ''), files))
}
```
## More information about our research group

[![GitHub Logo](https://github.com/Kohulan/DECIMER-Image-to-SMILES/blob/master/assets/CheminfGit.png?raw=true)](https://cheminf.uni-jena.de)

