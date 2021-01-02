Removing text from a PDF (like an auto-generated date header from Chrome's "save to PDF" feature if you forgot to turn off the headers)
turned out to be more involved than I expected. 
What I did is probably not the right way of doing things, but it worked. Some documentation.


The idea is:
- Read the PDF file using a text editor and figure out what fonts it uses for the offending text
- Write a script to get rid of anything using that font 
- Rebuild the pdf to repair the damage we just did

### Reading pdf files
Work on individual pages to avoid madness. PDFTK can split a pdf file into one pdf for each page with
```
pdftk input.pdf burst
```
Uncompressing the PDF makes it easier to read for a human (but it's still horrible)
```
pdftk input.pdf output output.pdf uncompress
```

### The PDF file format
Open an uncompressed PDF page in any text editor and get digging. It's somewhere between assembly and C.
Here are some things I could figure out thanks to the Adobe PDF Reference.
PDF files are not indented; I'm adding indentation to make it more obvious.

**Objects**  
This is an object/variable definition
```
98 0 obj 
  # content
endobj
```
This could be a page, a font, URL. 
Objects that reference other objects using the number, e.g. `98 0 R`.

For example, this object (`14 0 obj`) defines a font and it is used in the next object (`13 0 obj`)
```
14 0 obj 
  <<
    /FontName /TimesNewRoman-BoldItalic
    /StemV 144
    /FontFile3 16 0 R
    /Ascent 829
    /Flags 18
    /XHeight 352
    /Descent -240
    /ItalicAngle -14
    /FontBBox [-172 -240 1040 862]
    /Type /FontDescriptor
    /CapHeight 612
  >>
endobj
13 0 obj
  <<
    /LastChar 255
    /BaseFont /Times-BoldItalic
    /Subtype /Type1
    /FontDescriptor 14 0 R
    /Widths [500 500 500 500]
    /Encoding /WinAnsiEncoding
    /Type /Font
    /FirstChar 30
  >>
endobj 
```
The ``<< >>`` bit is a dictionary. A python `{"Entry": 2303213}` would be something like
```
  <<
    /Entry 2303213
  >>
```

**Streams**  
Objects that contain streams produce some kind of output. This is where to look to remove text. 
Streams contain binary data and are not human readable, but there are blocks separated by the tags `BT` and `ET`: `Begin Text` and `End Text`
```
98 0 obj 
  <<
    ...
  >>
  stream
    /SomeGibberish
    BT
      /T1 1 Tf
      0.1223 Tc 0.1478 Tw ( SERIE)Tj
      (Either some real text) Tj
      (or some horrible binary) Tj
    ET
  enstream
endobj
```
The first line after a `BT` specifies the font to use. It this example it is callind the T1 font.
If we remove the lines from `BT` to `ET`, the text is removed from the PDF. 
Without knowing what text it is (if it is some horrible binary), we can use the font to decide what to remove and what to keep.

**Font table**  
At the end of the PDF, there is a dictionary of fonts mapping this `/T1` to the definition of the font
```
/Font 
<<
  /T1_3 13 0 R
  /T1_2 16 0 R
>>
```
If you have issues finding this table, look for the first object (`1 0 obj`).
```
1 0 obj 
  <<
    /Kids [3 0 R]
    /Count 1
    /Type /Pages
  >>
endobj
```
Its kid contains a bunch of information on the PDF, including the font table:
```
3 0 obj 
  <<
    /CropBox [0 0 504 720]
    /Parent 1 0 R
    /Resources 
      <<
        /ExtGState 
          <<
          ...
          >>
        /ColorSpace 
          <<
          ...
          >>
        /Font 
          <<
            /T1_3 13 0 R
            /T1_2 16 0 R
          >>
        ...
      >>
    ...
  >>
endobj
```

### Filtering out bad fonts
Let's say the fonts we want to get rid off are called `/AN`, where `N` is any number. 
This python script does so by copying the file line by line, until it sees a `BT`. 
When it sees one, it does not copy the line but checks if the next line contains `/A0` or `/A1` or other numbers with a regex.
If there's no match and it's a line we want to keep, it copies the skipped `BT` and goes back to copying mode.
If it matches, it continues ignoring lines until it reaches an `ET`.
```
import re 

def b2s(line):
    return line.decode("windows-1252", errors="ignore")

def line_is_startblock(line):
    return "BT" in b2s(line)

def line_is_endblock(line):
    return "ET" in b2s(line)

def line_is_badfont(line):
    return re.search("\/A\d", b2s(line)) is not None

last_line_was_BT = False
ignoring_and_looking_for_ET = False
skipped_line = None

input_file = "uncompressed.pdf"
output_file = "filtered.pdf"

with open(input_file, "rb") as f_in:
    with open(output_file, "wb") as f_out:
        for cnt, line in enumerate(f_in):
            if ignoring_and_looking_for_ET:
                if line_is_endblock(line):
                    ignoring_and_looking_for_ET = False
            else:

                if last_line_was_BT:
                    if line_is_badfont(line):
                        ignoring_and_looking_for_ET = True
                    else:
                        f_out.write(skipped_line)
                        f_out.write(line)
                    last_line_was_BT = False
                elif line_is_startblock(line):
                    last_line_was_BT = True
                    skipped_line = line
                else:
                    f_out.write(line)
```

### Rebuilding the file 
This process is not "nice" to a pdf file. The internal data structures that keep track of how long the streams are get messed up. 
Chrome could still read it, but ghostscript can fix it. A `pdf2ps filtered.pdf filtered.ps` and `ps2pdf filtered.ps filtered.pdf` cleans it.
