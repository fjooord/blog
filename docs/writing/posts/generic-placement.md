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
slug: ai-writing-style-matching-case-study
tags:
- prompt engineering
- chain of thought prompting
- structured output
- case study
---

# How You Can Build an AI Report Generator That Perfectly Matches Your Writing Style

If you've ever used AI to help write reports or documentation, you know the frustration

**The output reads like it was written by someone else entirely.**

For businesses trying to maintain a consistent voice across thousands of documents, this isn't just an annoyance—it's a dealbreaker.

## The Challenge: Making AI Write Like Your Organization

Every organization has its own voice. Whether it's technical documentation, business reports, or customer communications, that consistent style builds trust and professionalism. 

When AI generates content that doesn't match this voice, it creates friction and erodes confidence in your product.

<!-- more -->
### Our Client
We encountered this firsthand with a client providing report writing services to psychologists. They needed to automate report generation while preserving the unique writing style of hundreds of different practitioners.

Each psychologist had spent years developing their own approach tailored to their specific patient populations.

The challenge wasn't just matching one organizational voice—it was dynamically adapting to however many psychologists joined their platform.

Generic AI outputs weren't just unsuitable—they risked undermining the trust built between each practitioner and their patients.

Here's how we solved this exponentially more complex challenge.

## Exploring the Solution Space

When tackling a complex challenge like this, it's crucial to explore every potential solution—even ones that might seem imperfect at first glance.

!!! abstract "Explore Every Option"

    By examining different approaches, we can better understand the tradeoffs and limitations of each option. 
    
    This knowledge is invaluable for making informed decisions about which direction to pursue.

    [Contact us](/services) to explore solutions for your project.

Let's walk through each solution we explored, including the approaches we ultimately decided against. 

After all, understanding why certain solutions may not be suitable is just as important as knowing which ones could work!

### 1. Prompt Engineering Solutions

### Basic Few-Shot Learning

Our first attempt at prompt engineering was straightforward:

- Include 2-3 examples of the user's previous reports
- Ask the model to mimic the style
- Generate new content

Issues faced:

- Inconsistent style matching
- Example data contaminating the new report generation
    - The meaning that data from the example such as tests results, diagnoses, etc. from the example were being referenced as part of the new report generation
    - This is a huge 

### Structured Output Monologues

We then tried a more sophisticated approach using structured monologues:

- First, analyze example structure through a detailed monologue
- Follow with a separate style analysis monologue examining tone, word choice, and flow
- Finally, generate new content that incorporates both structural and stylistic elements

Results:

- Much better style matching
- Example data still contaminating the new report generation

!!! info "Structured Monologues"

    Interested in this approach? Check out this post!

### 2. Model Fine-tuning Approaches

