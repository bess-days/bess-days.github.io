---
title: "Custom OCR Model"
excerpt: "Helping digitize documents for Indigenous tribes<br/><img src='/images/resized_ocr.jpg'>"
collection: portfolio
---

# Coeur d'Alene Online Language Resource Center

The Coeur d’Alene (CdA) tribe lost its last fluent speaker in 2018 and has since worked with the University of Arizona to revitalize the language. I joined the team three years ago, initially focusing on linguistics and research before leading my own OCR team.



## 🔎 What is Optical Character Recognition (OCR)

OCR is a technology that converts scanned documents, PDFs, or images into machine-readable, editable, and searchable digital text. By building an OCR model for Indigenous languages, we make these preserved documents accessible and searchable, enabling easier language teaching and learning and supporting ongoing language revitalization efforts in communities.

Most OCR models are trained on common languages. Popular ones include Tesseract, TrOCR, and GoogleVision. Tesseract gave the best results and supports over 100 languages, including some Indigenous ones, but not CdA.


## 📈 Data

### 👩‍💻 Ground Truths || Total: 350 pairs


To train a model, you need many ground-truth pairs: extracted lines from the target documents, presented as images, paired with a text file containing their transcriptions.

As mentioned, I lead a team. The team’s primary work was on more recent, clean, low-noise documents, while my side project focused on more complex, noisy source documents.

For my side project, I used stories typed on a special typewriter by Gladys Reichard in the Reichard Orthography, which differs from typical Latin characters.

These pages often exhibit significant photocopying noise and ink issues.

<img src='/images/real_01.png'>

Transcription: tᴇtcɩni’tkups tcäsx̥ä’tᴇms uᵘpɔ’tsᴇsᴇ ɫa’x̥ʷṕᴇm ɫuẃa

Some ground-truths are even slanted and have other noise, like:

<img src='/images/real_03.png'>

Transcription: tcilɩdju’sᴇnts kuḿ tćaḿ äya’ʀ guĺɩnt’a’q́ʷsus. tätc

There were also handwritten notes and symbols in the documents, but the model can’t train on both typed and handwritten text. Tesseract also isn’t strong with handwritten script. Markings like underlines and corrections weren’t included in the training. Example of excluded lines:

<img src='/images/handwritten.png'>

<img src='/images/slash.png'>

Some characters were very rare. I tried to capture as many as possible, and the data’s Zipf Score (measuring word diversity; >1 is imbalanced) was 1.2, which isn’t bad.

### 🧪 Synthetic Data || Total: 1000 pairs

