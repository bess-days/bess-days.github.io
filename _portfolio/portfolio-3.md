# Mental Health Natural Language Processing  
Github Repo: https://github.com/bess-days/mental-health-analysis

One in five people in the United States experiences mental illness. Recently, more individuals have openly discussed the effect of mental illness on their lives. Many organizations now collect public posts from people about mental health conditions. Sentiment Analysis for Mental Health is a public database containing tweets from users self-identifying with a particular condition or none. Each entry contains a unique ID, tweet, and the writer's condition. Categories include prominent conditions—Depression, Suicidal, Anxiety, Stress, Bipolar, and Personality Disorder—as well as a control category, Normal. The dataset combines eight separate Kaggle datasets with raw text.

## Steps for cleaning and processing the dataframe  
*(Refer to `group5_scriptfile_eda.ipynb`)*

| Action | Remaining Rows (starts at 53,043) |
|---|---|
| Dropped duplicates and missing texts | 51,073 |
| Removed non-English tweets (using Lingua) | 50,138 |
| Cleaning the text and removing any texts where nothing remains (lower casing, removing URLs, hashtags, etc), removing stop words, and lemmatizing the words | 50,056 |

The final breakdown of conditions in the 50,056 rows:
<img src="/images/conditions.png">
**Figure 0**

# Word Associations

## 1.1 Introduction

The research question explores whether linguistic patterns are distinctive to specific groups. Specifically, are certain emotions, concerns, or concepts prevalent among individuals with mental health conditions? While each condition's distinctiveness is intriguing, it may also offer practical advantages. For example, the dataset I used examines training chatbots to help doctors treat patients. If chatbots can detect emotions and identify condition-specific keywords, then customized treatment becomes more achievable.  
*(Refer to `group5_scriptfile_viz.ipynb`)*

## 1.2 Methodology

After following the preprocessing steps above, two methods were used:

1. A **TF-IDF** vectorizer transformed the text and identified terms that were frequent in a specific document but not across others. The goal was to reveal words more associated with one condition, which could help train chatbots to identify specific conditions.
2. A **WordCloud** visualized the most common words with more than three letters that appeared in more than five tweets. Weights and frequencies were based on each word's average frequency compared to its mean across all documents.

Condition-specific words (e.g., bipolar, depression) and the Normal category were removed from the dataframe before analysis.

## 1.3 Data Results


<img src="/images/anxiety.png">


### Figure 1.1 Anxiety Wordcloud

The Anxiety WordCloud includes words commonly associated with anxiety, such as worry, nervous, and restless. Additionally, many words align with medical themes. Terms such as heart, doctor, and cancer appear prominently, suggesting that medical concerns or fixations are frequently mentioned by individuals with anxiety in this dataset.


<img src="/images/depression.png"> 


### Figure 1.2 Depression WordCloud

People with depression focus on their emotions, using words like ‘feel’, ‘depress’, ‘hate’, ‘happy’, and ‘good’. This suggests they discuss their emotions more than other groups do; specifically, they _feel_ x, y, z, but _are_ not x, y, z.

<img src="/images/suicide.png">

### Figure 1.3 Suicide Depression Word

When someone is suicidal, they often discuss life and death, as well as aspirations they believe are unattainable. Common keywords in these tweets include 'kill,' 'die,' 'end,' and some expletives.

<img src="/images/stress.png">


### Figure 1.4 Stress WordCloud

The stress word cloud features many situational words, with ‘work,’ ‘job,’ ‘study,’ ‘pay,’ and ‘home’ appearing frequently. These typically routine subjects serve as the main triggers for people experiencing stress. While stress and anxiety are related, anxiety words concern the body and life-or-death issues, while stress words relate to uncontrollable situations.

<img src="/images/bipolar.png">


### Figure 1.5 Bipolar WordCloud

Bipolar disorder involves manic and depressive episodes. Words like ‘mania,’ ‘episodes,’ and ‘depressive’ are prominent. Most words focus on treatment rather than feelings. Bipolar disorder is a severe condition that, when untreated, can destroy lives, making treatment critical for a fulfilling life. Terms like ‘diagnose’, ‘diagnosis’, ‘psychiatrist’, ‘med’, ‘medicine’, and ‘doctor’ appear. Medicines that treat bipolar disorder, such as ‘Lithium,’ ‘Lamictal,’ ‘Seroquel,’ and ‘Latuda,’ are discussed.

<img src="/images/personality.png">


### Figure 1.6 Personality Disorder WordCloud

Personality disorders are defined by a person exhibiting unnatural patterns and unhealthy thinking that differ from societal norms. One of the most common and well-known personality disorders is Avoidant Personality Disorder (AvPD), which is one of the most prominent disorders. Descriptions of AvPD are also represented in the WordCloud, with sufferers discussing struggling ‘socially ’ and understanding relationships and ‘people.’

## 1.4 Conclusion

The results provided key insights into what distinguishes each mental health condition. One possible issue is the Depression TF-IDF. Because Depression is featured so prominently in the data set, I wondered if that affected the TF-IDF - indeed, it did. Upon further testing, words like ‘feel’ and ‘like’ are among the most common across all conditions, but because of the class imbalance, the inverse often portrays these words as uniquely associated with Depression.
To summarize: 
* Anxiety was linked with health concerns.
* Depression focused on strong emotions.
* Suicidal thoughts emphasized life-and-death language.
* Stress centered on life circumstances such as jobs and finances.
* Bipolar Disorder and Personality Disorder highlighted condition-related terminology, with Bipolar focusing more on treatment and Personality Disorder focusing more on social and life struggles.