!!! warning "Fine Tuning is the Last Resort"

    Always exhaust your prompting options before fine tuning.  Fine tuning is expensive, time consuming, and requires heavy monitoring and testing to ensure it's working.

    Meta has a great post on [when to fine tune](https://ai.meta.com/blog/adapting-large-language-models-llms/) that you should check out.

We are now at a fork in the road. We're having issues with prompt engineering not cutting it so far which is leading us to think that we could look into fine tuning but fine tuning is a very very difficult thing to do properly. 

You need to have clear look into your data and likely MLOps practitioners within your organization or your client's organization that can ensure continued training deployment and all the other things that come into play with fine tuning 

### Individual User Models

Our first instinct was to create personalized models for each psychologist. The concept was simple on paper:

- Collect writing samples as users interact with the system
- Fine-tune separate models for each user
- Use these personalized models for future generations

Why we didn't proceed:

- Resource intensive (hosting hundreds of models)
- Required significant amounts of training data per user
    - The clients system did not have an easy way to collect this data already built in
- Not feasible for a product still finding market fit

### Style Clustering

We then considered a middle-ground approach:

- Analyze writing patterns across all users
- Identify 3-4 distinct writing style clusters
- Fine-tune a model for each cluster
- Match new users to the closest style cluster

Limitations:

- Still required significant data collection
- Risk of oversimplifying unique writing voices
- User dissatisfaction with "close but not quite" matches, thus losing all the time and money invested in the style fine tunes

## The Breakthrough: Multi-Step Prompting with Sanitization

After these experiments, we realized we needed to solve two distinct problems:

1. Style preservation
2. Information containment

!!! tip

    Determine with your client the importance of each objective

    We found that the client was more concerned with information containment than with perfect style matching.

    At some point, you will need to make a trade off between the two objectives.

This led to our two-step approach:

### Step 1: Replacing Example Specific Data with Generic Placeholders

First, we developed a systematic way to strip examples of specific content while preserving style:
- Our previous experiments showed that the examples data were generally adjsuting the style and structure of the report
- The model was also while trying to mimic the style, determine what was "writing style" and "structure" and what was "client specific data"
- What if we break this process into multiple steps?
    - First, we identify what is client specific data and replace it with generic placeholders
    - Second, we ask the model to mimic the style and structure of the example, but now it doesn't have to determine what is client specific data and it only has to mimic the style and plug in the new patient data where necessary

The prompt below is designed to remove any information in the examples that is specific to the previous patient

#### Replacing Example Specific Data Prompt
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

Key benefits:

- Clear separation between:
    - what needs to be replicated (style and structure)
    - what needs to be ignored/replaced (example specific data)
- This resulting in much less likelihood of information leakage

### Step 2: Report Generation with Sanitized Examples

Our generation process stayed mostly the same:

- Inputs: 
    - Patient data
    - 2-3 examples of previous reports, but with the client specific data replaced with generic placeholders

- Process Change:
    - The only change was to inform the model in the system prompt that the examples are sanitized and what each of the generic placeholders mean

## Results
- Style matching performance was much better
- No information leakage

### Critical Components

1. **Identifier Definition**
    - Must be distinct and unambiguous
    - Should cover all variable content types
    - Need to be easily recognizable by the model
2. **Replacement Rules**
    - Clear guidelines for what should and shouldn't be replaced
    - Consistent formatting for identifiers
    - Preservation of surrounding context
3. **Validation Checks**
    - Verify no patient information leakage
    - Confirm style markers are preserved
    - Ensure all identifiers are properly replaced

## Ready to Implement This in Your Own System?

If you're building an AI report generation system that needs to match specific writing styles while handling sensitive information, here's how to adapt our approach :

1. Start with our prompt template below and customize the sections:

    - Replace [YOUR_DOCUMENT_TYPE] with "report" or your specific report type
    - Define [YOUR_DOMAIN_SPECIFIC_DATA] (patient data, test scores, etc)
    - List [EXAMPLES_OF_YOUR_DOMAIN_SPECIFIC_DATA] relevant to your reports

2. Create generic identifiers for your report data:

    - Map common data types (names, dates, metrics) to placeholders
    - Define clear rules for what should/shouldn't be replaced
    - Document identifier formats ([PATIENT_NAME], [TEST_DATE], etc)

3. Build validation checks specific to report generation:

    - Verify no sensitive information leakage
    - Confirm report structure remains intact
    - Check all identifiers are properly replaced

4. Test with a small batch of real reports:

    - Start with 2-3 example reports
    - Validate style preservation
    - Check for data contamination

5. Scale gradually and refine:

    - Monitor generated reports
    - Gather feedback from report writers
    - Update identifiers and rules as needed

### Prompt
```markdown
# Purpose
You are a system designed to identify and replace all [YOUR_DOMAIN_SPECIFIC_DATA] (e.g. [EXAMPLES_OF_YOUR_DOMAIN_SPECIFIC_DATA], etc) with generic identifiers.

# Context
Our users provide examples of [YOUR_DOCUMENT_TYPE] that they have written in the past so that their writing style can be used to generate new [YOUR_DOCUMENT_TYPE]
We have been having issues with the newly generated [YOUR_DOCUMENT_TYPE] containing [YOUR_DOMAIN_SPECIFIC_DATA] from the examples
    - This contamination is unacceptable since the data in the examples are not representative of the [YOUR_DOCUMENT_TYPE] for the new patient
You are going to replace the [YOUR_DOMAIN_SPECIFIC_DATA] with generic identifiers so the example clearly designates:
    - What needs to stay (ie the writing style and format)
    - What needs to be changed ([EXAMPLES_OF_YOUR_DOMAIN_SPECIFIC_DATA], etc)

# Task
Replace all [YOUR_DOMAIN_SPECIFIC_DATA] with generic identifiers

# Types of [YOUR_DOMAIN_SPECIFIC_DATA] to Replace
## [YOUR_DOMAIN_SPECIFIC_DATA_TYPE_1]
### Definition
[DEFINITION_OF_YOUR_DOMAIN_SPECIFIC_DATA_TYPE_1]
### Examples
[EXAMPLES_OF_YOUR_DOMAIN_SPECIFIC_DATA_TYPE_1]
### Replacement Identifier
[GENERIC_IDENTIFIER_FOR_YOUR_DOMAIN_SPECIFIC_DATA_TYPE_1]

...[REPEAT_FOR_ALL_YOUR_DOMAIN_SPECIFIC_DATA_TYPES] ...

# Rules
- Only replace the identifiers that are defined above
    - Do not make up new identifiers
- Do not change the format or structure of the [YOUR_DOCUMENT_TYPE] section, only replace the identifiers

# Instructions
- Read the [YOUR_DOCUMENT_TYPE] section thoroughly
- Identify all the domain-specific identifiers in the [YOUR_DOCUMENT_TYPE] section
- Replace all the domain-specific identifiers with the appropriate replacement identifier
```

## Key Takeaways

- Start with a minimal set of identifiers and expand based on needs
- Implement validation checks at every step of the pipeline
- Document everything - from identifier definitions to edge cases
- Focus on breaking down complex problems into manageable steps

The success of AI style matching isn't about throwing more compute at the problem. 

It's about thoughtful system design and careful handling of sensitive information.

!!! cta "Need Help?"
    Building AI systems can be complex. 

    [Let's discuss your project](/services) and find the right approach for your needs.