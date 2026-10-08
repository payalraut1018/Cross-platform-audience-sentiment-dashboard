# Cross-platform-audience-sentiment-dashboard
A Power BI dashboard analyzing cross-platform audience overlap, sentiment-weighted engagement, and content lifecycle

# Cross-Platform Audience & Sentiment Intelligence

A Power BI dashboard that estimates the number of unique individuals reached by a social media campaign across six platforms, assesses whether audience reactions are positive or negative, and examines how engagement evolves during the 48 hours following publication.

Platform-level reporting tends to overstate reach, disregard sentiment, and offer limited insight into how quickly audience attention declines. This project addresses all three limitations using a single dataset of approximately 150,000 posts.

![Page 1: Reach & Overlap](screenshots/page1-reach-overlap.png)
![Page 2: Sentiment & Fatigue](screenshots/page2-sentiment-fatigue.png)

## Business Problem

A campaign distributed across TikTok, Twitter, Instagram, Reddit, Facebook, and YouTube generates six independent sets of metrics. Reviewed together, they leave three questions unresolved:

1. Duplicate reach: Each platform reports its own views, making it impossible to determine whether the same individuals are counted again on another platform.
2. Fake hype: Likes, comments, and shares are totaled irrespective of tone, so a heavily criticized post can appear as successful as a well-received one.
3. Viewer fatigue: Engagement is seldom evaluated against post age, which makes it difficult to identify when interest declines and when additional spend ceases to be worthwhile.

## Goal

To provide marketing and analytics teams with a single view that answers:

- How many unique individuals were reached, and to what extent do platform audiences overlap?
- Was engagement predominantly positive or negative, rather than merely high in volume?
- How does engagement develop after publication, and what drives unusual spikes or declines?

## Key Visuals

Page 1: Reach & Overlap
- KPI cards: Total unique users, total views, average positive sentiment, and audience overlap, presented together to highlight the gap between raw views and actual individuals reached.
- Unique vs. Shared Audience by Platform: Each platform's total audience alongside the portion that also appears on other platforms.
- Platform Audience Summary:Exact unique-user counts for each platform.
- Filters:Topic, Platform, Date range, and a Clear all slicers button.

Page 2: Sentiment & Fatigue
- Engagement Trend & Anomaly Detection: Daily sentiment-weighted engagement, with unusual days flagged automatically.
- Engagement Decay by Time Since Posting: Average engagement at 0-6, 6-12, 12-24, and 24-48 hours after posting, shown as one line per platform.
- Engagement Driver Breakdown: A decomposition tree that segments the total by platform, media type, topic, and location.
- Filters: Topic, Platform, Date range, and a Clear all slicers button.

## Approach

### Data model
A dedicated date table is related to the primary table through a many-to-one relationship, and all measures are maintained in a separate `_Measures` table. Posts are grouped into time-since-posting intervals using a calculated column (`Hours Since Post Bucket`), supported by a numeric order column so that the intervals sort chronologically.

### Key measures

Unique audience: counts each individual once, regardless of the number of platforms they use.
```dax
Total Unique Users = DISTINCTCOUNT(Engagement[user_id])
```

Multi-platform audience: counts individuals who appear on more than one platform. `REMOVEFILTERS` is required so that a platform filter applied by a visual (for example, a single bar in the bar chart) does not restrict the count to one platform.
```dax
Multi-Platform User Count =
VAR UserPlatformCounts =
    ADDCOLUMNS(
        VALUES(Engagement[user_id]),
        "PlatformCount",
        CALCULATE(
            DISTINCTCOUNT(Engagement[platform]),
            REMOVEFILTERS(Engagement[platform])
        )
    )
RETURN
    COUNTROWS(FILTER(UserPlatformCounts, [PlatformCount] > 1))
```

Overlap share:
```dax
Pct Audience Overlap = DIVIDE([Multi-Platform User Count], [Total Unique Users])
```

Sentiment-weighted engagement: multiplies each post's engagement by its net sentiment, so that negative reactions reduce the total rather than increase it.
```dax
Sentiment Weighted Engagement =
SUMX(
    Engagement,
    Engagement[total_engagement] * (Engagement[sentiment_positive] - Engagement[sentiment_negative])
)
```

### Anomaly detection
Power BI's built-in anomaly detection is applied to the daily sentiment-weighted engagement trend.

## Key Findings

- 12% of users(14,899 of 129,556) appear on more than one platform.
- YouTube accounts for approximately 49% of total sentiment-weighted engagement, ahead of Instagram and Twitter.
- Across all platforms, average engagement declines by only about 7% between the first 6 hours and the 12-24 hour window, then partially recovers. YouTube declines by roughly 19% early on, while TikTok increases with post age.

## Business Impact & Insights

### 1. Reach is smaller than reported
- What the data shows: Approximately 12% of users appear on more than one platform.
- Why it matters: Summing platform-level reach counts these individuals more than once, so the true audience is smaller than reported figures suggest. Teams that evaluate success using combined views should track unique users instead.

### 2. One platform contributes most of the value
- What the data shows: YouTube accounts for roughly 49% of sentiment-weighted engagement.
- Why it matters: This indicates where content effort is generating returns and provides a reasonable starting point for allocating time and budget across platforms.

### 3. Engagement is sustained after posting
- What the data shows: Average engagement declines by only about 7% within the first 48 hours; YouTube declines by about 19%, while TikTok rises.
- Why it matters: The assumption that content loses momentum quickly is not supported by this data, so ending activity on a platform early would be difficult to justify.

### 4. Sentiment changes how success is measured
- What the data shows: Weighting by net sentiment allows negative reactions to lower a post's score.
- Why it matters: A heavily criticized post is no longer counted as a success, which gives a more accurate view of performance than raw likes, comments, and shares.

### 5. Unusual days can be traced to a cause
- What the data shows:Anomaly detection flags days outside the expected range, and the decomposition tree segments them by platform, media type, topic, and location.
- Why it matters: An unexpected change in the numbers becomes a specific starting point for investigation.

*These insights are descriptive and are drawn from a synthetic dataset. They demonstrate what the dashboard can surface and should not be treated as established conclusions about real campaigns.*

## Tools

Power BI Desktop: Power Query, DAX, data modeling, anomaly detection, and the decomposition tree visual.

## Data

[Multi-Platform Social Sentiment & Engagement Evolution Dataset](https://www.kaggle.com/datasets/sohumgokhale/multi-platform-social-sentiment-evolution): a synthetic Kaggle dataset containing 150,000 posts across six platforms. The `user_id` values are synthetic and do not correspond to real individuals.


## Files

- `Cross_platform_media_insights.pbit`: template version without the data
- `Cross_platform_audience_sentiment_dashboard.pdf`: static export of both pages
- `screenshots/`: images used above
