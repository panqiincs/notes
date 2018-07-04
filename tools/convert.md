## ImageMagick Tips

Split a pdf file to seperated pages:

```bash
convert -density 300 file.pdf page_%04d.jpg
```

Image Binarization:

```bash
convert page_0004.jpg -threshold 55% result.png
```

Merge ordered images to a pdf file:

```bash
ls *.png | sort -n | tr '\n' ' ' | sed 's/$/\ TCP-IP_Illustrated.pdf/' | xargs convert
```

