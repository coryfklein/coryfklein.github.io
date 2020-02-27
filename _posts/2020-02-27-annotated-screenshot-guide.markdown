---
layout: post
title:  "Simple, Quick, and Easy Annotated Screenshots in macOS"
date:   2019-02-27 09:02:03
comments: true
---

![Example annotated screenshot of highlighting Firefox in best browsers google search]({{ site.url }}/assets/annotated-screenshots-guide/best-browsers.png)

Here's how to make simple, clean, and professional-looking annotated screenshots. Annotating screenshots like this immediately draws visual attention to the item you care about and will help you communicate better. This kind of annotation prompts your audience to immediately pick out the pieces of information you want them to see.

Although this guide may make the process appear long, with practice it can take less than 15 seconds (yes, I timed myself) from start to finish to add one of these simple annotations to any given screenshot.

## Make The Screenshot

[⌃⇧⌘+4](https://support.apple.com/en-us/HT201361) is usually what you want - it will turn the cursor into a cross-hair and allow you to visually select a rectangle on the screen that you want to grab. The addition of the ⌃ key tells it to copy the screenshot to the clipboard (instead of writing it to file), which I strongly recommend.

![Example showing how the crosshairs selection works]({{ site.url }}/assets/annotated-screenshots-guide/crosshairs.jpg)

_Pro tip: If you want to capture the entire window, then hit the space bar and just click once on the window you want. MacOS will even add a pretty drop shadow. See the image at the top of this post for an example._

## Open

Open your favorite photo editor. Most Photoshop alternatives have the functionality you need. I personally recommend [Pixelmator](https://www.pixelmator.com/mac/), a Photoshop-like app made especially for macOS that does nearly everything Photoshop does, but for only $40 and none of Adobe's hostile annual licensing nonsense.

![The new file dialog in Pixelmator, showing the dimensions imported from the clipboard image]({{ site.url }}/assets/annotated-screenshots-guide/new-file-dialog.png)

⌘+N creates a new image to edit with, and Pixelmator will conveniently size it the exact same dimensions as the image in the clipboard.

![The empty canvas in Pixelmator, ready to paste in the image]({{ site.url }}/assets/annotated-screenshots-guide/empty-canvas.png)

⌘+V to paste the image in

![The canvas with the screenshot pasted in]({{ site.url }}/assets/annotated-screenshots-guide/pasted-screenshot.png)

## Make Your Selection

![The Pixelmator rectangular marquee tool]({{ site.url }}/assets/annotated-screenshots-guide/marquee-tool.png)

Use the _Rectangular Marquee Tool_ (keyboard shortcut is M in Pixelmator) to select the part of the screenshot you want to highlight.

![A rectangular selection has been made]({{ site.url }}/assets/annotated-screenshots-guide/rectangle-selected.png)

_Tip: You can add multiple selections by holding ⇧_

⇧⌘+I to "invert" the selection. This will allow you to fill in everything _except_ what you selected.

![The selection has been inverted]({{ site.url }}/assets/annotated-screenshots-guide/inverted-selection.png)

_Watch for the dashed lines crawling around the border of the image to know you've successfully inverted the selection_

## Shade Fill

![The fill dialog has been opened]({{ site.url }}/assets/annotated-screenshots-guide/fill-dialog-opened.png)

⌥⌘+F to open the Fill dialog. Type "50" for opacity (Pixelmator auto-focuses the input here, so no clicking required). Hit enter to apply.

![Opacity changed to 50 percent]({{ site.url }}/assets/annotated-screenshots-guide/50-opacity.png)

_What color should you use? See extra tips at the bottom for details._

## Share

![all of the image selected]({{ site.url }}/assets/annotated-screenshots-guide/all-selected.png)

⌘+A, ⌘+C to copy your nicely annotated image to the clipboard

⌘+V to paste your image into wherever! Most modern applications now accept pasting images into typical text fields. Some common applications for me are Slack, StackOverflow, presentation slides, and wikis.

![example of pasting an image into slack]({{ site.url }}/assets/annotated-screenshots-guide/paste-example.png)

## Extra Tips And Notes

When making multiple rectangles, consider using text to add simple numbers adjacent to the selections if you want to indicate ordering

![example of numbering multiple highlights]({{ site.url }}/assets/annotated-screenshots-guide/numbered-example.png)

I recommend always filling with black (#000000), except when the primary content is already black (like a terminal), then use pure white (#FFFFFF). You can play with opacity, but 50% is a solid choice that works well on most all images.

![example of white fill on black background]({{ site.url }}/assets/annotated-screenshots-guide/white-fill.png)

In Pixelmator you can set the default color to black by selecting the brush tool (B), changing the color to black, then quitting and re-opening Pixelmator

![how to change the default color]({{ site.url }}/assets/annotated-screenshots-guide/default-color.png)

## Clipboard vs Files

The instructions I give here are geared towards utilizing the clipboard. I find this to be much faster and easier, particularly because screenshots are so short-lived. But you should be able to use files as your intermediary storage between steps with minimal changes to the procedure.
