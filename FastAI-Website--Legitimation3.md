# FastAI-Website-Legitimation

## Introduction¶

This project part is about to demonstrate how to use FastAI to detect NSFW (Not Safe for Work) images on a website.

Flow:
![image.png](attachment:image.png)

## Part#1 - searchEngineMining & test websites images download

The SE (search engine downloader) is needed for training FastAI to distinct between 'Good' to 'Bad' (NSFW) images.
Since, we need to download amount of non pleasant images, we'll use this specific tool.


```python
#Step:1 - All search engine related func
import cv2
def getData(searchUrl, folder):
    print("Get main url")
    global data
    global justFile
    # Open the URL as Browser, not as python urllib
    page = urllib.request.Request(searchUrl, headers={
        'User-Agent': '"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.69 Safari/537.36"'})
    soup = urllib.request.urlopen(page).read()
    data = BeautifulSoup(soup, "html.parser")
    justFile = folder + '/search_page.txt'
    file2write = open(justFile, 'w', encoding='utf-8')
    file2write.write(str(data))
    file2write.close()
    return

#read line by line
def imgMethod2(justFile):
    global urls
    urls = []
    with open(justFile, encoding="mbcs", errors='ignore') as f:
        line = f.readline()
        cnt = 1
        while line:
            if line.find('"http') != -1:
                line = line[line.find('"http') + 1:]
                line = line.split('"')[0]
                if line.find(".jpg") != -1 or line.find(".jpeg") != -1:
                    urls.append(line)

            line = f.readline()
            cnt += 1

    urls = list(set(urls))
    if len(urls) > 500:
        urls = urls[0:500]
    return

#Download img
def imgDownloader(urls):
    for k, valueC in enumerate(urls):
        if valueC.find(".jpg") != -1:
            imgExt = ".jpg"
        elif valueC.find(".jpeg") != -1:
            imgExt = ".jpeg"
        imgFile = folder + '/' + str(k) + imgExt
        try:
            img = requests.get(valueC).content
            with open(imgFile, 'wb') as f:
                f.write(img)
                f.close()
        except:
            continue
    return


def imgValidnResize(folder):
    #Check image valid: read, size
    imgList = os.listdir(folder)
    # validate if file is too small 2KB
    for i, value in enumerate(imgList):
        imgFullPath = folder + '/' + value
        file_size = os.stat(imgFullPath).st_size
        if file_size < 4000 or file_size > 16000000000:
            os.remove(imgFullPath)
        else:
            try:
                img = cv2.imread(imgFullPath, cv2.IMREAD_UNCHANGED)
                img.shape
            except:
                os.remove(imgFullPath)

    #resize
    imgList = os.listdir(folder)
    # validate if file is too small 2KB
    for i, value in enumerate(imgList):
        imgFullPath = folder + '/' + value
        # Crop & Resize
        img = cv2.imread(imgFullPath, cv2.IMREAD_UNCHANGED)
        # dsize
        dsize = (224, 224)
        # resize image
        output = cv2.resize(img, dsize)
        cv2.imwrite(imgFullPath, output)
    return
```


