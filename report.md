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

