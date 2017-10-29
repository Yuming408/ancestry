# Ancestry data challenge
1. SQL
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

2. Experiment design
- To set up the experiment, first you need to see what is the traffic looks like on your website, how many samples that are available. Then you decide how you want to run your test, it could be through marketing campaign or homepage modification depending on your budget and implementation difficuties. Then your want to split your samples into control and treatment group. Make sure it is a randomly spliting between control and treatment and trying to eliminate all possible confounding factors. For control group, there is no modification; for treatment group, you want to add message about your new product through marketing campaign or homepage. You should define the metrics you want to track to evaluate your test
- To determine and sample size, you need to define what is the significant level, power, base line conversation rate for the metrics you choose, practical significant level. Use sample size/traffic to decide how long you want to run your experiment. Depends on how the test will be done, for example, if you choose to run the experiment through email campaign, the metrics will be email open rate, pruchase rate; If you choose to change the homepage, you can use click through rate, purchase rate
- I will construct the confidence interval to analyze the result. The reason I am choosing confidence interval over p value method is because as your sample size gets larger, you are more likely to a smaller p value and conclude a statistically significant result. However, you need to decide if it is practically useful to launch this product. So sometimes, you need to decide what is the practical significant level looks like. Use the confidence interval approach, you can compare your results and see if it is both statistically significant and practically useful.
- I will use the below figure to show use case, dmin is the practical significant level, the figure is from the [Udacity AB test class] (https://www.udacity.com/course/ab-testing--ud257)
For case 1, you should launch the product since it is practically significant
For case 2, you should not launch the product
For case 3, you should not launch the product, even if it is statistically significant, but not practically useful
For case 4- 6: more tests are needed to improve the power

![alt text](https://github.com/Yuming408/ancestry/blob/master/Screen%20Shot%202017-10-27%20at%203.49.53%20PM.png "ab test")

3 Case study
- For data cleaning and feature engineering, first 
create label for response variable: 0 if not a xseller, 1 is a xseller
- To predict the Xseller, weeks between DNA test is activated and DNA order is created are generated and binned into 12 levels(1 week, 2 weeks.....> 10 weeks).  If never activate the test, it will return -1. 
- The overall xseller rate is about *12%*
- The next step is to explore relationships between predict variables and response variable. First let's take a look at how customer type affect whether or not a Xseller

|customer_type_group| Xseller rate|
|------------------ |------------:|
| Acom Sub     | 3%|
|Existing Reg|18%|
|New Reg|15%|

- It seems cusomters who are existing registrant have a much higher chance to become Xseller for Acom subscriptions.
- Next I am looking at the relationship between week difference of dateDNA test is activated and date DNA order is created and the whether or not a Xseller.
- It shows customers who activate the test within a week after they place the order have higher chance to become Xseller. In other words, customers who react fast after their DNA kit purchase are more likely to become Xseller




