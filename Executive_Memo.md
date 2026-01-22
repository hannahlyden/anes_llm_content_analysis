# Executive Memo
## Evaluating LLMs for Automated Coding of Public Opinion Survey Responses

**Author:** Hannah Lyden  
**Date:** January 2026  

---

## 1. Background and Motivation
- The goal of this project is to explore whether large language models (LLMs) can assist in coding open-ended survey responses.  
- Open-ended responses, like “What is the most important problem facing the country?”, provide rich insight but are labor-intensive to code.  
- Automating coding could save time while maintaining consistency, but there are risks of bias or misclassification.

---

## 2. Data
- **Dataset:** ANES 2024 Time Series Survey  
- **Sample size:** ~4,500 respondents  
- **Key variables:**  
  - `response_text` — respondent’s text answer  
  - Demographics: age, gender, race, education, marital status, occupational status, income, and party ID  
- **Human-coded labels:** Coded into 10 categories (Economy, Healthcare, Politics, Social Issues, Crime, Immigration, Environment, Foreign Policy, Other, Unclear)  

---
## 3. Methods
- **Coding scheme:**  
  - 10 categories with definitions and examples  

| Code | Category | Definition | Example Response |
|---|----------|----------------------|---------------|
| 01 | Economy/Inflation/Jobs | Mentions inflation, jobs/unemployment, cost of living, recession, economic growth, poverty | “Prices are rising and wages aren’t keeping up.” |
| 02 | Health/Healthcare | Mentions healthcare access, cost of medical care, insurance, public health issues | “Healthcare is too expensive.” |
| 03 | Politics/Government/Democracy | Mentions government dysfunction, leadership, elections, political polarization, trust in institutions | “The government is broken.” |
| 04 | Social Issues/Inequality | Mentions racism, discrimination, inequality, social division (excluding strictly |political or economic framing) | “Racial inequality is my biggest concern.” |
| 05 | Crime/Public Safety | Mentions crime, violence, policing, safety | “Crime in my city is out of control.” |
| 06 | Immigration | Mentions immigration, border control, refugees, undocumented immigration | “Illegal immigration is the biggest issue.” |
| 07 | Environment/Climate Change | Mentions climate, environment, pollution | “Climate change is destroying our future.” |
| 08 | Foreign Policy/National Security | Mentions war, foreign relations, terrorism|  “Foreign policy is weak.” |
| 09 | Other | Doesn’t fit above categories or mentions something unique | “Education quality is my biggest concern.” |
| 10 | Unclear | Response is indecipherable, non-substantive, non-response, self-contradictory | “Things aren’t good.” |

### Decision Rules for Coding Open-Ended Responses

To ensure consistency between human and LLM coding, the following decision rules were applied:

1. **Primary Issue Rule**
   - Each response is assigned to **one and only one category**.
   - When multiple issues are mentioned, code the **first or most emphasized issue**.
   - Emphasis is determined by:
     - Order of mention
     - Amount of detail devoted to an issue
     - Use of intensifiers (e.g., “biggest,” “most important”)
  - When two or more issues receive approximately equal emphasis, assign the category that:
    - Appears first in the response 
    - Receives the most descriptive language
    - Is most consistent with common public discourse surrounding the stated concern

2. **Specific-over-General Rule**
   - If a response mentions both a general concern and a specific issue, code the **more specific issue**.
   - Example:
     - “The economy and inflation” → *Economy*
     - “Government corruption and politicians not doing their jobs” → *Politics*

3. **Issue vs. Evaluation Rule**
   - Responses that criticize leaders, parties, or institutions without naming a policy area are coded as *Politics*.
   - Responses that name a substantive issue (e.g., healthcare costs, immigration enforcement) are coded to that issue area.
   - Responses referencing social division, polarization, or lack of social cohesion without explicit reference to political institutions or actors were coded as *Social Issues*.
   - Human rights–related responses were coded as *Politics/Democracy* when framed in terms of government institutions, laws, or democratic norms, and as *Social Issues* when framed in terms of societal discrimination or group-level treatment without explicit reference to political institutions.

