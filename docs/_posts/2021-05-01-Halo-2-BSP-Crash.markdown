---
layout: post
title:  "Halo 2 'BSP Crash' Fix"
date:   2021-05-01 12:10:43 -0500
categories: halo bugfix
twitchclip1: PlayfulDelightfulHumanSeemsGood
twitchclip2: BoredRamshackleCaribouBleedPurple
twitchvideo1: video=v831432106&parent=blog.scal.es
youtubevideo1: cDY_cN5cQuc
youtubevideo2: POnqGtKdsFA
youtubevideo3: jsiu1WPDqJQ
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

## Crashes from the HaloRuns community:

### EggplantHydra:
{% include twitchClipPlayer.html id=page.twitchclip1 %}
### Dubhzo:
{% include twitchClipPlayer.html id=page.twitchclip2 %}
### Temperament:
{% raw %}
<iframe src="https://player.twitch.tv/?video=831432106&parent=blog.scal.es&autoplay=false" frameborder="0" allowfullscreen="true" scrolling="no" height="480" width="720"></iframe>
{% endraw %}
### ibigblue #1:
{% include youtubePlayer.html id=page.youtubevideo1 %}
### ibigblue #2:
{% include youtubePlayer.html id=page.youtubevideo2 %}
### Raiyuki:
{% include youtubePlayer.html id=page.youtubevideo3 %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

[implemented-fix]: https://github.com/Scaless/HaloTAS/blob/master/HaloTAS/HRPatcher/dllmain.cpp#L257
