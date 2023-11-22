---
layout: post
title: Create a contact sheet photo collage poster
author: Tomasz Janczuk
tags:
  - photo collage
  - contact sheet
  - poster
  - exif
  - travel
---

Creating a photo collage from your digital photos should be easy, right? It is unless you have very specific needs for how you want your collage to look like.

In my case, I have just returned from a [year-long, around-the-world travel](https://22.janczuk.org) with 25,000+ pictures, and I wanted to arrange some 500 of them in a contact-sheet-like 30"x20" photo collage poster to hang on a wall. I wanted each picture to have a custom description and the GPS location where the picture was taken, extracted from the EXIF metadata.

When I started looking for a solution, no tool out there did exactly what I wanted, so I created my own: [tjanczuk/mkcollage](https://github.com/tjanczuk/mkcollage). Here is a sample of what the tool can create, with a few pictures from south of the equator:

<a href="/assets/post_images/2023-09-14/0.webp" style="border-bottom:none;"><img src="/assets/post_images/2023-09-14/0.webp" class="tj-img-diagram-100" alt="Contact sheet photo collage created with tjanczuk/mkcollage"></a>

What can you do with [tjanczuk/mkcollage](https://github.com/tjanczuk/mkcollage)?

- Use image files from your local drive to create contact-sheet-like, poster-size photo collages.
- Export the final collage image as PNG, JPEG, or WEBP.
- Set the desired collage width and individual picture height.
- Annotate images on the collage with custom descriptions or based on EXIF information in the image.
- Select images to include or order images based on image metadata, including EXIF information (the example above used a filter that only included images with latitude below 8 degrees South).
- Rich customization using CSS.

Check out the documentation for details, and happy collaging!
