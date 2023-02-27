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
|`D0`   | a (nsbj X nscn) cell matrix |Each cell has a (nroi X ntimepoints) matrix of EPI timeseries. |
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
|      Category     |  Variable name | Description | Note   | 
|------------------|-----------------|--------|-------------|
|  Filepath  		|`data`   | the input filename |The input should has the filename `data`.mat |
|    	|`ext`    | the parameter filename extension | The parameter filename will be  Params_`data`\_`ext`.mat |
|  QPP global parameters|`nP`     | total # of QPPs to detect (nP<=5)| If nP=1, only detect the primary QPP (QPP1); if nP=2, detect both QPP1 & QPP2; etc.|
|	 	|`PL`     | a (nP X 1) vector of QPP window length | ~20s for humans (e.g., PL(ip)=20/TR), |
|  QPP detection	parameters |`cth13` & `cth34`     | a 2D vector of correlation threshold for QPP1-QPP3 (`cth13`) & for QPP4-QPP5 (`cth45`)| If you do not need to detect QPP4-QPP5, please assign `cth34` a random number (e.g., `cth34`=[0, 0]).|
|  QPP phase adjustment	parameters|`cthph` | similarity threshold when phase-adjusting (phadj) a QPP |Default value: cthph=0.88|
|	 	|`s`     | control for strict phase adjustment (`s`=1) or relaxed phase adjustment (`s`=0)||
|	  	|`sdph`     | a (nP X 1) cell array of reference parcels| Each cell may include >=1 parcel IDs. The phase adjusted QPP waveform will start from rising positive values for the selected parcels.|
|  Functional connectivity (FC) analysis parameters|`fz` | control for the output matrix `FCr` to be the pearson correlation (`fz`=1) or to be the Fisher Z-Transformation of the pearson correlaion (`fz`=1).|

<a name="section-2-2"></a>
### 2.2 (Step 2) Run 'st2_QPPanalysis.m'
<a name="section-2-1-1"></a>
#### 2.2.1 Prespecified parameters       
The following three parameters need to be prespecified at the beginning of this script.
|      Category    |  Variable name  | Description | Note   | 
|------------------|-----------------|-------------|--------|
|  Filepath  		|`dataext`   | parameter filename |The parameter .mat file generated from step 1, which has the filename Param_`dataext`.mat |
|  Data concatenation method |`runM`     | control the way to concatenate the data| If `runM`=1, concatenate all `D0{i,j}` as a whole group and detect group QPP; if `runM`=2, concatenate all `D0{i,:}` and detect QPP from all scans of each subject; if `runM`=3, concatenate all `D0{:,:}` and detect QPP from all subjects of each scan.|
| QPP detection	method|`rbstScrn`     | control for fast QPP dectection (`rbstScrn`=0) or robust QPP detection (`rbstScrn`=1)|The fast QPP detection selectes a limited number of starting points which was used in (Abbas, Bassil, et al., 2019; Abbas, Belloy, et al., 2019; Belloy et al., 2018; Majeed et al., 2011; Raut et al., 2021; Yousefi et al., 2018), whereas the robust detection selects all possible starting points which was used in (Maltbie et al., 2022; Xu et al., 2023; Yousefi & Keilholz, 2021).|

<a name="section-2-1-1"></a>
#### 2.2.2 Automated QPP analysis & Final key output variables
For the QPP analysis, follwoing analytical procedures will be executed: 1) QPP1 detection, 2) phase adjustment, 3) QPP2-5 detection from the residuals of QPP1, 4) reverse phase QPP detection, and 5) Functional connectivity computation after QPP regression. The key output variables are:
| Output variable   | Format | Note   | 
|----------------|-------------|-------------|
|`QPPs`| a (nP X 2) cell matrix| The 1st column includes the 2D template of QPP1-QPP`nP`, and the 2nd column includes the corresponding reverse phase QPPs.|
|`Cs`| a (nP X 1) cell vector| Each cell includes the QPP sliding correlation timecourse.|
|`TMXs`| a (nP X 2) cell matrix| The occuring time of the QPP (1st column) and the reverse phase QPP (2nd column).|
|`METs`| a (nP X 2) cell matrix| The meta data of maxima and minima of the QPP sliding timecourse.|
|`Ds`| a (nP X 1) cell vector| Each cell includes original EPI timeseries.|
|`Drs`| a (nP X 1) cell vector| Each cell includes the EPI timeseries after QPP regression.|
|`Crs`| a (nP X 1) cell vector| Each cell includes the QPP sliding correlation timecourses after QPP regression.|
|`FCrs`| a (nP X 1) cell vector| Each cell includes the functional connectivity map after the QPP regression.|

