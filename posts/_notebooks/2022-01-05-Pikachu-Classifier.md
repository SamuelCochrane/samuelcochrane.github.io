---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.13.6
  kernelspec:
    display_name: Python 3
    name: python3
---

<!-- #region id="CjuEoqyaFzWC" -->
---
title: I built a Pikachu classifer because there's like 8 of them.
# toc: false
style: border #fill, border
color: success #primary / secondary / success / danger / warning / info / light / dark (choose one only)
image: https://remnote-user-data.s3.amazonaws.com/x4aWaP4M69AxYfnmxF0EqJvbd2S-ZLwct9l6DkN5x-QO5pwQW3LhUbLsXO7b5WVtrB4Z4G2nnMp03JW1HzwqFFiwZMp7DT1nP8HQrMcSpWohVhjSTzO3E3De7ulKOubv.gif
categories:
  - Portfolio
tags: 
  - FastAI
  - CNN
  - Machine Learning
published: true
---
<!-- #endregion -->

<!-- #region id="Ff0xUCzq3dYs" -->
# Environment Setup (required)
<!-- #endregion -->

```python id="zpf_rV6ZhEZI"
# type the following to make any cell get skipped during a "Run All"
# %%skip True
def skip(line, cell=None):
    '''Skips execution of the current line/cell if line evaluates to True.'''
    if eval(line):
        return

    get_ipython().run_cell(cell)

def load_ipython_extension(shell):
    '''Registers the skip magic when the extension loads.'''
    shell.register_magic_function(skip, 'line_cell')

def unload_ipython_extension(shell):
    '''Unregisters the skip magic when the extension unloads.'''
    del shell.magics_manager.magics['cell']['skip']
    
    
load_ipython_extension(get_ipython())
```

<!-- #region id="D8elra6FX6AZ" -->
install fastai
<!-- #endregion -->

```python id="rNqx82Z2lhIz"
! [ -e /content ] && pip install -Uqq fastai --upgrade
```

```python colab={"base_uri": "https://localhost:8080/"} id="4Gkhaikt2ZV2" outputId="16e130cc-1294-4631-d62b-fb3b7477fa00"
import fastai
print(fastai.__version__) # should show >=2.0, not 1.0.61
```

<!-- #region id="IN89dQcl2VVx" -->
install imagemagick
<!-- #endregion -->

```python id="IgfqvJWI2Ubs" colab={"base_uri": "https://localhost:8080/"} outputId="0ce633e0-822a-42b7-9332-87e2185a382d"
!apt install imagemagick
```

<!-- #region id="E4DeR2KQX9OQ" -->
import libraries
<!-- #endregion -->

```python id="JpRlbFz7x5GL"
from fastai.vision.all import *
import fastai.vision.widgets as fastwidgets
from ipywidgets import widgets as iwidgets
from pandas.api.types import CategoricalDtype
import matplotlib as mpl
import graphviz
import math   
#from google.colab import widgets
```

<!-- #region id="gkIMapJ-X_P8" -->
Set up data directory (including Drive)
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/"} id="6UO4GIG4431C" outputId="7045d2ca-e5cf-48f3-8d7f-42f061c06fb1"
import os
from google.colab import drive
MOUNTPOINT = '/content/gdrive'
DATADIR = os.path.join(MOUNTPOINT, 'My Drive', 'Colab Data', 'Pikachu Classifier')
drive.mount(MOUNTPOINT)
```

```python colab={"base_uri": "https://localhost:8080/", "height": 35} id="oj44-WP06C-6" outputId="362b6729-19ce-4720-8e5d-16fb1780715e"
path = Path(DATADIR)
if not path.exists():
    os.makedirs(path)
os.chdir(path)

os.getcwd()
```

<!-- #region id="dGQkAVi7YE-9" -->
Set up image search
<!-- #endregion -->

```python id="aX6lYybL3mih"
bingApiKey = '4a19d080f5b24133ac280f96334932fa'
```

```python id="Q7bTi8H2x_xj"
def search_images_bing(key, term, min_sz=128, max_images=150):    
     params = {'q':term, 'count':max_images, 'min_height':min_sz, 'min_width':min_sz}
     headers = {"Ocp-Apim-Subscription-Key":key}
     search_url = "https://api.bing.microsoft.com/v7.0/images/search"
     response = requests.get(search_url, headers=headers, params=params)
     response.raise_for_status()
     search_results = response.json()    
     return L(search_results['value'])
