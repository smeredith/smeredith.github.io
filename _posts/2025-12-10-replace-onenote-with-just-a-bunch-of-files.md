---
layout: post
title: Replacing OneNote with Just a Bunch of Files
---

I used to use OneNote on Windows.
It worked nicely--I could add content easily without much thought or work.
I switched to Linux a couple of years ago and therefore stopped using OneNote on Windows.
I can still access my notebooks via the web interface, but I hate it.
The web version is buggy, missing features that the desktop version has, and slow.

As a replacement for OneNote, I use a set of Markdown files, image files, and PDFs.
I keep these files in GitHub, in private repositories.
I browse and edit them either in VS Code locally, via the GitHub web interface, or using the GitHub mobile app on my iPhone.

OneNote stores things in a complex proprietary format that makes migrating your files very difficult.
Having Just a Bunch of Files gives you ultimate flexibility.
I limit the file formats I use to those that the GitHub web interface can render.

I used Obsidian for a while and I prefer VS Code because it is such a good text editor.
If I wanted to, I could use any of my note repositories as an Obsidian vault, and it would just work.

I am not trying to persuade you to switch from OneNote to Just a Bunch of Files.
But if you do want to, these ideas might make the switch easier for you.
Or if you are trying to decide if you should or could, maybe this will help.

This document assumes you have some technical proficiency with VS Code and Git or could learn them.

## TL;DR

- Replace a OneNote notebook with a set of standard files.
- Organize them in a directory structure.
- Keep them under version control using Git.
- View and edit them with whatever programs work best for the file at hand.

The rest of this document describes how you might mimic some of the features of OneNote in other ways.

## Notebooks, Sections, and Pages

I think of a single Git repository as the equivalent of a OneNote notebook.
OneNote sections can be replicated with directories, nested arbitrarily deep.
Markdown files and PDFs are the equivalent of OneNote pages.

I use Markdown for text notes.
Sometimes those notes include images.
PDFs covers pretty much everything else.

## Tags