4. **Ambiguous or Vague Responses**
   - Responses are coded as *Unclear* when they do not express a discernible issue or policy domain, including:
    - Vague or non-substantive statements (e.g., “Everything is a mess,” “The way things are going”)
    - Expressions of uncertainty or nonresponse (e.g., “Don’t know,” “No opinion”)
    - Statements lacking sufficient information to infer a meaningful topic
  - Responses are coded as *Other* when they clearly reference a substantive concern but do not fit any of the predefined categories. These include:
    - Issues outside the listed policy domains
    - Idiosyncratic or uncommon concerns that are interpretable but uncategorizable within the scheme
  - Short but interpretable responses (e.g., “Prices,” “Crime,” “Healthcare”) are coded to the most appropriate substantive category, not *Unclear*.
  - *Unclear* should be used only when no reasonable inference about the respondent’s intended issue can be made.

5. **Out-of-Scope Responses**
   - Responses not directly related to public policy (e.g., personal problems, jokes, non-answers) are coded as *Other*.
   - Examples:
     - “My personal finances”
     - “I don’t know”
     - “Nothing really”

6. **Foreign vs. Domestic Issues**
   - Responses explicitly referencing other countries, wars, or international relations are coded as *Foreign Policy*.
   - Domestic consequences of international issues are still coded as *Foreign Policy* unless the respondent clearly emphasizes domestic policy impacts.

7. **Consistency Checks**
   - Human coders and the LLM were provided identical category definitions and decision rules.
   - Responses that did not clearly fit any category were flagged during evaluation.

### Research Process
- **Human coding:**  
  - Hand-coded ~300 responses to establish gold standard  
- **LLM coding:**  
  - Batch processed responses using structured prompt  
  - Output restricted to one category per response
  - The model was instructed to return a category label, a confidence assessment, and a brief rationale for each classification, enabling systematic audit of model decisions.
- **Evaluation:**  
  - Metrics: Accuracy, Precision, Recall, F1 per category, confusion matrix  
  - Subgroup analysis by party ID, age, education

  |   Economy |   Healthcare |   Politics/Democracy |   Social Issues |   Crime |   Immigration |   Environment |   Foreign Policy |   Other |   Unclear | 
  |----------|-------------|---------------------|----------------|--------|--------------|--------------|-----------------|--------|----------|
  |  87 |  0 |  0 |  1 |  0 |  2 |  0 |  0 |  1 |  0 |
  |  0 |   11 |   0 |   0 |  0 |  0 |   0 |   0 |   0 |   0 |
  |  4 |   0 |  47 |   1 |  0 |  0 |  0 |   0 |   1 |   1 |
  |  2 |   0 |   3 |  52 |  0 |   0 |  0 |  0 |   0 |  0 |
  |  0 |   0 |   0 |   1 |   6 |   0 |   0 |  0 |  0 |  0 |
  |  1 |   0 |   0 |   0 |   0 |   50 |   0 |   0 |    0 |  1 |
  |  0 |   0 |   0 |   0 |  0 |   0 |   5 |   0 |   0 |    0 |
  |  0 |   0 |   0 |   0 |   1 |   0 |   0 |   4 |   0 |   0 |
  |  0 |   0 |   0 |   0 |   0 |   0 |    0 |    0 |   3 |   1 |
  |  0 |   0 |   3 |   0 |  0 |    0 |   0 |   0 |  1 |  10 |

## Analysis
- **Outcome variables**: Binary indicators for LLM-coded primary issue categories (*Economy*, *Politics/Democracy*, *Unclear*).
- **Predictors**: Demographics including age, income, gender, race, marital status, occupation, education, and party ID.
- **Category collapsing**:
    - *Education*: Less than HS, High school, Some college, Bachelor’s degree, Graduate degree, No info
    - *Party ID*: Democrat, Republican, Independent, No info
- **Statistical approach**:
    - Logistic regression models estimated the probability of a response being coded to a given category (e.g., *Economy*, *Politics/Democracy*, *Unclear*) based on demographic predictors.
    - Predicted probabilities were calculated holding continuous variables (age, income) at their mean values to illustrate subgroup differences.
- **Visualization**: Predicted probabilities plotted by education and party ID to show demographic patterns in LLM-coded category assignments.

