---
title: "Mental Health NLP Analysis"
excerpt: "Transformer-based Text Classification <br/><img src='/images/crop_mh.jpg'>"
collection: portfolio
---
# Mental Health Natural Language Processing  
[Github Repo](https://github.com/bess-days/mental-health-analysis)

One in five people in the United States experiences mental illness. Recently, more individuals have openly discussed the effects of mental illness on their lives. Many organizations now collect public posts about mental health conditions. [Sentiment Analysis for Mental Health](https://www.kaggle.com/datasets/suchintikasarkar/sentiment-analysis-for-mental-health) is a public database containing social media posts from users self-identifying with a particular condition or none. This database aggregates 8 Kaggle datasets related to these conditions from Reddit, Twitter, and other sources. To gather one specific condition, they looked at subreddit thread titles or annotations done by humans with diagnostic mental health experience.

Each entry contains a unique ID, tweet, and the writer’s condition. Categories include prominent conditions—Depression, Suicidal, Anxiety, Stress, Bipolar, and Personality Disorder—as well as a control category, Normal.

## Steps for cleaning and processing the dataframe  
*(Refer to [eda.ipynb](https://github.com/bess-days/mental-health-analysis/blob/main/data/eda.ipynb))*

| Action | Remaining Rows (starts at 53,043) |
|---|---|
| Dropped duplicates and missing texts (using Pandas dataframe manipulation) | 51,073 |
| Removed non-English tweets (using Lingua) | 50,138 |
| Cleaning the text and removing any texts where nothing remains (lower casing, removing URLs, hashtags, etc), removing stop words, and lemmatizing the words using SpaCy and NLTK | 50,056 |

The final breakdown of conditions in the 50,056 rows:
<img src="/images/conditions.png">
**Figure 0**

# Word Associations

## 1.1 Introduction

The research question explores whether linguistic patterns are distinctive to specific groups. Specifically, are certain emotions, concerns, or concepts prevalent among individuals with mental health conditions? While each condition’s distinctiveness is intriguing, it may also offer practical advantages. For example, the dataset I used examines training chatbots to help doctors treat patients. If chatbots can detect emotions and identify condition-specific keywords, then customized treatment may become more achievable. 
*(Refer to [viz.ipynb](https://github.com/bess-days/mental-health-analysis/blob/main/viz/viz.ipynb))*

## 1.2 Methodology

After following the preprocessing steps above, two methods were used:

1. A **TF-IDF** vectorizer transformed the text and identified terms that were frequent in a specific document but not across others. The goal was to highlight words associated with a given condition, which could help train chatbots to identify it.
2. A **WordCloud** visualized the most common words with more than three letters that appeared in more than five tweets. Weights and frequencies were based on each word’s average frequency compared with its mean across all documents.

Condition-specific words (e.g., bipolar, depression) and the Normal category were removed from the dataframe before analysis.

## 1.3 Data Results


<img src="/images/anxiety.png">


### Figure 1.1 Anxiety Wordcloud

The Anxiety WordCloud includes words commonly associated with anxiety, such as 'worry', 'nervous', and 'restless'. Additionally, many words align with medical themes. Terms such as 'heart', 'doctor', and 'cancer' appear prominently, suggesting that medical concerns are frequently mentioned by individuals with anxiety in this dataset.


<img src="/images/depression.png"> 


### Figure 1.2 Depression WordCloud

People with depression focus on their emotions, using words like ‘feel’, ‘depress’, ‘hate’, ‘happy’, and ‘good’. This suggests they discuss emotions more than other groups do; specifically, they feel x, y, z, but are not x, y, z.

<img src="/images/suicide.png">

### Figure 1.3 Suicide Depression Word

When someone is suicidal, they often discuss life and death, as well as aspirations they believe are unattainable. Common keywords in these tweets include ‘kill,’ ‘die,’ ‘end,’ and some expletives.

<img src="/images/stress.png">


### Figure 1.4 Stress WordCloud

The stress word cloud features many situational words, with ‘work,’ ‘job,’ ‘study,’ ‘pay,’ and ‘home’ appearing frequently. These routine subjects are common triggers for people experiencing stress. While stress and anxiety are related, anxiety words concern the body and life-or-death issues, while stress words relate to uncontrollable situations. 'PTSD' and 'flashback' are mentioned which is interesting, most likely because PTSD doesn't correlate as much with other conditions, it was labled as stress by datacollectors.

<img src="/images/bipolar.png">


### Figure 1.5 Bipolar WordCloud

Bipolar disorder involves manic and depressive episodes. Words like ‘mania,’ ‘episodes,’ and ‘depressive’ are prominent. Many words focus on treatment rather than feelings. Terms like ‘diagnose’, ‘diagnosis’, ‘psychiatrist’, ‘med’, ‘medicine’, and ‘doctor’ appear. Medicines that treat bipolar disorder, such as ‘Lithium,’ ‘Lamictal,’ ‘Seroquel,’ and ‘Latuda,’ are discussed.

<img src="/images/personality.png">


### Figure 1.6 Personality Disorder WordCloud

Personality disorders are defined by a person exhibiting unnatural patterns and unhealthy thinking that differ from societal norms. One of the most common and well-known personality disorders is Avoidant Personality Disorder (AvPD). Descriptions of AvPD are also represented in the WordCloud, with sufferers discussing struggles ‘socially’ and with understanding relationships and ‘people.’

## 1.4 Conclusion

The results provided key insights into what distinguishes each mental health condition. One possible issue is the Depression TF-IDF. Because Depression is featured so prominently in the dataset, I wondered if that affected the TF-IDF - indeed, it did. Upon further testing, words like ‘feel’ and ‘like’ are among the most common across all conditions, but because of the class imbalance, the inverse often portrays these words as uniquely associated with Depression.
To summarize: 
* Anxiety was linked with health concerns.
* Depression focused on strong emotions.
* Suicidal thoughts emphasized life-and-death language.
* Stress centered on life circumstances such as jobs and finances.
* Bipolar Disorder and Personality Disorder highlighted condition-related terminology, with Bipolar focusing more on treatment and Personality Disorder focusing more on social and life struggles.

# Domain Specific Transformer Predictions

## 2.1 Introduction

My next research question addresses a model’s capacity to predict an author’s condition from a tweet. While there is literature concerning model training for depression detection, fewer studies compare multiple mental health conditions.

A model that can recognize linguistic patterns unique to conditions could:

* Help social workers find individuals in need of early prevention
* Serve as an educational resource so researches can better understand the experiences, challenges, and thought processes of individuals with a specific condition
* Support chatbots and therapists in responding to users more effectively.

That said, in my opinion, it should not be used in diagnosis or giving people labels. Rather serve as a tool for prevention and identifying individuals at risk.

This project specifically compared the performance of multiple transformer models on the classification task.

## 2.2 Methodology

The Normal category was removed because it was the majority class, and the goal was to identify which condition each tweet belonged to.

Tweets were cleaned by:

* Removing links, hashtags, and punctuation
* Converting text to lowercase
* Avoiding lemmatization for model training

Two transformer models were evaluated:

* DistilBERT
* [MentalRoBERTa](https://huggingface.co/mental/mental-roberta-base) (a transformer trained on mental health posts)

Based on documentation on the MentalRoBERTa Hugging Face page, the model’s training closely aligned with the goal of developing an automatic condition detector. MentalRoBERTa is for non-clinical uses and was trained on anonymous post to the public, not social media posts and not collecting user profiles.

I performed automatic tokenization and padded statements using the tokenizer specific to each model. To address class imbalance, I applied weighted training to balance class losses across groups. Training was accordingly weighted to compensate for class imbalance.

Shared Model Parameters
* Training epochs: 10
* Batch size: 24
* Early stopping patience: 2
* Learning rate: 2e-5
* Weight decay: 0.01
* Evaluation, saving, and logging strategy: epoch
* Best model loaded at the end based on Macro F1-score

## 2.3 Results

### 2.3.1 MentalRoBERTa  
*(Refer to [Mental_model.ipynb](https://github.com/bess-days/mental-health-analysis/blob/main/model/mental_model.ipynb))*

The results of running MentalRoBERTa were promising. Specifically, the model stopped early around epoch 9. The final training loss was 0.26, and the observed accuracy was 78%.

Therefore, emphasis was placed on the macro F1-score to account for class imbalance. Macro F1 considers both precision and recall, ensuring equal representation for all classes, including minority groups such as personality disorders. There was no need to penalize the model for failing to fit one class.

The highest macro F1-score achieved was 0.8213, within the 0.80-0.90 range, suggesting the model performs well.

This result is consistent with the macro F1-score calculated from the classification report following predictions.

<img src="/images/confusion.png">
### Figure 2.2 Confusion Matrix for Mental RoBERTa

The confusion matrix supported several initial hypotheses. Depression and Suicidal tweets were the most frequently confused categories, while Stress was occasionally mislabeled as Anxiety. Personality Disorder, being a minority class, also produced lower F1-scores and was more difficult to evaluate accurately.

<img src="/images/ROC.png">
### Figure 2.3 ROC curves for Mental RoBERTa

The ROC curves indicate that the results are relatively good, with the curves for Anxiety, Bipolar, Personality Disorder, and Stress close to 1 (AUC of .97-.99). This suggests that the model generally performs well at differentiating between the classes. However, Suicidal and Depression both have an AUC of .91, which is expected because those two conditions overlap.

### 2.3.2 Distill-Bert  
*(Refer to [distil.ipynb](https://github.com/bess-days/mental-health-analysis/blob/main/model/distil.ipynb))*

I will briefly discuss DistilBERT, as the main goal is to compare the performance of a base model with that of a mental health-specific pre-trained model.

Using identical parameters and data, I reran the model with DistilBertUncased as both the tokenizer and the pre-trained model. The results were comparable, with most metrics trailing the mental health pre-trained model by 1-5 points. The average Macro-F1 was 80%, which is at the lower end of what is considered very good for such models. The model struggles to accurately recognize and evaluate minority groups.


## 2.4 Conclusion

MentalRoBERTa outperformed DistilBERT across most evaluation metrics.  

<img src="/images/table.png">

Table 2.6 Comparison of DistilBERT and MentalRoBERTa - the cells marked in green are the highest scores between the two models

Key Takeaways
- Recall on Bipolar disorder meaning the model trained on Mental RoBerta produced fewer false negatives, which is important as missing a diagnosis can be critical. DistilBERT seems to only identify obvious Bipolar related text.
- Mental RoBERTa performed better in all metrics for the data's minority classes Personality Disorder (1.8% of data) and Stress (4.6% of data). This proves that its domain-specific training helps it identify more specific/minor patterns more accurately without missing users.
- While DistilBERT has slighlty higher precision that Mental RoBERTa in the Suicidal category, Mental RoBERTa is better at recognizing nuanced references to suicide. The latter model raises a few more false alarms (lower precision), but it catches more people in crisis (higher recall). This is critical for intervention.
- There was no difference in performance between the F1 in Depression and Anxiety, and similar performance on other metrics. As the most prevelant conditions, not just in this dataset, but in general, means Depression and Anxiety related language would be covered significantly by DistilBERT's training data.

The findings support the hypothesis that a domain-specific pre-trained transformer performs better than a broader general-purpose model for mental health classification tasks.

