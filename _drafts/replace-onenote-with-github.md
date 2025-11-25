# Replacing OneNote with Markdown and PDFs

I used to use OneNote on Windows.
It worked pretty well--I could add content easily without much thought or work.
I stopped using Windows a couple years ago, and therefor stopped using OneNote on Windows.
I can still access my notebooks via the web interface, but I hate it.
It's buggy, it's missing features, and pages often take forever to load.

As a replacement for OneNote, I use a set of markdown files, image files, and pdfs.
I keep these files in GitHub, in a private repo.
I browse and edit them either in VS Code locally, via the GitHub web interface, or using the GitHub mobile app on my iPhone.

This setup has a lot of benefits for me and I am happy to be done with OneNote.
One thing I love is the transparency and portability of the file format.
OneNote stores things in a complex propriety format that makes migrating your files very difficult.

I used Obsidian for a while in the past, and for some reason I prefer VS Code and GitHub.
If I want, I can use any of my note repos as an Obsidian vault, and it will just work.

I am not trying to persuade you to switch from OneNote to plain text and PDF files.
But if you do want to, these ideas might make the switch easier for you.
Or if you are trying to decide if you should or could, maybe this will help.

This document assumes you have some technical proficiency with VS Code and git.

## Notebooks, Sections, and Pages

I think of a single git repo as the equivalent of a OneNote notebook.
OneNote sections can be replicated with directories, nested arbitrarily deep.
A markdown file is the equivalent of a OneNote page.
I don't know what to say about OneNote sub-pages: they are an odd concept.
You can get more or less the same effect with an additional directory level.

If you don't want a hierarchical directory structure, you are free to organize your files however you like.
If you want to use tags instead, I recommend [front matter yaml](https://jekyllrb.com/docs/front-matter/) in your markdown files.
It's a block of yml at the start of the file that looks like this:
```
---
tags:
 - mytag1
 - mytag2
---
```
In addition to tags, you can put other metadata in this section if it helps you organize things.
As an example, this is how [GitHub Docs uses frontmatter](https://docs.github.com/en/contributing/writing-for-github-docs/using-yaml-frontmatter).

The primary goal of using the yml format is to provide a standard and consistant way to apply metadata to all my notes.
Nothing I currently use relies on them, but other apps, like Obsidian, use this format as well.

I use a combination of directories and metadata to organize my notes.
In addition to `tags:`, I use `source:` to record a URL if I am taking notes on a web page.
I use a few other random tags.

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

If you want <mark>yellow highlighting</mark> in VS Code and GitHub, you need to surround the text with `<mark>` and `</mark>`, which are not part of markdown.

## Tables

Tables are painful to work with in markdown.
There are some VS Code extensions that help, like "Markdown Table Prettifier."

Another option is to use a tool that is already excelent at working with tables.
Google Sheets, Excel, or LibreOffice Calc can all take a table (copied from the rendered view, not the raw markdown table) from the clipboard and turn it into a spreadsheet table.
After working with it in the spreadsheet app, you need it as a markdown table again to paste it back into your document.
[Tableconvert](https://tableconvert.com/excel-to-markdown) can to it: copy from the shreadsheet and paste into tableconvert, then copy the markdown and paste it into your file.
It's a couple of steps, but to me that's easier than trying to work with a markdown table in plain text.

## Spell Checking

When you edit a file in the GitHub web interface, it does not spell check your text.
Even the browser's built-in spell checker does not work in that interface.
To get spell checking, I use VS Code locally with a spell check extension to edit the files.

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

## Whiteboard / Drawing / Diagrams

For creating diagrams and drawings, I use the Excalidraw extension for VS Code by pomdtr.
This gives you an infinate canvas and the ability to use a stylus if you like.

To create a drawing, create a new file with a name ending in ".excalidraw.svg".
The drawing commands, which Excalidraw calls "the scene", are stored in the svg file itself.
This means VS Code and GitHub see the file as an image and render it as such, and Excalidraw uses the scene to let you continue to edit your drawing the next time you open it.

You can also edit these file in a browser with [excalidraw.com](https://excalidraw.com/).

## Version Control

OneNote has a built-in page history, but it is no where near as robust as git for version control.
For example, moving a page in OneNote to a different section destroys the page history.

It's automatic in OneNote, but you must think about it a little more when using git.
For example, if you edit files both locally and using the GitHub UI, you might need to do a little more work to sync.

## Mobile App

I use the GitHub app for iOS.
You need an Internet connection for this.
I can search, browse, and edit notes.
This has satisfied my mobile needs, which is usually to look something up or jot something down.

You could also just use the GitHub web UI in the phone's browser.

## Print to OneNote

I just print to a PDF and place the PDF in the folder where it belongs.
To view it in the GitHub web UI, select the file.
To view it in VS Code, install the vscode-pdf extension from tomoki1207 and from then on you can preview the file by clicking on it in.

## Annotate a PDF

To annotate a PDF, I use a native app to highlight text and insert text annotations.
I like this more than using OneNote tools to draw on top of a PDF printout in a page or inserting various text boxes around the page.
Okular is one app that works.
[BentoPDF](https://bentopdf.com/) is pretty good for text and stylus input, and it also allows you to insert images.
If you like to annotate with a sytlus, Google Chrome might be another good choice.

If you need more space on a PDF to take notes, you can increase the size of its margins using [the I2PDF tool found here](https://www.i2pdf.com/add-margin-to-pdf).
You can add up to 3 inches per side.
I wouldn't use this tool for any private files.

Or you can use `pdfcrop`, a command line tool for Linux.
This command adds a 2" right margin:
```
pdfcrop --margins '0 0 144 10' inpub  t.pdf output.pdf # left bottom right top in 72nds of an inch
```
I installed it with `sudo apt install texlive-extra-utils`.
The package is huge.

If I need even more space in your PDF for notes, I can use [BentoPDF](https://bentopdf.com/) to add blank pages anywhere in a PDF.

## Hand-written notes

I don't make handwritten notes using a stylus.
If you do, a good choice for that might be the native app Xournal++.
You can export your written notes as PDF so they will be rendered.
Xournal++ can open the PDF again if you want to add more notes to the file later, but you can't go back and edit your old ones.

## Dictation

I don't need to do this often, but if I want to dictate my notes, I would use the GitHub app on one of my iOS devices, open the file for edit, and use the iOS native speech-to-text feature to get this done.

## Encrypted Sections

OneNote allows you to encrypt an entire section.
As a replacement, I recommend you keep notes and documents that really need to be encrypted in a password manager.

Alternatively, you can encrypt your private PDFs with a password.

## VS Code Markdown Extensions

- For markdown, I use "Markdown All in One," "Markdown Paste," "Markdown Table Prettifier," and "Foam."
- For spellcheck, I use "Code Spell Checker" by Street Side Software.
- To view PDFs inside VS Code, I use vscode-pdf extension from tomoki1207.
- For creating diagrams and drawings, I use the Excalidraw extension for VS Code by pomdtr.

## Further Reading

- [Best ONENOTE ALTERNATIVES for Linux, Windows and MacOS](https://youtu.be/Ly10LxvzNGg?si=_zE0xco9_Xb_91SI)