```python
#Step:2 - run SE downloaed
#List keywords
searchUrl = 'https://www.google.com/search?tbm=isch&sxsrf=ALeKk01WaEv6oHDoZ_QusigxEC439dDEMQ%3A1591724524753&source=hp&biw=1680&bih=907&ei=7MnfXtXLK4-ZkgWAy5B4&q=cannabis&oq=cannabis&gs_lcp=CgNpbWcQAzICCAA6BwgjEOoCECc6BQgAELEDUJ4cWK0sYNg5aAFwAHgAgAGlAYgBhgmSAQMwLjiYAQCgAQGqAQtnd3Mtd2l6LWltZ7ABCg&sclient=img&ved=0ahUKEwjV8YD3o_XpAhWPjKQKHYAlBA8Q4dUDCAc&uact=5'
ligalList = ['tech']
for x, valueX in enumerate(ligalList):
    searchUrl = 'https://www.google.com/search?tbm=isch&sxsrf=ALeKk01WaEv6oHDoZ_QusigxEC439dDEMQ%3A1591724524753&source=hp&biw=1680&bih=907&ei=7MnfXtXLK4-ZkgWAy5B4&q=cannabis&oq=cannabis&gs_lcp=CgNpbWcQAzICCAA6BwgjEOoCECc6BQgAELEDUJ4cWK0sYNg5aAFwAHgAgAGlAYgBhgmSAQMwLjiYAQCgAQGqAQtnd3Mtd2l6LWltZ7ABCg&sclient=img&ved=0ahUKEwjV8YD3o_XpAhWPjKQKHYAlBA8Q4dUDCAc&uact=5'
    searchUrl = searchUrl.replace('cannabis', valueX)
    folder = 'G:/DataScienceProject/webcrawler/searchEngineImages/' + valueX
    if os.path.exists(folder) == False:
        os.mkdir(folder)
    getData(searchUrl, folder)
    imgMethod2(justFile)
    imgDownloader(urls)
    imgValidnResize(folder)

```

## Step:3 - Arrange the train folder
## Was to easy, i done it manually, as followed:
G:/DataScienceProject/webcrawler/train/nsfw/alcohol
G:/DataScienceProject/webcrawler/train/nsfw/cannabis
G:/DataScienceProject/webcrawler/train/nsfw/gambling
G:/DataScienceProject/webcrawler/train/nsfw/nude
G:/DataScienceProject/webcrawler/train/nsfw/porn
G:/DataScienceProject/webcrawler/train/nsfw/tobacco_smoking
G:/DataScienceProject/webcrawler/train/nsfw/violence
G:/DataScienceProject/webcrawler/train/regular/animal
G:/DataScienceProject/webcrawler/train/regular/beach
G:/DataScienceProject/webcrawler/train/regular/business
G:/DataScienceProject/webcrawler/train/regular/car
G:/DataScienceProject/webcrawler/train/regular/cellular
G:/DataScienceProject/webcrawler/train/regular/coffee
G:/DataScienceProject/webcrawler/train/regular/hotel
G:/DataScienceProject/webcrawler/train/regular/tech
G:/DataScienceProject/webcrawler/train/regular/telecom
G:/DataScienceProject/webcrawler/train/regular/vacation

## Note: before move image folder into train folder, make sure that each image has a unique name.


```python
#Step:4 -  Train/CV Splitter
import random
import shutil
import os
folder = 'G:/DataScienceProject/webcrawler/train/'
folderList = os.listdir(folder)
for i, valueA in enumerate(folderList):
    subfolder1 = folder + valueA + '/'
    subFolderList = os.listdir(subfolder1)
    for j, valueB in enumerate(subFolderList):
        subfolder2 = subfolder1 + valueB
        print(subfolder2)

        catSubFolderList = os.listdir(subfolder2)
        sampleSize = int(len(catSubFolderList) / 10)
        sampled_list = random.sample(catSubFolderList, sampleSize)
        for k, valueC in enumerate(sampled_list):
            imgSrcPath = subfolder2 + '/' + valueC
            imgDstPath = imgSrcPath.replace("train", "cv")
            shutil.copy(imgSrcPath, imgDstPath)
            os.remove(imgSrcPath)
```


