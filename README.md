# Geolog used to automate the Carbonate characterization workflow using Clerke’s Arab D Rosetta Stone calibration data. This technique provides for a full pore system characterization with modeled Capillary Pressure saturations for an Arab D complex carbonate reservoir.

### INSTALL NOTE: Please delete any old GitHub subdirectory from this repository and definitely the old Geolog Project because we have had some major changes in the workflow with the addition of the handling of the Thomeer parameters and FWL Search routine. To install the Geolog project, delete the old ArabD_GitHub Geolog project, unzip the project in this repository and copy this 'new and improved' version of the Geolog project to your PG_PROJECTS location.

#### You will also need to have loaded the following python libraries installed in your Geolog PG_PYTHON_EXE site area:
- import geolog
- import pandas as pd
- import numpy as np
- import matplotlib.pyplot as plt
- import math
- from pandas import DataFrame, read_csv
- import altair as alt
- import altair_transform
- alt.renderers.enable('altair_viewer')
- alt.data_transformers.disable_max_rows()

## Introduction:
Geolog has always been found to be a user-friendly Petrophysical software tool that allows the users to load, interrogate, process and display well log and production data with ease. With the introduction of Geolog18.0 (now Geolog21.0), we can also use python code in our Geolog loglans to push the analysis even further. Geolog python loglans can exploit state-of-the-art methods in python to interrogate and build probabilistic techniques to estimate additional well log results and calibrate to core analysis results, rock types, facies, engineering parameters and petrophysical properties.  

In this repository we include a comprehensive Geolog project primarily using Geolog python loglans. We supply an example well to utilize a proven workflow to interrogate and characterize a typical Arab D carbonate reservoir. This example serves as the basis for a full-field reservoir characterization workflow that would be used on all wells throughout the entire field. In this example we are showing the results for just one well, but in the full-field reservoir characterization we would follow the same workflow and generate the same results for all wells in the field. The final objective would be to use these well data results and create a 3D static model of the reservoir using the porosity, permeability, Petrophysical Rock Types (PRT), capillary pressure parameters and saturations determined in this workflow. Typically, this static model would then be used to initialize the dynamic model for reservoir simulation. 

![Geolog_Image](Results.png)

This project demonstrates tried and proven workflow with the techniques as described by Phillips(1) et al. used in the characterization of most Arab D reservoirs in Saudi Arabia. Permeability, Petrophysical Rock Types (PRT), Capillary Pressure and modeled saturations are all estimated or calculated in this workflow using the new python loglans in order to characterize this complex carbonate reservoir. Clerke’s(2) Arab D Rosetta Stone core analysis database is used as the calibration data. 

These calibration data are from Ed Clerke’s Rosetta Stone, Arab-D carbonate dataset from Ghawar field in Saudi Arabia. This is a very special carbonate dataset. Clerke randomly selected the final calibration samples from 1,000’s of core plugs for the final dataset.  The Rosetta Stone data cover the full range in poro-perm space and Petrophysical Rock Types (PRTs) observed in the Arab D reservoir. For each sample Clerke acquired High Pressure Mercury Injection (HPMI) and fit the capillary pressure curves to the Thomeer hyperbola (see Altair Plot of Capillary Pressure curves) created from the Initial Displacement Pressure (Pdi), curvature term Gi that relates to the variability of pore throats and Bulk Volume Occupied (BVocci) that is related to the Pore Volume for each pore system i.  From the results Clerke defined his Petrophysical Rock Types (PRT). For this Arab D reservoir, most PRTs have a dual-porosity system, and some PRTs have up to 3 pore systems. 

## Suggested Arab D Carbonate Workflow:
The following workflow is suggested to interrogate, process, interpret and model the petrophysical properties of a typical Arab D carbonate reservoir using Clerke’s Arab D Rosetta Stone Carbonate database as calibration. The workflow consists of the following steps:

1) We can interrogate both the Well Log data and Rosetta Stone calibration data using standard Geolog layouts, cross plots and histograms or we can use a python loglan featuring Altair, which is very interactive python software library driven from a Geolog Module Launcher. With Altair you can build the layout of your choice using depth plots, cross plots or histograms, where they are all interactive and dynamically linked.

