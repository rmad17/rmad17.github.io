+++
title = 'Database Design for Products'
date = 2025-09-06T01:00:28+05:30
tags = ["database", "api", "scaling", "product", "system design", "database design"]
draft = false
cover.image = '../../databasedesign.png'
cover.responsiveImages = true
cover.hidden = false
categories = ["tech", "database", "database-design", "system-design"]
disqus_url = 'https://souravbasu.xyz/posts/database-design/'
disqus_identifier = '2025-09-06T01:00:28+05:30'
+++

# Database Design: A Product-First Approach to Building Scalable Systems

Over the years wrestling with database architectures I've learned that the most elegant technical solution isn't always the right one. Database design is often viewed through the lens of technical optimization—normalized tables, efficient indexes, and query performance. While these technical aspects are crucial, the database disasters I've witnessed taught me that the most successful architectures emerge from a deep understanding of the product they serve.
This article distills hard-won lessons from building databases for fantasy sports platforms that needed to handle 100x traffic spikes during Premier League matches, airline systems that couldn't afford a single booking failure, and financial systems where a single data inconsistency could trigger regulatory audits. The fantasy platform that once struggled with Saturday afternoon traffic surges now gracefully handles millions of users checking their teams during Champions League nights. The airline system processes complex multi-leg journeys without breaking a sweat. Each project taught me something different about the gap between textbook database design and production reality.
What changed wasn't the technology—it was the approach. I stopped designing databases for technical perfection and started designing them for the products they serve. This shift in perspective has been the difference between systems that merely function and systems that excel under pressure.

This article shares insights from three distinct use cases, each with unique challenges that shaped my approach to product-centric database design.



## 🎧 Looking for a audio overview?

<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/2167608318&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/sourav-basu-952044690" title="Sourav Basu" target="_blank" style="color: #cccccc; text-decoration: none;">Sourav Basu</a> · <a href="https://soundcloud.com/sourav-basu-952044690/database-designs-for-product" title="Database Designs for Product" target="_blank" style="color: #cccccc; text-decoration: none;">Database Designs for Product</a></div>

---

## 🎯 Product-Centric Database Design Framework

```
┌─────────────────┐    ┌─────────────────┐    ┌────────────────────┐
│   🎯 Product    │───▶│   🏗️ Schema     │───▶│   🚀 Product       │
│    Analysis     │    │     Design      │    │    Success         │
│                 │    │                 │    │                    │
│ • User Workflows│    │ • API Patterns  │    │ • User Satisfaction│
│ • Business      │    │ • Scale Planning│    │ • Business Goals   │
│   Context       │    │                 │    │                    │
└─────────────────┘    └─────────────────┘    └────────────────────┘
```

**Core Principles:**
✅ Understand user workflows before schema design  
✅ Design for product roadmap evolution  
✅ Optimize for actual usage patterns  
✅ Measure product outcomes over technical metrics

---

## Three Case Studies: Different Products, Different Solutions

| **Use Case** | **Core Challenge** | **Key Strategy** | **Primary Focus** |
|--------------|-------------------|------------------|-------------------|
| 🏆 **Fantasy Gaming** | High reads + burst scaling | Denormalization + caching | Read optimization |
| ✈️ **Airline Reservations** | Schema evolution | Version management | Flexibility |
| 💰 **Loan Management** | Operational scaling | Strategic denormalization | Process efficiency |

---

# Case Study 1: Fantasy Football App - Optimizing for Read-Heavy Workloads

## The Challenge: 5x Traffic Spikes During Live Matches

The fantasy football platform presented a classic read-heavy scenario with extreme burst scaling requirements. During match days, concurrent users jumped from 300/min to 1500+/min (5x spike) as users obsessively checked live scores and player statistics.

## 📊 Traffic Pattern Analysis

**Key Insights:**
- **Read vs Write Ratio:** 15:1 (users check 15x more than they change)
- **Peak Duration:** 6-8 hours across multiple match kickoffs
- **Critical Path:** Live player statistics during matches

## Database Architecture: Strategic Denormalization

Instead of a normalized approach, I designed for query efficiency with strategic denormalization:
The data required during the peak hours could be resolved into three categories:
* Player and Team Info - This includes profile information on players and teams
* Live Updated Data - This is the match data that we are storing for each player at an appearance level. 
* Calculated Metrics and Points - This is the data that is calculated realtime based on the match data and the players role. This is the metrics that is used in ranking players and teams.

