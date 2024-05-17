# hw7data


name:  <unnamed>
       log:  /Users/kiyoi/Desktop/stata/stata II/Assignment 5/hw7.Du.Sijin.log
  log type:  text
 opened on:  16 May 2024, 23:20:00


//1. data
global repo "https://github.com/jhustata/project/raw/main/"

global nhanes "https://wwwn.cdc.gov/Nchs/Nhanes/"

 //2. code
do ${repo}followup.do
/*
> 
> 6.2 Code Development:
> Edit and Rename Provided Script:
> Download, modify, and upload the provided Stata .do file for linking the DEMO.XPT data to mo
> rtality follow-up data. 
> Rename this file to followup.do and commit it with the description: “Updated DEMO.XPT linkag
> e .do file”. 
> In otherwords, download, modify, and upload Stata_ReadInProgramAllSurveys.do. 
> You may watch the week 6 video for the key items to edit. 
> For instance, you may edit it so that it reads in the data directly from the website.
> 
> Instruction from: https://jhustata.github.io/intermediate/chapter6.html#code-development
> 
> */

//di "What is your work directory?" _request(workdir)
//cd "$workdir"

//cd "C:\PUBLIC USE DATA"    // SET DIRECTORY WHERE DATA ARE LOCATED, E.G. "C:\PUBLIC USE DA
> TA"
 global SURVEY NHANES_1999_2000     // REPLACE <SURVEY> WITH RELEVANT SURVEY NAME (IN ALL CAP
> S)

//the above scripts deal with a local file
//i've edited to work remotely, as we've learned in stata II
global mort "https://ftp.cdc.gov/pub/HEALTH_STATISTICS/NCHS/datalinkage/linked_mortality/"

global repo "https://github.com/jhustata/project/raw/main/"

* example syntax:
* global SURVEY NHIS_2018
* or
* global SURVEY NHANES_2017_2018

clear all

**************
*NHIS VERSION*
**************
 
// DEFINE VALUE LABELS FOR REPORTS
 label define eligfmt 1 "Eligible" 2 "Under age 18, not available for public release" 3 "Inel
> igible" 

label define mortfmt 0 "Assumed alive" 1 "Assumed deceased" .z "Ineligible or under age 18"

label define flagfmt 0 "No - Condition not listed as a multiple cause of death" 1 "Yes - Con
> dition listed as a multiple cause of death" .z "Assumed alive, under age 18, ineligible for 
> mortality follow-up, or MCOD not available"

label define qtrfmt 1 "January-March" 2 "April-June" 3 "July-September" 4 "October-December"
z "Ineligible, under age 18, or assumed alive"

label define dodyfmt .z "Ineligible, under age 18, or assumed alive"

label define ucodfmt 1 "Diseases of heart (I00-I09, I11, I13, I20-I51)"                     
      

 label define ucodfmt 2 "Malignant neoplasms (C00-C97)"                                      
>       , add

 label define ucodfmt 3 "Chronic lower respiratory diseases (J40-J47)"                       
>       , add

 label define ucodfmt 4 "Accidents (unintentional injuries) (V01-X59, Y85-Y86)"              
>       , add

 label define ucodfmt 5 "Cerebrovascular diseases (I60-I69)"                                 
>       , add

label define ucodfmt 6 "Alzheimer's disease (G30)"                                          
>       , add

label define ucodfmt 7 "Diabetes mellitus (E10-E14)"                                        
>       , add

 label define ucodfmt 8 "Influenza and pneumonia (J09-J18)"                                  
>       , add

label define ucodfmt 9 "Nephritis, nephrotic syndrome and nephrosis (N00-N07, N17-N19, N25-N
> 27)"  , add

 label define ucodfmt 10 "All other causes (residual)"                                       
>       , add

label define ucodfmt .z "Ineligible, under age 18, assumed alive, or no cause of death data"
>       , add


// READ IN THE FIXED-WIDTH FORMAT ASCII PUBLIC-USE LMF
//infix str publicid 1-14 eligstat 15 mortstat 16 ucod_leading 17-19 diabetes 20 hyperten 21
>  dodqtr 22 dodyear 23-26 wgt_new 27-34 sa_wgt_new 35-42 using ${SURVEY}_MORT_2019_PUBLIC.dat
>         
 
