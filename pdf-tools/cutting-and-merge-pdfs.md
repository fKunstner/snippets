# Cutting PDFs while keeping links working 

Requires `pdftk` ([Win](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/), [Linux](http://manpages.ubuntu.com/manpages/bionic/man1/pdftk.1.html), [GUI](http://pdfchain.sourceforge.net/)).

```
pdftk full-document.pdf cat 12-15 output only_pages_12_to_15.pdf
```

# Merging

```
pdftk file1.pdf file2.pdf file3.pdf cat output concatenated-1-2-3.pdf
```
