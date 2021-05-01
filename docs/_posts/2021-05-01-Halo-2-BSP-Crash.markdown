---
layout: post
title:  "Halo 2 'BSP Crash' Fix"
date:   2021-05-01 12:10:43 -0500
categories: halo bugfix
twitchvideo1: PlayfulDelightfulHumanSeemsGood
twitchvideo2: 
---

{% highlight cpp %}


// ...
void* p_buffer[32768]; Buffer of tag pointers, fixed-size 32k elements
// After the tag buffer exists some debug data
// ...
char show_bsp_debug; // Usually 0, if > 0 show debug string on-screen
// ...
// After the debug data is *very important* Scenario data
// ...
// ...
// The next index into p_buffer to allocate and hand out, starts at 1
int64_t next_tag_index = 1;
{% endhighlight %}

{% include twitchPlayer.html id=page.twitchvideo1 %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

[implemented-fix]: https://github.com/Scaless/HaloTAS/blob/master/HaloTAS/HRPatcher/dllmain.cpp#L257
