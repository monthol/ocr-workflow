# ocr-workflow
## Install Tesseract on mac
To compile with Docker follow this [Link](https://github.com/tesseract-shadow/tesseract-ocr-compilation)
After unzipped, edit file container-scripts/tessdata_download.sh
```
#!/bin/bash

# osd	Orientation and script detection
wget -O ${TESSDATA_PREFIX}/osd.traineddata https://github.com/tesseract-ocr/tessdata/raw/3.04.00/osd.traineddata
# equ	Math / equation detection
wget -O ${TESSDATA_PREFIX}/equ.traineddata https://github.com/tesseract-ocr/tessdata/raw/3.04.00/equ.traineddata
# eng English
wget -O ${TESSDATA_PREFIX}/eng.traineddata https://github.com/tesseract-ocr/tessdata/raw/4.00/eng.traineddata
# tha Thai
wget -O ${TESSDATA_PREFIX}/tha.traineddata https://github.com/tesseract-ocr/tessdata/raw/4.00/tha.traineddata
```
Then build docker container yourself by running shell script dockerfile.build.sh
```
sh dockerfile.build.sh
```
now you can skip script step 1 and 2. Run script 3 to 6
```
sh 3-run-new-container.sh
sh 4-update-src.sh
sh 5-compile-src.sh
docker images /list images
docker ps /list container
docker exec -it t4cmp tesseract \--list-langs
```
You should see **tha** in support languages. Run a test file by copy from local to container.
```
docker cp thatest.jpg t4cmp:/home/thatest.jpg
```
Now you can run tesseract (recommend setting for tha, psm=1 and oem=1) and copy from container back to local
```
docker exec -it t4cmp tesseract thatest.jpg thatest -l tha --psm 1 --oem 1 pdf hocr
docker cp t4cmp:/home/thatest.pdf ./thatest.pdf
```
### Output option
#### pdf with text only
```
docker exec -it t4cmp tesseract thatest.jpg thatest -l tha --psm 1 --oem 1 -c textonly_pdf=1 pdf hocr
```
#### tsv
```
docker exec -it t4cmp tesseract thatest.jpg thatest -l tha --psm 1 --oem 1 txt pdf hocr tsv
```
#### Create character box file use in hocr
This can be explored to be use to modify characters and bounding box for hocr. More [here](https://github.com/tesseract-ocr/tesseract/wiki/Training-Tesseract-%E2%80%93-Make-Box-Files).
```
docker exec -it t4cmp tesseract thatest.jpg thatest -l tha --psm 1 --oem 1 makebox txt hocr
```
#### To remove extra spaces and preserve only interword-spaces, as suggested in [#1009](https://github.com/tesseract-ocr/tesseract/issues/1009) and [#991](https://github.com/tesseract-ocr/tesseract/issues/991).
```
docker exec -it t4cmp tesseract thatest.jpg thatest -l tha --psm 1 --oem 1 -c preserve_interword_spaces=1 txt
```
#### List of parameters
```
docker exec -it t4cmp tesseract --print-parameters
```
#### Restart docker container
```
docker start t4cmp
```
#### Run Tesseract on images in a folder
```
for i in *.tif ; do tesseract $i outtext;  done;
```
### Scantailor
To install scantailor on Mac follow instruction [here](https://github.com/scantailor/scantailor/issues/273#issuecomment-357964331)

# Training with OCROpus
By using Conda, first create environment and activate it
```
>source activate ocropus_env
```
To list available environments:
```
>conda info -envs
```
To exit from environment:
```
>source deactivate
```

To create the TRUTH data, use command **ocropus-gtedit**
```
ocropus-gtedit html book/0001/??????.bin.png -o temp-correction.html
```
This created html file with each line the editable text box to manually type in.
```
>ocropus-gtedit extract temp-correction-gt.html
```
This extracted the html input data in the textbox in seperated file.

training with existing model **--load**
```
>ocropus-rtrain --load en-default-pyrnn.gz -o \[model-name] book/0001/*.bin.png
```
The training can not be done with existing model at the moment, as far as I know, the error of 
```
>ocropus-rtrain -o \[model-name] book/0001/*.bin.png
```
Run the prediction of test set
```
ocropus-rpred -m [model-name] 'book/0001/??????.bin.png'
```
### Resources
1. [work flow explained](https://graal.hypotheses.org/786)


