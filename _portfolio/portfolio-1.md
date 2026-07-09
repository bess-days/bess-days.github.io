---
title: "Custom OCR Model"
excerpt: "Helping digitize documents for Indigenous tribes<br/><img src='/images/resized_ocr.jpg'>"
collection: portfolio
---

# Coeur d'Alene Online Language Resource Center ([COLRC](https://thecolrc.org/))

The Coeur d’Alene (CdA) tribe lost its last fluent speaker in 2018 and has been working with the University of Arizona to revitalize the language. I joined the team three years ago, first focusing on linguistics and research, then leading my own OCR team.

## 🔎 What is Optical Character Recognition (OCR)
OCR is a technology that converts scanned documents, PDFs, or images into machine-readable, editable, and searchable text. By building an OCR model for Indigenous languages, we make these preserved documents accessible and searchable, supporting language teaching, learning, and revitalization.

Most OCR models are trained on widely spoken languages. Popular options include Tesseract, TrOCR, and Google Vision. 


## Models

### 🏺 Tesseract


#### What is Tesseract?

[Tesseract](https://tesseractocr.org/) is an open-source OCR engine that uses a Long Short-Term Memory (LSTM) neural network, a type of Recurrent Neural Network. It accepts images (PNG or JPG) and outputs text using various methods. Tesseract currently supports over 100 languages, including some Indigenous languages, but not CdA.

#### What is Tesstrain?

[Tesstrain](https://github.com/tesseract-ocr/tesstrain/tree/main) is the official training toolkit for creating and fine-tuning Tesseract OCR models. It allows training custom models for specific fonts, languages, or document styles not well covered by default Tesseract models.

Training a model requires a large annotated dataset of ground truths: images of extracted lines from target documents paired with text transcriptions (ground-truth pairs). This teaches the model what characters look like for your specific context.

## 📈 Data

### 👩‍💻 Ground Truths || Total: 350 pairs

For my [project](https://github.com/bess-days/colrc-ocr-model/tree/main), I used stories typed on a typewriter by Gladys Reichard in the Reichard Orthography, which differs from typical Latin characters.

These pages often exhibit significant photocopying noise and ink issues.

<img src='/images/real_01.png'>

Transcription: tᴇtcɩni’tkups tcäsx̥ä’tᴇms uᵘpɔ’tsᴇsᴇ ɫa’x̥ʷṕᴇm ɫuẃa

Some ground truths are even slanted and have other noise, like:

<img src='/images/real_03.png'>

Transcription: tcilɩdju’sᴇnts kuḿ tćaḿ äya’ʀ guĺɩnt’a’q́ʷsus. tätc

The model was trained exclusively on typed text, meaning lines with handwritten annotations, symbols, and underlines were excluded from training. Example of excluded lines:

<img src='/images/handwitten.png'>

<img src='/images/slash.png'>

Some characters were very rare, like ᵓ and r̥, so character coverage was prioritized during data collection. A survey of the data showed that the Zipf Score, which measures character diversity, was 1.2, an acceptable range for the available data.

### 🧪 Synthetic Data || Total: 1000 pairs

Generating synthetic data effectively supplemented the limited hand-annotated pairs. The Python packages and scripts used are available [here](https://github.com/bess-days/colrc-ocr-model/tree/main/data_scripts)
* [Markovify](https://github.com/jsvine/markovify): Using existing text, create novel words in the language using a Markov chain model that captures patterns in existing words and recombines them. [My Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L29-L44)
    * Sample novel words include xʷä'ntc
        * xʷ occurs in 98/897 words in ground truths, ä’ occurs in 208/897, ntc occurs in 18/897 words, and ä’n occurs in 22/897 words
    * The words used in all synthetic data are a combination of ground truth words and 1,000 synthetic words.

* [Augraphy](https://github.com/sparkfish/augraphy): Applies image and text augmentations to best approximate the appearance of the ground truths. [My Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L77-L100)
    * [Low Random Inklines](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/lowinkrandomlines.html): Adds ink lines randomly through the image
    * [Inkbleed](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/inkbleed.html): Captures all edges (i.e., letters) in the image and adds a slight blur.
    * [Letterpress](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/letterpress.html):  Mimics uneven ink dispersion on the image
    * [Subtle Noise](https://augraphy.readthedocs.io/en/latest/doc/source/augmentations/subtlenoise.html): Emulates the imperfections in scanning solid colors due to subtle lighting differences

* [Pillow](https://pypi.org/project/pillow/):  Using the Image functionality, I created the images [My Code](https://github.com/bess-days/colrc-ocr-model/blob/main/data_scripts/synthetic_gen.py#L49-L76)
      * I selected the Duolis font because it accurately represents all the characters. While it does not exactly replicate the typewriter font, prioritizing a font that at least resembles it while capturing all the characters was the priority.


<img src='/images/sample_4.png'>



#### My training

Multiple Runs:

* 350 ground truths

* 350 ground-truths + 1000 synthetic pairs

Parameters

* 20,000 iterations (number of times the model runs over the data)

    * The default is 10,000 iterations, but I increased it to 20,000 due to diverse training samples and noise, aiming for a lower error rate.

* Start model (this is a pretrained model by Tesseract that I fine-tuned)

    * Latin script (the base model was trained on many Latin characters)
    
    * Training from scratch would require a large annotated dataset, likely between 10,000 and 50,000 examples.

## Results
How did I evaluate the model?
Tesstrain tracks error rates across iterations, reporting the character error rate (CER) on evaluation data (the 10% not used for training) and on training data. It returns the best model, which I then evaluated further by testing on a diverse set of ground-truth pairs and comparing the outputs.

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
The most important metrics for evaluating a model are its performance on unseen data (lowest evaluation CER) and on hand-picked OCR samples. The model trained with synthetic data demonstrated better generalization in both areas.

The following comparison illustrates the differences between Tesseract’s base Latin model (the start model) and the fine-tuned model, with and without synthetic data.

The example below demonstrates a ground truth with significant noise, including black spots and crooked text.

<img src='/images/test_07.png'>

The top line shows the ground truth, while the bottom displays the model’s prediction.

Highlighted in red on the top line are characters that were deleted or replaced in the model’s prediction. The green characters on the bottom line are added by the model’s prediction and aren’t in the ground truth.


Latin Start Model (51% CER):
<img src='/images/7_base.png'>

Without synthetic data (4% CER)
<img src='/images/7_wo.png'>

With Synthetic Data (2% CER)
<img src='/images/7_w.png'>

Observed patterns:
The model trained with synthetic data outperformed the model without synthetic data in recognizing sentence and word boundaries. Omitted punctuation occurred at least four times in tests with the model without synthetic data. Using a start model has drawbacks compared to training from scratch. A start model includes a large unicharset for the script (in this case, Latin). When fine-tuned, it retains all Latin characters, but the likelihood of those characters appearing in the output decreases as training progresses.

This is an example that has both foreign characters and a missing period.

<img src="/images/test_16.png">

Latin Start Model (62%)
<img src="/images/16_b.png">

Without Synthetic data (8% CER):
<img src="/images/16_woo.png">

With Synthetic data (2% CER)
<img src="/images/16_with.png">


The reason for the inserted letters is unclear. Typically, such errors result from image noise, but in this case, there was no excessive noise. No consistent trend emerged: in about one-third of the test data, the model without synthetic data outperformed the model with synthetic data, although the latter performed better overall. 

I theorized that the model trained on synthetic data, which included more lexically diverse words from the Markov model, would perform better with longer words. This was true in some cases, but in others, performance was similar (but not less). Both models generally performed well on language-specific characters such as ᵘ, ᵃ̈, and x̥ that the Latin model was not heavily trained on. 

Recurring error patterns included:

* Missing accent (a vs ä was most common)

* Confusion between the letters (m, n, h were most common)

* Missing an apostrophe or mistaking for ᶥ



# Teamwork

I mentored a team of undergraduates through a similar process for different documents in CdA.

The team members brought diverse skills and backgrounds, allowing me to teach and develop new technical skills. At the project’s outset, I was still learning the OCR pipeline. We used trial and error, and I ensured we had the necessary tools, including a repository and relevant literature.

This project on language workbooks in the Nicodemus orthography required less fine-tuning than my work with the Reichard documents. The character diversity resembling Latin a-z and the high-quality source material made the model easier to train. The Latin model performed well but often misclassified ɫ as l or L, which led us to train our own model. In the end, the results achieved reduced character error rates; however, the unique layouts of these source documents pose a problem we have yet to tackle.

The team is well-positioned to continue, and I assured them I will remain reachable if they need any help.


# Can you do this for your Indigenous language?

This process can be done with any language; however, the task must be personalized for your language. Firstly, consider if your language’s orthography is similar to a script Tesseract is pre-trained on. If the answer is yes, this simplifies the training process. Training a custom model for a completely new script is possible but requires substantial training data and the creation of additional files, such as a unicharset.

Depending on how similar it is to the start model, you might need more iterations. If your script closely resembles the Latin start model, as the team’s orthography did, fewer iterations are needed to fine-tune the model. The team completed 10,000 iterations, whereas the Reichard model I produced separately, which was less similar to Latin, took twice that many.
You’ll also need at least a few hundred ground-truth examples under varying conditions, and then many more synthetic data points, though, again, this greatly depends on your language.

# Conclusion

OCR is a relatively new technology that is constantly improving, whether through advances in layout parsing or the addition of new languages. Tesseract and, in turn, Tesstrain are handy tools for training custom models. Their default performance might be insufficient for low-resource languages, necessitating synthetic data or parameter adjustments. I went through 15+ model variations, each time tweaking a single factor, whether it was the Markov model clusters, augmentations of the synthetic data, or the number of iterations. Getting the best model takes trial and error. There is no one-size-fits-all model.





