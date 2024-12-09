---
authors:
- fjooord
categories:
- Case Studies
comments: true
date: 2024-12-05
description: Discover insightful advice for young people, emphasizing choices, confidence,
  and personal growth through lived experiences.
draft: false
slug: ai-report-generation-style-matching-case-study
tags:
- prompt engineering
- chain of thought prompting
- structured output
- case study
---

# You Don't Need to Fine-Tune to Match YOUR Writing Style
## How You Can Build an AI Report Generator That Perfectly Matches Your Writing Style

**"This doesn't sound like us at all."**

It's the common refrain when organizations try using AI to generate reports, documentation, etc. 

While AI can produce grammatically perfect content, it often fails at the crucial task of matching an organization's voice - transforming from minor inconvenience to major blocker when scaling across thousands of documents.

Using a novel two-step approach focused on separating style from data, we achieved:

- Near perfect style matching according to practitioner feedback
- Zero data contamination from example reports across all test cases
- A scalable solution that works for 10 or 1000 users
- No need for expensive fine-tuning or ML expertise

## The Challenge: Beyond Simple Style Matching

Our client, a report writing service for psychologists, presented an even more complex challenge. 

They needed AI that could adapt to hundreds of distinct writing styles - one for each practitioner on their platform.

<!-- more -->

These weren't just stylistic preferences. Each psychologist had developed their reporting approach over years, carefully tailored to their specific patient populations. 

A generic AI voice wouldn't just feel wrong - it risked undermining the trust these practitioners had built with their patients.

The conventional solution would be fine-tuning - training custom AI models for style matching. 

But this approach brings its own challenges: significant data requirements, expensive MLOps expertise, and complex infrastructure maintenance. 

For many organizations, especially those supporting multiple users, it's prohibitively expensive and time-consuming.

We needed a different approach: one that could match writing styles precisely through prompt engineering alone, making it accessible to organizations without ML expertise.


## Iterating Toward a Solution

Since fine-tuning wasn't an option, we focused on pushing the boundaries of prompt engineering. 

Our journey through increasingly sophisticated prompting approaches revealed key insights about handling style and data separately.

### First Attempts: Simple Few-Shot Learning

Our initial approach was straightforward - feed the model example reports and ask it to generate new ones in the same style. While this worked occasionally, we quickly hit two major issues:

1. Inconsistent style matching across different reports
2. Information leakage - test results and diagnoses from example reports appearing in new ones

This second issue was particularly concerning. We couldn't risk patient data from example reports contaminating new ones.

### Moving to Multiple Chain of Thought Monologues

To address these issues, we tried breaking the problem down. We added structured analysis steps before generation:

- First analyzing the example's structure
- Then separately examining style elements like tone and word choice
- Finally generating new content

The style matching improved significantly, but we still couldn't reliably prevent information leakage from example reports.

We needed a fundamentally different approach to handling sensitive information while preserving style.

!!! info "Structured Monologues"

    Interested in this approach? Check out this post!

## The Breakthrough: Separating Style from Data

Our earlier attempts revealed a crucial insight: 

- **We were conflating two distinct challenges.**
- The model wasn't just struggling with style matching - it was simultaneously trying to manage sensitive patient data.

This realization led to our key innovation: what if we stripped the examples of all patient-specific information before using them for style matching?

Rather than asking the model to juggle multiple complex tasks:

- Understanding writing style
- Managing sensitive data
- Generating new content
- Ensuring data accuracy

We could break this into a clean, two-step process:

1. First, sanitize example reports by replacing specific data points with generic placeholders
2. Then use these "cleaned" examples to guide generation of new reports

This approach solved both our core challenges: maintaining style consistency while eliminating the risk of data contamination.

### Step 1: Example Sanitization

Our sanitization process uses a consistent set of generic placeholders to preserve report structure while removing sensitive data:

#### Prompt for Sanitization
```markdown
# Purpose
You are a system designed to identify and replace all client-specific data (e.g., Test Scores, Qualitative Descriptions, etc.) with generic identifiers.

# Context
Our users provide examples of reports that they have written in the past so that their writing style can be used to generate new reports
We have been having issues with the newly generated reports containing client-specific data from the examples
    - This contamination is unacceptable since the data in the examples are not representative of the report for the new patient
You are going to replace the client-specific data with generic identifiers so the example clearly designates:
    - What needs to stay (ie the writing style and format)
    - What needs to be changed (test names, scores, etc)

# Task
Replace all the client-specific data with generic identifiers

# Types of Client-Specific Data to Replace
## Test Names
### Definition
Names of specific psychological assessments or subtests.
### Examples
- Matrix Reasoning
- Block Design
- Vocabulary
### Replacement Identifier
[TEST_NAME]

## Test-Specific Metrics
### Definition
Quantitative measures or scores unique to psychological tests.
### Examples
- FSIQ
- VCI 
- PRI
- WMI
- PSI
- Scaled scores (e.g. "scaled score = 13")
### Replacement Identifier
[TEST_METRIC]

### Definition
Quantitative scores unique to psychological tests.
### Examples
- Standard scores (e.g. "Standard score = 100")
- Scaled scores (e.g. "scaled score = 13")
- T scores (e.g. "T score = 50")
- Composite scores (e.g. "Composite score = 100")
### Replacement Identifier
[TEST_SCORE]


## Qualitative Descriptions
### Definition
Qualitative descriptions of performance or ability.
### Examples
- Exceptionally Low
- Below average
- Low average
- Average
- High average
- Above average
- Exceptionally High
### Replacement Identifier
[QUALITATIVE_DESCRIPTION]


## Normative Comparisons
### Definition
Comparisons to peers or population norms using percentiles or proportions.
### Examples
- Better than 50% of peers
- Exceeds 75% of her peers
- Performs at the 80th percentile
### Replacement Identifier
[PEER_COMPARISON]

# Rules
- Only replace the identifiers that are defined above
    - Do not make up new identifiers
- Do not change the format or structure of the report section, only replace the identifiers

# Instructions
- Read the report section thoroughly
- Identify all the domain-specific identifiers in the report section
- Replace all the domain-specific identifiers with the appropriate replacement identifier
```
!!! info "Prompt Design"

    Interested in how to design highly performant prompts? Check out this post!

#### Example Input/Output

##### Input
```plaintext
PIIPERSON1's language skills were assessed using the WISC-V and BASC
His performance on the WISC-V Verbal Comprehension Index (VCI) resulted in a composite score at the 30th percentile, indicating his language comprehension and verbal processing are average for his age group
The BASC Functional Communication scale was used to evaluate how PIIPERSON1's communication skills are viewed in different environments
His mother reported his functional communication at home as average, with a T score of 59
His current (second-grade) teacher also rated his communication skills as average in the classroom with a T score of 49
His first-grade teacher, however, had concerns, noting his abilities as at-risk with a T score of 39
Below Average scores on the Functional Communication scale indicate that the student demonstrates poor expressive and receptive communication skills and/or may have difficulty seeking out and finding information on their own
```

##### Output
```plaintext
PIIPERSON1's language skills were assessed using the [TEST_NAME] and [TEST_NAME]
His performance on the [TEST_NAME] [TEST_METRIC] resulted in a [TEST_SCORE] at the [PEER_COMPARISON], indicating his language comprehension and verbal processing are [QUALITATIVE_DESCRIPTION] for his age group
The [TEST_NAME] Functional Communication scale was used to evaluate how PIIPERSON1's communication skills are viewed in different environments
His mother reported his functional communication at home as [QUALITATIVE_DESCRIPTION], with a [TEST_SCORE]
His current (second-grade) teacher also rated his communication skills as [QUALITATIVE_DESCRIPTION] in the classroom with a [TEST_SCORE]
His first-grade teacher, however, had concerns, noting his abilities as [QUALITATIVE_DESCRIPTION] with a [TEST_SCORE]
[QUALITATIVE_DESCRIPTION] scores on the Functional Communication scale indicate that the student demonstrates poor expressive and receptive communication skills and/or may have difficulty seeking out and finding information on their own
```

### Step 2: Report Generation
With sanitized examples in hand, generation becomes straightforward. The model receives:

- Patient data for the new report
- 2-3 sanitized example reports
- Clear mapping of what each placeholder represents

The only modification to our generation process is ensuring the model understands the placeholder system and its role in preserving style while inserting new patient data.

## Results: Precision Without Compromise

Our two-step approach delivered measurable improvements across key metrics:

- 0% data contamination incidents across our test set
- Feedback indicated that practitioners found the new reports matching their style

Most importantly, our approach scaled. 

Whether handling 10 practitioners or 1000, the system maintained consistent performance without requiring additional training or fine-tuning.

## Key Takeaways

Our experience revealed that successful AI style matching isn't about more complex models or larger datasets. 

Instead, focus on:

- Breaking complex tasks into manageable steps
- Prioritizing data safety from the ground up
- Building systems that scale without requiring ML expertise
- Starting simple and iterating based on real-world usage

The beauty of this approach lies in its simplicity - it delivers enterprise-grade results using prompt engineering alone.

## Interested in Implementing This Yourself?
Check out the [open source implementation](https://github.com/Lascari-AI/Report-Style-Transfer) to start building your own style-matching system.

!!! cta "Need Help?"
    Building AI systems can be complex. 

    [Let's discuss your project](/services) and find the right approach for your needs.

<script async data-uid="2c30d1cf0f" src="https://fjooord.ck.page/2c30d1cf0f/index.js"></script>