Note: Similar to detected QPPs, the phase-adjusted QPP and related variables are saved in the following cell matrices, `QPPas`, `TMXas`, `METas`, `Cas`. All these parameters are saved in the `dataext`\_`*`_QPPs.mat file, where `*` depends on the prespesified parameters.  
        
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
Abbas, A., Bassil, Y., & Keilholz, S. (2019). Quasi-periodic patterns of brain activity in individuals with attention-deficit/hyperactivity disorder. NeuroImage: Clinical, 21, 101653. https://doi.org/10.1016/j.nicl.2019.101653

Abbas, A., Belloy, M., Kashyap, A., Billings, J., Nezafati, M., Schumacher, E. H., & Keilholz, S. (2019). Quasi-periodic patterns contribute to functional connectivity in the brain. NeuroImage, 191, 193–204. https://doi.org/10.1016/J.NEUROIMAGE.2019.01.076

Belloy, M. E., Naeyaert, M., Abbas, A., Shah, D., Vanreusel, V., van Audekerke, J., Keilholz, S. D., Keliris, G. A., van der Linden, A., & Verhoye, M. (2018). Dynamic resting state fMRI analysis in mice reveals a set of Quasi-Periodic Patterns and illustrates their relationship with the global signal. In NeuroImage (Vol. 180, pp. 463–484). Academic Press Inc. https://doi.org/10.1016/j.neuroimage.2018.01.075

Majeed, W., Magnuson, M., Hasenkamp, W., Schwarb, H., Schumacher, E. H. H. E. H. H., Barsalou, L., & Keilholz, S. D. D. S. D. (2011). Spatiotemporal dynamics of low frequency BOLD fluctuations in rats and humans. Neuroimage, 54(2), 1140–1150. https://doi.org/10.1016/j.neuroimage.2010.08.030

Maltbie, E., Yousefi, B., Zhang, X., Kashyap, A., & Keilholz, S. (2022). Comparison of Resting-State Functional MRI Methods for Characterizing Brain Dynamics. Frontiers in Neural Circuits, 16. https://doi.org/10.3389/FNCIR.2022.681544/FULL

Raut, R. v., Snyder, A. Z., Mitra, A., Yellin, D., Fujii, N., Malach, R., & Raichle, M. E. (2021). Global waves synchronize the brain’s functional systems with fluctuating arousal. Science Advances, 7(30). https://doi.org/10.1126/SCIADV.ABF2709/SUPPL_FILE/SCIADV.ABF2709_SM.PDF

Xu, N., Smith, D. M., Jeno, G., Seeburger, D. T., Schumacher, E. H., & Keilholz, S. D. (2023). The interaction between random and systematic visual stimulation and infraslow quasiperiodic spatiotemporal patterns of whole brain activity. BioRxiv, 2022.12.06.519337. https://doi.org/10.1101/2022.12.06.519337
Yousefi, B., & Keilholz, S. (2021). Propagating patterns of intrinsic activity along macroscale gradients coordinate functional connections across the whole brain. NeuroImage, 231, 117827. https://doi.org/10.1016/j.neuroimage.2021.117827

Yousefi, B., Shin, J., Schumacher, E. H., & Keilholz, S. D. (2018). Quasi-periodic patterns of intrinsic brain activity in individuals and their relationship to global signal. NeuroImage, 167, 297–308. https://doi.org/10.1016/j.neuroimage.2017.11.043