```

```python id="VkcNISTlvIlq"
def remove_failed_downloads(path):
  fns = get_image_files(path) #get filenames
  failed = verify_images(fns) #find corrupted/failed files
  failed.map(Path.unlink)     #remove those files
```

```python id="amm7VSGE0oqW"
#sets a transparent background to white for all images
def remove_transparent_bg(path): 
  %cd $path
  ! mogrify -background white -alpha remove -alpha off
```

<!-- #region id="KMqQvjZD9kTm" -->
# Test run of image downloading code (optional)
<!-- #endregion -->

```python cellView="form" id="_2M9MJmBS1xs"

#@markdown Skip this section (Yes/No)
skip_section = True #@param {type:"boolean"}

```

```python id="N8UEvFtbzrm1"
%%skip skip_section
results = search_images_bing(bingApiKey, 'minun pokemon', max_images=10) # returns a list of image urls, among other things
urls = results.attrgot('contentUrl') #get the array of the URLs
len(urls), urls[0] #should show {max_images}, for {max_images} retrieved URLs
```

```python id="SeYDiK3l4ia8"
%%skip skip_section
path = os.path.join(DATADIR,"pokemon") #folder for all pokemon
dest = Path(os.path.join(path,"minun")) #subfolder for just this one
#create the data path if needed
if not Path(dest).exists():
  os.makedirs(dest)
```

```python id="wAWjL8JixfkF"
%%skip skip_section
download_images(dest, urls=urls, n_workers=0)
len(os.listdir(path)) #how many images did we download? (this may be less than {max_images} due to some urls being invalid)
```

```python id="o8kSEOPY5gd2"
%%skip skip_section
#what does the first image in this class' folder look like?
id = os.listdir(Path(path))[0]
im = Image.open(os.path.join(path, id))
im.to_thumb(128,128)
```

<!-- #region id="Hu24m8rVEA2c" -->
working ok? Now let's download our pokemon for real.
<!-- #endregion -->

```python id="hmk1sVeIcy_c"
#delete our test batch

```

<!-- #region id="ggusD0Ica6Cu" -->
# Setting up the data (required)
<!-- #endregion -->

<!-- #region id="RpZKybCdC2vg" -->
Download the image dataset
<!-- #endregion -->

```python id="Gc0b0D1Z8j-f" colab={"base_uri": "https://localhost:8080/", "height": 571} outputId="4b206abf-8f00-4379-d49f-16139f197a8d"
pokemon_list = ['pikachu','mimikyu',
                'plusle -minun', #adding the minus operator to exclude images 
                'minun -plusle', #of both together, since they're so common
                'dedenne', 'togedemaru', 
                'morpeko', 'emolga', 'pachirisu']
num_images = 50
path = os.path.join(DATADIR,"pokemon")
if not Path(path).exists():
  os.makedirs(path)

grid = widgets.Grid(3, 3)


for i, o in enumerate(pokemon_list):
  dest = Path(os.path.join(path,o))
  if not dest.exists(): ## skips existing datasets that are already DL'd
    dest.mkdir(exist_ok=True)
    results = search_images_bing(bingApiKey, f'{o} pokemon')
    urls = results.attrgot('contentUrl')
    download_images(dest, urls=results.attrgot('contentUrl'), max_pics=num_images, n_workers=0)
    remove_failed_downloads(dest)
    remove_transparent_bg(dest)
  with grid.output_to(i%3,math.floor(i/3)):
    print(o)
    print("DL'd Images: ", len(os.listdir(dest)))
    exampleimg = os.path.join(dest, os.listdir(Path(dest))[0])
    display(Image.open(exampleimg).to_thumb(128,128))
    print()

```

<!-- #region id="Ty3nV6nzAUzE" -->
Setting up data objects
<!-- #endregion -->

```python id="WKHlZuIJb_fv"
#setting up the datablock, this will act as a unified 'template'
#for our dataloaders for each pokemon.
pokemonBlock = DataBlock(
	blocks=(ImageBlock, CategoryBlock),    	
	get_items=get_image_files,    
	splitter=RandomSplitter(valid_pct=0.2, seed=42),    
	get_y=parent_label,    
	item_tfms=RandomResizedCrop(224, min_scale=0.5), #randomly crop the image on each Epoch
  batch_tfms=aug_transforms()# Utility func to flip, rotate, zoom, warp,etc. the images
 )
