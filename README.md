# pyRV

This code was developed by Christopher Sullivan and Lorenzo Magnolfi.  It largely adapts the pyblp source code (copyright Jeff Gortmaker and Christopher Conlon) to perform the procedure for testing firm conduct developed in "Testing Firm Conduct" by Marco Duarte, Lorenzo Magnolfi, Mikkel Solvsten, and Christopher Sullivan.


## Install
First, you will need to download and install python, which you can do from this [link](https://www.python.org/).

Then you will need to install four python packages: 
* numpy
* pandas
* statsmodels
* pyblp

These can be installed by running the pip3 install command in either terminal (Mac) or Command Prompt (Windows).  For example, to install numpy, run the following:

    pip3 install numpy

Finally, you should download the folder pyRV_folder.  


## Running the code
First open python3 (pyRV has been developed for releases of python 3.6 and later) in either terminal or command prompt

### Import python packages
Then you import the necessary packages:

        import numpy as np
        import pandas as pd
        import pyblp
        import sys

To import pyRV, you specify the path on your computer for the pyRV_folder you downloaded

        pyRV_path = '<user specified path>/pyRV_folder'
        sys.path.append(pyRV_path)
        import pyRV

### Load the main dataset
In this tutorial, we are going to use the Nevo (2000) fake cereal data which is provided in both the pyblp and pyRV packages.  pyblp has excellent [documentation](https://pyblp.readthedocs.io/en/stable/index.html) including a thorough tutorial for estimating demand on this dataset which can be found [here](https://pyblp.readthedocs.io/en/stable/_notebooks/tutorial/nevo.html).   

First you load the main dataset, which we refer to as `product_data`:

        product_data = pd.read_csv(pyblp.data.NEVO_PRODUCTS_LOCATION)

### Estimate demand with pyblp
Next, you extimate demand using pyblp.  

        pyblp_problem = pyblp.Problem(
            product_formulations = (
                pyblp.Formulation('0 + prices ', absorb = 'C(product_ids)' ),
                pyblp.Formulation('1 + prices + sugar + mushy'),
                ),
            agent_formulation = pyblp.Formulation('0 + income + income_squared + age + child'), 
            product_data = product_data,
            agent_data = pd.read_csv(pyblp.data.NEVO_AGENTS_LOCATION)
            )


        pyblp_results = pyblp_problem.solve(
            sigma = np.diag([0.3302, 2.4526, 0.0163, 0.2441]), 
            pi = [[5.4819,0, 0.2037 ,0 ],[15.8935,-1.2000, 0 ,2.6342 ],[-0.2506,0, 0.0511 ,0 ],[1.2650,0, -0.8091 ,0 ]  ],
            method = '1s', 
            optimization = pyblp.Optimization('bfgs',{'gtol':1e-5})  
            )

In this example the linear characteristics in consumer preferences are price and product fixed effects.  Nonlinear characteristics with random coefficients include a constant, `price`, `sugar`, and `mushy`.  For these variables, we are going to estimate both the variance of the random coefficient as well as demographic interactions for `income`, `income_squared`, `age`, and `child`. `price`, `sugar`, and `mushy` are variables in the `product_data`.  Draws of `income`, `income_squared`, `age`, and `child` are in the agent data.  More info can be found [here](https://pyblp.readthedocs.io/en/stable/_api/pyblp.Problem.html)

The first 9 lines of code set up the demand estimation problem.  Running them yeilds output which summarize the dimensions of the problem (see [here](https://pyblp.readthedocs.io/en/stable/_api/pyblp.Problem.html)) for description of each variable.  Also reported are the Formulations, i.e., the linear characteristics, non-linear characteristics for which we are estimating either variances or demographic interactions, and a list of the demographics being used.


        Dimensions:
        =================================================
         T    N     F    I     K1    K2    D    MD    ED 
        ---  ----  ---  ----  ----  ----  ---  ----  ----
        94   2256   5   1880   1     4     4    20    1  
        =================================================

        Formulations:
        ===================================================================
               Column Indices:           0           1           2      3  
        -----------------------------  ------  --------------  -----  -----
        X1: Linear Characteristics     prices                              
        X2: Nonlinear Characteristics    1         prices      sugar  mushy
            d: Demographics            income  income_squared   age   child
        ===================================================================


The second block of code actually runs the estimation. It output includes information on computation as well as tables witrh the parameter estimates. A full list of post-estimation output which can be queried is found [here](https://pyblp.readthedocs.io/en/stable/_api/pyblp.ProblemResults.html).

````
    Problem Results Summary:
================================================================================================================
GMM     Objective      Gradient         Hessian         Hessian     Clipped  Weighting Matrix  Covariance Matrix
Step      Value          Norm       Min Eigenvalue  Max Eigenvalue  Shares   Condition Number  Condition Number 
----  -------------  -------------  --------------  --------------  -------  ----------------  -----------------
 1    +4.561514E+00  +6.914932E-06  +3.442537E-05   +1.649684E+04      0      +6.927228E+07      +8.395896E+08  
================================================================================================================

Cumulative Statistics:
===========================================================================
Computation  Optimizer  Optimization   Objective   Fixed Point  Contraction
   Time      Converged   Iterations   Evaluations  Iterations   Evaluations
-----------  ---------  ------------  -----------  -----------  -----------
 00:00:21       Yes          51           57          45479       141178   
===========================================================================

Nonlinear Coefficient Estimates (Robust SEs in Parentheses):
=========================================================================================================================================================
Sigma:         1             prices            sugar            mushy       |   Pi:        income       income_squared         age             child     
------  ---------------  ---------------  ---------------  ---------------  |  ------  ---------------  ---------------  ---------------  ---------------
  1      +5.580936E-01                                                      |    1      +2.291971E+00    +0.000000E+00    +1.284432E+00    +0.000000E+00 
        (+1.625326E-01)                                                     |          (+1.208569E+00)                   (+6.312149E-01)                 
                                                                            |                                                                            
prices   +0.000000E+00    +3.312489E+00                                     |  prices   +5.883251E+02    -3.019201E+01    +0.000000E+00    +1.105463E+01 
                         (+1.340183E+00)                                    |          (+2.704410E+02)  (+1.410123E+01)                   (+4.122564E+00)
                                                                            |                                                                            
sugar    +0.000000E+00    +0.000000E+00    -5.783552E-03                    |  sugar    -3.849541E-01    +0.000000E+00    +5.223427E-02    +0.000000E+00 
                                          (+1.350452E-02)                   |          (+1.214584E-01)                   (+2.598529E-02)                 
                                                                            |                                                                            
mushy    +0.000000E+00    +0.000000E+00    +0.000000E+00    +9.341447E-02   |  mushy    +7.483723E-01    +0.000000E+00    -1.353393E+00    +0.000000E+00 
                                                           (+1.854333E-01)  |          (+8.021081E-01)                   (+6.671086E-01)                 
=========================================================================================================================================================

Beta Estimates (Robust SEs in Parentheses):
===============
    prices     
---------------
 -6.272990E+01 
(+1.480321E+01)
===============
````

### 
