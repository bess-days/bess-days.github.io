---
title: "Authorship Classification Competition"
excerpt: "Transformer-based Text Classification <br/><img src='/images/authorship_cropped.png'>"
collection: portfolio
---


<table>
  <caption>
    Class Competition Info
  </caption>
  <thead>
  <tr>
    <th></th>
    <th></th>
  </tr>
  </thead>
<tbody>
  <tr>
    <th><b>Leaderboard score</b></th>
    <td>68 (1st Place)</td>
  </tr>
  <tr>
    <th><b>Leaderboard team name</b></th>
    <td>Sydney Bess</td>
  </tr>
  <tr>
    <th><b>Kaggle username</b></th>
    <td>sydneybess</td>
  </tr>
  <tr>
    <th><b>Code Repository URL</b></th>
    <td>https://github.com/uazhlt-ms-program/ling-582-fall-2025-class-competition-code-bess-days</td>
  </tr>
</tbody>
</table>


## Task summary


The goal of this task is to see if two texts are by the same or different authors. This task has been studied in depth in linguistics; it even has its own field, Stylometry. For this Kaggle competition, we were given training and test datasets in CSV format. The train dataset, with 1,601 entries, was manually labeled: each entry (a combination of two snippets from the same or different authors) was assigned a 1 or 0 (more on this later). The test set was unlabeled and was what we were supposed to use for inference.

While Stylometry can be automated (i.e., feature engineering based on sentence length, punctuation, etc.), I wanted to know whether a sequence-classification pre-trained model could make the process more efficient.

## Exploratory data analysis

The data consisted of an ID, TEXT (which were actually two texts by the same or different author, combined with the string [SNIPPET]), and a LABEL of 0 or 1 depending on whether the texts were by the same or different author.
To investigate data dispersal, I checked several things. First, the labels.
In the full dataset:

| **Dataset**        | **Entries** | **Label 0** | **Label 1** |
|----------------|---------|---------|---------|
| Full Dataset   | 1,601   | 1,245   | 356     |
| Train Dataset  | 1,280   | 999     | 281     |
| Validation Set | 160     | 123     | 37      |
| Test Dataset   | 161     | 123     | 38      |

