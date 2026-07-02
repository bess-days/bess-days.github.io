---
title: "Custom OCR Model"
excerpt: "Helping digitize documents for Indigenous tribes<br/><img src='/images/resized_ocr.jpg'>"
collection: portfolio
---

# Coeur d'Alene Online Language Resource Center

The Coeur d’Alene (CdA) tribe lost its last fluent speaker in 2018 and has been working with the University of Arizona to revitalize the language. I joined the team three years ago, initially focusing on linguistics and research before leading my own OCR team.



## 🔎 What is Optical Character Recognition (OCR)

OCR is a technology that converts scanned documents, PDFs, or images into machine-readable, editable, and searchable digital text. By building an OCR model for Indigenous languages, we make these preserved documents accessible and searchable, enabling easier language teaching and learning and supporting ongoing language revitalization efforts in communities.

Most OCR models are trained on common languages. Popular ones include Tesseract, TrOCR, and GoogleVision. Tesseract gave the best results and supports over 100 languages, including some Indigenous ones, but not CdA.


## 📈 Data

### 👩‍💻 Ground Truths || Total: 350 pairs


To train a model, you need a large annotated dataset of ground truths: extracted lines from the target documents, presented as images, paired with a text file containing their transcriptions.

For my side project, I used stories typed on a special typewriter by Gladys Reichard in the Reichard Orthography, which differs from typical Latin characters.

These pages often exhibit significant photocopying noise and ink issues.

<img src='/images/real_01.png'>

Transcription: tᴇtcɩni’tkups tcäsx̥ä’tᴇms uᵘpɔ’tsᴇsᴇ ɫa’x̥ʷṕᴇm ɫuẃa

Some ground truths are even slanted and have other noise, like:

<img src='/images/real_03.png'>

Transcription: tcilɩdju’sᴇnts kuḿ tćaḿ äya’ʀ guĺɩnt’a’q́ʷsus. tätc

The model was trained exclusively on typed text, meaning lines with handwritten annotations were excluded from training. Markings like underlines and corrections weren’t included in the training. Example of excluded lines:

<img src='/images/handwitten.png'>

<img src='/images/slash.png'>

Some characters were very rare like the ᵓ and r̥. Character coverage was prioritized during data collection.  A survey of the data showed a Zipf Score (measuring character diversity; >1 is imbalanced) was 1.2, fell within an acceptable range for this situation.

### 🧪 Synthetic Data || Total: 1000 pairs