```python
#Step:5 - Download test webstie images
import requests
from bs4 import BeautifulSoup
import os
import cv2
'''
testList = ['https://www.hbo.com/']
'''
path = 'G:/DataScienceProject/webcrawler/test/'
for i, valueA in enumerate(testList):
    #Website name
    websiteName = valueA.split(".")[1]
    print(websiteName)
    #Prefix
    prefix = valueA.split(".")[0].split(":")[0]
    filename = path + websiteName + '/full_website.txt'

    try:
        #Load the full website html
        with open(filename, 'r', errors='ignore') as f:
            data = f.read()
            f.close()
        #Extract only image urls
        data = BeautifulSoup(data, "html.parser")
        img_tags = data.find_all('img')
        urls = [img['src'] for img in img_tags]
        urls[0] #If list emtpy create error to force exception
    except:
        #Read line by line - for handling Dell website type
        urls = []
        with open(filename, encoding="mbcs", errors='ignore') as f:
            line = f.readline()
            cnt = 1
            while line:
                if line.find('img src') != -1:
                    line = line[line.find('img src')+9:]
                elif line.find('img alt') != -1:
                    line = line[line.find('src') + 5:]
                elif line.find('meta content') != -1:
                    line = line[line.find('meta content') + 14:]
                line = line.split('"')[0]
                if line[0:2] == '/*' or line == '':
                    {}
                elif line[0] == '/' or line[0:4] == 'http':
                    urls.append(line)
                #print("Line {}: {}".format(cnt, line.strip()))
                line = f.readline()
                cnt += 1

    urls = list(set(urls))
    #Remove error image links - Cisco type
    for n, valueE in enumerate(urls):
        if valueE[0:2] == '//':
            del urls[n]

    #Handle relative urls
    for j, valueB in enumerate(urls):  ##to add website name
        if valueB == '':
            del urls[j]
        elif valueB[0] == '/':
            urls[j] = valueA + valueB

    #Create image folder
    imgPath = path + websiteName + '/img/'
    if os.path.exists(imgPath) == False:
        os.mkdir(imgPath)

    #Download all website images   #### Need to handle web img extension!!
    for k, valueC in enumerate(urls):
        imgExt = urls[k].split(".")[-1]
        if imgExt != 'svg':
            if len(imgExt) > 4 and imgExt[-4] == '=':
                imgExt = imgExt.split("=")[-1]
            imgFile = imgPath + str(k) + '.' + imgExt
        try:
            img = requests.get(valueC).content
            with open(imgFile, 'wb') as f:
                f.write(img)
                f.close()
        except:
            continue

    imgList = os.listdir(imgPath)
    # validate if file is too small 2KB
    for i, value in enumerate(imgList):
        imgFullPath = imgPath + value
        file_size = os.stat(imgFullPath).st_size
        if file_size < 4000 or file_size > 16000000000:
            os.remove(imgFullPath)
        else:
            try:
                img = cv2.imread(imgFullPath, cv2.IMREAD_UNCHANGED)
                img.shape
            except:
                os.remove(imgFullPath)

    imgList = os.listdir(imgPath)
    # validate if file is too small 2KB
    for i, value in enumerate(imgList):
        imgFullPath = imgPath + value
        # Crop & Resize
        img = cv2.imread(imgFullPath, cv2.IMREAD_UNCHANGED)
        xWD = int(img.shape[1] / 224)
        xHI = int(img.shape[0] / 224)
        if xWD > 2 and xHI >2:
            #crop = img[100:300, 100:300] #y: y+h, x: x+w
            for x in range(xWD):
                for y in range(xHI):
                    ystart = y * 224
                    yend = ystart + 224
                    xstart = x * 224
                    xend = xstart + 224
                    crop = img[ystart:yend, xstart:xend]
                    cropName = imgFullPath.split(".")[0] + str(y) + str(x) + str('.')+ imgFullPath.split(".")[1]
                    cv2.imwrite(cropName, crop)

            os.remove(imgFullPath)


```

# Part#2 - FastAI


```python
#Step:6 - FastAI
from fastai.vision import *
import warnings
warnings.filterwarnings('ignore')
path = 'G:/DataScienceProject/webcrawler/train'
folderList = os.listdir(path)
data = ImageDataBunch.from_folder(path,
                                  train=".",
                                  test="../cv",
                                  valid_pct=0.1,
                                  classes=folderList)
```


