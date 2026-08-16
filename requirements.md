# SaaS Analytics Platform
## AI-Assisted Development Training Project

### Project Overview

Build a complete end-to-end analytics platform for **Invented Software**, a fictional B2B SaaS company selling business-management software to small merchants — using AI-assisted development practices. This project simulates real-world data engineering and business intelligence workflows while teaching modern development practices with AI tooling.

The company's revenue model is **entirely recurring**: monthly fees on software
plans and add-ons. There is no transaction processing, no interchange and no
hardware, which makes the SaaS metric stack the natural analytical frame.

Note that `raw_products.type` does **not** reliably distinguish recurring
revenue — it is a catalogue label that has drifted from what the products
actually are. Derive the revenue-bearing SKUs from `raw_subscriptions` rather
than trusting the column.

**Dataset location**: the `data/` directory contains 6 CSV tables: merchants,
subscriptions, products, markets, acquisition costs, and operating costs. 8
markets, 160 merchants, 117 subscription records.

**Coverage is not uniform.** The two cost tables cover 2024–2025. The entity
tables predate that: merchant signups run 2022-06 → 2024-06 (114 of 160 before
2024) and subscriptions start 2022-06 → 2024-12 (75 of 117 before 2024). The
reporting window and the entity history are not the same range — scoping one
over the other drops most of the revenue.

**Data conventions** are documented once, in [`data/README.md`](data/README.md),
along with the field-level notes and the join keys. Read that before modelling;
several of the conventions corrupt results silently if missed.

### The KPI list

**This is the canonical list.** Other documents link here rather than restating
it. Phase 1 asks you to choose **three to five** of these and specify them
precisely.

| # | KPI | Notes |
|---|---|---|
| 1 | Monthly Recurring Revenue (MRR) | The spine. Everything else depends on it |
| 2 | Annual Recurring Revenue (ARR) | MRR × 12 |
| 3 | Logo Churn Rate | Customers lost. Mind the denominator |
| 4 | Revenue Churn Rate | Euros lost — a different question from logo churn |
| 5 | Net Revenue Retention (NRR) | Expansion minus contraction and churn |
| 6 | Gross Profit Margin | Which cost source? There is more than one |
| 7 | Customer Acquisition Cost (CAC) | Denominator matters as much as numerator |
| 8 | Customer Lifetime Value (LTV) | Depends on a churn rate and a margin |
| 9 | CAC Payback Period | Months of gross profit to repay acquisition |
| 10 | LTV : CAC Ratio | Both halves must describe the same population |
| 11 | Burn Multiple | Net burn ÷ net new ARR |
| 12 | Cash Runway | Cash ÷ burn. Decide what counts as burn |

Twelve entries. Several are ambiguous in ways the data will not resolve for you —
that ambiguity is deliberate, and documenting the choice you made is part of the
deliverable.

### Learning Objectives

By completing this project, you will demonstrate proficiency in:
- AI-assisted research and technology selection
- Database design and implementation
- Data modeling with dbt
- Learn about basic of containers, as setup should be containerized with Docker Compose
- Business intelligence platform setup and configuration
- End-to-end data pipeline orchestration
- Technical documentation and presentation skills

### Project Phases

#### Phase 1: Data Discovery & Business Understanding
**Deliverable**: Business requirements document

**⚡ FIRST STEP**: copy the project so the original stays available to compare
against — `cp -r . ../workspace && cd ../workspace` — and commit as you go. This
repository already ships an agent instruction file and git history; there is no
need to generate either.

- Analyze the provided dataset structure and relationships
- Document business logic and key performance indicators
- Identify analytical questions the platform should answer
- Define success metrics for the analytics platform

#### Phase 2: Architecture & Technology Research
**Deliverable**: Technical architecture document with AI research logs

- Research and select appropriate database technology (PostgreSQL, MySQL, SQLite, etc.)
- Research and select a business intelligence platform. Evaluate candidates
  against criteria rather than reputation: self-hostable? open licence?
  dbt-native semantic layer? containerisable? configurable from code, or
  click-through only?
- For each tool on your shortlist, check what assistance exists for it — an agent extension, an MCP server, an official container image, or nothing. A tool with good tooling around it has a much shorter learning curve. Record what you find in the ADR.
- Document technology choices with AI-assisted research rationale
- Create system architecture diagram


