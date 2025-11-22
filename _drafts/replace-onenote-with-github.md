I used to use OneNote on Windows.
It worked pretty well--I could add content easily without much thought or work.
I stopped using Windows a couple years ago, and therefor stopped using OneNote on Windows.
I can still access my notebooks via the web interface, but I hate it.
It's buggy, does not have all the same features, and pages take forever to load.

As a replacement for OneDrive, I use a set of markdown files, image files, and pdfs.
I keep these files in GitHub.
I browse and edit them either in VS Code locally or via the GitHub web interface.

This change has a lot of benefits for me and I am happy to be done with OneNote.
One thing I love is the transparency and portability of the file format.
OneNote stores things in a complex propriety format that makes migrating your files very difficult.

I am not trying to persuade you to switch from OneNote to plain files like I did.
But if you do want to, these ideas might make the switch easier for you.

## Notebooks, Sections, and Pages

How you organize your notes is probably at least as important the application you choose to create them.
OneNote is nice if you like a hierarchical organization.

I think of a git repo as the equivalent of a OneNote notebook.
OneNote sections can be replicated with directories, nested arbitrarily deep.
A markdown file is the equivalent of a OneNote page.
I don't know what to say about OneNote sub-pages: they are an odd concept.
You can get more or less the same effect with an additional directory level.

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
To get spell checking, I use VS Code locally to edit the files.

## Print to OneNote

I just print to a PDF and place the PDF in the folder where it belongs.
To view it, select the file in the GitHub UI.
To annotate it, I use a native app.
This is a better experience than using OneNote tools to draw on top of a PDF printout in a page or inserting text boxes around the page.

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

For creating diagrams and drawing, I use [excalidraw.com](https://excalidraw.com/).
When saving the file I created, I use "Export Image".
I select "svg" as the image format, check "Embed scene" option, and download the file and add it to my repo.
The GitHub UI displays the image when I select the file.

To edit the image, I upload the .svg file to excalidraw.com again.
Since I embedded the scene, the diagram is loaded back into the editor and I can carry on editing it.
Export it again and add it back to the repo.

## Version Control

OneNote has a built-in page history, but it no where near as robust as git.
For example, moving a page in OneNote to a different section destroys the page history.

## Mobile App

## Tags

## Hand-written notes

- xcalidraw
- xournal++
- rnote

## Dictation

## EncroS Sections

## Other VS Code Markdown Extensions

Have a look at "Markdown All in One," "Markdown Paste," "Markdown Table Prettifier," and "Foam."

## Further Reading

[](https://youtu.be/Ly10LxvzNGg?si=_zE0xco9_Xb_91SI)
