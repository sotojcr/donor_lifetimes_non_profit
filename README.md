# Donor lifetimes and values for a non-profit

Overview
========
Background
----------

- Only 20% of revenue of an average non-profit comes from donations (most of the rest of government contacts)
- There is often no-coherent strategy for recruiting or retaining donors.  Traditional methods range from holding expensive special events to personal connections.  Often this is done by hunches and without any analytical support.
- Questions to be answered:
	1.  What are the good predictors of donor lifetimes?  More importantly, how do these predictors influence them.
	2.  How to extend donor lifetimes?
	3.	What are the features of a good or bad donor?  (Determined by calculating the Donor lifetime values)

Data
----
- Donor database from South Bay non-profit. 2338 donors spanning over 40 years.
- 100 features.  Examples: ZIP code, martial status, gender...


Modeling
========
Approach
--------
- The Aalen Additive model, which is patient-survival model, was used to predict the lifetime. The model fits the data to a *hazard function* in turn is used to predict the lifetime.

- The hazard function are features which can influence the lifetime of the patient, which is the donor in our use case.

	λ(t) = b<sub>0</sub>(t) + b<sub>1</sub>(t)x<sub>1</sub> +. . . +b<sub>N</sub>(t)x<sub>T</sub>


- Why not a multi-decision tree model?  Because a patient-survival model is more interpretable.

- Why not linear or logistic regression?  Because the residuals are not uniform over time.

Data processing
---------------
- Data package "Lifelines" which contains the Aalen additive model

Bootstrapping
-------------
- The more than 100 features were whittled down to 76 based on the p values.  (Details not repeated here).  
	1.	ACGC (whether a donor comes from the ACGC branch of the organization)
	2.	COS breakfast (A $100/plate breakfast event with a celebrity speaker)
	3.	Golf tournament (A golf tournament held at a country club that cost $1000 to play)
	4.	Male (True = male, False = female)
	5.	Married (True = married or partnered, False = single, divorced, or widowed)
	6. 	Personal connection (True =  a personal friend or acquaintance of an employee of the agency.)
	7.  Predicted income (Using the zip code, we scrapped the US Census Bureau website for the average household income and use this as the feature.)
- Because of the relative small data set, a bootstrap approach was used to model the data.   Samples were drawn with replacement from the original data set 10 000 times to train 10 000 models.  These 10 000 models generated 10 000 hazard functions.  

Results
=======
1.	Hazards
	-------
The plots below show the hazard functions generated by the bootstrapped models.  The variance of our model can be nicely visualized by the spread of lines.   
-  The baseline hazard function represents all the hazards not accounted for by the hazards.  That is, it is the hazard function if all the categorical features were false and all the numerical features were at their average values.  

![Hazard function baseline](images/cum_haz_baseline0-5.png)

- The hazard function for each hazard were plotted below. Each function shows at what time in a donor's lifetime hazard will emerge.  Note that even though these functions are called "hazards," when they are negative, they are actually "benefits" that prolong the lifetime of the donor.  For example, a higher than average income would produce a negative hazard over much of the lifetime of a donor.

![Hazard function for ACGC](/images/cum_haz_ACGC0-5.png>)
![Hazard function for Celebrity breakfast](images/cum_haz_COS0-5.png)
![Hazard function for Golf tournatments](/images/cum_haz_golf0-5.png)
![Hazard function for Male](/images/cum_haz_male0-5.png)
![Hazard function for Married](/images/cum_haz_married0-5.png)
![Hazard function for Personal connection](/images/cum_haz_personal_connection0-5.png)
![Hazard function for Predicted income](/images/cum_haz_pred_income0-5.png)

2.	Predicting Hazards for individual donors
	----------------------------------------

- For any given donor (old or new), we can substitute his features into the model to predict the lifetime.  More importantly, we can customize a hazard function for each donor and predict at what times in her or his lifetime hazards will emerge (represented by steep slopes in the hazard function).  This is illustrated below by the plots of the hazard function 30 random donors, each with a different profile of features.   

![CumulativeHazfor15donors](/images/Cum_haz_15donors.png)

3.	Donor lifetime values
---------------------
- One important question that needs to be answered and what the donor lifetime values (DLV) are for donors with different characteristics.   We focus on three features: Golf, Celebrity breakfast, and Married.  We compare the DLV for those whose feature is True and False.  

	DLV = *Donor Lifetime* x (*annual donation<sub>avg</sub>* - *annual recruitment and maintenance costs*)

*Recruitment/maintenance costs* are the sum of
1.	One-time cost of recruitment (e.g., the cost of the golf event amortized for each successful donor recruited at the event)
2.	Recurring costs (e.g., periodic mailers, maintenance of donor database)

The results are plotted in the figures below.

![DLV violin plots](/images/DLV.png)

**Conclusions**
===============
1.	Donors recruited at *golf tournaments* have negative DLV of about -$1000.  This is not surprising because it cost $50 000 to hold each golf tounatment and the hazard function for golf is extremely high even after two years.

2.	Donors who are recruited at the *Celebrity breakfast* have higher DLV than those who are not (the rest), event though the event cost $30 000 to hold.   This is because each event generated more donors and they each donate more money than the golf tournaments.

3.  Unmarried donors have higher DLV than married donors, even though the hazard function for married is negative (indicating being married is beneficial).  This is also because unmarried donors tend to donor more than married donors.
