# Metrics Overview

### Why use quantifiable metrics?

-   Metrics help organizations systematically measure, assess, and improve performance

### Metric benefits and pitfalls

##### Benefits +

-   Quantifiable way to track business health and spot opportunities for improvement/experimentation
-   Drives organization alignment toward measurable target (often part of an OKR framework)
-   Used for comparison across time periods and/or segments
-   Input to data-informed decision making process
-   Standardize way to track performance
-   Online metric improvement hypotheses can often be tested via AB testing

##### Pitfalls -

-   Expecting a single metric to tell the full story
-   Over complicated metrics not understood by end users
-   Reliant on vanity metrics (e.g. always increasing metrics) that look good on paper but not tied to core business action
-   Easy to game goal metrics
-   Missing big picture and obsessing over small metric movements
-   Cherry picking metrics, time windows, and/or segments to prove a previously held opinion (confirmation bias)
-   Abandoning qual signals and over indexing on quant metrics due to ease of measurement
-   Overly sensitive or overly stable metrics that limit actionability
-   Not monitoring metric input data quality resulting in metric quality erosion over time

------------------------------------------------------------------------

# Business Metric Examples

##### Financial Metrics

-   **Sales**: total units or services sold (volume trends)
-   **Revenue**: income from all sources before deducting any costs (scales with business activity; nuances on how revenue recognition works for financial accounting)
-   **Cost of Goods Sold (COGS)**: direct variable costs of production (can scale up with more production)
-   **Gross Profit**: revenue minus COGS (assess efficiency of using resources)
-   **Gross Margin**: percentage of revenue that is gross profit (efficiency at converting revenue to profit)
-   **Burn Rate**: cash spent per time period (highlights cash flow sustainability)
-   **Runway**: the number of time periods (often months) a company can continue to operate at its current burn rate before running out of cash

##### Marketing Metrics

-   **Marketing Spend:** total spend on marketing across channels
-   **Customer Per Acquisition (CPA):** typical cost associated with acquiring a new customer
-   **Lifetime Value (LTV):** total profit one expects from customer relationship over time period
-   **Conversion Rate (CVR):** number of folks who purchased / number of folks who were exposed to offer
-   **LTV to CAC:** ratio of LTV to customer acquisition costs (used to assess marketing spend efficiency)

##### Product Usage & Growth

-   **Active Users:** number of unique users to interact with core value prop of product (often segmented as daily active users, weekly active users, monthly active users)
-   **Retention Rate:** percent of unique users who use the product period vs period (often segmented by user cohorts based on when the users joined the product)
-   **Feature Adoption:** percent of unique users who use a feature out of the total user base eligible to use the feature

##### Customer Satisfaction

