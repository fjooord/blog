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

If you've ever used AI to help write reports or documentation, you know the frustration: the output reads like it was written by someone else entirely. 

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

### Structured Output Monologues

We then tried a more sophisticated approach using structured monologues:

- Break down the writing style into explicit components
- Using multiple chain-of-thought sections
- Guide the model through style analysis and reproduction

While this showed promise, we hit two roadblocks:

- Implementation challenges in the client's .NET environment
- Complex prompt structures leading to occasional hallucinations

!!! info "Structured Monologues"

    Interested in this approach? Check out this post!

### 2. Model Fine-tuning Approaches

!!! warning "Fine Tuning is the Last Resort"

    Always exhaust your prompting options before fine tuning.  Fine tuning is expensive, time consuming, and requires heavy monitoring and testing to ensure it's working.

    Meta has a great post on [when to fine tune](https://ai.meta.com/blog/adapting-large-language-models-llms/) that you should check out.

### Individual User Models

Our first instinct was to create personalized models for each psychologist. The concept was simple on paper:

- Collect writing samples as users interact with the system
- Fine-tune separate models for each user
- Use these personalized models for future generations

Why we didn't proceed:

- Resource intensive (hosting hundreds of models)
- Required significant amounts of training data per user
- High computational costs for maintenance
- Not feasible for a product still finding market fit

### Style Clustering

We then considered a middle-ground approach:

- Analyze writing patterns across all users
- Identify 3-4 distinct writing style clusters
- Fine-tune a model for each cluster
- Match new users to the closest style cluster

Limitations:

- Still required significant data collection
- Writing styles often too nuanced for broad categorization
- Risk of oversimplifying unique writing voices
- User dissatisfaction with "close but not quite" matches, thus losing all the time and money invested in the style fine tunes

## The Breakthrough: Multi-Step Prompting with Sanitization

After these experiments, we realized we needed to solve two distinct problems:

1. Style preservation
2. Information containment

This led to our two-step approach:

### Step 1: Example Sanitization

First, we developed a systematic way to strip examples of specific content while preserving style:

The prompt below is designed to remove any information in the examples that is specific to the previous patient and thus can confuse the model and cause hallucinations

[EXAMPLE_REPLACEMENT_TAG]

Key benefits:

- Clear separation between what needs to be replicated (style and structure) and what needs to be ignore/replaced (client specific data)
- Reduced risk of information leakage
- More focused examples for the model

### Step 2: Structured Generation

Our generation process evolved to use these sanitized templates:

- Input: Patient data + sanitized style template
- Process: Guided replacement of generic identifiers
- Output: Styled content with correct patient information

## Implementation Details

The success of this approach relies heavily on careful prompt engineering. Here's what we learned:

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

If you're building an AI system that needs to match specific writing styles while handling sensitive information, here's how you can implement our approach:

1. Start with our generalization prompt framework [PROMPT_TAG]
2. Create your own set of generic identifiers based on your domain
3. Implement validation checks for your specific use case
4. Test thoroughly with a small set of users before scaling
5. Monitor and iterate based on user feedback

### Key Considerations

- Start small with a core set of identifiers
- Build robust validation into your pipeline
- Create clear documentation for your team
- Develop a process for handling edge cases

The key is to remember that sometimes, the best solution isn't about building more complex models—it's about breaking down the problem into manageable, safer steps.

Have you faced similar challenges with AI writing style matching? I'd love to hear about your experiences in the comments below.