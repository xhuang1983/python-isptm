python-isptm
============

Python scripts for ISPTM (iterative search for post-translantional modification) implementation

December, 2012

ISPTM: an Iterative Search Algorithm for Systematic Identification of 
Post-translational Modifications from Complex Proteome Mixtures

Xin Huang, Lin Huang, Shi-Jian Ding
University of Nebraska Medical Center, Omaha, NE


Instruction for the usage of python scripts


License:
      All scripts are written in Python programming language. The scripts currently use License GNU GPL v2 (1991). This scripts also contain software from a number of other libraries and utilities from the GNU project

System requirement:
      The Python platform is required to be installed on computers (Linux/Windows/Mac OS). Python can be freely downloaded from www.python.org. Current scripts have been tested in Python 2.5 or higher. All scripts are based on command lines. Please refer to www.python.org for more information about running a command line python program. 

General instructions:
      The files with extensions of .py are the scripts of Python. Users can open these file using a text editor (i.e.Notepad) and change it. To run a python file (pythonfile.py) in command line system, users have to navigate the current fold containing the python file, and type “python pythonfile.py”, and enter.
      The ISPTM scripts have two versions. The python files in folder “ISPTM” are used for the stand alone desktop computers (Windows/Linux/Mac OS). The files in subfolder “pbs_cluster_version” are used for computer clusters with parallel computing system Portable Batch System (PBS).

Parameter file: parameters.txt:
      A file named “parameters.txt” must be in the folder with python scripts, setting in the parameter file:

Parameters for ISPTM pipeline:

Fasta=<String>
      The file name of the “.fasta” file for database search, this is a required field. The “.fasta” file must be stored in the same folder with the scripts.

DI=<1|2>
      Depth of iterative search, this is a required filed, and can only be 1 or 2. Allowing one variable modification in each search is fast, wherase two is relative slow. 

REF=<0|1>
     Specify if DtaRefinery is needed in step1 of ISPTM. DtaRefinery calibrate the mass error of precursor ions in high-accuracy mass spectrometer. The default value is No (0).

MAX=<Integer>
      The maximum id of modification in OMSSA mods.xml file. In the version of 2.1.9, the MAX value is 207.

IE=<String>
      The list of either included (I) or excluded (E) variable modifications for iterative searches. The list begins with "I" or "E", followed by the ids of mods (in mods.xml), separated by "," with no space (i.e. IE=I0,1,2). Fixed modifications cannot be in the list. ISPTM will test all variable modifications (0 ~ MAX) if this option is empty.

TE=<Real>
      Tolerance of the precursor ions, it is used in OMSSA command option "-te", the default value is 0.02 (Da)

TO=<Real>
      Tolerance of the product ions, it is used in OMSSA command option "-to", the default value is 0.5 (Da)

V=<Integer>
      Number of missed cleavages allowed, it is used in OMSSA command option "-v", the default value is 2

II=<Real>
      E-value threshold to iteratively search a spectrum again, it is used in OMSSA command option “-ii”. In the basic search, a spectrum with an E-value below this threshold is removed from iterative searches. The default value is 0.01

MF=<Integer>
      The fixed modifications in all searches, it is used in OMSSA command option “-mf”. Ids of multiple mods must be separate by "," and with no space. 

Parameters for ISPTM on desktop version:

OM=<String>
      The name of the folder of OMSSA search engine. The default one is “omssa-2.1.9.win32”

P=<Integer>
      Number of processors used for iterative searches. The default value is 2

Parameters for ISPTM on PBS cluster version:

RPJ=<Integer>
      The OMSSA runs per job (RPJ). Total Runs = Jobs * RPJ. The default value is 500, which means every 500 individual runs are involved in each job

MEM=<Integer>
      Specify the memory (MB) allocated for each run, the default value is 8000 (MB).

MT=<Integer>
      Specify the maximum of run time (hours) of a job, the default value is 20 (hours).

Parameters for Spectrum annotation and site confidence score calculation:

RM=<Integer> and RI=<Integer>
      The maximum (RM) and minimum (RI) m/z for the annotation range of a spectrum, the default value of RM is 1500, and RI is 300. Only the product ions in this range of m/z are used for SC score calculation.

SP=<Real>
      The time interval for submitting annotation requests to MS-Product. It relies on the internet capacity. Increase it if internet speed is low. The default value is 0.2

NPW=<integer>
      The number of selected product ions in each 100 m/z window, the default value is 6. 

FIE=<String>
      The mods list for filtering based on site confidence score. A list starts with "I" indicates filtering will only apply for following mods, while a list starts with "E" indicates filtering will not apply for following mods. "I" or "E" followed by the Ids of mods, separated by "," with no space (i.e. FIE= I6,7,8 means SC score cutoff will apply to phosphorylation on S,Y,T). Ids in FIE must be a subset of the Ids in parameter “IE”. Emptying this field indicates cutoff will apply to all mods in list of IE. 

CC=<Real>
      Site confidence score cutoff for filtering, the default value is 0.8.


Instructions to run ISPTM scripts    

