# Community Resilience NLP: Emergency Detection and Prioritization from Hurricane Tweets

**An NLP and LLM-Based Framework for Emergency Detection and Prioritization in Natural Disaster Response**

Muna Kandel and Sujing Wang · Department of Computer Science, Lamar University

---

## Overview

During natural disasters, people post huge numbers of social media messages: requests for help, offers of support, and situational reports. This data is noisy, unstructured, and arrives too fast for response teams to sort by hand.

This project builds an end-to-end framework that turns disaster-related tweets into actionable intelligence. It does four things:

1. **Detects** emergency messages.
2. **Classifies** each message by its role and content.
3. **Ranks** emergencies by urgency.
4. **Matches** emergency requests with relevant support offers.

The framework is designed to work across disaster types. In this study, it is evaluated on **Hurricane Sandy** social media data.

## Two Classification Tasks

- **Task 1: Role-based classification.** Emergency, support offer, or neutral.
- **Task 2: Content-based classification.** Informative, personal, or other.

## Key Results

### Transformer models (test set)

| Task | Model | Accuracy | Macro F1 |
|---|---|---|---|
| Role-based | **DistilRoBERTa ★** | **0.8000** | **0.8036** |
| Role-based | RoBERTa | 0.7828 | 0.7787 |
| Role-based | BERT | 0.7690 | 0.7599 |
| Role-based | DistilBERT | 0.7517 | 0.7435 |
| Content-based | **DistilBERT ★** | **0.8226** | **0.8074** |
| Content-based | DistilRoBERTa | 0.8104 | 0.7936 |
| Content-based | BERT | 0.8012 | 0.7830 |
| Content-based | RoBERTa | 0.7982 | 0.7810 |

### Baseline models (test set, TF-IDF features)

| Task | Model | Accuracy | Macro F1 |
|---|---|---|---|
| Role-based | Logistic Regression | 0.6966 | 0.6928 |
| Role-based | Linear SVM | 0.7000 | 0.6916 |
| Role-based | Naive Bayes | 0.6897 | 0.6629 |
| Content-based | Logistic Regression | 0.8104 | 0.8039 |
| Content-based | Naive Bayes | 0.8043 | 0.7819 |
| Content-based | Linear SVM | 0.7890 | 0.7816 |

### Highlights

- **DistilRoBERTa** improved role-based macro F1 by **0.11** over the strongest baseline.
- **856 emergency tweets** were ranked by urgency: 44 Critical, 164 High, 349 Moderate, and 299 Low.
- **221 emergency–support pairs** were matched through semantic similarity.
- Zero-shot pseudo-labels reached **74.80% inter-model agreement** overall and **87.31%** for the emergency class.

## Methodology

The pipeline has six stages.

### 1. Integration
Eight Hurricane Sandy CSV files, totaling 13,858 tweets, were merged into a single dataset. Only fields relevant to the study were kept: tweet text, timestamps, user and location fields, and identifiers.

### 2. Preprocessing
Tweet text was lowercased and cleaned of URLs, mentions, repeated characters, and non-ASCII symbols. Hashtags were processed and whitespace was normalized. A **recency score** gives more weight to tweets posted closer to the disaster event.

### 3. Labeling
- **Role-based labels** come from zero-shot classification (cross-encoder/nli-deberta-v3-small) with confidence thresholds. Low-confidence tweets default to neutral.
- **Content-based labels:** missing labels were filled with a TF-IDF and Logistic Regression model. Low-confidence predictions were assigned to "other" and flagged.

### 4. Quality Control
- A retweet frequency feature captures broader public concern.
- **CleanLab** removed likely noisy labels: 893 records for role-based and 643 for content-based.
- Data was split 70/15/15 into train, validation, and test sets.

### 5. Training and Evaluation
- **Baselines:** Logistic Regression, Linear SVM, and Naive Bayes on TF-IDF unigram and bigram features.
- **Transformers:** DistilBERT, BERT, RoBERTa, and DistilRoBERTa, each fine-tuned for both tasks. Settings: max length 64, batch size 16, learning rate 2e-5, up to 10 epochs, early stopping, seed 42.
- Models were evaluated on accuracy and macro F1.

### 6. Actionable Intelligence
- **Urgency scoring:** emergency tweets with confidence of 0.70 or higher are scored as a weighted combination of model confidence (0.35), keyword severity (0.30), personal distress (0.20), and duplicate frequency (0.15). A message-type multiplier then adjusts each score, and tweets are grouped into Critical, High, Moderate, and Low tiers.
- **Ablation study:** removing model confidence caused the largest drop in mean urgency, from 0.4062 to 0.0731, confirming it as the strongest signal.
- **Semantic matching:** emergency and support tweets are encoded with all-MiniLM-L6-v2 and paired by cosine similarity. The mean match score is 0.4848 and the maximum is 0.6859.

## Repository Contents

- **hurricane_project.ipynb**: The main notebook covering the full pipeline, from integration through urgency ranking and matching.
- **data/**: Source tweet datasets.
- **Processed datasets**: The merged dataset, cleaned and labeled data, the final modeling dataset, and the imputed message types.
- **Output files**: Model comparison results, urgency-ranked emergency tweets, and final urgency tiers.
- **publishable_outputs/**: Figures and tables used in the paper.

Trained model weights (the BERT, DistilBERT, RoBERTa, and DistilRoBERTa checkpoints) are not included because of their size. They can be reproduced by running the notebook.

## Limitations

- The framework has been evaluated only on Hurricane Sandy data. Cross-disaster generalization is untested.
- All results come from historical, offline data. Real-time latency has not been benchmarked.
- CleanLab filtering left relatively small training sets, which raises the risk of overfitting.
- Urgency tiers and matches have not yet been validated by emergency management experts.

## Future Work

- Evaluate the framework on other disasters and on multilingual data.
- Have experts validate the urgency tiers and emergency–support matches.
- Add geospatial analysis.
- Benchmark throughput and latency.

## Authors

- **Muna Kandel**, M.S. Computer Science, Lamar University
- **Dr. Sujing Wang**, Department of Computer Science, Lamar University