```python
#Step:7 - Check accuricy over iteration 
from fastai.metrics import error_rate # 1 - accuracy
learn = create_cnn(data, models.resnet34, metrics=accuracy)
defaults.device = torch.device('cuda') # makes sure the gpu is used
learn.fit_one_cycle(30)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>4.019870</td>
      <td>2.513050</td>
      <td>0.224299</td>
      <td>00:24</td>
    </tr>
    <tr>
      <td>1</td>
      <td>3.214707</td>
      <td>1.195631</td>
      <td>0.616822</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2.438077</td>
      <td>0.699407</td>
      <td>0.794393</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1.832302</td>
      <td>0.534569</td>
      <td>0.841121</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1.389410</td>
      <td>0.470504</td>
      <td>0.841121</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1.068376</td>
      <td>0.472817</td>
      <td>0.859813</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>6</td>
      <td>0.822551</td>
      <td>0.467992</td>
      <td>0.850467</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>7</td>
      <td>0.642952</td>
      <td>0.423794</td>
      <td>0.859813</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>8</td>
      <td>0.516821</td>
      <td>0.455216</td>
      <td>0.831776</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>9</td>
      <td>0.416723</td>
      <td>0.428948</td>
      <td>0.869159</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>10</td>
      <td>0.353135</td>
      <td>0.441226</td>
      <td>0.831776</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>11</td>
      <td>0.297801</td>
      <td>0.447643</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>12</td>
      <td>0.247562</td>
      <td>0.535614</td>
      <td>0.822430</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>13</td>
      <td>0.202789</td>
      <td>0.522058</td>
      <td>0.831776</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>14</td>
      <td>0.166033</td>
      <td>0.478015</td>
      <td>0.859813</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>15</td>
      <td>0.141566</td>
      <td>0.492496</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>16</td>
      <td>0.120358</td>
      <td>0.481068</td>
      <td>0.850467</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.102586</td>
      <td>0.531820</td>
      <td>0.803738</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>18</td>
      <td>0.086209</td>
      <td>0.547795</td>
      <td>0.831776</td>
      <td>00:14</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.077022</td>
      <td>0.534279</td>
      <td>0.831776</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.071503</td>
      <td>0.541410</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.060186</td>
      <td>0.564443</td>
      <td>0.831776</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.056534</td>
      <td>0.565441</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.051183</td>
      <td>0.544058</td>
      <td>0.850467</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.045064</td>
      <td>0.539170</td>
      <td>0.850467</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>25</td>
      <td>0.041406</td>
      <td>0.537626</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>26</td>
      <td>0.039970</td>
      <td>0.532053</td>
      <td>0.850467</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>27</td>
      <td>0.037368</td>
      <td>0.539188</td>
      <td>0.850467</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.033319</td>
      <td>0.534159</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.030496</td>
      <td>0.537952</td>
      <td>0.841121</td>
      <td>00:15</td>
    </tr>
  </tbody>
</table>



```python
learn.unfreeze() # must be done before calling lr_find
learn.lr_find()
learn.recorder.plot()
```



    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='0' class='' max='7' style='width:300px; height:20px; vertical-align: middle;'></progress>
      0.00% [0/7 00:00<00:00]
    </div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table><p>

    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='11' class='' max='15' style='width:300px; height:20px; vertical-align: middle;'></progress>
      73.33% [11/15 00:08<00:03 0.0354]
    </div>



    LR Finder is complete, type {learner_name}.recorder.plot() to see the graph.
    


![png](output_11_2.png)



```python
learn.fit_one_cycle(30, max_lr=slice(3e-5, 3e-4))
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0.032195</td>
      <td>0.523867</td>
      <td>0.850467</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0.023974</td>
      <td>0.515463</td>
      <td>0.850467</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>2</td>
      <td>0.028097</td>
      <td>0.522798</td>
      <td>0.859813</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>3</td>
      <td>0.029878</td>
      <td>0.504512</td>
      <td>0.859813</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>4</td>
      <td>0.029407</td>
      <td>0.595161</td>
      <td>0.850467</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>5</td>
      <td>0.036940</td>
      <td>0.733822</td>
      <td>0.813084</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>6</td>
      <td>0.044722</td>
      <td>0.629720</td>
      <td>0.841121</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>7</td>
      <td>0.045579</td>
      <td>0.645625</td>
      <td>0.822430</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>8</td>
      <td>0.052527</td>
      <td>0.598010</td>
      <td>0.859813</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>9</td>
      <td>0.061957</td>
      <td>0.556766</td>
      <td>0.869159</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>10</td>
      <td>0.061259</td>
      <td>0.665028</td>
      <td>0.813084</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>11</td>
      <td>0.054622</td>
      <td>0.538378</td>
      <td>0.822430</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>12</td>
      <td>0.047597</td>
      <td>0.611345</td>
      <td>0.850467</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>13</td>
      <td>0.043337</td>
      <td>0.521048</td>
      <td>0.850467</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>14</td>
      <td>0.040517</td>
      <td>0.447970</td>
      <td>0.869159</td>
      <td>00:21</td>
    </tr>
    <tr>
      <td>15</td>
      <td>0.032732</td>
      <td>0.533471</td>
      <td>0.887850</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>16</td>
      <td>0.031231</td>
      <td>0.538213</td>
      <td>0.878505</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.026833</td>
      <td>0.535733</td>
      <td>0.869159</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>18</td>
      <td>0.030070</td>
      <td>0.515098</td>
      <td>0.878505</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.028632</td>
      <td>0.545439</td>
      <td>0.869159</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.024675</td>
      <td>0.525689</td>
      <td>0.869159</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.021947</td>
      <td>0.529568</td>
      <td>0.859813</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.017879</td>
      <td>0.533107</td>
      <td>0.859813</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.016757</td>
      <td>0.534468</td>
      <td>0.869159</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.016287</td>
      <td>0.528689</td>
      <td>0.850467</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>25</td>
      <td>0.014530</td>
      <td>0.526682</td>
      <td>0.859813</td>
      <td>00:20</td>
    </tr>
    <tr>
      <td>26</td>
      <td>0.013652</td>
      <td>0.526999</td>
      <td>0.859813</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>27</td>
      <td>0.013781</td>
      <td>0.520550</td>
      <td>0.859813</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.012946</td>
      <td>0.513816</td>
      <td>0.850467</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.012324</td>
      <td>0.537055</td>
      <td>0.859813</td>
      <td>00:19</td>
    </tr>
  </tbody>
