---
title: "Convert Image To Text"
last_modified_at: 2024-10-24T16:20:02-05:00
categories:
  - Blog
  - Productivity
tags:
  - Python
  - Converter
  - AI
classes: wide  
---

# How To Convert Image To Text On Your Local Machine?

In the last weeks I found myself converting images to text very often. For that task I did mainly use AI tools like [ChatGPT](https://chatgpt.com) or [Claude](https://claude.ai). As you may know using these tools is with costs. They come with a free plan, which is limited to several requests per day. 

That's way I did want to have a tool available on my machine, which can carry out that task. It should be able to process the image copied to the clipboard, which would make the usage of the tool very handy.

The good news is that there are free Python libraries available, allowing to implement this tool very easily.

## Requirements
You need to install the required applications and libraries first.

1. [Python 3](https://www.python.org) 
2. Pip 3 - Python's standard package manager: usually installed with Python 3
3. [Tesseract OCR - optical character recognition/reader](https://github.com/tesseract-ocr/tesseract)
4. [Python wrapper for Tesseract OCR](https://pypi.org/project/pytesseract)
5. [Pillow - Python Imaging Library](https://pillow.readthedocs.io/en/latest/installation/basic-installation.html)
6. macOS only:
  - [PyObjC - Bridge Between Python and Objective-C] (https://pyobjc.readthedocs.io/en/latest/install.html)
  - [Wrappers for the Quartz frameworks on macOS - Allows accessing the clipboard] (https://pypi.org/project/pyobjc-framework-Quartz)

## My Setup On macOS
As I'm on macOS, the installation process looks like:

```bash
$ brew install python@3.13
$ brew install tesseract
$ pip3 install pytesseract
$ pip3 install pillow 
$ pip3 pyobjc-framework-Quartz
$ pip3 install pyobjc
```
## Create The Script

Create a file (e.g. `img2txt.py`) with the following content:

```python
from PIL import Image
from io import BytesIO
import Quartz
import LaunchServices
import pytesseract

def get_image_from_clipboard():
    # Get contents from clipboard
    pasteboard = Quartz.NSPasteboard.generalPasteboard()

    # Define the type of data (image) we're interested in 
    classes = [Quartz.NSImage] 
    options = {}
    images = pasteboard.readObjectsForClasses_options_(classes, options)

    if images and len(images) > 0:
        # Image found: extract TIFF image data from the clipboard
        image_data = images[0].TIFFRepresentation()
        if image_data:
            # Convert the TIFF data into an image using Pillow
            img = Image.open(BytesIO(image_data))
            return img
    return None

#####################
## Execute the function defined above to read an image from clipboard
img = get_image_from_clipboard()
if img:
    # Convert to text
    text = pytesseract.image_to_string(img)
    print(text)
else:
    print("No image found on the clipboard")
```

How it works:
 - Uses the `Quartz` framework to access the macOS clipboard
 - Checks if there's an image in the clipboard by inspecting the types of data stored there
 - If an image is found, it converts it into a Pillow image object that you can manipulate (e.g., save, display, etc.)
 - Uses `tesseract` to convert the image to text
 - Prints the text

# Run The Script

Copy an image of text into the clipboard.  
For example, on macOS you can take a screenshot and put it into the clipboar via key keyboard shortcut &#8963;&#8679;&#8984;4 
Then go to your terminal and execute the script above:
```bash
$ python3 img2txt.py
```
Verify that the content of the image is printed.