### Altair Used to Interrogate the Well log data in Geolog:
![Geolog_Image](Geolog20_ArabD.gif)

		alt_logs.pysh

### Altair used to Interrogate the Rosetta Stone Thomeer Capillary Pressure curves and Petrophysical Rock Types (PRTs):
![Geolog_Image](Thomeer_Pc_and_Thomeer_Parameters2.gif)

		altair_thoreer_parameters.pysh

2) Typically, we run MultiMin for a solid log analysis model using the minerals found in the Arab D reservoir; Limestone, Dolomite, Anhydrite with Illite. Having learned MultiMin from Stepehen Cheshire, the author of MultiMifn, we always use environmentally corrected log data and use  calculated uncertainties for each log curve employed in the analysis. Stephen called this the 'Rifle Drill' approach to MultiMin.

We cannot show the MultiMin code for Optimization; but to serve as an example, we have created a python loglan that utilizes Scipy Optimize to estimate lithology:

   		optimize_lith.pysh 
    
This loglan first uses digitized chartbook data as the basis for our kNN Porosity (PHIT) and Rho Matrix density calculations used in this analysis. Once PHIT is estimated, then we then use Scipy Optimize (minimize) to optimize on our carbonate lithology. 

The primary function employed in Scipy is shown below. It is trying to estimate volumes of Calcite and Dolomite.

	fun = lambda x: (RHOB2 - (2.52*vol_illite + 2.71*x[0]+2.847*x[1]+PHIT*FD)) + (TNPH - (0.247*vol_illite + 0*x[0]+0.005*x[1]+PHIT*1))

Which is using two log response functions that are to be minimized:

	RHOB_theoretical =  2.52*vol_illite + 2.71*x[0]+2.847*x[1]+PHIT*FD

	TNPH_theoretical =  0.247*vol_illite + 0*x[0]+0.005*x[1]+PHIT*1

where res.x[0] = VOL_CALCITE and res.x[1] = VOL_DOLO. The objective is to minimize the difference between **RHOB - RHOB_theoretical** and **TNPH - TNPH_theoretical** while solving for lithology. Please consider this work in progress. We have much to learn on python Optimization. 

We would like to thank Andy McDonald and for the great ideas that come from his Petrophysics Python Series. We are using his hatch fill example in our optimization loglan depth plots. 

Please find below an example of the Scipy optimization output.

![Geolog_Image](Optimize_lith_Geolog.png)

3) Use available core calibration data from the representative reservoir/field to build a petrophysical model to estimate permeability from our python loglan using kNN. We are using normalized input data that is weighted by Euclidean distances for each of the nearest neighbors to make our estimation using this python loglan:
	
		perm_knn.pysh

4) Using the kNN estimated permeability and calculated Total Porosity (PHIT) from our optimization, we query Clerke’s Rosetta Stone core database to predict the Thomeer Capillary Pressure parameters (Pdi, Gi and BVocci) for each pore system i over the reservoir interval using this loglan;

		thomeer_parameters.pysh

 We also use a python loglan to predict the following Petrophysical Rock Types (PRT) also using kNN as defined by Clerke:
- M_1 Macro/Meso
- M_2 Macro/Micro
- M_1_2 Macro/Meso/Micro
- Type1 Meso
- Type 1_1 Meso/Micro
- Type 2 Micro PRTs

		thomeer_prt.pysh

![Geolog_Image](Thomeer_output.png)

5) We use the Thomeer Capillary Pressure parameters that we estimated over the entire reservoir interval to model saturations based on the buoyancy due to reservoir fluid density differences at each height above the Free Water Level (FWL). We usually compare the Bulk Volume Oil (BVO) from MultiMin vs. BVO from Thomeer-based capillary pressure saturations since BVO is pore volume weighted to test our results. 

#### Free Water Level Search:
We have provided a FWL Search routine in python too to estimate the FWL elevation (TVDss) in our example well. We typically use this on each key well in the field to create a FWL plane for the field. 

		fwl_search.pysh

