---
layout: post
title: jQuery Templates vs jQote 2
---

{{ page.title }}
================

<p class="meta">October 12, 2010</p>

Introduction
------------

My [original post](/2010/10/10/jquery-templates-vs-jqote-2.html) regarding a speed comparison between jQote2 and the new jQuery Templates plugin has generated more interest than I originally expected. In fact, several members of the jQuery Templates team have written to me either with questions or with suggestions about how to improve the benchmark to make it a more “apples to apples” comparison. To understand why the first benchmark wasn’t necessarily as fair as I had originally assumed, we have to dig a little deeper into the way the two plugins work.

The original jQote plugin worked very similarly to the way that I was apparently using the jQuery Templates plugin (and it was much slower than jQote 2). Both jQote 1 and the `.tmpl()` function of the jQuery Templates plugin apparently process the template and then create an unparented DOM fragment that is ready for insertion into the page. The jQote2 plugin, on the other hand, simply generates a string from the template. The overhead associated with creating the DOM fragment is apparently very high, and explained a great deal of the slow down in the jQuery Templates plugin benchmark.

Boris Rivers-Moore, the principal developer on the jQuery Templates plugin, was kind enough to write me an email explaining this, and offered a suggestion on how to improve the benchmark. Apparently, there is a different way of going about using the plugin, with the following method: `template_object($, payload).join("");`

This way of doing things works the same as the jQote2 plugin, generating a string from the template, rather than the unparented DOM object. It is also much faster.

He also pointed out that the jQuery Templates plugin automatically HTML encodes during the variable insertion, and offered a different set of tags for the template that would remove this functionality, again providing a better comparison for a benchmark.

Benchmarks
----------

Here is an updated benchmark, using this new method and with the template tag changes (time to render in milliseconds; lower bars are better):

<img src="/images/benchmark5.png" />

Average time for jQote2 was 11ms, for jQuery Templates 25ms. As you can see, the jQuery Templates plugin is now much, much more competitive in terms of speed compared to the jQote plugin. 

However, this got me thinking; the benchmark could really be considered rather synthetic at this point, because although it does measure the time to go from template + payload to HTML string, it doesn’t actually take into account doing anything with the resulting string. So, I added on some additional functionality to the benchmark, where the resulting string is appended to the DOM object (in this case, a `<div>` tag hidden by CSS). Something along these lines:

	$("#div").append(template_object($, payload).join("")); //jQuery Templates
	$("#div").append($.jqote(template_object, payload)); //jQote2

Here are the results of this adjustment:

<img src="/images/benchmark3.png" />

Average time for jQote2 was 66ms, for jQuery Templating, 76ms. As you can see, some time is added on by adding the string to the DOM, but not much, and again the plugins are very close in speed. This, however, led to one further thought: if adding the string to the DOM is so cheap, then why bother with using `.tmpl()` and creating the unparented DOM fragment?

As a comparison, I again modified the benchmark, this time to use the `.tmpl()` call from the jQuery Templates plugin, instead of the function which creates a string. Keep in mind that this is the “officially documented” way of using the plugin, and the way that all of the new tutorials being created out there seem to be using. The code was something like this (similar to what was used in the original benchmark): `$.tmpl(template_object, payload).appendTo("#div");`

And here were the results:

<img src="/images/benchmark4.png" />

Average time was 480ms. Over 6 times slower. From a very speedy 76ms, to a page-halting 480ms. 

I understand that having an unparented DOM object can have its advantages. For example, it allows you to chain together a series of statements, and work with the resulting object directly in jQuery if you need to do further processing. However, the vast majority of tutorials, and even the official documentation on the jQuery site, is simply taking this unparented DOM object and appending it to some other object on the page (as does the benchmark). It’s apparent from these results (as near as I can tell, anyway), that it would be much, much faster to simply create the string, and then append that string to the DOM (which I assume is why jQote2 switched to this as its primary modus operandi, instead of the DOM fragment functionality of jQote 1).

Conclusions
-----------

When doing a better, apples-to-apples comparison, it does become clear that the jQuery Templates plugin certainly can hold its own ground against the existing field of jQuery templating plugins, including jQote2. However, the majority of material that I have read about the new plugin (including the official documentation) uses a method which is apparently many times slower than it needs to be. I looked through the official jQuery documentation for any mention of the faster “string” method of using the plugin, and I was unable to find any. It is possible that I missed it, but I hope that these findings will encourage the jQuery Templates team to better publicize the faster method. Again, while the .tmpl() function may be useful if chaining or further processing is required, if your only goal is to get the resulting template into the DOM, the string method is much, much faster.

On the other hand, I should point out that each benchmark here is doing 1,000 template executions, so perhaps this is an unfair comparison of using the .tmpl() function. One could argue that if you needed to loop through 1,000 items, you would do so inside the template, and depending on how the .tmpl() function handles internal loops (I honestly have not looked), it might cut down significantly on the overhead, since it would only be presumably building one fragment instead of 1,000. But, to be on the safe side, it seems that if you just want to process the template and then insert the the result as a whole without any further processing, the string method seems to be the best bet.

Boris also took the time to point out a few other features that jQuery Templates has that I was unaware of:

> The jQuery Templates does include a number of other features in addition to encoding which may not be part of the other template engines. One is using with() so that you can say {{html name}} rather than {{html this.name}} as in jQote. Also, we test for whether fields are defined, so that we can accept ‘jagged’ arrays, where a field may be defined on some items and not on others. And we test for whether foo (in {{html foo}} or ${foo} ) is of type function, and if so, we call the function. And so on. These features impact the perf [sic] slightly.

Thanks to Boris Rivers-Moore and Marcus Tucker for taking the time to read my original post and respond. Keep up the good work!