1. Convert raw data to mgf.
      Move .RAW files and step1_ms_spectra_refining.py in one folder.
       python step1_ms_spectra_refining.py

      We provide a way to converts MS raw data (Thermo .RAW) to MS/MS spectra list files (.mgf). It requires additional programs (DeconMSn, DtaRefinery, and DTAtoMGFConverter) from PNNL site (http://omics.pnl.gov/software/) and .Net framework 1.1 pre-installed. 
      It converts every “.RAW” file to an “.mgf” file, with same file name. Users can remove all the intermediate files. For more details, please refer to the instructions of those programs. This step only runs in Windows.

2. Download OMSSA and generate sequence database.
      Move the “.mgf” files, the “.fasta” file, and the “.py” scripts in one folder (i.e. ISPTM). 
      Download the OMSSA engine form ftp://ftp.ncbi.nih.gov/pub/lewisg/omssa/2.1.9/, choose the right platform (i.e. omssa-2.1.9.win32) and release them under the folder of “ISPTM”. 
      Download the NCBI blast tools from ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.2.24/, choose the right platform (i.e. blast-2.2.24+) and release them under the folder of “ISPTM”.
      Run makeblastd: change the name of .fasta file in “makeblastdb.cmd”, and double click it.  
      
      Makeblastdb creates the sequences files .phr, .pin, and .ps1 accompanying the .fasta file.

3. ISPTM procedures.
python step2_basic_search_wo_mods.py
      It performs the basic OMSSA search for an .mgf file without variable modifications, and output an OMSSA “.csv” file. 

python step3a_remove_unmodified_spectra.py
       It refines the original “.mgf” file, removes the spectra of identified peptides in the basic search. For each fraction, it creates a new spectra file with extension of “.filtered.mgf”, and a new identification file with extension of “.unmodified.csv”.

python step3b_iterative_searches.py
      It performs iterative searches for each “.filtered.mgf” file. It creates a sub-folder for each fraction, and all the OMSSA “.csv” outputs by iterative searches are store in the sub-folders.

python step4_data_filtering_output.py
      For each fraction, it combine all the iterative search results, filter out the low-score and redundant identifications. It creates an “.isptm.csv” file, having the identifications of modified peptides by ISPTM approach. 

      ISPTM procedures generate lots of intermediate files, only “.unmodified.csv” (unmodified peptides) and “.isptm.csv” (modified peptides) are useful results. All other intermediate files can be removed.

4. Site confidence (SC) calculation.
Input files of the scripts for SC score calculation are the original “.mgf” files, “.isptm.csv” files, and the OMSSA mods file (mods.xml). Users need to put these files and python scripts in one folder, and run the scripts.

python sc1_ms_product_submitter.py
      It reads the identification results and submits them of MS-Product site. It creates the intermediate fragment “.frag.txt” and annotation “.anno.txt” files, and a folder “rawHTMLs” which contains the webpages from MS-product (http://prospector.ucsf.edu/prospector/cgi-bin/msform.cgi?form=msproduct) site. 
 
python sc2_score_calculator.py
      It calculates the SC score of identifications. This script reads the ISPTM output “.isptm.csv” files, and generates scored “.score.csv” files. It added a new column “site_confidence”, in which the SC score of each modified site were annotated.

python sc3_score_filter.py
      It filters the scored “.score.csv” files by a user-defined cutoff. The output file has an extension of “.filtered.csv”.
 
5. Run ISPTM on parallel computing system.

      ISPTM scripts are modified in order to run on the parallel computing system with a “job-scheduler”. Users need to upload the “.mgf” files, the “.fasta” file, “parameters.txt”, and the python scripts in “pbs_cluster_version” to the super-power computer. All files must be in the same path. The scripts have automatic procedures of downloading OMSSA and NCBI blast tools, and performing makebastdb (for UNIX/Linux only). Some steps generate bash (.sh) files for submitting jobs. Users need to run them after the python scripts.

Step2:
python step2_basic_search_wo_mods.py
sh submit_unmodified.sh

Step3:
python step3a_remove_unmodified_spectra.py
python step3b_iterative_searches.py
sh submit_isptm.sh

Step4:
python step4_data_filtering_output.py
sh submit_filtering.sh

5. Other useful python scripts

python final_cleaning_steps.py 
      It removes all useless intermediate files in the path of parallel system (for UNIX/Linux only)

python kill_all_jobs.py –au username
      It kills all the jobs submitted by the username. 

python csv_file_merger.py filename.csv
      Initial outputs of ISPTM are stored by fractions. Put this script and the .csv files of all fractions in one folder, this script merges them into one .csv file (filename.csv).

python csv_file_splitter.py filename.csv
      Opposite to the merger procedure, this script reads a .csv file (filename.csv) and split it by fraction names. 

python remove_fixed_in_mgf.py
      DtaRefinery appends a “_FIXED” in each .mgf file. Put this script and the .mgf files of all fractions in one folder, this script removes “_FIXED”.

Contacts:
      For general questions or collaborations, please contact: Shi-Jian Ding (dings@unmc.edu).
      For technique questions, feedbacks, and bugs, please contact: Xin Huang (xinhuang.zju@gmail.com).

Citation:
      Huang X, Huang L, Peng P, Guru A, Xue W, Hong SY, Liu M, Sharma S, Fu K, Swanson D, Zhang Z, and Ding S-J, ISPTM: an iterative search algorithm for systematic identification of post-translational modifications from complex proteome mixtures, Journal of Proteome Research (Submitted).