//let read in the ASCII PUBLIC-USE from the CDC/NCHS website instead of downloading it local
> ly 
infix str publicid 1-14 eligstat 15 mortstat 16 ucod_leading ///
>     17-19 diabetes 20 hyperten 21 dodqtr 22 dodyear ///
>         23-26 wgt_new 27-34 sa_wgt_new 35-42 using ///   
>     "${mort}NHANES_1999_2000_MORT_2019_PUBLIC.dat"

 // DEFINE VARIABLE LABELS 
label var publicid "NHIS Public-ID Number"

label var eligstat "Eligibility Status for Mortality Follow-up"

label var mortstat "Final Mortality Status"

label var ucod_leading "Underlying Cause of Death: Recode"

label var diabetes "Diabetes flag from Multiple Cause of Death"

label var hyperten "Hypertension flag from Multiple Cause of Death"

label var dodqtr  "Quarter of Death"

label var dodyear "Year of Death"

label var wgt_new  "Weight Adjusted for Ineligible Respondents: Person-level Sample Weight"

label var sa_wgt_new  "Weight Adjusted for Ineligible Respondents: Sample Adult Sample Weigh
> t"

// ASSOCIATE VARIABLES WITH FORMAT VALUES 

Contains data
 Observations:         9,965                  
    Variables:            10                  
----------------------------------------------------------------------------------------------
Variable      Storage   Display    Value
    name         type    format    label      Variable label
----------------------------------------------------------------------------------------------
publicid        str14   %14s                  NHIS Public-ID Number
eligstat        byte    %46.0g     eligfmt    Eligibility Status for Mortality Follow-up
mortstat        byte    %26.0g     mortfmt    Final Mortality Status
ucod_leading    float   %71.0g     ucodfmt    Underlying Cause of Death: Recode
diabetes        byte    %86.0g     flagfmt    Diabetes flag from Multiple Cause of Death
hyperten        byte    %86.0g     flagfmt    Hypertension flag from Multiple Cause of Death
dodqtr          byte    %42.0g     qtrfmt     Quarter of Death
dodyear         float   %42.0g     dodyfmt    Year of Death
wgt_new         float   %9.0g                 Weight Adjusted for Ineligible Respondents:
                                                Person-level Sample Weight
sa_wgt_new      float   %9.0g                 Weight Adjusted for Ineligible Respondents:
                                                Sample Adult Sample Weight
----------------------------------------------------------------------------------------------
Sorted by: 
     Note: Dataset has changed since last saved.

// ONE-WAY FREQUENCIES (UNWEIGHTED)
tab1 eligstat mortstat ucod_leading diabetes hyperten dodqtr dodyear, missing

-> tabulation of eligstat  

       Eligibility Status for Mortality |
                              Follow-up |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
                               Eligible |      5,445       54.64       54.64
Under age 18, not available for public  |      4,517       45.33       99.97
                             Ineligible |          3        0.03      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of mortstat  

    Final Mortality Status |      Freq.     Percent        Cum.
---------------------------+-----------------------------------
             Assumed alive |      3,770       37.83       37.83
          Assumed deceased |      1,675       16.81       54.64
Ineligible or under age 18 |      4,520       45.36      100.00
---------------------------+-----------------------------------
                     Total |      9,965      100.00

-> tabulation of ucod_leading  

      Underlying Cause of Death: Recode |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