As a one-person team, I didn’t have time to collect a significant number of ground-truth pairs. Because of that, I decided to create synthetic data using various Python scripts and packages found [here](https://github.com/bess-days/colrc-ocr-model/tree/main/data_scripts)
* [Markovify](https://github.com/jsvine/markovify): Using existing text, create novel words in the language using a Markov chain model that captures patterns in existing words and recombines them. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L29-L44)
    * Sample novel words include xʷä'ntc
        * xʷ occurs in 98/897 words in ground-truths, ä’ occurs in 208/897, ntc occurs in 18/897 words, ä’n occurs in 22/897 words
    * The words used in all synthetic data are a combination of ground-truth words and 1,000 synthetic words.

* [Augrpahy](https://github.com/sparkfish/augraphy):  This is how I altered the text on the image and the image itself to best replicate the style of the ground-truths. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L77-L100)
    * Tesseract prefers clear images, so I reduced noise and trained on less noisy ground truth.
    * [Low Random Inklines](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/lowinkrandomlines.html): Adds ink lines randomly through the image
    * [Inkbleed](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/inkbleed.html): Captures all edges (i.e., letters) in the image and adds a slight blur.
    * [Letterpress](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/letterpress.html):  Mimics uneven ink dispersion on the image
    * [Subtle Noise](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/subtlenoise.html): Emulates the imperfections in scanning solid colors due to subtle lighting differences

* [Pillow](https://pypi.org/project/pillow/):  Using the Image functionality, I created the images [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L49-L76)
    * For the font, I chose Duolis because it accurately represents all the characters, even though it doesn’t closely resemble a typewriter font.


<img src='/images/sample_4.png'>


## Models

### 🏺 Tesseract


#### What is Tesseract?

Tesseract is an open-source OCR (Optical Character Recognition) engine that uses a Long Short-Term Memory (LSTM) neural network, a special type of Recurrent Neural Network. It accepts an image (PNG or JPG) and outputs its text using various methods. Currently, Tesseract has 100+ languages, including some Indigenous languages.

#### What is Tesstrain?

The official training framework/toolkit for creating and fine-tuning Tesseract OCR models, and allows you to train custom models for specific fonts, languages, or document styles not well-covered by default Tesseract models.

To train it, you use ground-truth data (image-text pairs) to teach the model what characters look like in your specific context.

#### My training

Multiple Runs:

* 350 ground-truths (gt)

* 350 gt + 1000 synthetic

Paramaters

* 20,000 iterations (number of times the model runs over the data)

      * The default is 10,000, but because of unique images and noise, I set it to 20,000 to achieve a lower error rate.

* Start model (this is a pretrained model by Tesseract that I fine-tuned)

    * Latin script (the base model was trained on many Latin characters)
    
    * Doing the training from scratch would have taken many, many gt pairs (probably in the 10-50 thousand)

## Results
How did I evaluate the model?
Tesstrain itself at different iterations, reporting the character error rate (CER) on evaluation data (not used in training) and the CER on training data. Next, I handpicked a wide variety of ground-truth pairs (representing wide characters, different typewriter styles)

| Model                  | Lowest Eval CER (Iteration)    | Lowest Training CER (Iteration) | Hand-picked CER | Flawless Predictions |
|------------------------|--------------------------------|---------------------------------|-----------------|----------------------|
| Latin Start Model      | —                              | —                               | 49%             | 0                    |
| w/o Synthetic (Lon)    | 7.1% (16,100 / 20,000)         | 0.177% (19,900 / 20,000)        | 3.7%            | 13                   |
| w/ Synthetic (Tyn)     | 1.6% (18,400 / 20,000)         | 0.47% (18,400 / 20,000)         | 2.9%            | 15                   |

W/O Synthetic Data
<img src='/images/lon.plot_cer.png'>
W/ Synthetic Data
<img src='/images/tyn.plot_cer.png'>

Interpretation:
The most important things to look at when evaluating a model are how well it does on unseen data - that would be the model’s lowest eval CER and my hand-picked OCR - for both with synthetic data was better. While w/o synthetic data had a lower training CER, this suggests the model was overfitting, meaning it didn’t perform as well on unseen data.

Let's compare the three models: Tesseract’s base Latin model (my model's start model), w/o synthetic data, w/ synthetic data.

<img src='/images/test_07.png'>

As you can see, this is an excellent example of a challenging ground truth - there are black spots, and the text is crooked.

The top is the ground truth, and the bottom is what the specified model predicted.

Latin Start Model (51% CER):
<img src='/images/07_base.png'>

W/o synthetic data (4% CER)
<img src='/images/07_wo.png'>

W/ Synthetic Data (2% CER)
<img src='/images/07_with.png'>

Patterns I found in performance between the two were that the model with synthetic data did better at reading sentence ends and word boundaries. Missing periods occurred at least four times in my tests with the model without synthetic data. There are also cons with using a start model compared to training from scratch—the start model introduces the unichar set of that script (in this case, Latin), so letters appear that shouldn’t appear in the document’s orthography (though this only occurred a few times in testing).

This is an example that has both foreign characters and a missing period.

<img src="/images/test_16.png">

W/o Synthetic data (8% CER):
<img src="/images/16_wo.png">


W/ Synthetic data (2% CER)
<img src="/images/16_w.png">


To be honest, I’m not sure exactly why letters were inserted there - usually, when letters are inserted, it is because of noise in the photograph. Neither could I find very clear trends: one model did better than the other. In fact, in around a third of the test data, the model w/o synthetic data did better than the ones w/ synthetic data, even though synthetic data did better overall.

My original theory had been that because the synthetic data are trained on more unique words (3+ clusters) fabricated by the Markov model, the model w/ synthetic data would do better on longer words, and it did in some cases, but performed the same (but not less) than w/o synthetic data. Generally, both performed well on very unique characters like superscript u and a, along with voiceless markers (the dot underneath the letters)

I did find that the most common errors are:

* Missing accent (a vs ä was most common)

* Confusion between the letters (m, n, h were most common)

* Missing an apostrophe or mistaking for ᶥ

Overall, though, it did incredibly well on the unique characters the Latin model was most likely not heavily trained on, as seen in the Latin base predictions compared to the fine-tuned.



# Teamwork

I lead a team of undergraduates through a similar process for different documents in CdA.

All of them had different skills and backgrounds, giving me ample opportunity to both teach them new skills and advance existing ones. When I started the project with them, I had not yet fully learned the OCR pipeline. Together, we did trial and error, but I put my best foot forward by preparing a repository for us to work on and doing plenty of research to best teach the team.

Fortunately, this project took less work than my personal project with the Reidarch documents.  The orthography and quality of the sources were less of a problem with the pages being clean, and the main character used that was outside the typical ‘Latin’ set was l with a tilde. The Latin model on its own did well on our documents, but generally messed up the l with a tilde, so after trial and error, the team and I created our own Tesseract model. In the end, the results were near-perfect, with little character error; however, the unique layouts of these source documents, as language workbooks, pose a large problem we have yet to tackle.

I hope this amazing team can work together and carry on my work after I graduate. I assured them I will remain reachable if they need any help.




# Can you do this for your Indigenous language?

Yes! Though there are a few variables. Firstly, is your language's orthography similar to a script Tesseract is pre-trained on? If the answer is yes, you're on the right track. Not to say a completely new script isn't possible to train a custom model on; it will just take a lot of data and the creation of other files, like a manually created unicharset. 



Depending on the similarity to the start model, you might need more iterations - for the more similar to Latin I did with the team, it took the default 10,000 iterations, but mine took double that. You'll also need a few hundred ground truths under varying conditions, and then many more synthetic data points. Though again, this greatly depends on your language - my suggestion is to test many variations (I ran over 15 models with varying parameters and input data).









