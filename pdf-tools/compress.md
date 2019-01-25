# Compressing PDFs while keeping links working

From [askubuntu.com/questions/113544/how-can-i-reduce-the-file-size-of-a-scanned-pdf-file](https://askubuntu.com/questions/113544/how-can-i-reduce-the-file-size-of-a-scanned-pdf-file).

Requires [ghostscript](https://www.ghostscript.com/)

```
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile=output.pdf input.pdf
```

Other options for `-dPDFSETTINGS`:
```
-dPDFSETTINGS=/screen   Small (72 dpi)
-dPDFSETTINGS=/ebook    Medium (150 dpi)
-dPDFSETTINGS=/printer  Large (300 dpi)
```