I thought the data was imbalanced because most entries were not by the same author. I had a theory that the model would have lower recall because it would learn more from not the same author than from the same author. 
Hence, tried adding more data via AI augmentation, but it made things up. 
So next, I did research on other datasets and found this dataset on Kaggle [Spooky Authorship Dataset](https://www.kaggle.com/competitions/spooky-author-identification/data?select=train.zip)
It had text samples from three authors, with columns for ID, text, and author.
While I didn't care about who the author was, I set up a random selection of two rows, combined the two text columns with [SNIPPET], and, if the author was the same, set the corresponding label.
While I ended up not using the data because it actually made my model worse (I discuss that more in future improvements), even though it balanced the two categories, here was the breakdown



| **Dataset**       | **Entries** | **Label 0** | **Label 1** |
|---------------|---------|---------|---------|
| Full Dataset  | 2,997   | 1,725   | 1,273   |
| Train Dataset | 2,398   | 1,381   | 1,017   |
| Test Dataset  | 600     | 344     | 256     |




While this 'balanced' the data, I did some research and found that BERT does not need much for [finetuning](https://stackoverflow.com/questions/58696041/how-to-deal-with-a-highly-imbalanced-issuetext-classification-dataset)
To make the data easier to visualize, I added text1 and text2 columns, which split the single [Snippet] special token into two texts.



| **ID** | **TEXT**                                  | **LABEL** | **text1**                                 | **text2**                                   |
|-------:|-------------------------------------------|-----------|---------------------------------------------|-----------------------------------------------|
| 1596   | Kinton realized to his surprise that the effor... | 0         | Kinton realized to his surprise that the effor... | Their erect posture gave them a weirdly half-h... |


## Approach

My approach was to use a BERT-based text classifier to classify the texts.

1. Performed exploratory analysis and decided to add two new columns for each of the text snippets. This is also where I attempted to supplement with more data.
2. Separated the dataframe into train, validation, and test, and imported it into a Dataset for easier manipulation
3. Set up the tokenizer; I used bert-base-uncased, as suggested by the [Text classifier](https://huggingface.co/docs/transformers/en/tasks/sequence_classification). Though I didn’t just choose a Sequence classifier model because that post told me to. As I’ve learned in classes, Sequence models are great for NLP because they can learn features from a sequence of words by transforming it into a sequence of numerical representations. With these features, it can build dependencies, which is useful for this task because you want to learn patterns and the similarities among authors.
* For this, I mapped the dataset and made some alterations to the model to better understand it, such as renaming ‘LABEL’ and ignoring the actual text columns, focusing on the IDs, input_ids, token_type_ids, attention_mask, and labels.
* Next, I put it in the Torch format, based on this example: [Dealing with the tokenizer](https://huggingface.co/docs/datasets/en/process).
4. Next, I set up a data_collator with dynamic padding to better organize padding.
5. Creating a metric to test the success of the model using accuracy, a function taken from HuggingFace, along with other functions taken from class lectures that talked about evaluating binary classification models.
6. Set up the training arguments as suggested by Hugging Face. My initial run was 3 num epochs, but I thought if I ran it longer, it would be better - but that was not the case. I was trying to be novel and try new things, but HuggingFace knows what it is talking about. But at least I learned what didn’t work as well (though all my experimentations did help me learn more about my model, and were relatively close to each other, ie, 65%-67% compared to the top first model, which was 70%). In the end, I went with 5 epochs.
7. Run model on witheld data to see accuracy there
8. Apply the model to the unlabeled test csv file


## Results



| **Metric**                 | **Baseline** | **New Model** | **Delta (New − Base)** |
| -------------------------- | ------------ | ------------- | ---------------------- |
| **Accuracy**               | 0.781        | 0.844         | **+0.064**             |
| **Precision**              | 0.500        | 0.875         | **+0.375**             |
| **Recall**                 | 0.085        | 0.378         | **+0.293**             |
| **F1 Score**               | 0.146        | 0.520         | **+0.374**             |
| **AUC ROC**                | —            | 0.727         | —                      |
| **Average Precision (AP)** | —            | 0.600         | —                      |

First, I set up a base model that used a similarity score and a threshold to predict the label by turning each text into a TF-IDF vector. For this, I used the training data later used for the model.

While accuracy is relatively high, compared to the Sequence model, it tends to mislabel the positive (same author) class as negative (not same author) with high frequency. More specifically, among all the golden class 1 labels, it gets only 8% correct. This is a major issue.
The reason accuracy might be so high is that the data is imbalanced toward negative examples.

For my model, I saved the best accuracy at epoch 3, though in retrospect I should have tested with the model that achieved the best precision and recall.


Looking at the 'New Model' column, there is a high accuracy, which can be misleading because, as I predicted in the beginning, the data heavily favours the negative class. 

The high precision means the predicted positives are truly positive. However, on the flip side, recall is very low, as it accurately predicts actual positives only 38% of the time (so even though it is out of the positives predicted, a high percentage are actually positive, it only picks positives in a third of the cases where there are positives).  The F1 score better reflects the imbalance.

That is why I decided to also calculate the Area Under the ROC, especially the AP.
The ROC AUC says it does an average job separating positives and negatives. But because the data is imbalanced, I wanted to focus more on AUC-PR, where the positive cases are ranked above the negative cases. The 60% means it is more than random, which is good, suggesting the reason the recall is so low is not that the model can't predict positives, but that the threshold is too high.

All that said, the delta shows there is a 35% average improvement in Precision, Recall, and F1 score, which is good. This shows that using a transformer-based model improves performance compared to a standard vectorisation model. However, it follows the same proportional trend of high precision and low recall (though to a greater extent). This suggests that, even though the model improves matters, there is something to be said about the data imbalance (though again, when I tried to add data, it did worse).

## Error analysis


| **Metric**        | **Value** |
|-------------------|-----------|
| Eval Accuracy     | 0.8509    |
| Eval Precision    | 0.7917    |
| Eval Recall      | 0.5000    |
| Eval F1           | 0.6129    |
| Eval AUC ROC      | 0.7668    |
| Eval AP       | 0.6793    |

This trend of high accuracy, high precision, and low recall continues when evaluating the model on test data (separate from the validation and training data). The F1 score is higher because precision and recall are more balanced. This suggests it did a better job of identifying more true positives rather than categorizing them as negative (though only with 50% recall, which is still bad). This remains the main type of error. The higher, but similar, accuracy supports the idea that the datasets were relatively similar (as shown in the EDA).

## Future Improvements

I'd like to look at the specifics of the errors, like if the text pairs it predicts wrong or right have specific features like including dialogue, flowery language (ie distinct), or are just descriptive. 

The second thing I'd do is balance the data by augmenting it. I think when I tried to add more data from the Kaggle dataset I mentioned, I added too much, so it didn't actually represent what the test data would look like. It was 65% when I submitted the prediction (although at the time I added the data, I wasn't checking the Precision, Recall, or F1 score to tell - all I knew is it had an accuracy of around 75% (below both base and my model) and a 65% on Kaggle). I would go back and redo adding the data and try again, because I hadn't pursued it after it got my lowest results. If that still didn't work (or even along with that), I'd do a method like class weighting in machine learning where I'd add more weight to the minority class (label 1)). 




