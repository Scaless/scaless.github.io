---
layout: post
title:  "MCC Halo 2 'BSP Crash' Fix"
date:   2021-05-01 12:10:43 -0500
categories: halo
twitchclip1: PlayfulDelightfulHumanSeemsGood
twitchclip2: BoredRamshackleCaribouBleedPurple
twitchclip3: BoredAttractiveLegTebowing-fMMQkjhXDv6jC5gJ
youtubevideo1: cDY_cN5cQuc
youtubevideo2: POnqGtKdsFA
youtubevideo3: jsiu1WPDqJQ
youtubevideorepro: eh021lpMwO4
---

*2021-06-09 Update: 343 has responded to my support ticket that they are aware of the issue :)*

*2021-10-13 Update: 343 Fixed the crash in Season 8 (v2580)! Unfortunately this patch also broke a ton of speedrunning tricks, but at least there's no crashes?*

*2024-08-23 Update: Fixed some broken links and removed Twitter link, since I deleted my account after Musk purchased it.*

---

For many months, a game crash has been plaguing the MCC Halo 2 speedrunning community. Seemingly random and inconsistent, it mostly struck players doing IL (Individual Level) speedruns. The crash gained its name from the bright green text that appeared in the bottom right of the screen during the game's final moments.

![BSP Crash](/assets/crash.jpg)

{% include twitchClipPlayer.html id=page.twitchclip2 %}
Credit: [Dubhzo][dubhzo-twitch] 

The oldest clip I could find of this style of crash is from early August 2020, and we believe the crash originated from one of the MCC patches around that period. Curiously, the [patch notes][july-patch-notes] for the July 14th 2020 release mention "Halo 2: Resolved a crash that could occur during long playthroughs". Hmmm.

# Digging Deeper

After some hot tips from the HaloRuns community for reproducing the crash and some analysis of crash dumps, I figured out where in memory the relevant data was stored. The memory layout for that section looks something like this:

{% highlight cpp %}
int32_t next_index = 1; // The next index into p_buffer to allocate and hand out, starts at 1
void* p_buffer[32768]; // Buffer of pointers, fixed-size 32k elements
[Debug Data]
bool show_bsp_debug; // Usually 0, if > 0 show debug string on-screen
[More Debug Data]
[Scenario Data]
{% endhighlight %}

* `next_index` starts at 1 because the 0th element in the buffer is used as an offset to add to pointers read from the buffer. On PC this offset seems to always be 0 and is not relevant to the bug itself.

Every time the game loads content, a new value is stored in `p_buffer` at the index of the current value of `next_index`. `next_index` is then incremented by 1 and the value of the previous index is returned. The pseudocode for the function looks like this: 

{% highlight cpp %}
int64_t insert_value_into_p_buffer(int64_t value)
{
  int64_t allocated_index = next_index;
  p_buffer[next_index] = value;
  next_index++;
  return allocated_index;
}
{% endhighlight %}

The bug here is that `next_index` has no upper bound and only has 1 reset condition: the first time the level is loaded. It grows and grows until it is larger than 32768, pointing beyond the end of `p_buffer` and corrupting other regions of memory. First it overruns onto the debug information which causes the familiar green BSP and POS counters to be visible. After that, critical runtime and scenario data is overwritten, at which point the game cannot cope and crashes.

Data corruption bugs are usually some of the most difficult to trace and debug. We were extremely lucky that the debug register for showing the BSP text on screen was the first thing to get corrupted. Being able to find the format string for that text and working backwards from there made the process very smooth. The full format string for that text actually looks like this:

{% highlight cpp %}
[bsp %d][pos %4.1f %4.1f %4.1f][fps %2.1f] %2.2f
{% endhighlight %}

This itself might have been an earlier version of the [pan-cam][pan-cam-wiki] stats that later games used. I really like finding these little debugging tools that developers made for themselves.

# Repro

Since we now know the exact cause of the crash, making a repro was easy. Quarantine Zone was by far the worst level for triggering the crash, so the optimal strategy for reproducing it is the following:

1. Start up QZ
2. Drive to the 2nd shutter door
3. Restart the level
4. Repeat for roughly 9 minutes until `next_index` grows above 32768 and corrupts memory.

{% include youtubePlayer.html id=page.youtubevideorepro %}

A faster alternative with access to debugging tools is to just set `next_index` to a value close to the threshold, say 32000. This should cause the crash  quickly depending on the load zones in the level.

# Resolution

Typical players will almost never run into this crash. It relies on restarting and running through the level over and over, dozens of times so that the index grows large enough to crash. What kind of player would do such a thing?

Oh right, speedrunners.

If you're grinding for a good IL time, you will constantly be restarting the level after any mistake. This could be dozens or hundreds of attempts in a single session. That's why this crash hasn't been more widely reported by the playerbase, they just don't play enough!

Until this is officially fixed by 343, I have released a code patch to fix this bug. The basic gist of the solution is to hook the functions that modify the `p_buffer` table and redirect them to write into a `std::vector` that can dynamically grow to any size. The `p_buffer` table is then not used any further. While this doesn't solve the underlying issue of poor memory management, it does allow for vastly longer play sessions without crashing.

[The code for the patch is available here][implemented-fix]. 

# Final Thoughts

Big thanks to Harc for helping me with debugging and testing: [Check out his Twitch][harc-twitch] 

Did somebody say [HaloRuns.com][haloruns-link]? If you're interested in speedrunning any game in the Halo series, give the [HaloRuns site][haloruns-link] and [Discord][haloruns-discord] a peek.

If anyone from 343 is reading, the relevant functions and addresses used are here:

| Game Version | MCC 2282 , Steam |
| next_index | halo2.dll+CD8098 |
| p_buffer table | halo2.dll+E22370 |
| clear_pointer_table | halo2.dll+6DF770 |
| get_pointer_by_index | halo2.dll+6DF7A0 |
| insert_pointer | halo2.dll+6DF7B0 |

| Game Version | MCC 2406 , Steam |
| next_index | halo2.dll+CD9098 |
| p_buffer table | halo2.dll+E23110 |
| clear_pointer_table | halo2.dll+6DF710 |
| get_pointer_by_index | halo2.dll+6DF740 |
| insert_pointer | halo2.dll+6DF750 |

pls fix

## More examples of crashes:

#### Harc
{% include twitchClipPlayer.html id=page.twitchclip3 %}

#### EggplantHydra:
{% include twitchClipPlayer.html id=page.twitchclip1 %}

#### Temperament:
{% raw %}
<iframe src="https://player.twitch.tv/?video=831432106&parent=blog.scal.es&autoplay=false" frameborder="0" allowfullscreen="true" scrolling="no" height="480" width="720"></iframe>
{% endraw %}
#### ibigblue:
{% include youtubePlayer.html id=page.youtubevideo1 %}
{% include youtubePlayer.html id=page.youtubevideo2 %}
#### Raiyuki:
This one is really interesting because it overflowed just enough to write into the debug data and enable the BSP debug text, but not far enough to actually cause a crash until the next load. The artifacts you see flailing around are caused by other parts of code still reading/writing to the addresses that are now shared with the overflowed `p_buffer`.
{% include youtubePlayer.html id=page.youtubevideo3 %}

# Bonus

Halo 2 Mobile?

![Halo 2 Mobile](/assets/mobile_halo.jpg)

[pan-cam-wiki]: https://www.halopedia.org/Panoramic_Camera_Mode
[haloruns-link]: https://haloruns.com/
[haloruns-discord]: https://haloruns.com/discord
[harc-twitch]: https://www.twitch.tv/harctehshark
[dubhzo-twitch]: https://www.twitch.tv/dubhzo
[implemented-fix]: https://github.com/Scaless/HaloTAS/blob/0706b2bf5150ef00c9ed520fe21dde2d3304d31c/HaloTAS/HRPatcher/dllmain.cpp#L268
[july-patch-notes]: https://support.halowaypoint.com/hc/en-us/articles/360045689232-Halo-The-Master-Chief-Collection-Patch-Notes-7-14-20