To model Capillary Pressure saturations, it is essential to have a proper Free Water Level (FWL). Reservoir Capillary Pressure or buoyancy is dependent upon the height above the FWL. 

On new discoveries the FWL is usually determined from Formation Test data plotting the pressure data vs. TVDss to find the intersection of the water gradient vs. hydrocarbon gradient. The elevation of this intersection is the FWL or zero Capillary Pressure. However, on older fields, this type of data is typically not available prior to pressure depletion and/or fluid contact movements in the field. Therefore, we need another way to estimate the FWL for the field. In the python software used in our Notebook we offer a FWL search technique that has been shown to work very well in numerous fields.

We perform this well-by-well FWL search by varying the FWL elevation from an estimated highest FWL to the lowest expected FWL (spill point...) for the reservoir.  We then calculate the error difference between the Bulk Volume Oil (BVO) from logs vs. BVO from Thomeer Capillary Pressure at each new FWL estimate for that well. The final fwl_est is the FWL estimated with the lowest Bulk Volume Oil (BVO) error for that well. This fwl_est is then used in our final Thomeer BVO Oil calculations.

The FWL search is usually run on all wells with a fwl_est for each well. In many instances in fields with large hydrocarbon columns, the wells near the crest will be too high above the FWL to give valid results. We have found that the wells near the edge usually give the best estimation.  However, those wells affected by water encroachment will also not give valid results. In the end it is usually a small percentage of wells near the edge of the field that will give valid FWL estimates that are consistent. The search results from these wells are then typically used to construct a plane in the 3D fine-grid model to represent the FWL for the field.

It should be noted, that not all FWL surfaces are flat. Structural tilting, subduction, and dynamic aquifers... can result in a tilted FWL elevations with the possibility of residual oil below the FWL, depending in the situation. 

![Geolog_Image](FWLSearch.png)

Finally, we estimate the Thomeer Capillary Pressure BVO using the FWL Search results and

		thomeer_pc_sats.phsy

to calculate our final estimate of BVO from Capillary Pressure data. 

6) As a secondary technique to estimate PRTs, we also tested another applications in Geolog employing python’s Sklearn as published by Hall(3). We could have estimated Depositions of Environment or other types of categoric geologic facies used in this Sklearn prediction process. There is also a GitHub repository containing the Jupyter Notebook at the following link to use as a help file to understand the process and set model parameters: 

https://github.com/Philliec459/SKLEARN-used-to-predict-Petrophysical-Rock-Types-in-Arab-D-Carbonate

The same data used in Geolog can be evaluated within this Jupyter Notebook using Seaborn matrix plots, various types of a Confusion Matrix plots and the pick the SVM model C and gamma parameters from the Heat Map shown below. However, be sure to input the C_REG and GAMMA values in the Geolog python loglan using the most accurate C and gamma combination presented from the Heat Map.

![Geolog_Image](evaluate.png)

## RESOURCES:
https://www.pdgm.com/products/geolog/

https://github.com/Philliec459?tab=repositories

https://github.com/Philliec459/SKLEARN-used-to-predict-Petrophysical-Rock-Types-in-Arab-D-Carbonate



1.	Phillips, E. C., Buiting, J. M., Clerke, E. A, “Full Pore System Petrophysical Characterization Technology for Complex Carbonate Reservoirs – Results from Saudi Arabia”, AAPG, 2009 Extended Abstract.
2.	Clerke, E. A., Mueller III, H. W., Phillips, E. C., Eyvazzadeh, R. Y., Jones, D. H., Ramamoorthy, R., Srivastava, A., (2008) “Application of Thomeer Hyperbolas to decode the pore systems, facies and reservoir properties of the Upper Jurassic Arab D Limestone, Ghawar field, Saudi Arabia: A Rosetta Stone approach”, GeoArabia, Vol. 13, No. 4, p. 113-160, October, 2008. 
3.	Hall, Brendon, “Facies classification using Machine Learning”, The Leading Edge, 2016, Volume 35, Issue 10
 
