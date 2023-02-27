# A generalized toolbox for QPP detection, analyzsis, and visulization for rodent or human brains
This is a generally applicable MATLAB toolbox, which detects, analyzes, and visualizes up to 5 QPPs and QPP regressed functional connectivity maps from fMRI timeseries of the brain across rodents and humans. This toolbox is a significant modification and extension of [QPP_Scripts_v0620](https://github.com/GT-EmoryMINDlab/QPP_Scripts_v0620), and can analyze QPPs for resting brain and for task-evoked/stimulated brains across species.

# Table of Contents
* 1 - [Prerequisite](#section-1)
    * 1.1 [Software dependencies](#section-1-1)    
    * 1.2 [Input File (./input/)](#section-1-2)
      * 1.2.1 -[Input variables in `data`.mat](#section-1-2-1)
      * 1.2.2 -[(optional step) Run 'st0_ROIreOrg.m' for variable generations](#section-1-2-2)        
* 2 - [Main Pipeline](#section-2)
    * 2.1 [(Step 1) Run 'st1_ParamsSet.m'](#section-2-1)
    * 2.2 [(Step 2) Run 'st2_QPPanalysis.m'](#section-2-2)
        * 2.2.1 -[Prespecified parameters](#section-2-2-1)
        * 2.2.2 -[Automated QPP analysis & Output variables](#section-2-2-2)
    * 2.3 [(Step 3) Run 'st3_QPPFCvisual.m'](#section-2-3)
        * 2.3.1 -[Prespecified parameters](#section-2-3-1)
        * 2.3.2 -[Generated figures](#section-2-3-2)
* 3 - [Output Files](#section-4)
* 4 - [References](#section-4)


<a name="section-1"></a>
## 1. Prerequisite 
<a name="section-1-1"></a>
### 1.1 Software dependencies
This is a [Matlab](https://www.mathworks.com/) (The Mathworks Inc., Natick, MA, USA, R2018a or a later version) toolbox. Any operational system with Matlab (R2018) installed will be sufficient to run this toolbox. All supporting Matlab functions to be called in the main pipeline are included in ./QPPv0922/ folder.

### 1.2 Input File (./input/`data`.mat)
The input file `data`.mat is included in ./input/ folder whereas `data` is the input filename. Five distinct input samples are enclosed in the current ./input/, which includes 1) a visual stimuated human brain dataset, 2) a Human Connectome Project resting human brain dataset, 3) a resting rat brain dataset, and 4) a resting mice brain dataset.
#### 1.2.1 Input variables    
The input `data`.mat must include the following four variables.
| Variable name | Description | Note   | 
|--------|--------|-------------|
|`D0`   | a (nsbj X nscn) cell matrix |Each cell has a nroi X ntimepoints matrix of EPI timeseries. |
|`MotionInf`| a (nsbj X nscn) cell matrix | Each cell includes >=1 segment(s) of timepoints without significant motions.|
|`ROI2Net`| a (nroi X 1) vector| Each entry is the network index corresponding to each ROI.|
|`NetLB`| a (nnet X 1) cell vector| Each cell includes the (shorthand) label of each network.|

Note: nsbj= number of subjects, nscn=number of EPI scans, nroi=number of ROIs, ntimepoints=number of timepoints, nnet= number of networks.
#### 1.2.2 (optional step) Run 'st0_ROIreOrg.m' for variable generations
The `ROI2Net` and `NetLB` variables can be generated by running 'st0_ROIreOrg.m'. In this case, the EPI data cell matrix should at least be included in your original input file; if you did not prepare the `MotionInf` parameter, the entire timepoints of each scan will be included in `MotionInf` when running 'st0_ROIreOrg.m'. 

In addition, you will also need to prepare a atlas-network file. This file should be a spreadsheet including at least two columns, one column including the numerical label of ROIs/Parcels corresponing to your atlas, and the other column including the network name each ROI belonging to. In ./resources/ folder, 5 atlas-network files are included: 1) the Schaefer-Yeo 400 parcels to 7 Yeo network, 2) Glasser 360 parcels to 7 Yeo network, 3) Brainnetome 360 parcels to 7 Yeo network, 4) SIGMA_Wistar rat brain atlas, and 5) a modified Allen Brain Institute Mouse atlas.

<a name="section-2"></a>
## 2. Main Pipeline
The main pipeline consists 3 steps. The 1st step (st1_ParamSt.m) sets up the intial parameters. The 2nd step (st2_QPPanalysis.m) detects and analyzes QPPs. The 3rd step (st3_QPPFCvisual.m) visualizes QPP and related results given the outputs of step 2. 
<a name="section-2-1"></a>
### 2.1 (Step 1) Run 'st1_ParamsSet.m'
Following variables will be predefined, and a parameter file Params_`data`\_`ext`.mat will be generated after running this script.
|      Purpose     |  Variable name | Description | Note   | 
|------------------|-----------------|--------|-------------|
|  Filepath  		|`data`   | the input filename |The input should has the filename `data`.mat |
|                  	|`ext`    | filename extension for the parameter file| The parameter filename will be  Params_`data`\_`ext`.mat |
|  QPP global parameters|`nP`     | total # of QPPs to detect (nP<=5)| If nP=1, only detect the primary QPP (QPP1); if nP=2, detect both QPP1 & QPP2; etc.|
|		   	|`PL`     | a (nP X 1) vector of QPP window length | ~20s for humans (e.g., PL(ip)=20/TR), |
|  QPP detection	|`cth13` & `cth34`     | a 2D vector of correlation threshold for QPP1-QPP3 (`cth13`) & for QPP4-QPP5 (`cth45`)| If you do not need to detect QPP4-QPP5, please assign `cth34` a random number (e.g., `cth34`=[0, 0]).|
|  QPP phase adjustment	|`cthph` | similarity threshold when phase-adjusting (phadj) a QPP |Default value: cthph=0.88|
|		   	|`s`     | control for strict phase adjustment (`s`=1) or relaxed phase adjustment (`s`=0)||
|		   	|`sdph`     | a (nP X 1) cell array of reference parcels| Each cell may include >=1 parcel IDs. The phase adjusted QPP waveform will start from rising positive values for the selected parcels.|
|  Functional connectivity (FC) analysis|`fz` | control for the output matrix `FCr` to be the pearson correlation (`fz`=1) or to be the Fisher Z-Transformation of the pearson correlaion (`fz`=1).|

<a name="section-2-2"></a>
### 2.2 (Step 2) Run 'st2_QPPanalysis.m'
<a name="section-2-1-1"></a>
#### 2.2.1 Prespecified parameters       
The following three parameters need to be prespecified at the beginning of this script.
|      Purpose     |  Variable name  | Description | Note   | 
|------------------|-----------------|-------------|--------|
|  Filepath  		|`dataext`   | parameter filename |The parameter .mat file generated from step 1, which has the filename Param_`dataext`.mat |
|  Data concatenation method |`runM`     | control the way to concatenate the data| If `runM`=1, concatenate all D{i,j} as a whole group and detect group QPP; if `runM`=2, concatenate all D{i,:} and detect QPP from all scans of each subject; if `runM`=3, concatenate all D{:,:} and detect QPP from all subjects of each scan.|
| QPP detection	method|`rbstScrn`     | control for fast QPP dectection (`rbstScrn`=0) or robust QPP detection (`rbstScrn`=1)|The fast QPP detection selectes a limited number of starting points which was used in XXXX, whereas the robust detection selects all possible starting points which was used in (XXX).|

<a name="section-2-1-1"></a>
#### 2.2.2 Automated QPP analysis & Output variables
The analytical procedures to be executed and corresponding outputs to be generated are listed below.

<a name="section-2-3"></a>
### 2.3 (Step 3) Run 'st3_QPPFCvisual.m'
<a name="section-2-3-1"></a>
#### 2.3.1 Prespecified parameters       
In addition to the 3 parameters described in [Section 2.2.1](#section-2-2-1), three additional parameters can be prespecified at the beginning of this script.
|     Variable name  | Description | Purpose   | 
|-----------------|-------------|--------|
|`Pselect`   | a vector of integers | to specify the QPP #s to be visualized |
| `Gselect`   | a vector of integers | to specify the group# (e.g. which scans/subjects) to be compared; if `runM`=1, by default `Gselect`=1. |
| `bin`| a decimal | to specify bin size of the histogram of sliding correlation timecouses|

<a name="section-2-3-2"></a>
#### 2.3.2 Generated figures
| Index  | Figure Description |
|--------|--------------------|
| 1 | specified QPPs of each group|
| 2 | phase reversed QPPs of each group |
| 3 | sliding correlation timecourses with maxima and minima labeled |
| 4 | sliding correlation timecourses before and after QPP regression|
| 5 | histograms of sliding correlation timecourses before and after QPP regression|
| 6 | functional connectivity (FC) matrix before (upper triangule) and after (lower triangule) QPP regression |

In figures 1, 2, and 6, ROIs are reorganized based on networks.

<a name="section-3"></a>
## 3. Output Files(./Output/)
Examples of output files are listed.

      ./Output    
         ├── GrpQPP                                   # Concatenated all EPI scans of all subjects as a whole group (`runM`=1) 
         │   ├── interm                               # Intermediate output files
         │   │      ├── `dataext`_Grp1_rbst1_qpp1.mat # QPP1 detected with robust detection 
         |   │      ├── `dataext`_Grp1_rbst1_qpp2.mat # QPP2 detected with robust detection 
         |   │      ├── `dataext`_Grp1_rbst1_qpp3.mat # QPP3 detected with robust detection 
         |   │      ├── `dataext`_Grp1_rbst0_qpp1.mat # QPP1 detected with fast detection 
         |   │      ├── `dataext`_Grp1_rbst0_qpp2.mat # QPP2 detected with fast detection 
         |   │      ├── `dataext`_Grp1_rbst0_qpp3.mat # QPP3 detected with fast detection 
         |   │      ├── `dataext`_Grp1_rbst0_qpp4.mat # QPP4 detected with fast detection 
         |   │      ├── `dataext`_Grp1_rbst0_qpp5.mat # QPP5 detected with fast detection          
         │   │      └── ...                           
         │   ├── `dataext`_Grp1_rbst1_QPPs.mat        # Key output variables for QPP1-QPP3 with robust detection (`rbstScrn`=1)
         │   ├── `dataext`_Grp1_rbst0_QPPs.mat        # Key output variables for QPP1-QPP5 with fast detection (`rbstScrn`=0)       
         │   └── ...   
         │
         ├── SbjQPP                                   # Concatenated all EPI scans of each subject as a whole group (`runM`=2) 
         │   ├── interm                               # Intermediate output files
         │   │      ├── `dataext`_Sbj1_rbst1_qpp1.mat # QPP1 detected with robust detection for subject 1
         |   │      ├── `dataext`_Sbj1_rbst1_qpp2.mat # QPP2 detected with robust detection for subject 1      
         |   │      ├── `dataext`_Sbj1_rbst1_qpp3.mat # QPP3 detected with robust detection for subject 2      
         |   │      ├── `dataext`_Sbj2_rbst1_qpp1.mat # QPP1 detected with robust detection for subject 2   
         |   │      ├── `dataext`_Sbj2_rbst1_qpp2.mat # QPP2 detected with robust detection for subject 2      
         |   │      ├── `dataext`_Sbj2_rbst1_qpp3.mat # QPP3 detected with robust detection for subject 2               
         │   │      └── ...                           
         │   ├── `dataext`_Sbj1_rbst1_QPPs.mat        # Key output variables for QPP1-QPP3 of subject 1 with robust detection (`rbstScrn`=1)
         │   ├── `dataext`_Sbj2_rbst1_QPPs.mat        # Key output variables for QPP1-QPP3 of subject 2 with robust detection (`rbstScrn`=1)  
         │   └── ...               
         │
         └── ScnQPP                                   # Concatenated all subjects' EPI data of each scan as a whole group (`runM`=3) 
             ├── interm                               # Intermediate output files
             │      ├── `dataext`_Scn1_rbst0_qpp1.mat # QPP1 detected with fast detection for the 1st scan
             │      ├── `dataext`_Scn1_rbst0_qpp2.mat # QPP2 detected with fast detection for the 1st scan         
             │      ├── `dataext`_Scn2_rbst0_qpp1.mat # QPP1 detected with fast detection for the 2nd scan    
             │      ├── `dataext`_Scn2_rbst0_qpp2.mat # QPP2 detected with fast detection for the 2nd scan           
             │      └── ...                           
             ├── `dataext`_Scn1_rbst0_QPPs.mat        # Key output variables for QPP1-QPP2 of the 1s scan with fast detection (`rbstScrn`=0)
             ├── `dataext`_Scn2_rbst0_QPPs.mat        # Key output variables for QPP1-QPP2 of the 2nd scan with fast detection (`rbstScrn`=0)  
             └── ...


<a name="section-5"></a>
## 4. References
Avants, B., Tustison, N. J., & Song, G. (2022). Advanced Normalization Tools: V1.0. The Insight Journal. https://doi.org/10.54294/UVNHIN

Barrière, D. A., Magalhães, R., Novais, A., Marques, P., Selingue, E., Geffroy, F., Marques, F., Cerqueira, J., Sousa, J. C., Boumezbeur, F., Bottlaender, M., Jay, T. M., Cachia, A., Sousa, N., & Mériaux, S. (2019). The SIGMA rat brain templates and atlases for multimodal MRI data analysis and visualization. Nature Communications, 10(1), 1–13. https://doi.org/10.1038/s41467-019-13575-7

Chou, N., Wu, J., Bai Bingren, J., Qiu, A., & Chuang, K. H. (2011). Robust automatic rodent brain extraction using 3-D pulse-coupled neural networks (PCNN). IEEE Transactions on Image Processing : A Publication of the IEEE Signal Processing Society, 20(9), 2554–2564. https://doi.org/10.1109/TIP.2011.2126587

Chuang, K.-H., Lee, H.-L., Li, Z., Chang, W.-T., Nasrallah, F. A., Yeow, L. Y., & Singh, K. K. D. /O. R. (2018). Evaluation of nuisance removal for functional MRI of rodent brain. NeuroImage. https://doi.org/10.1016/J.NEUROIMAGE.2018.12.048

Cox, R. W. (1996). AFNI: Software for analysis and visualization of functional magnetic resonance neuroimages. Computers and Biomedical Research, 29(3), 162–173. https://doi.org/10.1006/cbmr.1996.0014

Cox, R. W., & Hyde, J. S. (1997). Software tools for analysis and visualization of fMRI data. NMR in Biomedicine, 10(4–5), 171–178. https://doi.org/10.1002/(SICI)1099-1492(199706/08)10:4/5<171::AID-NBM453>3.0.CO;2-L

Jenkinson, M., Beckmann, C. F., Behrens, T. E. J., Woolrich, M. W., & Smith, S. M. (2012). FSL. NeuroImage, 62(2), 782–790. https://doi.org/10.1016/j.neuroimage.2011.09.015

Lee, H. L., Li, Z., Coulson, E. J., & Chuang, K. H. (2019). Ultrafast fMRI of the rodent brain using simultaneous multi-slice EPI. NeuroImage, 195, 48–58. https://doi.org/10.1016/j.neuroimage.2019.03.045

Lein, E. S., Hawrylycz, M. J., Ao, N., Ayres, M., Bensinger, A., Bernard, A., Boe, A. F., Boguski, M. S., Brockway, K. S., Byrnes, E. J., Chen, L., Chen, L., Chen, T.-M., Chi Chin, M., Chong, J., Crook, B. E., Czaplinska, A., Dang, C. N., Datta, S., … Jones, A. R. (2006). Genome-wide atlas of gene expression in the adult mouse brain. Nature 2006 445:7124, 445(7124), 168–176. https://doi.org/10.1038/nature05453

Pan, W.-J., Thompson, G. J., Magnuson, M. E., Jaeger, D., & Keilholz, S. (2013). Infraslow LFP correlates to resting-state fMRI BOLD signals. Neuroimage, 74(0), 288–297. https://doi.org/10.1016/j.neuroimage.2013.02.035

Thompson, G. J., Pan, W. J., Magnuson, M. E., Jaeger, D., & Keilholz, S. D. (2014). Quasi-periodic patterns (QPP): Large-scale dynamics in resting state fMRI that correlate with local infraslow electrical activity. NeuroImage, 84, 1018–1031. https://doi.org/10.1016/j.neuroimage.2013.09.029

Chen, J. (2022). Tools for NIfTI and ANALYZE image. MATLAB Central File Exchange. https://www.mathworks.com/matlabcentral/fileexchange/8797-tools-for-nifti-and-analyze-image

Xu, N., LaGrow, T. J., Anumba, N., Lee, A., Zhang, X., Yousefi, B., Bassil, Y., Clavijo, G. P., Khalilzad Sharghi, V., Maltbie, E., Meyer-Baese, L., Nezafati, M., Pan, W.-J., & Keilholz, S. (2022). Functional Connectivity of the Brain Across Rodents and Humans. Frontiers in Neuroscience, 0, 272. https://doi.org/10.3389/FNINS.2022.816331

