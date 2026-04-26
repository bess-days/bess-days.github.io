---
title: "Custom OCR Model"
excerpt: "Helping digitize documents for Indigenous tribes<br/><img src='/images/resized_ocr.jpg'>"
collection: portfolio
---

# Coeur d'Alene Online Language Resource Center

The Coeur d'Alene tribe lost their last fluent speaker in 2018, and have since been working with University of Arizona to revitalize the language. I joined the team 3 years ago, mainly doing linguistic work and research before heading my own team in Optical Character Recognition (OCR)

## 🔎 What is Optical Character Recognition

OCR is a technology that converts scanned documents, PDFs, or images into machine-readable, editable, and searchable digital text. Many Indigenous tribes have hundreds of documents in their language preserved, but it isn't easily accessible or searchable. Building an OCR model to do that opens many pathways to teach the language and engage future language learners.


However, most OCR models are trained on the world's more common languages. Popular OCR models include Tesseract, TrOCR, and GoogleVision. Tesseract, of which most of better results came from, supports over 100 languages, including some Indigenous languages. But not CdA.


## 📈 Data

### 👩‍💻 Ground Truths || Total: 350 pairs


To train a model, you need many "ground-truths" which are extracted lines from the target documents in way of screenshots, paired with a text file with the transcription.

As mentioned, I lead a team. The team's primarily work was in more recent typed documents that were clean and with little noise, while my side project was to do it on more complex, noisy source documents.

The documents I did for my side project were stories typed on a special typewriter by Gladys Reichard in the Reichard Orthography. This orthography is notably different then what is considered typical Latin characters.

As you'll notice, there is a lot of noise from photo copying and ink clarity issues.

<img src='/images/real_01.png'>

Transcription: tᴇtcɩni'tkups tcäsx̥ä'tᴇms uᵘpɔ'tsᴇsᴇ ɫa'x̥ʷṕᴇm ɫuẃa

However, some characters were incredibly rare, and while I tried to capture as many as I could equally, the resulting data had a Zipf Score (something that measures diversity of words in a vocabulary with everything >1 leaning towards imbalanced) of 1.2. However, that isn't _too_ bad of a score. 

### 🧪 Synthetic Data || Total: 1000 pairs

Due to be a one-woman team, I hadn't the time to take a significant amount of ground-truth pairs. Because of that, I decided to create synthetic data using various Python scripts and packages found [here](https://github.com/bess-days/colrc-ocr-model/tree/main/data_scripts)
* [Markovify](https://github.com/jsvine/markovify): Using existing text, create novel words in the language using a Markhov chain model that captures patterns in existing words and recombines them [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L29-L44)
    * Sample novel words include xʷä'ntc
        * xʷ occurs in 98/897 words in ground-truths, ä' occurs in 208/897, ntc occurs 18/897 words, ä'n occurs 22/897 words
    * The words used in all synthetic data is a combination of ground-truth words and 1,000 synthetic words

* [Augrpahy](https://github.com/sparkfish/augraphy): This is how I altered the text on the image and image itself for the synthetic data to best replicate the style of the ground-truths. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L77-L100)
    * Because Tesseract prefers clean images, I limited the noise (along with limiting the ground-truths I trained on)
    * [Low Random Inklines](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/lowinkrandomlines.html): Adds ink lines randomly through the image
    * [Inkbleed](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/inkbleed.html): Captures all edges (ie letters) in the image and adds a slight blur
    * [Letterpress](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/letterpress.html):  Mimics uneven ink dispertion on the image
    * [Subtle Noise](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/subtlenoise.html): Emulates the imperfections in scanning solid colors due to subtle lighting differences

* [Pillow](https://pypi.org/project/pillow/):  Utilizing the Image functionality was able to create the images. [Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L49-L76)
    * For the font I chose Duolis because it could accuratly represent all the characters, even though it doesn't that closely resemble a typewriter font.


<img src='/images/sample_4.png'>


## Models

I used two different models for OCR

### 🏺 Tesseract


#### What is Tesseract?

Tesseract is an open-source OCR (Optical Character Recognition) engine using a Long Short-Term Memory neural network which is a special type of Recurrent Neural Network. It accepts an image (PNG, JPG) and outputs the image's text using various methods. Currently, Tesseract has 100+ languages, including some Indigenous languages.

#### What is Tesstrain?


The official training framework/toolkit for creating and fine-tuning Tesseract OCR models and allows you to train custom models for specific fonts, languages, or document styles not well-covered by default Tesseract models.

To train it you have ground truth data (image + text pairs) as input to teach the model what characters look like in your specific context

#### My training

Multiple Runs:

* 350 ground-truths (gt)

* 350 gt + 1000 synthetic

Paramaters

* 20,000 iterations (number of times the model runs over the data)

    * Default is 10,000 but because of the uniqueness and noise of the images, I had to do 30,000 to reach the lower error rate

* Start model (this is a pretrained model by Tesseract that I fine-tuned)

    * Latin script (the base model was trained on many Latin characters)

    * Doing the training from scratch would have take many many gt pairs (probably in the 10-50 thousand)

## Results
How I evaluated the model?
Tesstrain itself at different iterations runs the character error rate on evaluation data (not used in training) and error rate of training data. Next, I handpicked a wide variety of ground truths (representing wide characters, different typewriter styles)

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
The most important things to look at when evaluating a model is how well it does on unseen data - that would be the model's lowest eval CER and my hand-picked OCR - for both with synthetic data was better. While w/o synthetic data had a lower training CER, that suggests the model was overfitting, meaning it didn't do as well as unseen data.

Let's compare the three models: Tesseract's base Latin model (my model's start model), w/o synthetic data, w/ synthetic data. 

<img src='/images/test_07.png'>
Top is ground truth, bottom is what the specified model predicted.

Latin Start Model (51% CER):
<img src='/images/07_base.png'>

W/o synthetic data (4% CER)
<img src='/images/07_wo.png'>

W/ Synthetic Data (2% CER)
<img src='/images/07_with.png'>