The first kind of data changes very little and is easier to cache. Invalidating the cache every week or twice a week would be sufficient 
to ensure that the profile information is always updated. This also reduced the initial load time for our stats dashboard as we could present this
data while the rest of the data loaded.

The next piece of the puzzle was fetching the match data. We had indexes setup to improve the speed of fetching from the database. The challenge with indexes is that they slow down the writes of the database. To create a balance we used sockets to push real time updates, using Elixir to manage concurrency and indexes on database to manage the fast reads. We preferred to optimise queries over read replicas to ensure the user has access to almost real time data.

The third part is where the majority of challenges lie. This is the most critical data and a large part of it is derived from the other sets of data. The derivation process is also based on algorithms created by us and this created the scoring process. This is also to be factored that post the metrics and points calculations these needs to be store to the database for further reads. This could not be solved by one single way so we broke it down to multiple approaches. 
  1. The first approach was to have our point allocating algorithm in a json format. This allowed us to fetch the points to be allocated at a greater efficiency. The scoring would then be calculated.
  2. We would then update the relavant tables with the scores calculated at the previous step.
  3. The final retrieval of stats involved real time calculations as well. Like for example the stats and points achieved in home games and away games. To address these kind of scenarios we denormalized the data when storing and stored in seperate tables to make the fetching faster. On the UI as well we made these data available on request. This would ensure there would be zero latency issues for the user.

The last step of denormalization requires a tradeoff to use more storage which was an acceptable condition to us as storage is significantly cheaper than cpu cycles and provides us with a better experience for our users. At the same time to minimize storage usage we would archive data that is older than three years. This also keeps our size of the stats tables limited and ensures our existing process remains scalable. 


## API Query Optimization Examples

**Before Optimization**
Common API call: "Get Cristiano Ronaldo's season summary with home and away points"
- Required tables: Players → Teams → Match Stats → Game Results → Team Schedules → Opponent Stats   
- Response time: 2-5 seconds
- Database load: High CPU usage during peak traffic

**Post Optimization**
Same API call using seperate home and away table :
- Required tables: Single table lookup from player_season_summary
- Response time: <500ms
- Database load: Minimal CPU usage
- Storage trade-off: Duplicate data adds extra storage but improves performance


## Results Achieved

| Metric | Before | After | Improvement |
|--------|--------|--------|-------------|
| **Peak Load Handling** | 500 concurrent users per minute | 2.5k concurrent users per minute | 2k concurrent users per minute |
| **API Response Time** | 3-8 seconds | <500ms | 6-15x faster |
| **Database Load** | >90% during peaks | 40% during peaks | >50% reduction |
| **Business Gains** | Smoother user experience | Cost reduction due to more efficient queries |


---

<br><br>


# Case Study 2: Airline Reservation System - Schema Evolution Management

## The Challenge: NDC Version Compatibility

The airline industry's challenge centered on IATA's New Distribution Capability (NDC) schema evolution. Airlines implement different NDC versions (17.2, 18.1, 21.3, 24.1) with no standardization, requiring support for multiple versions simultaneously while converting between client-specific versions.

## NDC Version Management Architecture

### Core Challenge Visualization

```
Client Request (NDC 21.1) ──┐
                            │
Client Request (NDC 22.2) ──┼──▶ Our System (NDC 22.2 Base) ──┐
                            │                                 │
Client Request (NDC 24.1) ──┘                                 │
                                                              ▼
                                           ┌─────────────────────────────┐
                                           │  Version Conversion Engine  │
                                           │                             │
                                           │ Input: Any NDC Version      │
                                           │ Storage: Our Base Version   │
                                           │ Output: Client's Version    │
                                           └─────────────────────────────┘
```