```

```python colab={"base_uri": "https://localhost:8080/", "height": 514} id="Z76nH_F6rTma" outputId="a86dc70d-03a7-40f3-e6ad-58abba5a8ce2"
#setting up our dataloaders
dls = pokemonBlock.dataloaders(path)
dls.train.show_batch(max_n=8, nrows=2) #show me some data
```

<!-- #region id="nSNyV4H6AX_m" -->
# Build our model (required)
<!-- #endregion -->

```python colab={"base_uri": "https://localhost:8080/", "referenced_widgets": ["f6e0421a05f04bc2a16554c84789dc32", "cd83495db3ce48199cf0af8d7123f7e6", "acada7b81d8f43ee9b785aa63545b4fd", "fe122d55c0234242bf87827c399db871", "26afc051715543cebe23756901ceea86", "d2ec78855d714b979c25de04bf9247a4", "d7c801786abb4473bc1d39892c19665e", "7596db078e554dd8acfc6c1d35bdab33", "e5e711519fcc4d4796cecc66db4d4575", "78ce32d2093f4d48baccc0535e5d9e3f", "ff1a581689014a11a841c367d1230404"]} id="dKAZtSxKtXYV" outputId="dffc1b26-2eed-4b3c-9faa-d8fda10a5794"
learn = cnn_learner(dls, resnet18, metrics=error_rate)
learn.fine_tune(4)
```

```python id="3-tGTETI1XS7" colab={"base_uri": "https://localhost:8080/"} outputId="25625c32-c932-418a-f75c-c87f6d783557"
#graph how well it's doing (confusion matrix)
interp = ClassificationInterpretation.from_learner(learn)
interp.plot_confusion_matrix()
```

```python id="U5MgeWlTNIC3"
#save our model
learn.export("pikachu_classifier")
```

<!-- #region id="aSFbNqE-9YCH" -->
# Clean the data (optional)
<!-- #endregion -->

```python cellView="form" id="FjeFlihWR2RI"

#@markdown Skip this section (Yes/No)
skip_section = True #@param {type:"boolean"}


```

```python id="AZ9w1wMb32I7"
%%skip skip_section
cleaner = fastwidgets.ImageClassifierCleaner(learn)
cleaner
```

```python id="WzLTPSA68yA-"
%%skip skip_section
#move all the data we re-labeled to the proper folder
for idx,cat in cleaner.change(): 
  shutil.move(str(cleaner.fns[idx]), Path(os.path.join(path, cat)))
```

```python id="hBBGPGh96rF4"
%%skip skip_section
#move all the data we labeled as deleted to a "deleted" folder
dest = os.path.join(Path(path).parent,"deleted")
if not Path(dest).exists():
  os.makedirs(dest)
for idx in cleaner.delete(): 
  shutil.move(str(cleaner.fns[idx]), dest) #*
  #*or use this to simply delete them: 
  # cleaner.fns[idx].unlink() 
  # and you can always delete the full folder later using 
  # shutil.rmtree(dest)
```

```python id="ESraaXy1_y99"
#DONE! :)
```

```python id="0cozuaBxmz5H"
!pip install voila
!jupyter serverextension enable voila --sys-prefix
```

<!-- #region id="BatehbherKE2" -->
# Testing It Out
<!-- #endregion -->

```python id="1y8_Z8UkWTC3"
#@markdown Enable this to use the .pkl file exported by the model. Leave it unchecked to use the model you just trained.
use_existing_model = True #@param {type:"boolean"}

if(use_existing_model):
  learn = load_learner("pikachu_classifier")
```

```python colab={"base_uri": "https://localhost:8080/"} id="FDSksq9WuGDC" outputId="91047add-787f-438b-97a2-daedfee2084f"
!pip install ipywebrtc    

```

```python id="ugKSIERTf2IV"
import requests
from io import BytesIO  
```

```python id="FfwbCgljph8p"
def on_new_image_upload(change):
  img = PILImage.create(btn_upload.data[-1])    
  img_display.from_file(img)
  on_new_image(img)
