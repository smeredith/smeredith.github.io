I used to use OneNote on Windows.
It worked pretty well--I could add content easily without much thought or work.
I stopped using Windows a couple years ago, and therefor stopped using OneNote on Windows.
I can still access my notebooks via the web interface, but I hate it.
It's buggy, does not have all the same features, and pages take forever to load.

As a replacement for OneNote, I use a set of markdown files, image files, and pdfs.
I keep these files in GitHub.
I browse and edit them either in VS Code locally or via the GitHub web interface.

This change has a lot of benefits for me and I am happy to be done with OneNote.
One thing I love is the transparency and portability of the file format.
OneNote stores things in a complex propriety format that makes migrating your files very difficult.

I am not trying to persuade you to switch from OneNote to plain files.
But if you do want to, these ideas might make the switch easier for you.
Or if you are trying to decide if you should or could, maybe this will help.

## Notebooks, Sections, and Pages

I think of a git repo as the equivalent of a OneNote notebook.
OneNote sections can be replicated with directories, nested arbitrarily deep.
A markdown file is the equivalent of a OneNote page.
I don't know what to say about OneNote sub-pages: they are an odd concept.
You can get more or less the same effect with an additional directory level.

OneNote forces this Notebook/Section organization.
If you don't want a hierarchical directory structure, you are free to organize your files however you like.
If you want to use tags instead, I recommend [front matter yaml](https://jekyllrb.com/docs/front-matter/).
It's a block of yml at the start of the file that looks like this:
```
---
tags:
 - mytag1
 - mytag2
---
```

I use a combination of directories and tags.

## Search

Searching for text in the markdown files is easy, fast, and works well.
Plus, I get to use regular expressions.
And I can search the entire repo whereas the OneNote web interfaces limits me to searching the current page or the current section.

However, OneNote will search for text within images on its pages using OCR.
This includes images that are printouts of PDF.
That's a win for OneNote, but this is not a very common scenario for me.
To search for a PDF in my repo that contains some given text, I need to use an external tool.
My file manager (Nemo) will do it fine.

## Text Formatting

I get the formatting I need from [markdown syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

If you want <mark>yellow highlighting</mark>, you need to surround the text with `<mark>` and `</mark>`.

## Tables

Tables are painful to work with in markdown.
There are some VS Code extensions that help, like "Markdown Table Prettifier."
That implies that when working with tables I will edit locally instead of using the GitHub web UI.

## Spell Checking

When you edit a file in the GitHub web interface, it does not spell check your text.
Even the browser's built-in spell checker does not work in that interface.
To get spell checking, I use VS Code locally with a spell check extension to edit the files.

## Print to OneNote

I just print to a PDF and place the PDF in the folder where it belongs.
To view it, select the file in the GitHub UI.

## Annotate a PDF

To annotate a PDF, I use a native app.
I like this more than using OneNote tools to draw on top of a PDF printout in a page or inserting various text boxes around the page.
To highlight text and insert text annotations, I use Okular.
If you like to annotate with a sytlus, Xournal++ might be a good choice.

## Images

To insert an image on a page, I upload the image to the same folder as the markdown file, and give it the same base filename as the markdown file, with a number at the end if there are multiple images.
So `filename.md` might embed `filename-1.jpg` and `filename-2.jpg`.
I experimented with putting all the images in a separate folder, but now I prefer them together with base file.

The markdown syntax for inserting an image is:
```
![](filename.jpg)
```
However, if you need to change the size of the image, you need to use HTML syntax:
```
<img src="filename.jpg" width="500" />
```

## Links

You can use standard markdown links to link between pages in your repo, even referencing headers within a page if you like.
You can also link to files like images and PDFs.
And you can add external links as well.

## Drawing

For creating diagrams and drawings, I use [excalidraw.com](https://excalidraw.com/).
This gives you an infinate canvas and the ability to use a stylus if you like.

When saving the file I created, I use "Export Image".
I select "svg" as the image format, check "Embed scene" option, and download the file and add it to my repo.
The GitHub UI displays the image when I select the file.

To edit the image, I upload the .svg file to excalidraw.com again.
Since I embedded the scene, the diagram is loaded back into the editor and I can carry on editing it.
Export it again and add it back to the repo.

## Version Control

OneNote has a built-in page history, but it no where near as robust as git for version control.
For example, moving a page in OneNote to a different section destroys the page history.

It's automatic in OneNote, but you must think about it a little more when using git.
For example, if you edit files both locally and using the GitHub UI, you might need to do a little more work to sync.

## Mobile App

I use the GitHub app for iOS.
You need an Internet connection for this.
I can search, browse, and edit notes.
This has satisfied my mobile needs.

You could also just use the GitHub web UI in the browser.

## Hand-written notes

### Xournal++

I don't make handwritten notes using a stylus.
If you do, a good choice for them might be the native app Xournal++.
You can export your written notes as PDF and they will fit right in with the rest of your files.
Xournal++ can open the PDF again if you want to add more notes to the file later.

### rnote

## Dictation

## EncroS Sections

## Other VS Code Markdown Extensions

Have a look at "Markdown All in One," "Markdown Paste," "Markdown Table Prettifier," and "Foam."

## Further Reading

[](https://youtu.be/Ly10LxvzNGg?si=_zE0xco9_Xb_91SI)
