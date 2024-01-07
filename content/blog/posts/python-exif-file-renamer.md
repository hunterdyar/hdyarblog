---
title: Batch Rename Images with their EXIF Data
date: 2024-01-07
---

While I work on an educational photography tool, I encountered a specific problem where the solution is worth sharing.

My problem was loading images by their [exposure setting metadata](https://en.wikipedia.org/wiki/Exif). Instead of creating a database of images and settings, it seems much easier to just rename the files with some encoding scheme.

So that's what I did. [This python script](https://github.com/hunterdyar/image-exif-batch-renamer/blob/main/main.py) will take a folder and a 'pattern', and rename all image files in that folder with the pattern.

Using it, I was able to turn my folder of images from my camera into files usable for my learning tools.

## How the Python Script Works
Here are the core problem-solving parts of the script.

- [argparse](https://docs.python.org/3/library/argparse.html) is the [not so] secret sauce for dealing with command line arguments.
- [OS.listdir](https://docs.python.org/3/library/os.html#os.listdir) allows me to iterate through all of the files in a folder.
- [OS.path.splitext](splitext) is the preffered way to split out the extension from a filename.
- The Python [EXIF](https://exif.readthedocs.io/en/latest/usage.html) package pulls the exposure data from the image metadata, and implicitly works to filter out non-image files.
- The [Fraction](https://docs.python.org/3/library/fractions.html) library helps me deal with shutter speeds; ie: grab the denominator from fractions.
- And, of course, [OS.rename](https://docs.python.org/3/library/os.html?highlight=os%20rename#os.rename) to actually rename a file.


The script & documentation are available here: [hunterdyar/image-exif-batch-renamer](https://github.com/hunterdyar/image-exif-batch-renamer)
