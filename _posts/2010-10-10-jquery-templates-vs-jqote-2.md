---
layout: post
title: jQuery Templates vs jQote 2
---

{{ page.title }}
================

<p class="meta">October 10, 2010</p>

The jQuery team recently released an “official” jQuery plugin called Templates to address the increasing need for client-side templating when working with web applications. I have been using this technique for quite some time in my own projects through a different jQuery plugin called jQote2. Of course, the first thought that came to my mind was, “should I switch?”. For me, at least, the two primary concerns are feature set and speed.

*Feature Set*

The feature sets of the two plugins are pretty similar at first glance. Both offer you the basic ability to use Javascript variables inside of an HTML template stored in a `script` tag. The syntax is a little different ( `${varname}` vs `<%=varname%>`), but honestly that’s a pretty minor difference, and it’s really going to become second nature depending on which plugin you end up using. It does meant that I would have to re-write my existing templates to make the switch, though.

Both plugins offer the ability to pre-compile templates at page load so that there is minimal overhead associated with using a template over and over again on a single page. This compiling feature is also very important for those of us who work with Adobe AIR, as without the ability to compile, AIR’s `eval()` after onload restriction makes templating impossible.

So, as far as feature sets go, at least in my eyes it’s pretty much a tie, with a slight edge to jQote for my own purposes since there’s no point in converting all my existing templates to a new syntax if I don’t gain any new features.

*Speed*

The official jQote site has an interesting grouping of benchmarks published, and the author of the plugin was kind enough to post this benchmark page on Github. However, I noticed it had not been updated to include the newly-released official templating plugin, and instead includes a much older version of the plugin. I downloaded and modified the benchmark to include the latest version of the Templates plugin found on Github. 

The benchmark works by pre-compiling two templates (a simple template with a few variable insertions, and a more complicated template featuring a for loop) for all the tested plugins (it includes a number of other plugins besides just jQote and the official Templates plugin, which I have also included for consistency’s sake). It then passes two payloads to each templating system 1,000 times for a single benchmark, and this benchmark is repeated 25 times. It should be noted that by precompiling the templates, the actual compilation time is not taken into account in these benchmarks — this is strictly a test of the processing time it takes to process a payload into a pre-compiled template.

I ran the benchmark on Chrome 6.0.472.63 on Mac OS X Snow Leopard 10.6.3. Here are the results (average time to render, in milliseconds; lower bars are better):

<img src="/images/benchmark1.jpg" />

For reference, jQote2 had an average time of 11ms, maximum time of 15 ms. The official jQuery Templates plugin had an average time of 510ms, with a maximum time of 560ms.

The official plugin is off the charts. It is incredibly slow. For reference, here is the benchmark as it originally appears on the jQote site…here “jQuery Templating” refers to a very old version of the jQuery Templates plugin (which at the top reads “for demonstration purposes only”):

<img src="/images/benchmark2.jpg" />

So at some point in the development of the plugin, things got incredibly slow. Just to be sure, I re-ran these benchmarks on Safari and Firefox (both on Mac), and found similar results (the overall time of all the plugins actually increased since the Javascript engines in those browsers are currently slower than the one in Chrome on Mac, but the relative results were the same).

*Conclusions*

Of course, the new official Templates plugin for jQuery is still beta software, and I expect as they continue to develop it, the speed increases will come. However, as of right now, I will most definitely be sticking with jQote2 for all current and future websites, until the software has more time to mature. It is good to know, though, that the feature sets are very similar, as is the syntax, so that if a time comes in the future that the Templates plugin surpasses jQote in speed, it won’t be too terribly difficult to switch.

*Small Rant*

I am left to wonder, however, why the jQuery team decided to create their own templating code from scratch when there were already so many other jQuery templating plugins out there, including one such as jQote which already does everything they want their plugin to do, and faster to boot. Why reinvent the wheel? I am sure that since it is the official implementation, the Templates plugin will get a lot of use, have many tutorials incorporate it, be used in other plugins, etc. Which just drives home the point further that the jQuery community as a whole might have been better served by using a better, more mature implementation of this feature, rather than starting from scratch.