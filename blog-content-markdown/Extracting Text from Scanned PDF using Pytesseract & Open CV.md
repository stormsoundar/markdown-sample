The process of extracting information from a digital copy of invoice can be a tricky task. There are various tools that are available in the market that can be used to perform this task. However there are many factors due to which most of the people want to solve this problem using Open Source Libraries.

I came across a similar set of problem a few days back and wanted to share with you all the approach through which I solved this problem. The libraries that I used for developing this solution were **pdf2image** (for converting PDF to images), **OpenCV** (for Image pre-processing) and finally **PyTesseract** for OCR along with **Python**.

## Converting PDF to Image

pdf2image is a python library which converts PDF to a sequence of PIL Image objects using pdftoppm library. The following command can be used for installing the pdf2image library using pip installation method.

> _pip install pdf2image_

Note: pdf2image uses **Poppler** which is a PDF rendering library based on the xpdf-3.0 code base and will not work without it. Please refer to the below resources for downloading and installation instructions for Poppler.

_[https://anaconda.org/conda-forge/poppler](https://anaconda.org/conda-forge/poppler)_

_[https://stackoverflow.com/questions/18381713/how-to-install-poppler-on-windows](https://stackoverflow.com/questions/18381713/how-to-install-poppler-on-windows_

After installation, any pdf can be converted to images using the below code.

```python
from pdf2image import convert_from_path

pdfs = r"provide path to pdf file"
pages = convert_from_path(pdfs, 350)

i = 1
for page in pages:
    image_name = "Page_" + str(i) + ".jpg"
    page.save(image_name, "JPEG")
    i = i+1
```

After converting the PDF to images, the next step is to highlight the regions of the images from which we have to extract the information.

> Note: Before marking regions make sure that you have preprocessed the image for improving its quality (DPI ≥ 300, Skewness, Sharpness and Brightness should be adjusted, Thresholding etc.)

## Marking Regions of Image for Information Extraction

Here in this step we will mark the regions of the image from where we have to extract the data. After marking those regions with the rectangle, we will crop those regions one by one from the original image before feeding it to the OCR engine.

> _Most of us would think to this point — why should we mark the regions in an image before doing OCR and not doing it directly?_

> _The simple answer to this question is that **YOU CAN**_

> _The only catch to this question is sometimes there are hidden line breaks/ page breaks that are embedded in the document and if this document is passed directly into the OCR engine, the continuity of data breaks automatically (as line breaks are recognized by OCR)._

Through this approach, we can get maximum correct results for any given document. In our case we will be trying to extract information from an invoice using the exact same approach.

The below code can be used for marking the regions of interest in the image and getting their respective co-ordinates.

```python
# use this command to install open cv2
# pip install opencv-python

# use this command to install PIL
# pip install Pillow

import cv2
from PIL import Image

def mark_region(imagE_path):

    im = cv2.imread(image_path)

    gray = cv2.cvtColor(im, cv2.COLOR_BGR2GRAY)
    blur = cv2.GaussianBlur(gray, (9,9), 0)
    thresh = cv2.adaptiveThreshold(blur,255,cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY_INV,11,30)

    # Dilate to combine adjacent text contours
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (9,9))
    dilate = cv2.dilate(thresh, kernel, iterations=4)

    # Find contours, highlight text areas, and extract ROIs
    cnts = cv2.findContours(dilate, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    cnts = cnts[0] if len(cnts) == 2 else cnts[1]

    line_items_coordinates = []
    for c in cnts:
        area = cv2.contourArea(c)
        x,y,w,h = cv2.boundingRect(c)

        if y >= 600 and x <= 1000:
            if area > 10000:
                image = cv2.rectangle(im, (x,y), (2200, y+h), color=(255,0,255), thickness=3)
                line_items_coordinates.append([(x,y), (2200, y+h)])

        if y >= 2400 and x<= 2000:
            image = cv2.rectangle(im, (x,y), (2200, y+h), color=(255,0,255), thickness=3)
            line_items_coordinates.append([(x,y), (2200, y+h)])


    return image, line_items_coordinates
```

![](https://miro.medium.com/max/700/1*eTJ5_mezfc-K3WKyxpls8Q.jpeg)

![](https://miro.medium.com/max/700/1*YiYMZB6XMV3s8R0tNrqI1A.jpeg)

## Applying OCR to the Image

Once we have marked the regions of interest (along with the respective coordinates) we can simply crop the original image for the particular region and pass it through pytesseract to get the results.

For those who are new to Python and OCR, pytesseract can be an overwhelming word. According to its official website -

> _Python-tesseract is a wrapper for [Google’s Tesseract-OCR Engine](https://github.com/tesseract-ocr/tesseract). It is also useful as a stand-alone invocation script to tesseract, as it can read all image types supported by the Pillow and Leptonica imaging libraries, including jpeg, png, gif, bmp, tiff, and others. Additionally, if used as a script, Python-tesseract will print the recognized text instead of writing it to a file._

Also, if you want to play around with the configuration parameters of pytesseract, I would recommend to go through the below links first.

_[https://pypi.org/project/pytesseract/](https://pypi.org/project/pytesseract/)_

_[https://stackoverflow.com/questions/44619077/pytesseract-ocr-multiple-config-options](https://stackoverflow.com/questions/44619077/pytesseract-ocr-multiple-config-options)_

The following code can be used to perform this task.

```python
import pytesseract
pytesseract.pytesseract.tesseract_cmd = r'C:\Users\Akash.Chauhan1\AppData\Local\Tesseract-OCR\tesseract.exe'

# load the original image
image = cv2.imread('Original_Image.jpg')

# get co-ordinates to crop the image
c = line_items_coordinates[1]

# cropping image img = image[y0:y1, x0:x1]
img = image[c[0][1]:c[1][1], c[0][0]:c[1][0]]

plt.figure(figsize=(10,10))
plt.imshow(img)

# convert the image to black and white for better OCR
ret,thresh1 = cv2.threshold(img,120,255,cv2.THRESH_BINARY)

# pytesseract image to string to get results
text = str(pytesseract.image_to_string(thresh1, config='--psm 6'))
print(text)
```

![](https://miro.medium.com/max/700/1*Omos0aoeTCSk2DXdl97KLQ.jpeg)

Output from OCR:

```
Payment:

Mr. John Doe

Green Street 15, Office 4
1234 Vermut

New Caledonia
```

![](https://miro.medium.com/max/700/1*L_kWBVRIPoGTAVooqOFZhw.jpeg)

Output from OCR

```
COMPLETE OVERHAUL 1 5500.00 5500.00 220
REFRESHING COMPLETE CASE 1 380.00 380.00 220
AND RHODIUM BATH
```

As you can see, the accuracy of our output is 100%.

So this was all about how you can develop a solution for extracting data from a complex document such as invoices.

There are many applications to what OCR can do in term of document intelligence. Using pytesseract, one can extract almost all the data irrespective of the format of the documents (whether its a scanned document or a pdf or a simple jpeg image).

Also, since its open source, the overall solution would be flexible as well as not that expensive.
