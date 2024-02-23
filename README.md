# utl-calculate-regression-coeficients-in-base-sas-fcmp-proc-reg-r-and-python
Calculate regression coeficients in base sas fcmp proc reg r and python 
    %let pgm=utl-calculate-regression-coeficients-in-base-sas-fcmp-proc-reg-r-and-python;

    Calculate regression coeficients in base sas fcmp proc reg r and python

    github
    http://tinyurl.com/5db925v3
    https://github.com/rogerjdeangelis/utl-calculate-regression-coeficients-in-base-sas-fcmp-proc-reg-r-and-python

    In matrix algebra
       beta(intercept & coeficients) = inverse(Xtranpose * X) * Xtranpose * Y

     SOLUTIONS

         1 proc reg
           model y=x1 x2;

         2 fcmp
           https://www.lexjansen.com/pharmasug-cn/2016/DS/PharmaSUG-China-2016-DS06.pdf

         3 r matrix algebra

         4 r lm function
           model <- lm(Y ~ X1 + X2)
           Similar to SAS

         5 python OLS.from_formula
           The only output is a text file
           You almost have to learn a new langusge

           lm = sm.OLS.from_formula('Y ~ X1 + X2', have);
           result = lm.fit();

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";

    data
         sd1.x (keep=x0 x1 x2)
         sd1.y (keep=y)
         sd1.xy
         ;
       retain y x0 x1 x2;
       set sashelp.class end=eof;
       x0=1;
       y=weight;
       x1=height;
       x2=age;
       keep x: y;
       output;
    run;quit;

    /**************************************************************************************************************************/
    /*                                      |                               |                                                 */
    /*     d:/sd1/xy.sas7bdat               |        d:/sd1/x.sas7bdat      |   d:/sd1/y.sas7bdat                             */
    /*     ==================               |        =================      |   ================                              */
    /*                                      |                               |                                                 */
    /* Obs      Y      X1     X2            |     Obs    X0     X1     X2   |       Obs      Y                                */
    /*                                      |                               |                                                 */
    /*   1    112.5   69.0    14            |       1     1    69.0    14   |         1    112.5                              */
    /*   2     84.0   56.5    13            |       2     1    56.5    13   |         2     84.0                              */
    /*   3     98.0   65.3    13            |       3     1    65.3    13   |         3     98.0                              */
    /*   4    102.5   62.8    14            |       4     1    62.8    14   |         4    102.5                              */
    /*   5    102.5   63.5    14            |       5     1    63.5    14   |         5    102.5                              */
    /*   6     83.0   57.3    12            |       6     1    57.3    12   |         6     83.0                              */
    /*   7     84.5   59.8    12            |       7     1    59.8    12   |         7     84.5                              */
    /*   8    112.5   62.5    15            |       8     1    62.5    15   |         8    112.5                              */
    /*   9     84.0   62.5    13            |       9     1    62.5    13   |         9     84.0                              */
    /*  10     99.5   59.0    12            |      10     1    59.0    12   |        10     99.5                              */
    /*  11     50.5   51.3    11            |      11     1    51.3    11   |        11     50.5                              */
    /*  12     90.0   64.3    14            |      12     1    64.3    14   |        12     90.0                              */
    /*  13     77.0   56.3    12            |      13     1    56.3    12   |        13     77.0                              */
    /*  14    112.0   66.5    15            |      14     1    66.5    15   |        14    112.0                              */
    /*  15    150.0   72.0    16            |      15     1    72.0    16   |        15    150.0                              */
    /*  16    128.0   64.8    12            |      16     1    64.8    12   |        16    128.0                              */
    /*  17    133.0   67.0    15            |      17     1    67.0    15   |        17    133.0                              */
    /*  18     85.0   57.5    11            |      18     1    57.5    11   |        18     85.0                              */
    /*  19    112.0   66.5    15            |      19     1    66.5    15   |        19    112.0                              */
    /*                                      |                               |                                                 */
    /**************************************************************************************************************************/

    ods output ParameterEstimates=beta_reg;
    proc reg data=xy;
      model y=x1 x2;
    run;quit;
    ods select all;

    proc print data=beta_reg;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* MODEL     DEPENDENT    VARIABLE     DF    ESTIMATE     STDERR     TVALUE      PROBT                                    */
    /*                                                                                                                        */
    /* MODEL1        Y        Intercept     1    -141.224    33.3831    -4.23040    0.00064                                   */
    /* MODEL1        Y        X1            1       3.597     0.9055     3.97259    0.00109                                   */
    /* MODEL1        Y        X2            1       1.278     3.1101     0.41104    0.68649                                   */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___     __
    |___ \   / _| ___ _ __ ___  _ __
      __) | | |_ / __| `_ ` _ \| `_ \
     / __/  |  _| (__| | | | | | |_) |
    |_____| |_|  \___|_| |_| |_| .__/
                               |_|
    */

    %symdel nrow ncol / nowarn;

    proc fcmp;

    /*----                                                                   ----*/
    /*---- get number of rows and number of columns                          ----*/
    /*----                                                                   ----*/

    %dosubl('proc sql; select count(*) into :nrow from sd1.x;quit;');
    %dosubl('data;set sd1.x(obs=1);array t _numeric_;call symputx("ncol",dim(t),"G");run;quit;');

    %put &=ncol &=nrow;  /*---- NCOL=3   NROW=19                             ----*/

    /*----                                                                   ----*/
    /*----  using matrix algebra to calculate regression coeficients         ----*/
    /*----  beta = xPrime * xInverse * y                                     ----*/
    /*----                                                                   ----*/

    array x[&nrow.,&ncol]/nosymbols;
    readx=read_array('x',x);

    array y[&nrow.,1]/nosymbols;
    ready=read_array('y',y);

    array x_t[&ncol,&nrow.]/nosymbols;
    call transpose (x,x_t);

    array xx_t[3,3]/nosymbols;
    call mult(x_t,x,xx_t);

    array xx_t_inv[&ncol,&ncol]/nosymbols;
    call inv(xx_t,xx_t_inv);

    array xx_t_inv_x[3,&nrow.]/nosymbols;
    call mult(xx_t_inv,x_t,xx_t_inv_x);

    array beta[3,1]/nosymbols;
    call mult(xx_t_inv_x,y,beta);

    rc=write_array('beta',beta);

    run;quit;

    proc print data=beta;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*      Obs       BETA1                                                                                                   */
    /*                                                                                                                        */
    /*       1     -141.224                                                                                                   */
    /*       2        3.597                                                                                                   */
    /*       3        1.278                                                                                                   */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                          _        _              _            _
    |___ /   _ __   _ __ ___   __ _| |_ _ __(_)_  __   __ _| | __ _  ___| |__  _ __ __ _
      |_ \  | `__| | `_ ` _ \ / _` | __| `__| \ \/ /  / _` | |/ _` |/ _ \ `_ \| `__/ _` |
     ___) | | |    | | | | | | (_| | |_| |  | |>  <  | (_| | | (_| |  __/ |_) | | | (_| |
    |____/  |_|    |_| |_| |_|\__,_|\__|_|  |_/_/\_\  \__,_|_|\__, |\___|_.__/|_|  \__,_|
                                                              |___/
    */

    %utlfkil("d:/xpt/want.xpt");

    /*---- in R solve function is used to invert a matrix                    ----*/

    %utl_rbegin;
    parmcards4;
    library(haven)
    library(SASxport)
    X<-as.matrix(read_sas("d:/sd1/x.sas7bdat"))
    Y<-as.matrix(read_sas("d:/sd1/y.sas7bdat"))
    want<-as.data.frame(solve(t(X) %*% X) %*% t(X) %*% Y)
    want
    for (i in seq_along(want)) {
              label(want[,i])<- colnames(want)[i]
           }
    write.xport(want,file="d:/xpt/want.xpt")
    ;;;;
    %utl_rend;

    /*--- handles long variable names by using the label to rename the variables  ----*/

    proc datasets lib=work mt=data mt=view nodetails nolist; delete want want_r_long_names; run;quit;

    libname xpt xport "d:/xpt/want.xpt";
    proc contents data=xpt._all_;
    run;quit;

    data want_py_long_names;
      %utl_rens(xpt.want) ;
      set want;
    run;quit;
    libname xpt clear;

    proc print data=want_py_long_names ;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Up to 40 obs from WANT_PY_LONG_NAMES total obs=3                                                                      */
    /*                                                                                                                        */
    /*  Obs           y                                                                                                       */
    /*                                                                                                                        */
    /*   1     -141.224                                                                                                       */
    /*   2        3.597                                                                                                       */
    /*   3        1.278                                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _            _              __                  _   _
    | || |    _ __  | |_ __ ___    / _|_   _ _ __   ___| |_(_) ___  _ __
    | || |_  | `__| | | `_ ` _ \  | |_| | | | `_ \ / __| __| |/ _ \| `_ \
    |__   _| | |    | | | | | | | |  _| |_| | | | | (__| |_| | (_) | | | |
       |_|   |_|    |_|_| |_| |_| |_|  \__,_|_| |_|\___|\__|_|\___/|_| |_|

    */
    %utlfkil("d:/xpt/want.xpt");

    %utl_rbegin;
    parmcards4;
    library(haven)
    library(SASxport)
    library(data.table)
    xy<-read_sas("d:/sd1/xy.sas7bdat")
    model <- lm(xy$Y ~ xy$X1 + xy$X2)
    want<-as.data.table(model$coefficients)
    write.xport(want,file="d:/xpt/want.xpt")
    ;;;;
    %utl_rend;

    /*--- handles long variable names by using the label to rename the variables  ----*/

    proc datasets lib=work mt=data mt=view nodetails nolist; delete want want_r_long_names; run;quit;

    libname xpt xport "d:/xpt/want.xpt";
    proc contents data=xpt._all_;
    run;quit;

    proc print data=xpt.want;
    run;quit;

    data want_py_long_names;
      %utl_rens(xpt.want) ;
      set want;
    run;quit;
    libname xpt clear;

    proc print data=want_py_long_names ;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Obs           Y                                                                                                        */
    /*                                                                                                                        */
    /*  1     -141.224                                                                                                        */
    /*  2        3.597                                                                                                        */
    /*  3        1.278                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                _   _                         _
    | ___|   _ __  _   _| |_| |__   ___  _ __     ___ | |___
    |___ \  | `_ \| | | | __| `_ \ / _ \| `_ \   / _ \| / __|
     ___) | | |_) | |_| | |_| | | | (_) | | | | | (_) | \__ \
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_|  \___/|_|___/
            |_|    |___/
    */

    %utl_submit_py64_310("
    import pandas as pd;
    import numpy as np;
    import statsmodels.api as sm;
    from sas7bdat import SAS7BDAT;
    with SAS7BDAT('d:/sd1/xy.sas7bdat') as m:;
    .   clas = m.to_data_frame();
    lm = sm.OLS.from_formula('Y ~ X1 + X2', clas);
    result = lm.fit();
    f = open('d:/txt/aov.txt', 'a');
    print(result.summary(), file=f);
    f.close();
    ");

    data _null_;
     infile 'd:/txt/aov.txt';
     input;
     putlog _infile_;
    run;quit;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  d:/txt/aov.txt                                                                                                        */
    /*                                                                                                                        */
    /*                              OLS Regression Results                                                                    */
    /*  ==============================================================================                                        */
    /*  Dep. Variable:                      Y   R-squared:                       0.773                                        */
    /*  Model:                            OLS   Adj. R-squared:                  0.745                                        */
    /*  Method:                 Least Squares   F-statistic:                     27.23                                        */
    /*  Date:                Mon, 23 May 2022   Prob (F-statistic):           7.07e-06                                        */
    /*  Time:                        14:26:03   Log-Likelihood:                -71.750                                        */
    /*  No. Observations:                  19   AIC:                             149.5                                        */
    /*  Df Residuals:                      16   BIC:                             152.3                                        */
    /*  Df Model:                           2                                                                                 */
    /*  Covariance Type:            nonrobust                                                                                 */
    /*  ==============================================================================                                        */
    /*                   coef    std err          t      P>|t|      [0.025      0.975]                                        */
    /*  ------------------------------------------------------------------------------                                        */
    /*  Intercept   -141.2238     33.383     -4.230      0.001    -211.993     -70.455                                        */
    /*  X1             3.5970      0.905      3.973      0.001       1.678       5.517                                        */
    /*  X2             1.2784      3.110      0.411      0.686      -5.315       7.872                                        */
    /*  ==============================================================================                                        */
    /*  Omnibus:                        0.572   Durbin-Watson:                   1.935                                        */
    /*  Prob(Omnibus):                  0.751   Jarque-Bera (JB):                0.604                                        */
    /*  Skew:                           0.108   Prob(JB):                        0.739                                        */
    /*  Kurtosis:                       2.153   Cond. No.                         809.                                        */
    /*  ==============================================================================                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*        _       _           _
     _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
    | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
    | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
    |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                              |_|
    */

    REPO
    ------------------------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/utl-drop-down-to-python-for-a-regression-sas-python-interface
    https://github.com/rogerjdeangelis/utl-linear-regression-in-python-R-and-sas
    https://github.com/rogerjdeangelis/utl-betas-for-rolling-regressions
    https://github.com/rogerjdeangelis/utl-calculate-the-regression-slope-for-each-patient-by-treatment
    https://github.com/rogerjdeangelis/utl-drop-down-to-python-for-a-regression-sas-python-interface
    https://github.com/rogerjdeangelis/utl-generate-all-possible-paiwise-interactions-products-regression
    https://github.com/rogerjdeangelis/utl-linear-regression-in-python-R-and-sas
    https://github.com/rogerjdeangelis/utl-locating-breakpoints-for-dogleg-mutiple-regression-lines
    https://github.com/rogerjdeangelis/utl-outlier-analysis-based-on-robust-regression
    https://github.com/rogerjdeangelis/utl-piecewise-regression-find-the-breakpoint
    https://github.com/rogerjdeangelis/utl-random-forest-regression-vs-linear-regression-with-uncorrelated-independent-variables-in-r
    https://github.com/rogerjdeangelis/utl-regression-line-plus-and-minus-the-interquartile-range-of-dependent-variable
    https://github.com/rogerjdeangelis/utl-simple-example-of-meta-regression-using-SAS-and-R
    https://github.com/rogerjdeangelis/utl-using-linear-regression-with-base-sas-and-r-to-interpolate-missimg-values
    https://github.com/rogerjdeangelis/utl-using-the-regression-equation-to-score-a-table-like_a_weighted-sum
    https://github.com/rogerjdeangelis/utl_dosubl_do_regressions_when_data_is_between_dates
    https://github.com/rogerjdeangelis/utl_excluding_rolling_regressions_with_one_on_more_missing_values_in_the_window
    https://github.com/rogerjdeangelis/utl_how_to_automate_a_series_of_logistic_regressions
    https://github.com/rogerjdeangelis/utl_multiple-regressions-using-arrays
    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
