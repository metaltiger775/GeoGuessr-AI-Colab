# GeoGuessr AI Colab
*Credits to [Stelath](https://github.com/Stelath)*

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/metaltiger775/geoguessr-ai-colab/blob/main/notebook/Geoguessr_AI_Trainer.ipynb) ![License](https://img.shields.io/github/license/Stelath/geoguessr-ai)

This is a repo for running [Stelath](https://github.com/Stelath)'s GeoGuessr AI trainer on Google Colab, for people who don't have access to powerful GPUs and don't want to pay for them using other online services such as [LambdaLabs](https://lambdalabs.com/service/gpu-cloud), [Azure](https://azure.microsoft.com/), or [Vultr](https://www.vultr.com/).


# How to run (Google Colab)

**!!Make sure you Change runtime type to GPU!!** 

<ins>Setup:</ins>
- Open the [Geoguessr_AI_Trainer.ipynb](/notebook/Geoguessr_AI_Trainer.ipynb) file in Google Colab with a GPU instance
- Choose between making your own dataset or using the premade dataset:
  - **Using premade dataset** Edit the "*Download dataset*" cell under the "*Setup and Installation*" cell to add your [Kaggle](https://kaggle.com) username & token ([tutorial](#Tutorial))
  - **Making your own dataset** Edit the settings to your liking and run the cell but **don't forget to add your [Google Street View API key](https://developers.google.com/maps/documentation/streetview/overview)**
- Run the whole "*Setup and Installation*" cell

<ins>Training:</ins>
- Start by editing the settings to your liking **if you know what you're doing** *(all of the settings you see are the default settings)*
- Run the cell
- Wait
- *If you enabled the **save_to_drive** setting, you will find the models in your drive inside the folder **"Geoguessr/models"***


# Tutorial
### How to get your [Kaggle](https://kaggle.com) API token + username

1. Create an account if you haven't already
2. Go to your [Settings](https://www.kaggle.com/settings)
3. Find the **"API"** section
4. Click on **Create New Token**
5. It will save a *.json* file to your computer
6. Open the file in the text editor of your chcoice
7. Replace the line in the Google Colab Notebook with the content of the json file




## Instructions

### Run Pretrained Model

In order to run a pretrained model you can download it from Google Drive: [![Open In Drive](https://img.shields.io/badge/Google%20Drive-5383ec?style=flat&logo=googledrive&logoColor=5383ec&label=%E2%80%8B)](https://drive.google.com/file/d/1VJpeLJp6jC8IUfKy6cAtZ9WZcX1TTutW/view?usp=sharing) or you can use the Google Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/metaltiger775/geoguessr-ai-colab/blob/main/notebook/Geoguessr_AI_Trainer.ipynb).

### Train a Model
You can use the [get_images.py](https://github.com/metaltiger775/geoguessr-ai-colab/blob/main/get_images.py "get_images.py") script to download a database of images through Google Cloud, they allow 28,500 free Google Street View API calls each month so keep that in mind (anything more and you will be charged $0.007 per image), you will also have to set up a Google Cloud account. You can also download the database of Google Street View images I created [here](https://www.kaggle.com/stelath/city-street-view-dataset).

After you have a database of images running the [dataset_builder_multi_label.py](https://github.com/metaltiger775/geoguessr-ai-colab/blob/main/dataset_builder_multi_label.py) script will preprocess all of the images, then running [main.py](https://github.com/metaltiger775/geoguessr-ai-colab/blob/main/main.py) will begin training the model.

Here is a set of commands that would be used to train a model on 25,000 images (keep in mind you will need a cities folder containing `.geojson` files from [Open Addresses](https://openaddresses.io/)):
```
python -m get_images --cities cities/ --output images/ --icount 25000 --key (YOUR GSV API KEY HERE)
python -m dataset_builder_multi_label --file images/picture_coords.csv --images images/ --output geoguessr_dataset/
python -m main geoguessr_dataset/ -a wide_resnet50_2 -b 16 --lr 0.0001 -j 6 --checkpoint-step 1
```


## Project Overview

This project was done by [Stelath](https://github.com/Stelath) as an opportunity to learn more about CNN & RNNs by creating a ML Model that could reliably guess a random location in one of five US cities given only a Google Street View image. The idea was inspired by the game GeoGuessr where the user is given a random Google Street View location and have to guess based on the Street View their location in the world.

### Creating a Dataset

In order to train the model I first had to create a reasonably large dataset of Google Street View images to train it on. To do this I wrote a python script ([get_images.py](https://github.com/metaltiger775/geoguessr-ai-colab/blob/main/get_images.py "get_images.py")) to download a large set of photographs from 5 cities in the US from Google Street View API. In order to download images from the Google Street View API latitude and longitude coordinates were needed, to solve this I utilized an address book from [Open Addresses](https://openaddresses.io/) to get the latitude and longitude data of random street addresses for each of the 5 cities.

### Formatting the Images

Before training the model I decided to format the targets in order to create a way for the AI to better guess the location by turning the GPS coordinates into multi class targets, through formatting each number in the coordinate as a one hot array with numbers ranging one through ten.

### Training the Model

After using a couple different model architectures I settled on a wideresnet with 50 layers which gave comparable results to a wideresnet with 100 layers but took far less GPU memory. The model was trained on a dataset of 50,000 images and had the best performance 20 epochs in and then began to slowly overfit.

### How Could it be Improved?

Using a custom model that is better suited to guessing locations rather than just image classification. Adding far more layers so the model can pick up more complexity however this would require a larger GPU. Arguably the best performance improver - and something that would more accurately follow the GeoGuessr - idea would be to train a 3D CNN on an array of images from the location giving the model far more data to work with and make accurate predictions.

### Takeaway

While the model is by no means perfect, it is suprisingly accurate given the limited input it recives. This interestingly enough reveals that while many would regard American cities as similar there are clearly significant differences in their landscapes so much so that the AI was able to take advantage of them to at least correctly predict the city it was in the majority of hte time.