</table>



```python
learn.unfreeze() # must be done before calling lr_find
learn.lr_find()
learn.recorder.plot()
```



    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='3' class='' max='7' style='width:300px; height:20px; vertical-align: middle;'></progress>
      42.86% [3/7 00:36<00:48]
    </div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0.010775</td>
      <td>#na#</td>
      <td>00:12</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0.010371</td>
      <td>#na#</td>
      <td>00:11</td>
    </tr>
    <tr>
      <td>2</td>
      <td>0.016333</td>
      <td>#na#</td>
      <td>00:11</td>
    </tr>
  </tbody>
</table><p>

    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='4' class='' max='15' style='width:300px; height:20px; vertical-align: middle;'></progress>
      26.67% [4/15 00:06<00:18 0.0269]
    </div>



    LR Finder is complete, type {learner_name}.recorder.plot() to see the graph.
    


![png](output_13_2.png)



```python
#Step:8 - Cleansing images
from fastai.widgets import *


ds, idxs = DatasetFormatter().from_toplosses(learn)
ImageCleaner(ds, idxs, path)
if os.path.exists('G:/DataScienceProject/webcrawler/train/models/tmp.pth'):
    os.remove('G:/DataScienceProject/webcrawler/train/models/tmp.pth')
    os.rmdir('G:/DataScienceProject/webcrawler/train/models')
else:
    {}

if os.path.exists('G:/DataScienceProject/webcrawler/train/cleaned.csv'):
    df = pd.read_csv('G:/DataScienceProject/webcrawler/train/cleaned.csv', header='infer')
    df['class'] = df.name.str.rsplit("\\", expand=True)[0]


if (df['class'].all() != df['label'].all()) == True:
    os.remove('G:/DataScienceProject/webcrawler/train/cleaned.csv')
else:
    {}
```


    HBox(children=(VBox(children=(Image(value=b'\xff\xd8\xff\xe0\x00\x10JFIF\x00\x01\x01\x01\x00d\x00d\x00\x00\xff…



    Button(button_style='primary', description='Next Batch', layout=Layout(width='auto'), style=ButtonStyle())



```python
#Step:9 - FastAI re-run after cleansing images
from fastai.vision import *
import warnings
warnings.filterwarnings('ignore')
path = 'G:/DataScienceProject/webcrawler/train'
folderList = os.listdir(path)
data = ImageDataBunch.from_folder(path,
                                  train=".",
                                  test="../cv",
                                  valid_pct=0.1,
                                  classes=folderList)
```


```python
#Step:11 
learn.data = data
learn.freeze()
learn.fit_one_cycle(30)
learn.unfreeze()
learn.lr_find()
learn.recorder.plot()
learn.fit_one_cycle(30, max_lr=slice(3e-5, 3e-4))
#learn.save('G:/DataScienceProject/webcrawler/fastai')
learn.export('G:/DataScienceProject/webcrawler/fastai.pkl')
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>4.388728</td>
      <td>3.792895</td>
      <td>0.719626</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>1</td>
      <td>3.409734</td>
      <td>1.285964</td>
      <td>0.719626</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2.127457</td>
      <td>0.066224</td>
      <td>0.971963</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1.360148</td>
      <td>0.036858</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>4</td>
      <td>0.918974</td>
      <td>0.045735</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>5</td>
      <td>0.644385</td>
      <td>0.056198</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>6</td>
      <td>0.463193</td>
      <td>0.048133</td>
      <td>0.990654</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>7</td>
      <td>0.337698</td>
      <td>0.049918</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>8</td>
      <td>0.251245</td>
      <td>0.072447</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>9</td>
      <td>0.189782</td>
      <td>0.078490</td>
      <td>0.981308</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>10</td>
      <td>0.150335</td>
      <td>0.090115</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>11</td>
      <td>0.115437</td>
      <td>0.080476</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>12</td>
      <td>0.093794</td>
      <td>0.071528</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>13</td>
      <td>0.076764</td>
      <td>0.069016</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>14</td>
      <td>0.062704</td>
      <td>0.087429</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>15</td>
      <td>0.055909</td>
      <td>0.077544</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>16</td>
      <td>0.048595</td>
      <td>0.072639</td>
      <td>0.981308</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.045188</td>
      <td>0.064403</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>18</td>
      <td>0.035209</td>
      <td>0.063259</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.030674</td>
      <td>0.063957</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.033988</td>
      <td>0.064423</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.030383</td>
      <td>0.066299</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.029165</td>
      <td>0.070754</td>
      <td>0.990654</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.024103</td>
      <td>0.071962</td>
      <td>0.981308</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.021907</td>
      <td>0.071542</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>25</td>
      <td>0.020802</td>
      <td>0.075296</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>26</td>
      <td>0.018870</td>
      <td>0.075254</td>
      <td>0.981308</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>27</td>
      <td>0.016905</td>
      <td>0.075799</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.018403</td>
      <td>0.077508</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.017417</td>
      <td>0.074403</td>
      <td>0.990654</td>
      <td>00:15</td>
    </tr>
  </tbody>
</table>




    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='0' class='' max='7' style='width:300px; height:20px; vertical-align: middle;'></progress>
      0.00% [0/7 00:00<00:00]
    </div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table><p>

    <div>
        <style>
            /* Turns off some styling */
            progress {
                /* gets rid of default border in Firefox and Opera. */
                border: none;
                /* Needs to be in here for Safari polyfill so background images work as expected. */
                background-size: auto;
            }
            .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
                background: #F44336;
            }
        </style>
      <progress value='1' class='' max='15' style='width:300px; height:20px; vertical-align: middle;'></progress>
      6.67% [1/15 00:05<01:21 0.0008]
    </div>



    LR Finder is complete, type {learner_name}.recorder.plot() to see the graph.
    


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>epoch</th>
      <th>train_loss</th>
      <th>valid_loss</th>
      <th>accuracy</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0.011374</td>
      <td>0.074097</td>
      <td>0.990654</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0.011084</td>
      <td>0.071579</td>
      <td>0.990654</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>2</td>
      <td>0.016282</td>
      <td>0.073622</td>
      <td>0.990654</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>3</td>
      <td>0.014621</td>
      <td>0.079858</td>
      <td>0.990654</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>4</td>
      <td>0.017397</td>
      <td>0.072116</td>
      <td>0.981308</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>5</td>
      <td>0.018927</td>
      <td>0.088501</td>
      <td>0.990654</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>6</td>
      <td>0.020343</td>
      <td>0.070200</td>
      <td>0.981308</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>7</td>
      <td>0.035146</td>
      <td>0.286476</td>
      <td>0.971963</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>8</td>
      <td>0.052318</td>
      <td>0.409916</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>9</td>
      <td>0.060069</td>
      <td>0.227677</td>
      <td>0.981308</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>10</td>
      <td>0.058007</td>
      <td>0.233905</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>11</td>
      <td>0.053529</td>
      <td>0.191048</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>12</td>
      <td>0.044918</td>
      <td>0.230210</td>
      <td>0.943925</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>13</td>
      <td>0.039017</td>
      <td>0.247123</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>14</td>
      <td>0.034531</td>
      <td>0.200228</td>
      <td>0.971963</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>15</td>
      <td>0.030813</td>
      <td>0.200558</td>
      <td>0.953271</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>16</td>
      <td>0.027267</td>
      <td>0.191412</td>
      <td>0.962617</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.026371</td>
      <td>0.205397</td>
      <td>0.953271</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>18</td>
      <td>0.022412</td>
      <td>0.201516</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.019796</td>
      <td>0.202040</td>
      <td>0.962617</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.017534</td>
      <td>0.202850</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.014566</td>
      <td>0.203923</td>
      <td>0.971963</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.013925</td>
      <td>0.218110</td>
      <td>0.971963</td>
      <td>00:17</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.014358</td>
      <td>0.216289</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.011916</td>
      <td>0.218032</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>25</td>
      <td>0.010464</td>
      <td>0.215107</td>
      <td>0.943925</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>26</td>
      <td>0.009369</td>
      <td>0.213085</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>27</td>
      <td>0.008085</td>
      <td>0.210171</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.008314</td>
      <td>0.214848</td>
      <td>0.953271</td>
      <td>00:16</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.008517</td>
      <td>0.207528</td>
      <td>0.962617</td>
      <td>00:17</td>
    </tr>
  </tbody>
</table>



![png](output_16_4.png)



```python
interp = ClassificationInterpretation.from_learner(learn)
interp.plot_confusion_matrix()
```






![png](output_17_1.png)



```python
import fastai

learn = load_learner('G:/DataScienceProject/webcrawler/', 'fastai.pkl')

nsfwList = ['alcohol', 'cannabis', 'gambling', 'nude', 'porn', 'tobacco_smoking', 'violence']
testFolders = os.listdir('G:/DataScienceProject/webcrawler/test/')

for i, valueA in enumerate(testFolders):
    imgPath = 'G:/DataScienceProject/webcrawler/test/' + valueA + '/img/'
    imgTestList = os.listdir(imgPath)
    if len(imgTestList) != 0:
        nswfCounter = 0
        for j, valueB in enumerate(imgTestList):
            file = imgPath + valueB
            img = open_image(file)
            pred_class, pred_idx, output = learn.predict(img)

            for k, valueC in enumerate(nsfwList):
                if valueC == str(pred_class):
                    nswfCounter += 1

        msg = "NSFW images: " + str(int( (nswfCounter / len(imgTestList)) *100)) + "%"
        #print(msg)
        reportFile = 'G:/DataScienceProject/webcrawler/test/' + valueA + '/' + valueA + '.txt'
        with open(reportFile, 'a+') as f:
            f.write(f"{msg}\n")
            f.close()
```


```python

```