# Domain Specific Transformer Predictions

## 2.1 Introduction

My next research question addresses a model's capacity to predict an author's condition from a tweet. While there is literature concerning model training for depression detection, fewer studies compare multiple mental health conditions. 

A model of this type could:

* Identify Twitter users discussing mental health concerns.
* Help algorithms surface condition-specific content.
* Support chatbots and therapists in responding to users more effectively.

This project specifically compared the performance of multiple transformer models on the classification task.

## 2.2 Methodology


The Normal category was removed because it was the majority class, and the goal was to identify which condition each tweet belonged to.

Tweets were cleaned by:

* Removing links, hashtags, and punctuation
* Converting text to lowercase
* Avoiding lemmatization for model training

Two transformer models were evaluated:

* DistilBERT
* MentalRoBERTa (a transformer trained on mental health posts)

Based on documentation from the MentalRoBERTa Hugging Face page, the model’s training aligned closely with the goal of developing an automatic condition detector.


I performed automatic tokenization and padded statements using the tokenizer specific to each model. To address imbalanced data, I applied weighted training to balance class loss across groups. Training was accordingly weighted to compensate for class imbalance.

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
*(Refer to `group5_scriptfile_mental.ipynb`)*

The results of running MentalRoBERTa were promising. Specifically, the model stopped early around epoch 9. The final training loss was 0.26, and the observed accuracy wasn’t the best (78%). 

Therefore, emphasis was placed on the macro F1-score to account for class imbalance. Macro F1 considers both precision and recall, ensuring equal representation for all classes, including minority groups such as personality disorders. There was no need to penalize the model for failing to fit one class. 

The highest macro F1-score achieved was 0.8213, within the range of 0.80 to 0.90, suggesting the model performs well.

This result is consistent with the macro F1-score calculated from the classification report following predictions.

### Table 2.2 Classification Report for Mental RoBERTa

When we look more closely at the confusion matrix, we see incorrect labels, which support the specific results for each class.

### Figure 2.3 Confusion Matrix for Mental RoBERTa

The confusion matrix supported several initial hypotheses. Depression and Suicidal tweets were the most frequently confused categories, while Stress was occasionally mislabeled as Anxiety. Personality Disorder, being a minority class, also produced lower F1-scores and proved more difficult to evaluate accurately.

### Figure 2.4 ROC curves for Mental RoBERTa

The ROC curves indicate that all results are relatively good, with the curves for Anxiety, Bipolar, Personality Disorder, and Stress close to 1 (AUC of .97-.99). This suggests that the model generally performs well at differentiating between the classes. However, Suicidal and Depression both have an AUC of .91, which is expected because those two conditions overlap.

### 2.3.2 Distill-Bert  
*(Refer to `group5_scriptfile_distil.ipynb`)*

I will discuss DistilBERT briefly, as the main goal is to compare the performance of a base model with a mental health-specific pre-trained model. 

Using identical parameters and data, I reran the model with DistilBertUncased as both the tokenizer and the pre-trained model. The results were comparable, with most metrics trailing the mental health pre-trained model by 1-5 points. The average Macro-F1 was 80%, which is at the lower end of what is considered very good for such models. The model struggles to recognize and accurately evaluate minority groups.

## 2.4 Conclusion

MentalRoBERTa slightly outperformed DistilBERT across most evaluation metrics. The largest improvements appeared in:

* Bipolar recall, reducing false negatives
* Personality Disorder classification
* Stress classification

Although both models used class-balanced weights, DistilBERT required more domain-specific information to make strong predictions for minority conditions.
| Model         | Macro F1 | Weighted F1 | Accuracy |
| ------------- | -------- | ----------- | -------- |
| DistilBERT    | 0.80     | 0.79        | 0.78     |
| MentalRoBERTa | 0.82     | 0.79        | 0.79     |


The findings support the hypothesis that a domain-specific pre-trained transformer performs better than a broader general-purpose model for mental health classification tasks.


| Class | DistilBERT Precision | DistilBERT Recall | DistilBERT F1 | MentalRoBERTa Precision | MentalRoBERTa Recall | MentalRoBERTa F1 |
|---|---|---|---|---|---|---|
| Anxiety | .86 | .91 | .88 | .87 | .90 | .88 |
| Bipolar | .88 | .79 | .83 | .87 | .85 | .86 |
| Depression | .81 | .75 | .78 | .83 | .73 | .78 |
| Personality Disorder | .77 | .69 | .73 | .82 | .71 | .76 |
| Stress | .81 | .82 | .82 | .86 | .86 | .86 |
| Suicidal | .71 | .79 | .75 | .70 | .82 | .76 |
| Accuracy | .78 |  |  | .79 |  |  |
| Macro F1 | .80 |  |  | .82 |  |  |
| Weighted F1 | .79 |  |  | .79 |  |  |

Table 2.6 Comparison of DistilBERT and MentalRoBERTa - the cells marked in green are the highest scores between the two models