---

## 4. Results
**Human vs. LLM Coding Agreement**
- Overall agreement of LLM vs human coding for 300 hand-coded responses (accuracy 91.6%)  
- Category-level performance highlights:
  - Best-performing categories: *Healthcare*, *Environment*, and *Foreign Policy*
  - Worst-performing categories: *Other*, *Unclear*, and *Crime*
- Confusion matrix shows which categories are most often misclassified  
- Subgroup analysis: small variations by party ID and education

**LLM Coding Results**
- Large amount of *Unclear*-coded responses (42.6%)
  - Audit of 50 *Unclear*-coded responses revealed that:
    - Most *Unclear* responses were not truly ambiguous.
    - Misspellings and multi-category responses caused LLM misclassification.
    - A human could assign many of these to Economy, Healthcare, etc.
    - Only ~2 of 50 cases were genuinely uninterpretable.


| LLM Code Label   |   proportion |
|-------------------|-------------|
| Unclear    |  42.6 |
| Economy    |  20.6 |
| Politics/Democracy |   10.2 |
| Social Issues      |          9.7 |
| Immigration        |          7.8 |
| Healthcare         |          3.3 |
| Crime              |          2   |
| Environment        |          1.6 |
| Foreign Policy     |          1.1 |
| Other              |          1   |

### Qualitative Audit Findings
A qualitative audit of misclassified responses indicates that LLM errors are systematic and interpretable, rather than random. The most common error type involved responses that mentioned multiple policy areas. In these cases, the LLM frequently defaulted to *Economy* when economic terms (e.g., “inflation,” “spending”) appeared, even when *Politics/Democracy* or *Social Issues* were more emphasized overall. This pattern suggests that the model overweighted salient economic keywords relative to respondent emphasis, contrary to the primary issue decision rule.

A second recurring error type involved abstract or evaluative statements (e.g., “lack of integrity,” “progressive agenda,” “fundamental human rights values”). Human coding treated these responses as *Politics/Democracy* or *Unclear* depending on whether a specific policy domain could be reasonably inferred. By contrast, the LLM inconsistently distinguished between issue-based content and normative evaluations, occasionally assigning substantive categories to responses that lacked a clearly articulated policy focus.

Additional discrepancies arose in boundary cases between closely related categories, particularly *Crime* versus *Foreign Policy* (e.g., national security framed as domestic protection) and *Social Issues* versus *Politics/Democracy* (e.g., social division or media polarization). A small number of cases involved out-of-scope or nonresponsive answers; in these instances, the LLM appropriately flagged *Unclear*, while human coding sometimes retained *Other* to preserve respondent intent.

Importantly, the high prevalence of *Unclear* classifications does not primarily reflect random model error. Instead, it reflects a conservative and rule-consistent application of the coding scheme to abstract, diffuse, or multi-issue responses. While many such responses are substantively interpretable to a human reader, recoding them would require additional inference beyond respondents’ explicit statements. We therefore retain *Unclear* as a valid analytic category, consistent with best practices in open-ended survey coding. Future applications could pair LLM coding with targeted human review to recover additional cases where interpretability is high but automated classification is uncertain.

Overall, the audit indicates that LLM performance is strongest for single-issue, concrete responses, with errors concentrated in multi-issue statements, abstract evaluations, and category-boundary cases. These findings informed refinements to the decision rules and provide clear guidance for prompt design and post-coding validation in similar applications.

### Demographic Predictors of Issue Classification

To assess whether LLM-coded issue categories varied systematically across respondent characteristics, logistic regression models were estimated predicting the likelihood that a response was coded to selected categories as a function of party identification, education, age, and income. These models reveal clear and substantively interpretable demographic patterning in LLM-coded outputs.

Responses coded as *Unclear* were significantly more likely among respondents with lower levels of education. Compared to respondents with a bachelor’s degree (the reference category), those with less than a high school education, a high school diploma, or some college were all more likely to receive an *Unclear* classification. Respondents with missing party identification were also more likely to be coded as *Unclear*. Figure 1 presents predicted probabilities of an *Unclear* classification by education level, holding age and income at their sample means and party identification constant. The figure shows a pronounced education gradient, with the highest predicted probability among respondents with less than a high school education and progressively lower probabilities among more highly educated respondents.