As a one-person team, I didn’t have time to collect a sufficient quantity of ground truth pairs. I decided to generate synthetic training data using various Python scripts and packages found [here](https://github.com/bess-days/colrc-ocr-model/tree/main/data_scripts)
* [Markovify](https://github.com/jsvine/markovify): Using existing text, create novel words in the language using a Markov chain model that captures patterns in existing words and recombines them. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L29-L44)
    * Sample novel words include xʷä'ntc
        * xʷ occurs in 98/897 words in ground truths, ä’ occurs in 208/897, ntc occurs in 18/897 words, ä’n occurs in 22/897 words
    * The words used in all synthetic data are a combination of ground truth words and 1,000 synthetic words.

* [Augraphy](https://github.com/sparkfish/augraphy):  This is how I applied image and text augmentations to best approximate the appearance of the ground truths. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L77-L100)
    * [Low Random Inklines](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/lowinkrandomlines.html): Adds ink lines randomly through the image
    * [Inkbleed](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/inkbleed.html): Captures all edges (i.e., letters) in the image and adds a slight blur.
    * [Letterpress](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/letterpress.html):  Mimics uneven ink dispersion on the image
    * [Subtle Noise](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/subtlenoise.html): Emulates the imperfections in scanning solid colors due to subtle lighting differences

* [Pillow](https://pypi.org/project/pillow/):  Using the Image functionality, I created the images [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L49-L76)
    * For the font, I selected Duolis because it accurately represents all the characters. This does not exactly replicate type writer font because finding a font that could capture the characters and at least loosely resembled the source text was a priority.


<img src='/images/sample_4.png'>


## Models

### 🏺 Tesseract


#### What is Tesseract?

[Tesseract](https://tesseractocr.org/) is an open-source OCR (Optical Character Recognition) engine that uses a Long Short-Term Memory (LSTM) neural network, a special type of Recurrent Neural Network. It accepts an image (PNG or JPG) and outputs its text using various methods. Currently, Tesseract has 100+ languages, including some Indigenous languages.

#### What is Tesstrain?

[Tesstrain](https://github.com/tesseract-ocr/tesstrain/tree/main) is the official training framework/toolkit for creating and fine-tuning Tesseract OCR models, and allows you to train custom models for specific fonts, languages, or document styles not well-covered by default Tesseract models.

To train it, you use ground truth data (image-text pairs) to teach the model what characters look like in your specific context.

#### My training

Multiple Runs:

* 350 ground truths (gt)

* 350 gt + 1000 synthetic

Parameters

* 20,000 iterations (number of times the model runs over the data)

    * The default is 10,000, but I increased it because of diverse training samples and noise, I set it to 20,000 to achieve a lower error rate.

* Start model (this is a pretrained model by Tesseract that I fine-tuned)

    * Latin script (the base model was trained on many Latin characters)
    
    * Doing the training from scratch would a large annotated dataset (probably in the 10-50 thousand)

## Results
How did I evaluate the model?
Tesstrain tracks the error rate at different iterations, reporting the character error rate (CER) on evaluation data (the 10% not used in training) and the error rate on the trained data. It returns the best model which I use for a more catered evaluation. Next, I handpicked a wide variety of ground truth pairs (representing diverse characters, varying document styles) and ran inferences and compared the outputs.

| Model                  | Lowest Eval CER (Iteration)    | Lowest Training CER (Iteration) | Hand-picked CER | Flawless Predictions |
|------------------------|--------------------------------|---------------------------------|-----------------|----------------------|
| Latin Start Model      | —                              | —                               | 49%             | 0                    |
| without Synthetic (Lon)    | 7.1% (16,100 / 20,000)         | 0.177% (19,900 / 20,000)        | 3.7%            | 13                   |
| with Synthetic (Tyn)     | 1.6% (18,400 / 20,000)         | 0.47% (18,400 / 20,000)         | 2.9%            | 15                   |

Without Synthetic Data
<img src='/images/lon.plot_cer.png'>
With Synthetic Data
<img src='/images/tyn.plot_cer.png'>

Interpretation:
The most important metrics to look at when evaluating a model are how well it does on unseen data - that would be the model's lowest eval CER- and how well it does on my hand-picked OCR; the model trained with synthetic data demonstrated improved generalization on both of these types of data.

The following comparison illustrates the differences between Tesseract’s base Latin model (my model's start model), the finetuned model with and without synthetic data.

<img src='/images/test_07.png'>

The example below demonstrates a ground truth with high-noise. The image has black spots, and the text is crooked.

The top image sho the ground truth, and the bottom is what the specified model predicted.

Latin Start Model (51% CER):
<img src='/images/7_base.png'>

Without synthetic data (4% CER)
<img src='/images/7_wo.png'>

With Synthetic Data (2% CER)
<img src='/images/7_with.png'>

Observed patterns:
The model with synthetic data outperformed without synthetic data when it came to recognizing sentence e and word boundaries. Omitted punctuation occurred at least four times in my tests with the model without synthetic data. There are also cons with using a start model compared to training from scratch—the start model. A start model includes a massive unichar set of all the characters in that script (in this case, Latin). When the model finetunes a specific unicharset for your data, it includes all the characters from Latin, however, the probabilities those characters show up in the pipeline are greatly diminished. 

This is an example that has both foreign characters and a missing period.

<img src="/images/test_16.png">

without Synthetic data (8% CER):
<img src="/images/16_wo.png">


With Synthetic data (2% CER)
<img src="/images/16_w.png">


The exact cause of why letters were inserted there is unclear. When letters are inserted, it is because of noise in the photograph, in this case there wasn't any excessive noise. Neither could I find clear trends: one model did better than the other. In around a third of the test data, the model without synthetic data outperformed than the ones with synthetic data, even though synthetic data performed better overall.

My theory had been that because the synthetic data are trained on greater lexically diverse words (3+ clusters) fabricated by the Markov model, the model with synthetic data would do better on longer words, and it did in some cases, but performed the same (but not less) than without synthetic data. Generally, both performed well on very unique characters like superscript u and a, along with voiceless markers (the dot underneath the letters)

Recurring error patterns included:

* Missing accent (a vs ä was most common)

* Confusion between the letters (m, n, h were most common)

* Missing an apostrophe or mistaking for ᶥ

Overall, though, it achieved strong performance on language-specific characters the Latin model was most likely not heavily trained on, as seen in the Latin base predictions compared to the fine-tuned.



# Teamwork

I mentored a team of undergraduates through a similar process for different documents in CdA.

All of them had different skills and backgrounds, giving me ample opportunity to both teach them new technical skills and develop existing ones. When I started the project with them, I had not yet fully learned the OCR pipeline. Together, we did trial and error, while I ensured we had the tools needed to start including a repository and extensive literature for us to review.

Fortunately, this project took less work than my personal project with the Reichard documents. The character diversity that most resembled Latin's a-z and high-quality source made the model easier to train. The Latin model on its own performed well on our documents, but generally misclassified the ɫ as l or L, so after trial and error, the team and I created our own Tesseract model. In the end, the results achieved very reduced character error rates; however, the unique layouts of these source documents, as language workbooks, pose a large problem we have yet to tackle.

The team is well-positioned to continue and I assured them I will remain reachable if they need any help.




# Can you do this for your Indigenous language?

This process can be done with any language, however, the task must be personalized for your language. Firstly, is your language's orthography similar to a script Tesseract is pre-trained on? If the answer is yes, this simplifies the training process. Not to say a completely new script isn't possible to train a custom model on; it will just take a substantial amount of training data and the creation of other files, like a unicharset. 



Depending on the similarity to the start model, you might need more iterations. If your script more closely resmebles the Latin characters and in turn the data used to train the Latin start model, as the team's orthography did, few iterations are needed to properly finetune the model. The team did 10,000 iterations, however, as the Reichard model needed to learn characters not as common in the Latin start model, it took 20,000 iterations.
Training requires at least a few hundred ground truths under varying conditions, and then many more synthetic data points, though again, this greatly depends on your language. My suggestion is to test many variations (I ran over 15 models with varying parameters and input data).