Diseases of heart (I00-I09, I11, I13, I |        468        4.70        4.70
          Malignant neoplasms (C00-C97) |        348        3.49        8.19
Chronic lower respiratory diseases (J40 |         94        0.94        9.13
Accidents (unintentional injuries) (V01 |         59        0.59        9.72
     Cerebrovascular diseases (I60-I69) |         95        0.95       10.68
              Alzheimer's disease (G30) |         56        0.56       11.24
            Diabetes mellitus (E10-E14) |         74        0.74       11.98
      Influenza and pneumonia (J09-J18) |         45        0.45       12.43
Nephritis, nephrotic syndrome and nephr |         36        0.36       12.79
            All other causes (residual) |        399        4.00       16.80
Ineligible, under age 18, assumed alive |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of diabetes  

   Diabetes flag from Multiple Cause of |
                                  Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
No - Condition not listed as a multiple |      1,451       14.56       14.56
Yes - Condition listed as a multiple ca |        223        2.24       16.80
Assumed alive, under age 18, ineligible |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of hyperten  

  Hypertension flag from Multiple Cause |
                               of Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
No - Condition not listed as a multiple |      1,395       14.00       14.00
Yes - Condition listed as a multiple ca |        279        2.80       16.80
Assumed alive, under age 18, ineligible |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of dodqtr  

                       Quarter of Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
Ineligible, under age 18, or assumed al |      9,965      100.00      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of dodyear  

                          Year of Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
Ineligible, under age 18, or assumed al |      9,965      100.00      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

 // SAVE DATA FILE IN DIRECTORY DESIGNATED AT TOP OF PROGRAM AS **SURVEY**_PUF.DTA
 // replace option allows Stata to overwrite an existing .dta file

> - I briefly changed the line below to "save demo/${SURVEY}_PUF, replace"
> - Then discovered that students were using MY followup.do instead of their own!!!
save ${SURVEY}_PUF , replace
(file NHANES_1999_2000_PUF.dta not found)
file NHANES_1999_2000_PUF.dta saved

****************
*NHANES VERSION*
****************
clear all
// DEFINE VALUE LABELS FOR REPORTS
label define premiss .z "Missing"

# hyperten "Hypertension flag from Multiple Cause of Death"

Contains data
 Observations:         9,965                  
    Variables:             8                  
----------------------------------------------------------------------------------------------
Variable      Storage   Display    Value
    name         type    format    label      Variable label
----------------------------------------------------------------------------------------------
seqn            float   %9.0g                 NHANES Respondent Sequence Number
eligstat        byte    %46.0g     eligfmt    Eligibility Status for Mortality Follow-up
mortstat        byte    %26.0g     mortfmt    Final Mortality Status
ucod_leading    float   %71.0g     ucodfmt    Underlying Cause of Death: Recode
diabetes        byte    %86.0g     flagfmt    Diabetes flag from Multiple Cause of Death
hyperten        byte    %86.0g     flagfmt    Hypertension flag from Multiple Cause of Death
permth_int      float   %9.0g      premiss    Person-Months of Follow-up from NHANES Interview
                                                date
permth_exm      float   %9.0g      premiss    Person-Months of Follow-up from NHANES Mobile
                                                Examination Center (MEC) Date
----------------------------------------------------------------------------------------------
Sorted by: 
     Note: Dataset has changed since last saved.


-> tabulation of eligstat  

       Eligibility Status for Mortality |
                              Follow-up |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
                               Eligible |      5,445       54.64       54.64
Under age 18, not available for public  |      4,517       45.33       99.97
                             Ineligible |          3        0.03      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of mortstat  

    Final Mortality Status |      Freq.     Percent        Cum.
---------------------------+-----------------------------------
             Assumed alive |      3,770       37.83       37.83
          Assumed deceased |      1,675       16.81       54.64
Ineligible or under age 18 |      4,520       45.36      100.00
---------------------------+-----------------------------------
                     Total |      9,965      100.00

-> tabulation of ucod_leading  

      Underlying Cause of Death: Recode |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
Diseases of heart (I00-I09, I11, I13, I |        468        4.70        4.70
          Malignant neoplasms (C00-C97) |        348        3.49        8.19
Chronic lower respiratory diseases (J40 |         94        0.94        9.13
Accidents (unintentional injuries) (V01 |         59        0.59        9.72
     Cerebrovascular diseases (I60-I69) |         95        0.95       10.68
              Alzheimer's disease (G30) |         56        0.56       11.24
            Diabetes mellitus (E10-E14) |         74        0.74       11.98
      Influenza and pneumonia (J09-J18) |         45        0.45       12.43
Nephritis, nephrotic syndrome and nephr |         36        0.36       12.79
            All other causes (residual) |        399        4.00       16.80
Ineligible, under age 18, assumed alive |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of diabetes  

   Diabetes flag from Multiple Cause of |
                                  Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
No - Condition not listed as a multiple |      1,451       14.56       14.56
Yes - Condition listed as a multiple ca |        223        2.24       16.80
Assumed alive, under age 18, ineligible |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

-> tabulation of hyperten  

  Hypertension flag from Multiple Cause |
                               of Death |      Freq.     Percent        Cum.
----------------------------------------+-----------------------------------
No - Condition not listed as a multiple |      1,395       14.00       14.00
Yes - Condition listed as a multiple ca |        279        2.80       16.80
Assumed alive, under age 18, ineligible |      8,291       83.20      100.00
----------------------------------------+-----------------------------------
                                  Total |      9,965      100.00

tab permth_int if permth_int==.z, missing

Person-Mont |
      hs of |
  Follow-up |
from NHANES |
  Interview |
       date |      Freq.     Percent        Cum.
------------+-----------------------------------
    Missing |      4,520      100.00      100.00
------------+-----------------------------------
      Total |      4,520      100.00

tab permth_exm if permth_exm==.z, missing

Person-Mont |
      hs of |
  Follow-up |
from NHANES |
     Mobile |
Examination |
     Center |
 (MEC) Date |      Freq.     Percent        Cum.
------------+-----------------------------------
    Missing |      4,992      100.00      100.00
------------+-----------------------------------
      Total |      4,992      100.00


// SAVE DATA FILE IN DIRECTORY DESIGNATED AT TOP OF PROGRAM AS **SURVEY**_PUF.DTA
// replace option allows Stata to overwrite an existing .dta file
/*
> - I briefly changed the line below to "save demo/${SURVEY}_PUF, replace"
> - Then discovered that students were using MY followup.do instead of their own!!!
save ${SURVEY}_PUF, replace
file NHANES_1999_2000_PUF.dta saved

save followup, replace 
(file followup.dta not found)
file followup.dta saved
import sasxport5 "${nhanes}1999-2000/DEMO.XPT", clear
merge 1:1 seqn using followup, nogen

    Result                      Number of obs
    -----------------------------------------
    Not matched                             0
    Matched                             9,965  
    -----------------------------------------


 //3. parameters
import sasxport5 "${nhanes}1999-2000/HUQ.XPT", clear

tab huq010 

    General |
     health |
  condition |      Freq.     Percent        Cum.
------------+-----------------------------------
          1 |      3,002       30.14       30.14
          2 |      2,464       24.74       54.88
          3 |      2,867       28.79       83.66
          4 |      1,306       13.11       96.78
          5 |        316        3.17       99.95
          9 |          5        0.05      100.00
------------+-----------------------------------
      Total |      9,960      100.00

merge 1:1 seqn using survey_followup, nogen keep(matched)

    Result                      Number of obs
    -----------------------------------------
    Not matched                             0
    Matched                             9,965  
    -----------------------------------------

rm followup.dta

rm survey_followup.dta

Survival-time data settings

         Failure event: mortstat!=0 & mortstat<.
Observed time interval: (0, years]
     Exit on or before: failure

--------------------------------------------------------------------------
      9,965  total observations
      4,520  event time missing (years>=.)                  PROBABLE ERROR
          2  observations end on or before enter()
--------------------------------------------------------------------------
      5,443  observations remaining, representing
      1,673  failures in single-record/single-failure data
  91,800.25  total analysis time at risk and under observation
                                                At risk from t =         0
                                     Earliest observed entry t =         0
                                          Last observed exit t =  20.83333

replace huq010=. if huq010==9
(5 real changes made, 5 to missing)
label define huq 1 "Excellent" 2 "Very Good" 3 "Good" 4 "Fair" 5 "Poor"
label values huq010 huq 
levelsof huq010, local(numlevels)
1 2 3 4 5
local i=1
foreach l of numlist `numlevels' {
  2.     local vallab: value label huq010 
  3.         local catlab: lab `vallab' `l'
  4.         global legend`i' = "`catlab'"
  5.         local i= `i' + 1
  6. }
 save week7, replace 
(file week7.dta not found)
file week7.dta saved

sts graph, ///
>     by(huq010) ///
>         fail ///
>         per(100) ///
>         ylab(0(20)80 , ///
>             format(%2.0f) ///
>         ) ///
>         xlab(0(5)20) ///
>         tmax(20) ///
>         ti("Self-Reported Health and Mortality") ///
>         legend( ///
>             order(5 4 3 2 1) ///
>                 lab(1 "$legend1") ///
>                 lab(2 "$legend2") ///
>                 lab(3 "$legend3") ///
>                 lab(4 "$legend4") ///
>                 lab(5 "$legend5") ///
>                 ring(0) pos(11) ///
>         )

        Failure _d: mortstat
  Analysis time _t: years

Iteration 0:   log likelihood = -14053.711
Iteration 1:   log likelihood = -13842.257
Iteration 2:   log likelihood = -13815.469
Iteration 3:   log likelihood = -13815.219
Iteration 4:   log likelihood = -13815.219
Refining estimates:
Iteration 0:   log likelihood = -13815.219

Cox regression with Breslow method for ties

No. of subjects =       5,436                           Number of obs =  5,436
No. of failures =       1,671
Time at risk    = 91,679.7499
                                                        LR chi2(4)    = 476.98
Log likelihood = -13815.219                             Prob > chi2   = 0.0000

------------------------------------------------------------------------------
          _t | Haz. ratio   Std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
      huq010 |
  Very Good  |   1.347475   .1251746     3.21   0.001     1.123176    1.616567
       Good  |   1.880346   .1619641     7.33   0.000     1.588251    2.226159
       Fair  |   2.985347   .2634849    12.39   0.000     2.511125    3.549126
       Poor  |   7.475088   .7565146    19.88   0.000     6.130144    9.115111
------------------------------------------------------------------------------

matrix define mat = r(table)

matrix list mat 

mat[9,5]
               1b.         2.         3.         4.         5.
           huq010     huq010     huq010     huq010     huq010
     b          1  1.3474753  1.8803457  2.9853475  7.4750881
    se          .  .12517462  .16196412  .26348489  .75651458
     z          .  3.2104048  7.3309743  12.392068  19.876295
pvalue          .  .00132548  2.285e-13  2.885e-35  6.528e-88
    ll          .  1.1231762  1.5882512  2.5111252  6.1301437
    ul          .  1.6165672   2.226159  3.5491258  9.1151112
    df          .          .          .          .          .
  crit   1.959964   1.959964   1.959964   1.959964   1.959964
 eform          1          1          1          1          1

matrix mat = mat'

svmat mat

preserve 

keep mat*

graph export semipara_unadj.png, replace 
file /Users/kiyoi/Desktop/stata/stata II/Assignment 5/semipara_unadj.png saved as PNG format

graph save semipara_unadj.gph, replace 
(file semipara_unadj.gph not found)
file semipara_unadj.gph saved

Iteration 0:   log likelihood = -14053.711
Iteration 1:   log likelihood = -12507.535
Iteration 2:   log likelihood = -12336.034
Iteration 3:   log likelihood = -12330.399
Iteration 4:   log likelihood = -12330.368
Iteration 5:   log likelihood = -12330.368
Refining estimates:
Iteration 0:   log likelihood = -12330.368

Cox regression with Breslow method for ties

No. of subjects =       5,436                          Number of obs =   5,436
No. of failures =       1,671
Time at risk    = 91,679.7499
                                                       LR chi2(6)    = 3446.69
Log likelihood = -12330.368                            Prob > chi2   =  0.0000

------------------------------------------------------------------------------
          _t | Haz. ratio   Std. err.      z    P>|z|     [95% conf. interval]
-------------+----------------------------------------------------------------
      huq010 |
  Very Good  |    1.14715   .1066937     1.48   0.140     .9559872    1.376539
       Good  |   1.522007   .1313003     4.87   0.000     1.285243    1.802387
       Fair  |   1.777098    .157479     6.49   0.000     1.493762    2.114177
       Poor  |   3.169804   .3213622    11.38   0.000     2.598577      3.8666
             |
    ridageyr |   1.089492   .0020788    44.92   0.000     1.085425    1.093574
    riagendr |   .7039323   .0347043    -7.12   0.000     .6390961    .7753461
------------------------------------------------------------------------------

. return list

scalars:
              r(level) =  95

matrices:
              r(table) :  9 x 7

matrix define mat_adj=r(table)

matrix define mat_adj=mat_adj'

matrix list mat_adj

mat_adj[7,9]
                    b          se           z      pvalue          ll          ul          df
1b.huq010           1           .           .           .           .           .           .
 2.huq010   1.1471502   .10669366   1.4760168   .13993937    .9559872   1.3765388           .
 3.huq010   1.5220069   .13130029    4.868902   1.122e-06   1.2852431   1.8023867           .
 4.huq010   1.7770978   .15747898   6.4884757   8.671e-11    1.493762   2.1141766           .
 5.huq010   3.1698039   .32136218   11.379393   5.297e-30   2.5985769   3.8665996           .
 ridageyr   1.0894921   .00207879   44.921463           0   1.0854254   1.0935741           .
 riagendr   .70393228   .03470427  -7.1210742   1.071e-12   .63909607   .77534611           .

                 crit       eform
1b.huq010    1.959964           1
 2.huq010    1.959964           1
 3.huq010    1.959964           1
 4.huq010    1.959964           1
 5.huq010    1.959964           1
 ridageyr    1.959964           1
 riagendr    1.959964           1

svmat mat_adj

keep mat_adj*

e(0, lcol(lime)) ///

graph export semipara_adj.png, replace 
file /Users/kiyoi/Desktop/stata/stata II/Assignment 5/semipara_adj.png saved as PNG format

 graph save semipara_adj.gph, replace 
(file semipara_adj.gph not found)
file semipara_adj.gph saved

 
 closed on:  16 May 2024, 23:20:10
----------------------------------------------------------------------------------------------
