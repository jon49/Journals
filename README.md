# Journals of My Ancestors

### Markdown

Using [`Pandoc`](http://johnmacfarlane.net/pandoc/) style markdown with underscores referring to underlines.

The command line used to create the markdown files.

~~~
perl -pe 's/_(.+?)_/\\underline{$1}/g' source.md | pandoc -o output.pdf 
~~~