### Database Schema Design for Version Flexibility
The challenge in this case was airlines follow different versions of NDC. The airline data had to be parsed and responded to the client in a our base version. With different schemas we had the challenges of same data in different data structures as well as specific data in higher versions. We came up with the idea of storing the data in the base versions format, with two other tables where we store extra details in a seperate table `version_extras  and we would add these based on the version number. The primary table itself would also capture the version details so that the data could be constructed back in the source format. Every time a new version is released we store the extra data in version extensions while maintaining the version info. When we upgrade the base version, we use the version_extras to modify the primary data details. The APIs would follow this pattern to provide the response and this process would make this scalable. 


## Results Achieved

| Metric | Result | Description |
|--------|---------|-------------|
| **Version Support** | `10+ versions` | Concurrent NDC version support |
| **Migration Time** | `Less than 1 day` | Time to add new version support |
| **Zero Downtime** | `✓` | No service interruption for updates |
| **Client Compatibility** | `100%` | All client versions supported |
| **Business Gains** | `Better support for multiple clients` | Increased potential customers |

---

# Case Study 3: Loan Management System - Scaling for Operational Complexity

## The Challenge: Evolving Regulatory Requirements

The loan management system needed to scale not just for volume, but for operational complexity. As the business evolved—from basic lending to obtaining NBFC license the number of process steps, compliance requirements, and data relationships grew exponentially.

## Operational Lifecycle Process

```
Loan Request → Loan Approval → KYC → Loan Disbursal → EMI Collection → Loan Closure
```

<br>

### Loan Lifecycle Database Design

The loan management system follows a clear lifecycle from application to closure. Here's the list of the primary database tables that we would use for most queries:

**User Table** - Primary user details that includes sensitive and non sensitive data

**Loan Request Table**  - Details regarding the requested loan amount, proposed tenure, etc.

**Loan Approval Table** - Contains information related to approval amount, approved tenure, rejection_reason(if rejected), etc.

**Loan Disbursal Table**  - Here we have the financial specifics of the loan - emi_amounts, interest rate, disbursal date, disbursal amount, etc. 

**EMI Table** - Contains records of each EMI and is updated as each are paid or missed. This also contains records of any emi based penalties.

**Loan Closure Table** - The final step when the loan is completed. This table also captures other informations like loan penalties, prepayment based closure, loan_written_off, etc.

**Payment Table** - Any kind of payment related transactions 

<br>

### Strategic Denormalization for Operations
---
**Performance Problem:**
The normalized design required joining multiple tables for a single query to gather any meaningful information for the user, resulting in 3-8 second response times. 

Lets take the example of a loan status query:
We would need the information of approved amount, loan details, EMIs paid and pending, prepayments made, penalties, etc. This would usually require significant joins and would be a performance bottleneck for the application. A very simple solution is to have the most ferquently used data in a denormalized format in a seperate tables. In this case a Loan Status table which contains the data from multiple tables into this single one. This reduces the time taken for each query improving API performance. This is not a major issue in scale as well because the nature of the application is read heavy.  

### Query Performance Optimization

**Before (Normalized - Multiple Joins):**
Loan Status query requiring 6 table joins:
- Response time: 3-8 seconds at 100 users per minute
- Query complexity: JOIN across User, Loan_Request, Loan_Approval, Loan_Disbursal, EMI tables

**After (Denormalized - Single Table):**
Direct lookup from loan_status table:
- Response time: <400ms at 100 users per minute
- Query complexity: Simple SELECT with WHERE clause

<br>

## Results Achieved

| Metric | Before | After | Improvement |
|--------|--------|--------|-------------|
| **Loan Officer Query Time** | 3-8 seconds | <400ms | 6-10x faster |
| **Compliance Report Generation** | 2 hours | 5 minutes | 24x faster |
| **Business Gains** | Faster turnaround for reports | Reduced Customer Queries |

---

# Key Learnings: When to Normalize vs. Denormalize

## Decision Framework

| **Scenario** | **Approach** | **Reason** |
|--------------|--------------|------------|
| **High Read/Write Ratio (>10:1)** | Denormalize | Query performance over storage |
| **Complex Operational Workflows** | Denormalize | Reduce join complexity |
| **Transactional Integrity Critical** | Normalize | ACID compliance |
| **Rapid Schema Evolution** | Hybrid | Flexibility with performance |

## Strategic Denormalization Guidelines

✅ **When to Denormalize:**
- Query patterns are predictable and repetitive
- Read performance directly impacts user experience
- Operational workflows require quick decision-making

❌ **When to Stay Normalized:**
- Data consistency is more critical than performance
- Schema changes frequently
- Storage costs are a major concern
- Write patterns are unpredictable

## Conclusion

Database design success comes from understanding your product's specific needs and user behavior patterns. Each of these three systems succeeded because the database architecture aligned perfectly with the product requirements:

- **Fantasy Football:** Optimized for burst reads and predictable query patterns
- **Airline Reservations:** Designed for schema evolution and version compatibility  
- **Loan Management:** Structured for operational efficiency and regulatory compliance

The key insight is that there's no universal "best" database design—only designs that work well for specific product contexts. Understanding your users' workflows, your business constraints, and your operational requirements is far more valuable than following theoretical database design rules.