def on_new_image_webcam(change):
  #camera.close()
  #image_recorder.close()
  img = (image_recorder.image.value)  #PILImage.create(btn_upload.data[-1])    
  img_display.value = (img) #img_display.from_file(img)
  on_new_image(img)
def on_new_image(img):    
  out_pl.clear_output()
  img_display.layout.visibility = 'visible'
  img_display.layout.height = 640
  with out_pl:  
    pred,pred_idx,probs = learn.predict(img)    
    lbl_pred.value = f'Prediction: {pred}; Probability: {probs[pred_idx]:.04f}'
```

```python id="nZ-WoFYUtBK0"
from ipywebrtc import CameraStream, ImageRecorder
from google.colab import output
output.enable_custom_widget_manager()

camera = CameraStream(constraints=
                      {'facing_mode': 'user',
                       'audio': False,
                       'video': { 'width': 640, 'height': 480 }
                       })
image_recorder = ImageRecorder(stream=camera)
image_recorder.image.observe(on_new_image_webcam, names=['value'])

def take_photo(change):
  img_display.layout.visibility = 'hidden'
  img_display.layout.height = 1
  with out_pl:
    display(iwidgets.VBox([camera, image_recorder]))
  
```

```python colab={"base_uri": "https://localhost:8080/", "height": 239} id="muQ9h_cRuOPu" outputId="d9e9614a-2769-4b81-d9af-b19d2d6e63dd"
btn_upload = iwidgets.FileUpload(multiple=False)
btn_upload.observe(on_new_image_upload, names='_counter')
btn_camera = iwidgets.Button(description="Camera")
btn_camera.on_click(take_photo)
out_pl = iwidgets.Output()
response = requests.get("https://cdn.vox-cdn.com/uploads/chorus_image/image/53254027/who_pokemon.0.jpg")
img = Image.open(BytesIO(response.content)).to_bytes_format() ##TODO fix namespace conflict for "Image" between PIL and something else...
img_display = iwidgets.Image(
    value=img,
    format='png',
    width=640,
    height=480
)
lbl_pred = iwidgets.Label()
```

```python id="Hfiy_MYGtxeq"

```

```python colab={"base_uri": "https://localhost:8080/", "height": 477, "referenced_widgets": ["944dea4154e34f518b7e1a259ed4ef38", "e14d628ff92f40268359cc791ab7a914", "3764fc61d3414379badf700242ec664d", "f8c20c0316354dd6bd7d609bb160ecb1", "7dab6b050ab64ea481b8451402b37a23", "b05f7e2e0e7f4f6d81b41d58ad0b4b9e", "efe25a6403b346dbafedbf04117ee051", "f6ac44426e574efab1193b5fbef41bac", "9cb712f7cacc42dda3755c5c2f34d4c2", "be78cc01f4844a43be69d62a026c666c", "ac38ef3276ef4da494703451bac885e9", "c10da1be6a7544e3bf162d82c83ce7fa", "4ab1e5380e5f4902a895620a8ff535dc", "7eed640eaecd4a96b13f8cfd0cf9b590", "32c1052cfec44b008c8fb2a05996f642", "c9bc5f9f3f274a2aabf1f3645afab74c", "de2423669f5e4cb99a77e67b219dc98b", "d66e883c7ee14ebd8eaf7da9914d9219", "6adcc99357f74a238836e8fa2740534c", "6f4090ef81304ff896ab6973a58239b0"]} id="K970mki7t98l" outputId="ea0cb936-4d2e-4a03-b1da-7c7e640a775e"
iwidgets.VBox([
  iwidgets.Label('Select your \'mon!'),
  img_display,
  out_pl,
  iwidgets.HBox([btn_camera, btn_upload]), 
  lbl_pred
]) 
```

```python id="xdbhPxj05eM_"

```

```python colab={"base_uri": "https://localhost:8080/"} id="xPWTA-ms_E2i" outputId="68167010-ef8e-4683-cf8d-c37257f3a79a"
from IPython.display import Image
try:
  filename = take_photo()
  print('Saved to {}'.format(filename))
  
  # Show the image which was just taken.
  display(Image(filename))
except Exception as err:
  # Errors will be thrown if the user does not have a webcam or if they do not
  # grant the page permission to access it.
  print(str(err))
```