-   **Net Promoter Score (NPS):** survey question on scale of 0 to 10 asking about willingness to recommend (NPS Score = % of 9 or 10 respondents - % of 6 or less respondents)
-   **Customer Satisfaction Score (CSAT):** measures how satisfied respondents are with a product and/or service (typically a survey question with a [Likert scale](https://en.wikipedia.org/wiki/Likert_scale) and response rate of top 1 or 2 answers used)

##### Operational Metrics

-   **Time to close new customer:** time duration between first contact with prospect till they become a paying customer
-   **Time to hire:** time duration between job posting and offer acceptance
-   **Employee turnover:** rate of folks leaving the company (number of employees leaving / average number of employees during time period)
-   **Employee DEI:** company representation across factors like race, gender, age, and more

------------------------------------------------------------------------

# Metric Frameworks

### AARRR (Pirate metrics)

-   **Overview**
    -   [popularized by Dave McClure (2007)](https://www.slideshare.net/dmc500hats/startup-metrics-for-pirates-long-version)
    -   a systematic way to optimize the customer funnel
    -   Dave suggests startups foucs on 5 to 10 conversion steps to measure and iterate on via AB tests (based on cost, conversion, and business revenue model)
    -   Dave recommends 80% effort deployed on existing feature optimization and 20% on new feature dev
    -   funnel metrics segmented by audience, channel, campaign theme, landing page & CTA, copy & graphics

1.  **Acquisition:** how users find out about the product
    -   channel optimization/prioritization based on volume, cost, efficiency
    -   example funnel metrics
        -   site visits
        -   engaged prospect rate (pages viewed \>= x, time spent \>= y, clicks \>= z)
2.  **Activation:** first happy experience and account creation
    -   getting folks to key feature usage and actions that signal future usage
    -   example funnel metrics
        -   account completion rate
        -   key feature usage rate
3.  **Retention:** repeat usage of the product
    -   marketing, content, features, processes, and offers to pull folks back to engage with the product
    -   example funnel metrics
        -   reactivation campaign CTR
        -   percent days engaged \>= x during first 30 days
4.  **Referral:** users like the product and tell others
    -   encourage users to refer after they have a happy experience
    -   example funnel metrics
        -   percent users who send invites \>= x
        -   percent users who have referral activate \>= x
5.  **Revenue:** users complete monetization actions
    -   aligns to how the business model captures value from customers
    -   example funnel metrics
        -   percent users who pay
        -   percent users who break-even

### Google HEART

-   **Overview**
    -   proposed by Googlers Kerry Rodden, Hilary Hutchinson, and Xin Fu in [2010 paper](https://research.google/pubs/measuring-the-user-experience-on-a-large-scale-user-centered-metrics-for-web-applications/)
    -   set of user-centered metrics to measure progress toward user experience goals
    -   metrics tracked for a given time period and/or cut by user segments/cohorts
    -   some metric categories may not be applicable for certain products/business use cases

1.  **Happiness**
    -   attitudinal metrics
    -   i.e. CSAT, NPS, and ease of use
2.  **Engagement**
    -   behavioral metrics on level of involvement with the product
    -   i.e. frequency, depth, breadth of interaction
    -   averages per user and rate of actives to do action are often good starting points
    -   engagement metrics that predict future product retention or revenue tend to be prioritized
3.  **Adoption**
    -   new users to start using a product
    -   expressed as absolute count, proportion of total users, proportion of active users, proportion of new/existing users, etc
    -   defining "using" is product/feature/business case specific
    -   i.e. accounts created last 7 days, number of users to turn on a feature, etc
4.  **Retention**
    -   percent of users who use the product during period X and then use it again during period Y
    -   new products tend to have evolving retention metrics vs mature products tend to have more stable retention metrics outside of seasonal/macro events
    -   cohort comparison is common here (i.e. cohorts based on when users first joined the product or adopted the product/feature)
    -   i.e. percent of 7 day actives this week who were also active last week, percent of cohort users who use the product month 2, month 3, etc
5.  **Task success**
    -   is the product UX/design intuitive and efficient at getting users to desired objective
    -   task metrics help pinpoint areas of opportunity to reduce friction and improve the user experience
    -   behavioral metrics assessing efficiency, effectiveness, error rates
    -   efficiency examples: time to complete task, clicks to complete task, steps complete per time window x
    -   effectiveness examples: step completion rate, rate of users who stick to optimal path, rate of users complete task without assistance
    -   error rates examples: page error rate, abandonment rate by sub task location, issues reported rate

### North Star Metric Framework

-   **Overview**
    -   inspired by Sean Ellis and the growth hacking movement
    -   a single metric to guide strategy, inform decision making, and set internal focus
    -   ideally as the north star metric improves business results improvement follows
    -   [Amplitude's North Star Playbook](https://amplitude.com/books/north-star/about-the-north-star-framework)
-   **Framework key elements**
    -   lagging business results/value: often revenue, continued paying customers, etc
    -   north star metric (NSM): captures the core value prop of the product and is a leading indicator of business value (i.e. critical rate, count, ratio, etc)
    -   input metrics: 3 to 5 sub-metrics that influence the north star metric
    -   work: activities that hope to move input metrics
-   **Example for subscription business**
    -   business result: renewing subscription revenue
    -   NSM: core value prop action that correlates with future sub revenue
    -   inputs: supporting tasks/actions that improve customer volume and frequency of core value prop action
    -   work: hypotheses to move and improve the input metrics
-   **NSM checklist**
    -   aligns to customer value exchange with the product
    -   represents the company's vision and product strategy
    -   leading indicator of critical business results
    -   measurable via product analytics tracking
    -   produces actionable set of sub-metrics teams can run experiments on
    -   simple to explain and understand for technical and non-technical team members
-   **Inputs**
    -   starting point ideas: breadth, depth, frequency, efficiency
    -   not too sensitive or too broad
    -   teams can generate large volume of actionable ideas to move the inputs

### Meta Product Market Fit

-   **Overview**
    -   [Meta Analytics PMF Playbook](https://medium.com/@AnalyticsAtMeta/analytics-and-product-market-fit-11efaea403cd)
    -   product market fit (PMF): the value a product delivers for a specific market segment
    -   in other words, are we building something people want? are folks using the product? do folks come back to use it?
    -   valuable products with high product market fit: keep people coming back, sustainability acquire new users, and have long stretches of usage
-   **PMF measurement criteria**
    -   stable retention: do folks keep coming back after using the product?
    -   sustainable growth: is there a healthy (ROI efficient) approach to acquire, retain, and resurrect folks over time?
    -   deep engagement: do folks have long usage duration on core value prop actions?
-   **Stable retention**
    -   active: defined based on core value prop of product
    -   volume of active cohort users in period N+1 / Volume of active cohort users in period N
-   **Sustainable growth**
    -   Meta often waits to ramp up upper funnel acquisition till after products can sustainability add and retain folks (e.g. product growth is likely not sustainable long-term if customer acquisition costs greatly exceed value extracted from the customer)
    -   growth accounting states model used to dissect active user growth ([Duolingo example](https://blog.duolingo.com/growth-model-duolingo/))
    -   i.e. is active user growth being fueled by new users, existing active users, and/or reactivated existing users?
-   **Deep engagement**
    -   time spent, days engaged out of the last 28 days, purchases/revenue per user, etc

### Growth Equation

-   **Overview**
    -   part growth framework + part metrics framework
    -   framework for how consumer products grow active users / customer base
    -   inspired by [Chamath Palihapitiya](https://youtu.be/raIUQP71SBU) (Facebook VP of Growth: 2007 - 2011) and [Andy Johns](https://review.firstround.com/indispensable-growth-frameworks-from-my-years-at-facebook-twitter-and-wealthfront/) (early growth employee at Facebook, Twitter, Quora, Wealthfront exec, former VC)
-   **Chamath's description** [^wip_metrics_foundation-1]
    -   acquisition: get people to the front door
    -   activation: get them to an "Aha" moment as quickly as possible
    -   engagement: deliver core value as often as possible
-   **Andy Johns's description (from Andy's work with Chamath at early days Facebook)**
    -   top of funnel (traffic, conversion rate) x
    -   magic moment (create emotional response) x
    -   core product value (solves real problems) =
    -   sustainable growth
-   **Facebook early days example**
    -   grow active users (active user growth makes the social network more valuable, provides evidence for additional funding/IPO, monetization opportunities, etc)
    -   active users = active new users + active existing users
    -   hypothesized set of key metrics and bets aligned to each growth equation pillar
    -   acquisition (top of funnel sign ups)
        -   universitys adopted
        -   expansion to high schools and non-students
        -   international expansion / localization
    -   activation (7 friends in 10 days)
        -   user onboarding optimized for profile completion and connecting with friends
    -   engagement (time spent & frequency)
        -   news feed innovation and time spent
        -   platform integrations and expanding engagement sources (i.e. game developers building games for FB; FarmVille)
        -   "Like" button as a mechanic for how users interact with content
        -   positioning profile content as a "Timeline" to more easily see a users history and life events
        -   Facebook mobile app and Instagram acquisition to capitalize on shift to mobile
-   **Notes on operationalizing the growth equation framework**
    -   build a culture of measuring, testing, and try new things
    -   requires teams to be unemotionally detached from what is getting built
    -   don't overly rely on gut as most folks can't predict correctly
    -   growth equation specifics differ by company type (core elements persist: acquisition, activation, engagement)

[^wip_metrics_foundation-1]: **Virality wasn't a core focus initially (how existing users recruit other users); Chamath put the emphasis on acquisition, activation, and engagement vs K factor (virality)**

------------------------------------------------------------------------

# Best Practices for Defining Metrics

1.  **Keep it simple**
    -   use the simplest metric logic to achieve the desired measurement objective
    -   ideally new metrics should be easy to understand and explain
2.  **Self-documenting metric names**
    -   aim for metric names that describe the metric for tech and non-tech stakeholders
    -   "last_28_days_trailing_total_sales" vs "total_sales" to represent the last 28 days of a trailing total
3.  **Actionable**
    -   source of inspiration for ways to improve the product experience or business
    -   hypothesized cause and effect relationship (i.e. doing X will lead to Y metric improvement)
4.  **Sensitive enough to move but not volatile**
    -   tends to require a balance between broad/high-level metrics and granular/detailed metrics
    -   actionable lagging metric or overaly broad metric might not be sensitive enough to move in the near term
    -   volatile metrics can lead to hard to explain fluctuations
5.  **Analyze historical metric trend, fluctuations, and underlying data quality**
    -   how does seasonality and user mix impact the metric?
    -   has the underlying data passed QAed for x new metric?

------------------------------------------------------------------------

# Investigating Metric Change

1.  **Understand the metric definition, data source, filters being used, and the time window**
    -   fact gathering step before jumping into problem solving mode
2.  **Determine if the metric change is meaningful to the business**
    -   if the change is within the expected fluctuation range, it may not require further investigation
    -   however, if the change is small but unexpected, it might be worth investigating as a signal for a larger underlying issue (business/product/data issue)
3.  **Consider any seasonal, business, or product changes that could influence the metric**
    -   metric changes during specific events may be expected (i.e. holiday shopping surge for an ecommerce company)
4.  **Check for any data pipeline or experience bugs that might be distorting the metric**
    -   recommended to start with the business context and then move to data/experience QA
5.  **Conduct a drivers analysis to identify the root cause of the deviation**
    -   use segmentation analysis to understand if a specific segment is driving the change (i.e. due to large weight/influence on the metric or outlier type performance compared to other segments)
    -   e.g. compare new users to existing ones, device types, geographic locations, power users versus non-power users, and paid versus free users
    -   i.e. new users during a particular period are taking certain actions at a lower rate than in previous periods

------------------------------------------------------------------------

# Defining Success Metrics for a New Product Experience

1.  **Context Collection**
    -   what customer problem/need/pain is the new product experience solving?
    -   what would the high-level experience look like?
2.  **Connecting the New Product Experience to Company Objectives**
    -   is the new product experience expected to deliver on a key company objective (i.e. establish product market fit, increase acquisition, improve engagement depth or retention, drive average revenue per user or LTV higher, etc)?
3.  **Primary Success Metric**
    -   align the primary metric to the core value prop the new experience is delivering and to the company objective being targeted
    -   one key success metric is preferred as it helps drive focus
4.  **Guardrail Metrics**
    -   make sure other parts of the experience are not eroding as the primary success metric improves
    -   metrics that should be stable are stable
    -   key segments to watch might also be part of the guardrail metrics
5.  **Measurement Window and Segments**
    -   do the metrics need a time filter?
    -   who should the metric target? (i.e. all users, only paying users, or new vs existing users, etc)

### Example Scenario: Audio Streaming Company ABC

1.  **Context Collection**
    -   customer pain
        -   qual: when using ABC company's mobile app, subscribers mention it is difficult to find and consume newly released audio offerings (music, podcasts, audioboks, lectures, etc)
        -   quant: consumption rates are low on newly released audio offerings outside of the new user onboarding experience (suggesting discovery might be a challenge)
    -   high-level experience improvement solution
        -   content discovery AI chat assistant (within the mobile app experience)
        -   users select a mood or input a prompt and the AI returns newly released content recommendations
2.  **Connecting the New Product Experience to Company Objectives**
    -   ABC company is looking to increase engagement depth on it's mobile app
    -   ABC has identified that new content discovery is predictive of higher future engagement depth and retention
    -   as a result, the new product experience is aiming to drive new content discovery and engagement depth in the near-term and retention in the long-term
3.  **Primary Success Metric**
    -   new content discovery rate
    -   defined as the % of users who open the app and consume a newly released audio offering
    -   the metric is generic enough to work in an AB test (e.g. applicable to the control group / status quo experience)
    -   newly released defined as content released within the last 30 days
4.  **Guardrail Metrics**
    -   listening time per user: check if listening time per user is stable or increasing
    -   session length: could potentially decrease if users are finding content faster
    -   CSAT: qual survey asking users if they are finding the new AI content discovery experience helpful
5.  **Measurement Window and Segments**
    -   measurement window: of all the users who open the app on X day, what percent consume new content within 24 hours of their last app open
    -   segments: all users
    -   additional segments to explore for deeper dive analysis:
        -   different user types: free, paid tier 1, paid tier 2, etc
        -   content types driving the new content discovery rate (i.e. music, podcasts, audiobooks, lectures, etc)
        -   new users (signed up in the past 4 weeks), power users (engaged with the app weekly for past 4 weeks), causal users (engaged with the app 1 to 3 weeks in the past 4 weeks), reactivated (not active in the past 4 weeks)

------------------------------------------------------------------------

# Active User Ratios: DAU, WAU, MAU, QAU

-   help explain the type of usage patterns folks have and gauge stickiness
-   when the ratios are close to 1 it suggests high stickiness during the metric frame
-   when the ratios are low it suggests less frequent usage during the metric frame

### DAU / WAU

-   helps identify if WAU is driven by folks coming back each day of week (ratio close to 1) or many users coming back once per week

### DAU / MAU

-   higher ratio means folks are coming back frequently and there's more of a daily value prop
-   lower ratio suggests folks are coming less frequency during a month

### WAU / MAU

-   if metric is close to 1 it suggest folks are coming back each week
-   if metric is low suggests that folks are not coming back multiple weeks in a month

### MAU / QAU

-   helpful if a product is used on less frequent basis
-   is the QAU driven by folks coming back each month or folks coming back once per quarter

### Why look at active user ratios vs absolute counts

-   both are helpful to give a read on the engagement health of the business
-   counts can indicate if the active user base is growing or not
-   active user ratios help assess the stickiness of the user base even if overall counts are declining (i.e. small active user base but those that are active are highly active)
-   i.e. an active count increase could be driven up by large sign up volume and ratios can spotlight if folks are sticking around
-   additionally, product teams look at proportion of actives of vs all qualified users(absolute counts could be increasing but share of active population could be declining)

------------------------------------------------------------------------

# Is Active User Growth Healthy/Sustainable?

### Potential signs of healthy growth

-   true test of sustainability is time (e.g. does the progress/momentum persist)
-   growth is being driven after product market fit has been established
-   organic word of mouth has started to drive new user growth
-   LTV to CAC ratios make economical sense across channels
-   some growth acquisition channel diversification
-   engagement depth and retention are solid/improving
-   acquiring new user cohorts leads to more new users
-   natural pattern of engagement followed by more engagement

### Potential signs of unhealthy growth

-   active user growth is being propped up by spikey aggressive paid marketing efforts and word of mouth/referrals is minimal
-   existing users are not engaging deeply and coming back to the product
-   new user growth is hiding existing user churn
-   satisfaction decays quickly as users use the product
-   growth is highly dependent on one channel (resulting in single channel risk)
-   deep discounting or free trials not optimized for LTV to CAC and boosting active user growth
-   easily copied marketing tactics on primary channel(s)

------------------------------------------------------------------------

# Case Studies

### Product X has 100% user turnover week to week but the count of 7 day actives is still growing. How might this be?

-   100% user turnover week to week implies users who are active for a given week don't return the next week
-   as a result, new user acquisition and/or reactivated users must be fueling the 7 day active count
-   in other words, count of new users who become active + count of dormant users in the previous week who become active \> previous week active users
-   note this is an extreme case (likely not realistic in practice for most products)
-   however, useful thought exercise to conceptualize active user growth accounting basics
-   total active users = active new users + returning active existing users + reactivated existing users

+----------+----------+----------+-----------+-------------+
| Week     | Total    | Active   | Returning | Reactivated |
|          |          |          |           |             |
| Number   | Active   | New      | Active    | Existing    |
|          |          |          |           |             |
|          | Users    | Users    | Existing  | Users       |
|          |          |          |           |             |
|          |          |          | Users     |             |
+:========:+:========:+:========:+:=========:+:===========:+
| 1        | 500      | 250      | 0         | 250         |
+----------+----------+----------+-----------+-------------+
| 2        | 600      | 300      | 0         | 300         |
+----------+----------+----------+-----------+-------------+
| 3        | 700      | 350      | 0         | 350         |
+----------+----------+----------+-----------+-------------+
| 4        | 800      | 400      | 0         | 400         |
+----------+----------+----------+-----------+-------------+

### What metrics would you look at to understand if a product is sticky?

-   product stickiness tends to be defined based on core action usage and frequency
-   core action usage should align to the main value prop of the product
-   frequency interval depends on the product domain
    -   consumer social media product might focus on daily usage intervals
    -   US tax preparation software might focus on annual usage intervals
-   what's an example of a product stickiness metric for Spotify?
    -   core action framing: session with 5 mins or more media consumption
    -   frequency goal: users return week over week to do the core action
    -   metric: percent of qualified users who return in week N and week N-1 with 5 mins or more media consumption in 1 or more sessions (future todo: how might one simplify this metric)

### What metrics would you look at to understand activation?

-   activation metrics tend to align to the "ah ha" moment of the product
-   for Facebook in the early days it was 7 friends in 10 days
-   metric would vary by product
-   base metric elements: user actions/behaviors that positively correlation with future usage/value extraction + time window for when the action(s) should occur

### How to handle a macro trend inflating primary metric?

-   attempt to gauge if the macro trend is temporary or likely to persist
-   i.e. COVID stay at home orders (temporary) vs shift to smartphones (likely to persist)
-   if the macro trend is likely to persist, the product might be experiencing a cultural or technological wave in which case the primary metric could benefit or be dragged down (depending on the business case, teams might leave the primary metric as is)
-   if the macro trend is temporary/seasonal, techniques could be explored to measure the primary metric impact or control for the temporary/seasonal trend
    -   simple pre vs post is the most basic starting point
    -   more advanced techniques to isolate the primary metric impact could involve time series decomposition, regression analysis, causal impact analysis, etc

### Why are vanity metrics context dependent?

-   vanity metrics tend to be metrics that look good on the surface but don't tie with business objectives and/or lack actionability
-   upper funnel metrics like app installs can be a vanity metric for a mature business but a critical metric for a new business trying to grow discovery
-   social media likes might be an OKR vanity metric; however, if likes correlate with downstream business objectives/goals then it could be a useful metric
-   vanity metric pressure testing
    -   does the metric align to business objectives/goals?
    -   is the metric actionable given business maturity and product type?

### Unintentional consequences of goal metrics?

-   could lead to short-term optimization vs long-term thinking
-   look out for path of least resistance to hit goal metrics which could lead to negative knock-on effects
-   many corporate world examples of this (individuals/teams pressured to hit a goal metric and damage the business/customer experience in the process)
-   if career incentives are tied to goal metrics, are there "unhealthy" ways to move the metric?
-   are some actions to move the metric better than others? is the goal metric robust enough to account for this?
