# Ancestry data challenge
1. *SQL*
```
select DNAtest_date, sum(count(testgid)) over (order by DNAtest_date) as cum_ct
from DNA_purchase where DNAtest_date >= '2017-01-01' and DNAtest_date <= '2017-01-31'
group by DNAtest_date
```

```
select country, age_group, lastname, firstname from(
select *, rank() over (partition by country, age_group order by ct) as rk
from(
select country, age_group, lastname, firstname, count(*) as ct
from DNA_purchase a
inner join
users b
on a.testgid = b.testgid
where Acom_subscriber = 1
and DNAtest_date >= '2017-01-01'
and DNAtest_date <= '2017-05-24'
group by 1,2,3,4) inq) inq_1
where rk = 1
```

2. *Experiment design*
- To set up the experiment, first you need to see what is the traffic looks like on your website, how many samples that are available. Then you decide how you want to run your test, it could be through marketing campaign or homepage modification depending on your budget and implementation difficulties. Then your want to split your samples into control and treatment group. Make sure it is a randomly splitting between control and treatment and trying to eliminate all possible confounding factors. For control group, there is no modification; for treatment group, you want to add message about your new product through marketing campaign or homepage. You should define the metrics you want to track to evaluate your test
- To determine and sample size, you need to define what is the significant level, power, base line conversation rate for the metrics you choose, practical significant level. Use sample size/traffic to decide how long you want to run your experiment. Depends on how the test will be done, for example, if you choose to run the experiment through email campaign, the metrics will be email open rate, purchase rate; If you choose to change the homepage, you can use click through rate, purchase rate
- I will construct the confidence interval to analyze the result. The reason I am choosing confidence interval over p value method is because as your sample size gets larger, you are more likely to get a smaller p value and conclude a statistically significant result. However, you need to decide if it is practically useful to launch this product. So sometimes, you need to decide what is the practical significant level looks like. Use the confidence interval approach, you can compare your results and see if it is both statistically significant and practically useful.
- I will use the below figure to show use case, dmin is the practical significant level, the figure is from the 
[Udacity AB test class](https://www.udacity.com/course/ab-testing--ud257) For case 1, you should launch the product since it is practically significant. For case 2, you should not launch the product For case 3, you should not launch the product, even if it is statistically significant, but not practically useful. For case 4- 6: more tests are needed to improve the power

![alt text](https://github.com/Yuming408/ancestry/blob/master/Screen%20Shot%202017-10-27%20at%203.49.53%20PM.png "ab test")

3 *Case study*
- For data cleaning and feature engineering, first 
create label for response variable: 0 if not a xseller, 1 is a xseller
- To predict the Xseller, weeks between DNA test is activated and DNA order is created are generated and binned into 12 levels(1 week, 2 weeks.....> 10 weeks).  If activate date is missing, it will return -1. 
- The overall xseller rate is about *12%*
- The next step is to explore relationships between predict variables and response variable. First let's take a look at how customer type affect whether or not an Xseller

|customer_type_group| Xseller_rate|
|------------------ |------------:|
| Acom Sub     | 3%|
|Existing Reg|18%|
|New Reg|15%|

![alt text](https://github.com/Yuming408/ancestry/blob/master/Screen%20Shot%202017-10-28%20at%2010.12.10%20PM.png)

- It seems customers who are existing registrant have a higher chance to become Xseller for Acom subscriptions.
- Next I am looking at if days difference of DNA test is activated and DNA order is created affects whether or not an Xseller.

|week_diff|Xseller_rate|
|---------|-----------:|
|-1|	5%|
|1 week|16%|
|2 weeks|14%|
|3 weeks|15%|
|4 weeks|14%|
|5 weeks|14%|
|6 weeks|13%|
|7 weeks|14%|
|8 weeks|13%|
|9 weeks|12%|
|10 weeks|12%|
|>10 weeks|10%|

![alt text](https://github.com/Yuming408/ancestry/blob/master/week_diff.png)

- It shows customers who activate the test within a week after they place the order have higher chance to become Xseller. In other words, customers who react fast after their DNA kit purchase are more likely to become Xseller

- I am also interested to see if days takes the result ready affect the Xseller conversion. Using the same approach. We can see the only significant difference is whether or not the result is ready. From our previous analysis, we know some users do not activate their DNA test so they never get their result. 

|daystogetresult_grp|Xseller_rate|
|-------------------|------------|
|-1|7%|
|1 week|16%|
|2 weeks|16%|
|3 weeks|15%|
|4 weeks|15%|
|5 weeks|17%|
|6 weeks|17%|
|7 weeks|16%|
|8 weeks|15%|
|9 weeks|16%|
|10 weeks|17%|
|>10 weeks|15%|


- Predictive model: 

To further understand how we can leverage these data to make recommendations for cross sell, I build a model to predict which users are more likely to become Xseller. Since all of the variables are categorical and have some noise, I used random
forest to predict whether or not this user is an Xseller. Random forest is a tree based ensemble
method. It can handle categorical features very well. Random forest is robust to noise data and preventing
the model from over fitting. For this problem, I used H2O random forest library to speed up the model
training. Alternatively, logistic regression or SVM can be used for this problem. But logistic regression is
more appropriate for numerical variables and SVM can be inefficient to train. Since random forest produced
a good result, so I did not try other models.

The variables I use to predict the results are: customer_type_group, week_diff, dna_visittrafficsubtype, regtenure. daystogetresult_grp is not chosen because from previous anaysis, if the user never activate the DNA test, they will not get their test result, if they do get their results, how long the days take does not seem to be an important factor.

Use 3 fold cross validation method, the model produce AUC of 0.74. It might not be an ideal result in terms of model performance, however we can still identify feature importance. Here is the feature importance:


|variable|	relative_importance|	scaled_importance	percentage|
|--------|---------------------|------------------------------|
|customer_type_group|1.0|	0.6170893|
|week_diff|0.5014742|	0.3094543|
|dna_visittrafficsubtype|0.0752522|	0.0464373|
|regtenure|0.0437847|	0.0270191|

From the model, we can see customer type and week difference to activate the test are the most important features, which confirms with the previous finding.

- *Recommendation*:

1. Existing registrant have a higher chance to become Xseller than other types of customers, they also contribute the highest population compare to other groups. when we trying to allocate marketing campaign effort, probably we can target more existing users.

2. New registrant also consists an important component in terms of population, we can think of ways to improve their experience with Ancestry product. Make sure to identify any friction for new customers when they are using Ancestry product.

3. The faster a user activates their test the more likely they will convert to Xseller. After a user place an DNA order, we can remind them to activate their test as soon as possible and educate them on what are the benefits of activate the test. About 75k users do not activate their test and lose the chance to experience Ancestry DNA product, they might forget or not sure what to do to activate the test. 