#### Phase 3: Database Implementation
**Deliverable**: Functional database with imported data

- Install and configure chosen database system
- Design and implement database schema
- Import CSV data with data quality validation
- Create database documentation


#### Phase 4: Data Modeling with dbt
**Deliverable**: dbt project with analytical models

- Initialize a **dbt Core** project and configure the database connection. dbt
  Cloud is a hosted service and is excluded by the local-deployment constraint.
  dbt runs **in a container**, reaching the database by its compose service name
- Develop staging models for raw data cleaning
- Build dimensional models for analytical use cases
- Implement data quality tests and documentation
- Generate and review dbt docs


#### Phase 5: Business Intelligence Platform (3-4 hours)
**Deliverable**: Configured BI platform with dashboards

- Install and configure chosen BI platform
- Connect BI platform to analytical database models
- Design and build executive dashboard with key metrics
- Create detailed operational reports
- Implement user access and security


#### Phase 6: System Integration & Automation
**Deliverable**: Automated deployment scripts

- Create startup scripts for all services
- Implement data refresh functionality with validation
- Build error handling and monitoring
- Create user documentation for system operation


### Technical Requirements

**Version Control Setup**:
- Initialize a local git repository at project start (no remote publishing required)
- Use git to track decision evolution and maintain project history
- Per-phase feature branches (`phase-1-discovery`, `phase-2-architecture`, etc.) are recommended practice but optional for solo work — commits on `main` are fine if branches feel like overhead

**Architecture Constraints**:
- Open source technology stack only
- Local deployment (no cloud SaaS solutions — this rules out dbt Cloud and any
  hosted BI offering)
- **Containerize the entire stack** with Docker Compose — database, dbt and the
  BI platform. The deliverable is a **reproducible package**: someone who has
  only a container runtime clones the repository, runs one command, and gets a
  working platform. Nothing else should need installing on the host — no
  database, no Python environment, no dbt, no BI tool.

  Concretely, that means dbt runs as a service against the database over the
  compose network rather than from a virtual environment on your machine, and
  anything that provisions the BI tool runs in a container too. Reproducibility
  is the point: a host-installed dependency is a step that works on your machine
  and fails on someone else's
- Production-ready configuration
- Scalable design patterns

**Functional Requirements**:
- One-command service startup (`./start.sh` or equivalent)
- Data refresh capability with schema validation
- Automated error handling and logging
- Mobile-responsive dashboard design

**Performance Requirements**:
- Dashboard load times under 3 seconds
- Data refresh completion under 1 minute
- Support for concurrent users (simulated)

### Bonus Challenges

#### Advanced Analytics (Optional)
- Implement customer segmentation analysis
- Add predictive analytics for sales forecasting
- Create cohort analysis for customer retention
- Build real-time alerting for key metrics

#### DevOps & Automation (Optional)
- Implement CI/CD pipeline for dbt models
- Add automated testing for dashboard functionality
- Create backup and recovery procedures
- Review the finished repository and propose two automations that would improve the workflow, whatever form your agent supports

#### Advanced Visualizations (Optional)
- Interactive drill-down capabilities
- Custom KPI widgets
- Embedded analytics in mock web application
- Mobile-first dashboard design

### Final Deliverables

#### 1. Technical Documentation Package
- Architecture decision record (ADR) for all technology choices
- Database schema documentation
- dbt model documentation (auto-generated)
- BI platform configuration guide
- System operation manual

#### 2. Live Demonstration (20 minutes)
- System startup demonstration
- Dashboard walkthrough with business insights
- Data refresh process demo
- Q&A session on technical decisions

#### 3. AI Development Process Report
- Log of AI interactions and research queries
- Documentation of how AI assisted in each phase
- Log of which agent capabilities you used, when, and which paid off most
- Reflection on AI-assisted vs traditional development approaches
- Recommendations for future AI-assisted projects

### Success Criteria

**Technical Excellence**:
- All services start successfully with single command
- Data pipeline processes without errors
- Dashboard loads and displays accurate data
- System handles data schema changes gracefully

**Business Value**:
- Dashboard answers key business questions
- Insights are actionable and clearly presented
- Data storytelling effectively communicates findings

**Professional Development**:
- Clear documentation of all technical decisions
- Confident presentation of system capabilities
- Demonstrated understanding of AI-assisted development workflows