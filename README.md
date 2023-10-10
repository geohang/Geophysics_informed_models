# Geophysics_informed_models

This code is provided for paper: Geophysics-informed hydrologic modeling of a mountain headwater catchment for studying hydrological partitioning in the critical zone
We thank the codes provided at https://modflow6-examples.readthedocs.io/en/master/_notebooks/ex-gwf-sagehen.html. Our code is basically modified from their example

# Steps for establishing a geophysics-informed hydrologic model
We are going to show the detailed steps to establish geophysics-informed hydrologic models with seismic refraction data. The codes in https://github.com/geohang/Geophysics_informed_models further help readers to reproduce this work. Note that all of the codes we used in this research are open source and it helps everyone to follow our work. Here are the steps:

Step 1: Invert the seismic refraction data. In this step, we need to invert the travel-time data into the subsurface velocity. There are many tools to achieve this step. In our research, we use the open-source Python package pyGIMLi https://www.pygimli.org/ (Rücker et al., 2017) to process the seismic data. We also provide examples and seismic refraction data in our GitHub link.
Step 2: Build a 3D seismic velocity model from 2D velocity models. In this step, we need to have a 3D seismic velocity model by Kriging. The procedure is following the work (Flinchum et al. 2018). There are also many open-source tools to achieve this step. In the work (Flinchum et al. 2018), they used the Stanford Geostatistical Modeling Software (Remy et al., 2009) and it is based on Matlab. In our work, we used the GSTools toolbox https://github.com/GeoStat-Framework/GSTools (Müller et al., 2021), which is based on Python.
Step 3: Extracting CZ structures from the velocity model. After having the 3D velocity model, we need to extract the depth of regolith and fractured bedrock from the velocity model. In this step, we need to pick these representative velocity values for the interfaces. Sometimes, rock physics knowledge is used to guide the selection process. Here, we follow the method proposed by Chen and Niu (2022) and use the trend of velocity gradient to determine these representative values. We admit that this step may introduce some uncertainty into the final results but the depth would not change significantly if you pick velocity values in a reasonable way. 
Step 4: Putting CZ structures into the hydrologic models. This step is a key gap from geophysics to hydrologic models. In this study, we chose MODFLOW 6 (Bakker et al., 2016; Hughes et al., 2017) to do the hydrologic modeling. There might be some differences in setup in different software. In MODFLOW 6, you can set layers for the groundwater model and for each layer you can assign elevation pix by pix. In this way, you can assign the CZ structures to the MODFLOW 6 to consider the subsurface heterogeneity. In this step, we recommend using open source Python package Flopy https://github.com/modflowpy/flopy (Bakker et al., 2016) to generate the input files for MODFLOW 6, which will be much easier to manage the data.
Step 5: Establish integrated hydrologic modeling which can reliably characterize surface-water/groundwater interactions in complex hydrological systems. In this step, we need to establish integrated hydrologic modeling to simulate the water processes in the catchment as accurately and comprehensively as possible. We need to use many advanced packages in MODFLOW 6 to achieve this goal. Here we will introduce how we integrate these advanced packages into our models and briefly explain the functions of these packages in catchment hydrologic modeling. The design of this model mostly refers to the work (Niswonger et al., 2006; Daoud et al., 2022) and open-source tutorial code from Flopy:
https://modflow6-examples.readthedocs.io/en/master/_notebooks/ex-gwf-sagehen.html. 
We will briefly introduce the most important packages we used in our study and how we manage the input data. Still, if you want to check these packages further, you can refer to the manual (Hughes et al., 2017). Here are the packages:
	DIS: This is for the discretization information for structured grids. In this package, you can input the CZ structure information inside as described in step 4. In our study, we set three layers for regolith, fractured bedrock and bedrock. If you have more information about the subsurface, you can set more layers and heterogeneity as you want. 
	NPF: It is a package to assign the hydraulic conductivity parameters to each node. The parameters we used in this research could be referred to as Tabel 1.
	DRN: It is a head-dependent boundary simulating the draining process. In this study, we use the DRN package to simulate groundwater discharge to the land surface to keep this water separate from rejected infiltrated simulated by the UZF package. The usage is originally from MODFLOW 6 tutorial and code https://modflow6-examples.readthedocs.io/en/master/_notebooks/ex-gwf-sagehen.html. To achieve this function, we used surface elevation as the input and a big hydraulic conductivity to make groundwater discharge to the land surface directly when the head is larger than the surface elevation. 
	SFR: SFR package is the streamflow routing package that simulates the flow interaction between the streams and the groundwater. The data needed for the streams’ properties are the streams’ length, width, slope, Manning coefficient, bed level and bed hydraulic conductivity. Reach connectivity must be explicitly specified for this version of the SFR Package. To prepare the input data for the SFR package, we first use ESRI ArcGIS software to identify streamflow reaches in the catchment (Figure 1a). The streams’ geometric properties, such as length, width, slope and bed level, are either estimated from ESRI ArcGIS software or empirically estimated by field observations. Note that ESRI ArcGIS software is commercial software but you can use open-source software QGIS https://qgis.org/en/site/ to achieve the same function. The detailed properties can refer to our code and properties file “sfr_pack.txt” in GitHub. Following work (Daoud et al., 2022), the Manning coefficient was assumed as 0.035 for all the stream reaches. The streams’ bed hydraulic conductivity was adjusted during the calibration as dependent on the 𝐾𝑣 of the matching cells that contain the streams, following the work (Daoud et al., 2022; Niswonger et al., 2006). The connection of the streams was first extracted automatically by the open-source software ModelMuse (Winston, 2019) and then manually checked based on the elevations of the reaches. 
	MOV: MVR is a new water mover package designed in MODFLOW 6 to move the water from a feature in one package as a provider to a feature in another package as a receiver. The MOV package is used to simulate the groundwater and surface water interaction in this study. Basically, it simulates the re-infiltration concepts, i.e., allowed the transferring of the water from rejected infiltration (UZF) and groundwater exfiltration (DRN). For the water from UZF, we use the top elevation array and SFR locations to calculate an array that contains id of neighboring UZF cells to the id of the SFR locations. The function is originally from MODFLOW 6 tutorials and can be seen in provided GitHub code. Similarly, it also can decide the location of groundwater exfiltration (DRN) to the specific SFR cells. Following the work (Langevin et al., 2017), we assume the factor for water moving fraction is 1 in this study.  
	UZF: UZF package is the package used to simulate the flow through the unsaturated zone and add the simulated flow to the groundwater zone. The UZF package simulates only the vertical unsaturated flow using the kinematic wave approximation to Richard’s equation and is solved by the method of characteristics (Niswonger et al., 2006). The land surface driving forces are introduced to the UZF package as inputs (infiltration rate and potential evapotranspiration rate), and both are applied at the surface. The hydrometeorological input data is introduced in section 2.2 in the main content. Other parameters used in the UZF package are either in Table 1 or Table S3. 
	GHB: The General-Head Boundary package is used to simulate head-dependent flux boundaries. As shown in equation (5), we use the GHB package to simulate the deep infiltration. The conductance is a calibrated parameter and the boundary head is the bottom of fresh bedrock.

STEP 6: Calibration of the model parameters. The calibration process was carried out using the shuffled complex evolution method (Duan et al., 1994) with the Python package SPOTPY https://spotpy.readthedocs.io/en/latest/ (Houska et al., 2015). The objective function in calibration only includes the measured discharge in this study. Further details of the calibration can be found in section 4.2 and Table 1. 
STEP 7: Water budget analysis. Flopy packages provide many useful functions to help analyze the water budget. It can be referred to the codes we provided or paper (Hughes et al., 2023).


