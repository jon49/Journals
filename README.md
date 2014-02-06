# Journals of My Ancestors

### Markdown

Using [`Pandoc`](http://johnmacfarlane.net/pandoc/) style markdown with underscores referring to underlines.

The command line used to create the markdown files.

##### To PDF:

~~~
perl -pe 's/_(.+?)_/\\underline{$1}/g' source.md | pandoc -o output.pdf
~~~

##### To ePub:

~~~
perl -pe 's/_(.+?)_/\<u\>$1\<\/u\>/g' source.md | pandoc -S --epub-metadata=../metadata.xml --epub-stylesheet=../Styles/template.css --epub-cover-image=../Images/coverImage.jpg -o ../output.epub
~~~

##### To HTML

~~~
cat file1 file2 | perl -pe 's/_(.+?)_/\<u\>$1\<\/u\>/g' | pandoc -s -S --toc -c ../../Styles/template.css -o OurLetters.html
~~~
### JPG Manipulation

Use `ImageMagick`

~~~
sudo apt-get install imagemagick
~~~

##### To PDF:

~~~
convert *.jpg pictures.pdf
~~~

##### To Lower Quality PDF:

~~~
convert -quality 25 -resize 50% *.jpg -adjoin output.pdf
~~~

##### Combine PDFs

~~~
pdftk *.pdf cat output mergedfile.pdf
~~~