![Figure 1: Likelihood of Unclear LLM-coded Response by Education Level](<Figures/unclear by education.png>)

Substantive issue categories also exhibited systematic demographic variation. Independent respondents, respondents with no party identification, and Republicans were significantly more likely than Democrats to cite the *Economy* as the most important problem, while respondents with graduate degrees were less likely than those with bachelor’s degrees to do so. By contrast, *Politics/Democracy* responses were disproportionately concentrated among Democrats and among more highly educated respondents; individuals with less than a bachelor’s degree were significantly less likely to be classified into this category. Age was positively associated with *Politics/Democracy* and negatively associated with *Economy*, indicating that older respondents were more likely to focus on institutional and democratic concerns, while younger respondents were more likely to emphasize economic issues.

Taken together, these results suggest that LLM-coded categories capture meaningful and theoretically plausible demographic patterns, lending additional support to the validity of the automated coding approach.

![Figure 2: Predicted Probabilites for Citing Economy or Politics/Democracy by Education and Party ID](<Figures/economy:politics by educ and party.png>)
---

## 5. Discussion

This analysis demonstrates that large language models can reliably code open-ended public opinion responses into broad issue categories, with high agreement relative to human coding for concrete, single-issue responses. Categories such as Healthcare, Environment, and Foreign Policy show particularly strong performance, indicating that LLMs are well suited for domains with distinctive vocabulary and clear issue boundaries.

At the same time, the results highlight important and predictable limitations. The LLM struggled most with responses that were abstract, evaluative, or referenced multiple policy areas. In these cases—especially when economic language appeared alongside political or social concerns—the model often defaulted to salient keywords rather than applying higher-order judgment about respondent emphasis. This behavior is consistent with known properties of language models and underscores the importance of carefully specified decision rules.

Notably, the large share of responses coded as Unclear reflects a conservative and rule-consistent application of the coding scheme rather than model instability or noise. Qualitative audit evidence shows that many of these responses are interpretable to a human reader but require inferential judgment beyond what is explicitly stated. Retaining Unclear as a substantive category therefore preserves analytic transparency and avoids over-interpreting ambiguous responses.

Demographic analysis further suggests that Unclear classifications are systematically patterned, particularly by education level and information availability, reinforcing that ambiguity in open-ended responses is itself a meaningful feature of the data rather than a technical artifact. Together, these findings suggest that LLM-based coding is best understood as a complement to—rather than a replacement for—human judgment in open-ended survey analysis.

---

## 6. Recommendations

Based on the results of this evaluation, the following best practices are recommended for applying LLMs to open-ended survey coding:

  1. **Use LLMs as a first-pass coder, not a full substitute for humans.**
    LLMs perform well for clearly articulated, single-issue responses and can dramatically reduce manual coding burden. However, abstract and multi-issue responses benefit from human review.

  2. **Retain Unclear as a valid analytic category.**
    Attempts to force classification of ambiguous responses risk introducing researcher bias. Treating Unclear as substantively meaningful aligns with established survey coding norms and preserves interpretive discipline.

  3. **Pair automated coding with targeted human review.**    
    Rather than re-coding the full dataset, researchers can selectively audit:
      - Responses flagged as Unclear
      - Low-confidence LLM outputs
      - Boundary cases between closely related categories
    This hybrid approach balances scalability with accuracy.

  4. Design prompts and decision rules to reflect analytic priorities.
     If recovering multi-issue responses is a core goal, prompts can be extended to allow secondary categories or hierarchical coding. Alternatively, analysts can prioritize high-precision primary issue identification, as done here.

  5. Incorporate validation checks into the workflow.
      Confusion matrices, subgroup analyses, and qualitative audits should be treated as standard components of any automated coding pipeline to ensure transparency and reproducibility.

Overall, this project demonstrates that LLMs are a powerful tool for open-ended survey analysis when used thoughtfully, with explicit decision rules and validation procedures. Their greatest value lies not in replacing human coders, but in enabling faster, more consistent analysis while preserving methodological rigor.  