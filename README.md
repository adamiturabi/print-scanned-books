# Setting up scanned PDFs for printing

## Introduction

Scanned text PDFs/JP2s are often skewed and uncropped. To set up for printing follow these instructions:

## Required software (Linux)

1. Install image codecs for ImageMagick:
   1. OpenJPEG: https://github.com/uclouvain/openjpeg/blob/master/INSTALL.md
   2. pkg-config: `sudo apt-get install pkg-config`
   3. Latest versions of TIFF and JPEG from http://www.imagemagick.org/download/delegates/
      1. tiff-4.0.8.tar.gz
      2. jpegsrc.v9b.tar.gz  
      For installing:
      ```
      ./configure
      make
      sudo make install
      ```
2. Install ImageMagick from source: https://imagemagick.org/script/install-source.php
   1. Verify that JP2 and TIFF are configured for ImageMagick:
      ```
      convert -list format | grep -i tiff
      convert -list format | grep -i jp2
      ```
3. Install scantailor: `sudo apt-get install scantailor`
4. Install PDF-Booklet: http://pdfbooklet.sourceforge.net/
5. Install img2pdf: `sudo apt-get install img2pdf`

## Procedure

1. If the original file is a PDF, first extract images. Use ImageMagick
2. If images are JPEG2000 then convert them to TIFF:
   1. Write shell script `convert_jp2_2_tiff`:
      ```
      #!/bin/sh
      dirname_val=`basename "$d"`
      echo $file
      cd "$dirname_val"
      mkdir skewed
      for filename in *.jp2; do
        base_filename=`basename "$filename" .jp2`
        /usr/local/bin/convert "$base_filename.jp2" TIFF64:"skewed/$base_filename.tiff"
      done
      ```
   2. `./convert_jp2_2_tiff dir_with_jp2s`
3. Scantailor:
   1. Figure out DPI of input using pixel resolution and/or PDF page size.
   2. Create project in scantailor. Add TIFF files in skewed/ and apply calculated DPI value.
   3. Check "RTL" for Arabic texts
   4. Apply "deskew" on "Auto" for all pages, and then run
   5. 1. Apply "Select content" on "Auto" for all pages, and then run.
      2. In rightmost pane change sorting order from "Natural order" to "Order by increasing width". Go to the lowest (widest) page and manually shrink width to correct textbox. Keep doing for for the widest pages until they normalize. See https://github.com/scantailor/scantailor/wiki/Page-Layout
      3. Now change sorting order to "Order by increasing height" and repeat.
   6. Set the correct margins and keep same page size for all pages. Verify that the page size for all pages is not much larger than the margins. If it is then repeat manual shrinking of content for offending pages.
   7. In the "Output" stage, set mode to "grayscale" and set white margins. Apply to all pages, and then run output.
4. Join scantailor output TIFFs to a PDF using ImageMagick or img2pdf. ImageMagick hangs if there are too many so do them in batches of 100 or 128.
   1. img2pdf:
      ```
      img2pdf scantailor_output_*.tif -o out.pdf
      ```
   2. ImageMagick:
      ```
      magick scantailor_output_*.tif output.pdf
      ```
   
5. Set up PDF for printing in PDF-Booklet