If you want to use tags, I recommend [front matter YAML](https://jekyllrb.com/docs/front-matter/) and inline tags like `#mytag1` in your Markdown files.

Front matter is a block of YAML at the start of the file that looks like this:
```
---
tags:
 - mytag1
 - mytag2
---
```

The Foam VS Code extension has a tags explorer feature that recognizes both front matter tags and inline tags.
It presents them as a tree as if the tags were directories and lets you quickly jump to any tags it finds.

## Search

Searching for text in the Markdown files is easy and fast.
Plus, I get to use regular expressions.
And I can search the entire repository whereas the OneNote web interface limits me to searching the current page or the current section.

However, OneNote will search for text within images on its pages using OCR.
This includes images that are printouts of PDFs.
That is a win for OneNote, but it is not a very common scenario for me.
To search for a PDF in my repository that contains some given text, I need to use an external tool.
My file manager (Nemo) does it fine.
Or I can use `grep`.

## Text Formatting

I get the formatting I need from [Markdown syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

If you want <mark>yellow highlighting</mark> in VS Code and GitHub, you need to surround the text with `<mark>` and `</mark>`, which are not part of Markdown.

## Tables

Tables are painful to work with in Markdown.
There are some VS Code extensions that help, like "Markdown Table Prettifier."

Another option is to use a tool that is already excellent at working with tables.
Google Sheets, Excel, or LibreOffice Calc can all take a table (copied from the rendered view of a Markdown file, not the raw Markdown table) from the clipboard and turn it into a spreadsheet table.
After working with it in the spreadsheet app, you need it as a Markdown table again to paste it back into your document.
[TableConvert](https://tableconvert.com/excel-to-markdown) can do this: copy the table from the spreadsheet and paste it into TableConvert, then copy the Markdown output and paste it into your file.
It is a couple of steps, but to me that is easier than trying to work with a Markdown table in plain text.

## Spell Checking

When you edit a file in the GitHub web interface, it does not spell-check your text.
Even the browser’s built-in spell checker does not work in that interface.
To get spell checking, I use VS Code locally with a spell check extension to edit the files.

## Images

To insert an image on a page, I save the image to the same folder as the Markdown file, and give it the same base filename as the Markdown file, with a number at the end if there are multiple images.
So `filename.md` might embed `filename-1.jpg` and `filename-2.jpg`.
I experimented with putting all images in a separate folder, but I prefer keeping them together with the base file.

The Markdown syntax for inserting an image is:
```
![](filename.jpg)
```
However, if you want to change the size of the image, you need to use HTML syntax instead:
```
<img src="filename.jpg" width="500" />
```

## Links

You can use standard Markdown links to link between pages in your repository, even referencing headers within a page if you like.
You can also link to files like images and PDFs.
And you can add external links to the Internet as well.

## Whiteboard / Drawing / Diagrams

For creating diagrams and drawings, I use the Excalidraw extension for VS Code by pomdtr.
This gives you an infinite canvas and the ability to use a stylus if you like.

To create a drawing, create a new file with a name ending in ".excalidraw.svg".
The data the app needs, which Excalidraw calls "the scene", is stored in the SVG file itself.
This means VS Code and GitHub see the file as an SVG image and render it as an image, and Excalidraw uses the scene to let you continue to edit your drawing the next time you open it.

You can also edit these files in a browser with [excalidraw.com](https://excalidraw.com/).

## Version Control

OneNote has built-in page history, but you do not get to control it, and it is nowhere near as robust as Git for version control.
For example, moving a page in OneNote to a different section destroys the page history.

It's automatic in OneNote, but you must think about it a little more when using Git.
For example, if you edit files both locally and using the GitHub UI, you may need to resolve a conflict at some point.

## Mobile App

I use the GitHub app for iOS.
An Internet connection is required for this.
I can search, browse, and edit notes.
This has satisfied my mobile needs, which is usually to look something up or jot something down.

You could also just use the GitHub web UI in your phone’s browser.

## Print to OneNote

I just print to a PDF and place the PDF in the folder where it belongs.
To view a PDF in the GitHub web UI, select the file.
To view it in VS Code, first install the `vscode-pdf` extension by tomoki1207.
After that, you can preview the file by clicking on it.

## Screen Clipper

There are a few options if you want to capture the contents of a web page into your notes.

The easiest option is to print to a PDF directly from your browser.
This is not ideal because you end up with a paginated PDF.

If you want a single long-page PDF, you can go to [Sejda](https://www.sejda.com/html-to-pdf) and paste the URL there.
You have the option of choosing between the normal formatting of the page or the version formatted for printing, which is better when it doesn't include ads.

If you want one long PNG of the page, you can save that directly from Chrome following the instructions in [this article.](https://www.lifewire.com/take-screenshot-using-dev-tools-on-google-chrome-5097913)

If you have an iPhone handy, you can take a screenshot while viewing the page, then select "Full Page" at the top.
After that, you can save the page as a PDF or image.
I don't see a way to do the same thing on an iPad.

If you want to save the contents of the page as Markdown, you can use the Reader View, MarkDownload or MarkSnip Chrome extensions.
I like Reader View.
To save the contents as Markdown, you hold shift while pressing the save button.
To capture only part of a page, you can select it before opening the extension.

## Annotate a PDF

To annotate a PDF, I use a native app to highlight text and insert text annotations.
I like this more than using OneNote tools to draw on top of a PDF printout in a page or inserting various text boxes around the page.
Okular is one app that works.
Xournal++ works as well.

> Note: 
> Xournal++ does not store your notes as PDF annotations the way a PDF annotation tool would.
> When you export as PDF, your notes are baked into the PDF.
> See [Hand-written Notes](#hand-written-notes) below for more about Xournal++.

If you need more space on a PDF to take notes, you can increase the size of its margins using [the I2PDF tool](https://www.i2pdf.com/add-margin-to-pdf).
You can add up to 3 inches per side.
The site is full of ads, and I would not use this tool for anything else.
I also would not trust it with private files.

If I need even more space in a PDF for notes, I use [BentoPDF](https://bentopdf.com/) to add blank pages anywhere in a PDF and then write on that.
This is a great tool for doing various kinds of PDF manipulation, like merging and splitting PDFs and extracting pages from them.

## Hand-written Notes

I don't make handwritten notes using a stylus.
If you do, VS Code with the Excalidraw extension is a good choice.
I'd start with a 70% zoom for writing.

You could also use the native app Xournal++.
You can use it to annotate a PDF or just take notes from scratch.
Stylus support is very good in this app and you get pressure sensitivity.
You can export your written notes as PDF so they will be rendered when you browse them.
Xournal++ can open the PDF again if you want to add more notes to the file later, but you can't go back and edit your old notes unless you keep the .xopp file around too.

If your primary use of OneNote was handwritten notes, Xournal++ could be a complete replacement.
But you would still have to organize your notes as files instead of Notebooks, Sections, and Pages.

As a reminder, I stick with common file types (PDF, plain text, and images) so I can read my files without special software.
In support of that goal, one could export to PDF at the end of a Xournal++ editing session.
That would leave a standard PDF version of the Xournal++ file.

To take that one step further, one could use `pdfattach` (from Poppler) to attach the .xopp file and the PDF being annotated (if there is one) to the exported PDF, then delete the stand-alone .xopp file and PDF file being annotated (if there is one).
That way, when browsing the files all you see is the final exported PDF with all your hand-written notes.
If you want to make edits, you could use `pdfdetach` to get the attached .xopp file and PDF being annotated (if there is one) back from the exported PDF and open it in Xournal++ again.
Repeat the attachment process once you are done editing.

## Dictation

I don't need to do this often, but if I want to dictate my notes, I would use the GitHub app on my phone, open the file for edit, and use the [iOS native dictation feature](https://support.apple.com/guide/iphone/dictate-text-iph2c0651d2/ios) to get this done.

## Encrypted Sections

OneNote allows you to encrypt an entire section.

To replicate this, you can encrypt your private PDFs with a password.
For example, the following encrypts input.pdf and creates encrypted.pdf. 
```
qpdf --encrypt USER-PASSWORD OWNER-PASSWORD 256 -- input.pdf encrypted.pdf
```
Now encrypted.pdf requires a password to open.

To remove the encryption:
```
qpdf --password=OWNER-PASSWORD --decrypt encrypted.pdf unencrypted.pdf
```

For background, see [qpdf documentation](https://qpdf.readthedocs.io/en/stable/encryption.html#pdf-encryption).

## Collaboration

OneNote has real-time collaboration.
I don't need that.
But your Bunch of Files is on GitHub, and Git was born for collaboration.

## VS Code Markdown Extensions

- For Markdown, I use "Markdown All in One," "Markdown Paste," "Markdown Table Prettifier."
- For personal knowledge base features, I use "Foam."
- For spellcheck, I use "Code Spell Checker" by Street Side Software.
- To view PDFs inside VS Code, I use vscode-pdf extension from tomoki1207.
- For creating diagrams and drawings, I use the Excalidraw extension for VS Code by pomdtr.
