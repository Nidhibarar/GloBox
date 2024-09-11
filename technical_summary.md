# A/B Test Report: Food and Drink Banner

## Purpose
Briefly describe the goal of the A/B test.

The purpose of conducting A/B test is to assess the impact of introducing highlight key products in the food and drink category as a banner at the top of the website. The control group does not see the banner, and the test group sees it

The purpose of conducting A/B test is to assess the impact of introducing food and drink banner on the homepage with a goal to check if there is an increament in the sale of food and drink. The primary metric of interest is the conversion rate. The experiment is designed to assess whether the banner significantly influences user behavior and contributes to increased sales in the food and drink category.

## Hypotheses
What are the null and alternative hypotheses we are testing?
Two Hypotheses Tests are conducted.

## Hypotheses for Conversion Rate:
Null Hypothesis (H0):
There is no significant difference in the conversion rate between the control group (group A) and the treatment group (group B) when exposed to the food and drink banner.

Alternative Hypothesis (H1):
There is a significant difference in the conversion rate between the control group (group A) and the treatment group (group B) when exposed to the food and drink banner.

Statistical Notation:
H0: p1 = p2 
H1: p1 != p2

Here, p1 is the population in the control group and p2 is the population in the treatment group.

## Hypotheses for Average Amount Spent per User:
Null Hypothesis (H0):
There is no significant difference in the average amount spent per user between the control group (group A) and the treatment group (group B) when exposed to the food and drink banner.

Alternative Hypothesis (H1):
There is a significant difference in the average amount spent per user between the control group (group A) and the treatment group (group B) when exposed to the food and drink banner.

Statistical Notation:
H0: μ1 = μ2, 
H1: μ1 != μ2

Here, μ1 is the population in the control group and μ2 is the population in the treatment group.

## Methodology
Average Amount Spent:

Test Used: Independent samples t-test.
Rationale: To compare mean spending between control (Group A) and treatment (Group B).
Procedure: Utilized the provided sample mean 
(x1ˉ1=3.37xˉ1=3.37, x2ˉ2=3.39xˉ2=3.39) 
standard deviations (s1=25.94s1=25.94, s2=25.41s2=25.41), and sample sizes (n1=24343n1=24343, n2=24600n2=24600) to calculate the t-statistic (T=−0.070T=−0.070).
Significance Level: α = 0.05.


### Test Design
- **Population:** Describe the test and control groups.
- **Duration:** Test start and end dates.
- **Success Metrics:** What metrics are used to measure success?

Test Group (Group B - Treatment):

This group consists of users who are exposed to the food and drink banner at the top of the GloBox mobile website.
Users in this group see the banner highlighting key products in the food and drink category when they visit the main page.
The purpose is to assess the impact of the banner on user behavior, specifically focusing on conversion rates and potentially the average amount spent per user.
Control Group (Group A - Control):

This group serves as the baseline or reference group for comparison.
Users in this group do not see the food and drink banner when they visit the GloBox main page.
The behavior of this group is used as a benchmark to evaluate whether the introduction of the banner in the test group has a significant impact on user engagement and conversion rates.
In summary, the test group experiences the experimental condition (exposure to the food and drink banner), while the control group represents the standard or unchanged condition (no exposure to the banner). By comparing the outcomes between these two groups, the A/B test aims to identify any statistically significant differences in user behavior and conversion metrics.


## Results
### Data Analysis
- **Pre-Processing Steps:** Include any SQL queries and describe data cleaning steps.

