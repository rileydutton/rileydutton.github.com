---
layout: post
title: Getting MSSQL to Work with PHP on Ubuntu 10.0.4
---

{{ page.title }}
================

<p class="meta">September 27, 2010</p>

After about 2 hours of Googling, I finally found this gem showing you how to get the MSSQL module installed on Ubuntu for PHP. Take my advice, DON’T go the ODBC route — it’s frought with peril. 

The only thing that seems to have changed between that post and 10.0.4 is that `sudo apt-get source php5` put the php5 source in my home directory (`~`) instead of the a system directory. I also had to install `php5-dev` in order to get access to `phpize`. Everything else was a breeze!

Just in case it should ever go away, here’s the exact steps I took to install on Ubuntu 10.0.4:

	sudo apt-get install php5-dev
	sudo apt-get source php5
	cd ~/php5-5.3.2/ext/mssql
	sudo phpize
	sudo ./configure
	sudo make
	sudo make install
	
*Viola!*