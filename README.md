# CCFatigue Platform

CCFatiguePlatform is an initiative from CCLab that aims to develop a web application to faciltate manipulation and harmonized storage of composite materials testing datasets. 

[Demo](https://ccfatigue-test.epfl.ch/)



# Web app: Usage
## Setup & run locally (without CCFatigue modules)

1. run backend according to [backend/README.md](backend/README.md)
2. run frontend according to [frontend/README.md](frontend/README.md)

## Run on server
See https://github.com/EPFL-ENAC/SB_Sysadmin/tree/enacvm0056



# Data
## Standard data format
[Example of data in standard format](https://drive.google.com/file/d/1-SuUHPbW-xFr65yqVIbrqHl1vb4ejMzo/view?usp=sharing "Shayan's data in standard format")




<a href="https://drive.google.com/uc?export=view&id=1GOUXB5KgDwiakTjMCoWDSEOsvrMtjIb6"><img src="https://drive.google.com/uc?export=view&id=1GOUXB5KgDwiakTjMCoWDSEOsvrMtjIb6" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />



|Directory naming convention|
|---------------------------|
|\Laboratory\Researcher\Test Type\Date\File type\ *filename
|Laboratory: Full accronym (i.e. CCLab, RESSLab, IBeton,...)
|Researcher: Last name
|Test type: Fatigue, Combined Fatigue Fracture, Standard Quasi-Static, Combined Quasi-Static Fracture
|Date: YYMMDD
|File type:  {RAW} = raw data, {STD} = standard format, {AGG} = aggregated format, {CYC} = Cycle counting (module 1) output, {SNC} = S-N curve (module 2) output, {CLD} = CLD (module 3) output, {FAF} = Fatigue failure (module 4) output, {DAS} = Dammage summation (module 5) output, {HYS} = Hysteresis_loops.py output


|Filenaming conventions|
|-----------------------|
|{File type} _ {Date} _ {Test type} _ {###}|
|File type: {RAW} = raw data, {STD} = standard format, {AGG} = aggregated format, {CYC} = Cycle counting (module 1) output, {SNC} = S-N curve (module 2) output, {CLD} = CLD (module 3) output, {FAF} = Fatigue failure (module 4) output, {DAS} = Dammage summation (module 5) output, {HYS} = Hysteresis_loops.py output
|Date: YYYY-MM-DD
|Test type: {FA} = standard fatigue, {QS} = standard quasi-static, {FF} = combined fatigue/fracture, {SF} = combined quasi-static/fracture
|###: numerical identifier
|Example: RAW _ 2021-04-20 _ FA _ 001

# Contents

## Code structure
<a href="https://drive.google.com/uc?export=view&id=148oUhO3_aIb8mcaS0nXiJbwZnd1sHW6X"><img src="https://drive.google.com/uc?export=view&id=148oUhO3_aIb8mcaS0nXiJbwZnd1sHW6X" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />

Visual representation of how different data are generated and treated

## Standardizing data
Code Standardizing_data.py converts raw data from use cases to the standard format
Raw data format supported: The raw data is read in CSV format

Usage: python run Standardizing_data.py

This code takes inputs in their raw format and transforms them in a standard format used for analysis

## Hysteresis analysis
The code section 'Hysteresis_loops.py' uses the data in it's standard form and performs computations to extract:

Hysteresis Area:
>Area within each of the loading/unloading cycles

Stiffness:
>Slope of linear regression for each loading/unloading cycles

Creep:
>Residual value of strain for each loading/unloading cycle

Output is a csv file with 4 columns: Number of cycles (n_cycles), Hysteresis area (hysteresis_area), Stiffness (stiffness), Creep (creep)

The code is structured into a certain number of functions:

### read\_std(data\_directory, data\_type, res, date, test\_type, test\_number, lab, researcher, loading):
    
    Arguments:
    	data_directory, data_type, res, date, test_type, test_number, lab, researcher, loading:
    	Information relative to file structure

   	Returns:
   		pandas dataframe with values in standard format

    Description: 
    	This function reads the csv file created with the Standardizing_data.py method
    	
### read\_met(data_directory, data\_type, res, date, test\_type, test\_number, lab, researcher, loading):    	
    
    Arguments:
    	data_directory, data_type, res, date, test_type, test_number, lab, researcher, loading:
    	Information relative to file structure

	Returns:
		pandas dataframe with values in standard format

    Description: 
    	This function reads the JSON file created with the Standardizing_data.py method

### write\_hys(hyst\_df, data\_directory, data\_type, res, date, test\_type, test\_number, lab, researcher, loading):  
	Arguments:
		hyst_df: DataFrame containing the following computed values:
		n_cycles, hysteresis_area, stiffness, creep_strain
		
		data_directory, data_type, res, date, test_type, test_number, lab, researcher, loading:
		Information relative to file structure
	
	Returns: Void
	
	Description: 
		This function takes all the data computed within Hysteresis_loops.py and 
		writes them as CSV to a specified location
	
### create\_hyst\_df()
	Arguments: 
		Void
	
	Returns: 
		Initializes a dataframe with 4 empty columns
		
	Description: 
		In this function we use the columns described by hyst_col and initialize an empty list
		that will contain all the computed information
		
		
### get\_stress\_strain(df, n_cycles):
	Arguments:
		df: df is the dataframe generated by the Standardizing_data.py method with information
		about Stress and Strain arranged in standard format
		n_cycles: List with n_cycles without repetitions
	
	Returns: 
		Void
	
	Description:
		This function takes the Machine_Load and Machine_Displacement columns from the standard
		format dataframe and isolates all the stress strain values for each cycles into a 
		list of lists
		
		
### fill\_hyst\_df(df, meta\_df, hyst\_df):

    Arguments:
        df: dataframe with raw data in standard format
        meta_df: metadata information contained in JSON file
        hyst_df: initialized dataframe with four columns generated with create_hyst_df()

    Returns: 
    	Void

    Description: 
    	This function does all the computations on the hysteresis loops allowing to
    	find values for hysteresis area, stiffness, creep and the number of cycles
    	In the first part we take the column Machine_N_cycles from the standard dataframe and
    	make it more compact by allowing one row per cycles
    	We then initialize 3 empty lists for the computations of hysteresis area, creep and
    	stiffness
    	The function get_stress_strain is then called, creating a list of len(n_cycles) lists
    	each containing stress/strain information for a single hysteresis loop
    	Then using a shoelace algorithm, we compute the area of every polygon described by
    	hysteresis loops and store them in the Hysteresis_Area list
    	Using the same points as for area computations, we evaluate the value for stiffness by
    	performing a linear regression on each loop and extracting the slope
    	We then evaluate the value for creep by taking the arithmetic mean the maximum and
    	minimum value for strain at each cycle
    	
### PolyArea(x, y)
    Arguments:
        x: values measured along the x axis (strain)
        y: values measured along the y axis (stress)

    Returns: 
        Single value for the area of the loop

    Description:
        The PolyArea function computes the area of each hysteresis loops using a shoelace algorithm


## Plots
test_dashboard.py creates the following plots:

Stress / strain curve:



<a href="https://drive.google.com/uc?export=view&id=1-SIpuYornVK4nhq8atgAOistwIiKig5A"><img src="https://drive.google.com/uc?export=view&id=1-SIpuYornVK4nhq8atgAOistwIiKig5A" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />

>On this graph we use the raw inputs for Stress and Strain, specific loops have been selected in order for the graph not to be too busy

<a href="https://drive.google.com/uc?export=view&id=1-LDUhePw0yY00AnlKi3Rts-FyJ8mLl6Z"><img src="https://drive.google.com/uc?export=view&id=1-LDUhePw0yY00AnlKi3Rts-FyJ8mLl6Z" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />

>This graph shows the evolution of the sample's stiffness. Stiffness corresponds to the slope of hysteresis loops and is closely linked to hooke's law. E is comparable to the constant k in the context of springs. The analysis is made from the stress - strain raw data, for each hysteresis loop, we evaluate the slope with a linear regression for each hysteresis loops for stress and strain.

<a href="https://drive.google.com/uc?export=view&id=1-DAFgVc8wKqIOcSvKcXtc7fzzkf8XjbX"><img src="https://drive.google.com/uc?export=view&id=1-DAFgVc8wKqIOcSvKcXtc7fzzkf8XjbX" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />

>Here we see how the area of each hysteresis loop evolves as the fatigue test goes on. The hysteresis loop area is closely related to the amount of energy dissipated through deformations, the sum of hysteresis areas is denoted as TDE, or total dissipated energy

<a href="https://drive.google.com/uc?export=view&id=1-JyI8mopYgwfF1QvsY_bx17DqilkfFTD"><img src="https://drive.google.com/uc?export=view&id=1-JyI8mopYgwfF1QvsY_bx17DqilkfFTD" style="width: 650px; max-width: 100%; height: auto" title="Click to enlarge picture" />

>Creep corresponds to the residual deformations during a test or life cycle of a sample

%>>>> 

### Aggregating data
aggregate.py takes all files in an experiment in standard format, analyses them and creates an aggregated set with the right format for analysis in CCFatigue. Output is a csv file with 6 columns: Stress ratio, Reliability level, Stress level no., Stress parameter, Number of cycles to failure, Residual strength 

The aggregate.py code is structured into a sequence of functions that allow to generate data in the appropriate format

### read\_df\_list()

    Arguments: 
        Void

    Returns: 
        dataframe containing as many lists as there are files in the respository

    Description: 
        We specify a path to a repository containing all the tests in an experiment in their
        standard format
        Then a list is created for each test and stored in a dataframe with all other tests

### read\_met()
    Arguments: 
        Void

    Returns: 
        dataframe containing as many lists as there are files in the respository

    Description: 
        We specify a path to a repository containing all the metadata in an experiment
        Then a list is created for each test and stored in a dataframe with all other tests
        
### stress\_n\_cycles()

    Arguments:
        df: dataframe generated by the read_df_list() function
        meta_df: dataframe generated by the read_met() function

    Returns:
        agg_stress: list with the maximum stress for each test
        agg_n_cycles: list with the maximum number of cycles for each test

    Description: 
        In this function, we first go across all the individual testings in order to find the
        maximum value of stress. We then fetch inside the metadata files the maximum number of
        cycles for all testings. The values are stored in the lists agg_stress and
        agg_n_cycles respectively
        
### calculate\_stress\_lev()

    Arguments:
        meta_df: dataframe containing metadata of all testings
        agg_stress: list containing the maximum stress value for all testings
        agg_n_cycles: list containing the number of cycles to failure for all testings

    Returns:
        sortStress: Same values as agg_stress only arranged in descending order
        sortNCycles: Same values as agg_n_cycles arranged following the order of sortStress
        stressLev: Different levels of stress defined for the experiment, takes integer values
        R_ratio: list of stress ratio for all testings
        rel_level: list of reliability level for all testings

    Description:
        This function creates all the columns necessary for the aggregated format to be
        compatible with CCFatigue
        First step is to create the R ratio and reliability level columns
        Then we arrange the stress and cycles to failure columns and evaluate the stress level
        number by bunching up the tests that are supposed to have the same values for stress
        (may vary slightly due to geometric factors)

### create\_agg(R\_ratio, rel\_level, stressLev, sortStress, sortNCycles):

    Arguments:
        R_ratio: list with the values of R_ratio of the individual tests
        rel_level: list with the reliability levels of the individual tests
        stressLev: list of the stress level for the individual tests
        sortStress: list of maximum stress values for the individual tests
        sortNCycles: list of number of cycles to failure for the individual tests

    Returns:
        SNdf: data frame with 6 columns described by 'cols' and as many rows as there are tests

    Description:
        Takes the individual columns generated by calculate_stress_lev() and arranges them
        within a dataframe

### Modules
Using the notebooks in the modules folder, one can compute the right parameters for plotting i.e. the notebook 'Hysteresis loops.ipynb' uses stress/strain info to compute the TDE and evolution of stiffness


# Reproduce

[Environment file](environment.yml "Environment file")

# License
TBD.

# Authors
Charlotte Weil, Scott M. Salmon, Samuel Bancal.