```sql

select u.id, sum(spent), g.group, a.device, U.gender,u.country,
CASE 
WHEN SUM(a.spent) is null
THEN 0
ELSE 1
END AS conversions
FROM users u
left join activity a
on u.id = a.uid
left join groups g
on u.id = g.uid
group by u.id, g.group, a.device

Data cleaning steps -
--2 type of join should we use to join the users table to the activity table
SELECT *
FROM users
LEFT JOIN activity
ON users.id = activity.uid

--3 SQL function can we use to fill in NULL values
SELECT Max(join_dt)
FROM groups

--4 the start and end dates of the experiment
select MIN(join_dt)
from groups 
;
select MAX(join_dt)
from groups 

--5 total users were in the experiment
select count(id)
from users

--6 How many users were in the control and treatment groups
select count(id)
, groups.group
from users
left join groups
ON users.id = groups.uid
group by groups.group

--7 conversion rate of all users
WITH total_users AS(
select id	
,
CASE 
WHEN SUM(activity.spent) is null
THEN 0
ELSE 1
END AS conversions
from users 
left join activity
ON users.id = activity.uid
group by id
)

SELECT sum(conversions)/ count(distinct(id))::decimal * 100
FROM total_users
--Select round((select count(distinct(uid)) from activity)::decimal/
(select count(distinct(id)) from users u) *100, 2)

--8 average amount spent per user for the control and treatment groups, including users who did not convert
WITH total_users AS(
select id,
CASE 
WHEN SUM(activity.spent) is null
THEN 0
ELSE 1
END AS conversions
from users 
left join activity
ON users.id = activity.uid
group by id
)
SELECT g.group 
, sum(conversions)/ count(distinct(id))::decimal * 100
FROM total_users
left join groups g
on total_users.id = g.uid
group by g.group


--What is the average amount spent per user for the control and treatment groups, 
--including users who did not convert?

with total_users as(
select u.id 
,coalesce(SUM(spent), 0) spent     
from users u
left join activity a
on u.id = a.uid
group by u.id
)
select g.group, avg(spent)
from groups g
left join total_users t
on t.id = g.uid
group by g.group

or

with total_users as(
select id,
CASE 
WHEN SUM(a.spent) is null
THEN 0
ELSE SUM(a.spent)
END AS total_spent --AVG(coalesce(spent, 0)) as spent, g.group
from users u 
left join activity a
on u.id = a.uid)
--left join groups g

select avg(total_spent), g.group from total_users t
inner join groups g
on t.id = g.uid
group by g.group


```

- **Statistical Tests Used:** Specify tests (e.g., t-test, chi-squared).
- **Results Overview:** High-level summary.

Average Amount Spent:
Test: Independent samples t-test.
Explanation: Assesses if the mean spending in the treatment group significantly differs from the control group mean.
Conversion Rate: 
Test: Two-sample z-interval test with pooled proportion.
Explanation: Determines if the difference in conversion rates between the groups is statistically significant.

Why These Tests:
t-test: Ideal for comparing means in small to moderate-sized samples.
z-interval test: Suitable for assessing proportions and well-suited for binary outcomes like user conversion.
Validity Measures: Tests chosen for appropriateness to data types and ensured reliability with predefined significance levels (α).

The independent samples t-test yielded a non-significant result (p = 0.9438).
Implication: No statistically significant difference in the mean spending between the control and treatment groups.

The two-sample z-interval test demonstrated a significant increase in the treatment group (p = 0.0001).
Implication: The new webpage design significantly influences user engagement and conversion rates.
Key Insights:

Results indicate varied impacts on spending and engagement metrics.
A cautious, iterative approach is advised, necessitating further testing before broad implementation.

### Findings

## Interpretation
- **Outcome of the Test(s):** Did it support the hypothesis?
- **Confidence Level:** State the statistical confidence

## Conclusions
- **Key Takeaways:** What did we learn?
- **Limitations/Considerations:** Any potential biases or anomalies.

Spending Behaviour: No significant change observed in the average amount spent per user with the new design.
User Engagement: Marked improvement in user engagement, evident in a significant increase in conversion rates.
Recommendation: Cautious optimism is advised, emphasizing the need for further testing before widespread implementation.
Insights for Action:
Prioritize user engagement elements for sustained positive impact.
Plan for iterative testing, allowing for adjustments that resonate with user behavior and preferences.


Possoble limitations -
Sample Size Variability: Limited dataset might not capture diverse user behaviors, influencing the generalizability of results.
Short-Term Observation: The test period may not capture long-term effects; user behaviors might evolve over time.
External Factors: Unaccounted external influences, like seasonality or marketing campaigns, may impact user behavior independently.
Sensitivity to Design Changes: User response may vary based on the nature and extent of design alterations, influencing test outcomes.

## Recommendations
- **Next Steps:** Based on results, what should the business do next?
- **Further Analysis:** What are some ways to iterate on this analysis?

Explore Further: Dive deeper into user feedback and behavior to understand nuances.
Iterative Testing: Consider small tweaks to the webpage design and run additional tests for clearer insights.
User Feedback: Gather user opinions through surveys or feedback forms for a holistic perspective.
Visual:

Incorporate visuals like a magnifying glass (exploration), a gear (iteration), and speech bubbles (user feedback) to represent each point.