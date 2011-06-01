---
layout: post
title: An easy way to change the settings of an S3 Bucket’s files (in Ruby)
---

{{ page.title }}
================

<p class="meta">May 20, 2011</p>

I recently needed to go back and change the settings of the files in an S3 bucket. I wanted to set the permissions to “pubic read” and also set a cache control header so that the images would be cached in browsers approparitely. After a bit of Googling, I wasn’t able to find anything particularly simple and straightforward. It’s really a pretty easy task, though. The one “gotcha” is that S3 will only return up to 1,000 keys for a given bucket at a time, so you have to be sure to loop through until you get them all. Here’s what I came up with:

<script src="https://gist.github.com/982938.js?file=setheaders.rb"></script>