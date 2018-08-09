---
layout: inner
title: 'Lucky Eggs and Friendship'
date: 2018-02-18 17:00:00
categories: blog pokemon
tags: pokemon 
lead_text: 'Lucky Egg usage and Friendship XP in Pokémon Go'
---

### Intro

A lot of people are starting to hit the "Ultra Friends" level of friendship in Pokémon Go and are finding that it grants 50,000 experience when you level up. Many have coordinated the level up moment to happen when a lucky egg is active to double that experience. I found myself hesitant to use an egg "just" for this moment rather than a full 30 minutes of gameplay at a park with lots of Pokémon to catch. However, does this make sense? Do I actually gain more experience in 30 minutes of playing the game than this one single moment? 

Let's take a look.

There are a few ways to gain experience in Pokémon Go including catching, evolving, and raiding which are typically the most lucrative. Let's evaluate if any of these methods alone (or combined) can do better than 50,000 xp. Note: Of course you are not restricted between these 2 choices. You can level up friendship AND do these activities as a win/win. You won't always have that option especially with a full work day so logging in at the agreed upon time for leveling up friendship is a lot less of a time investment.

### Catch XP

I caught nearly 900 pokémon the day of GoFest where maybe 800 were at my ~6 hours at GoFest using the Fast Catch method on everything in sight. Assuming best case scenario, that amounts to 800/6 = ~133/hr or ~67/30min (egg duration). This is probably under the absolute best with fast catch as we're still around 26s a catch. 7-10s is more typical for the actual catch but that assumes you have no lag and always have spawns within reach.

We can evaluate a **67** catch session and **180** catches (10s / (30min * 60s/min))

Moving with speed in mind your throws won't be the best. Best case you're probably throwing great curve balls consistently. If you're fast catching, all your catches can be assumed to be 'first throw' catches if you stay moving to get the most attempts in at spawns without wasting time checking pokemon for repeat attempts.


{% highlight python %}
import math

def calculate_catch_xp(num_catches, accuracy='great', first_throw=True):
    xp_values = {
             'catch': 100,
             'curve': 10,
             'nice': 10,
             'great': 50,
             'excellent': 100,
             'first_throw': 50
            }
    val = num_catches * (xp_values['catch'] + xp_values[accuracy] + xp_values['first_throw'] * first_throw)
    return val
{% endhighlight %}


{% highlight python %}
"{:,}".format(calculate_catch_xp(67, accuracy='great', first_throw=True))
'13,400'

"{:,}".format(calculate_catch_xp(180, accuracy='great', first_throw=True))
'36,000'

"{:,}".format(calculate_catch_xp(180, accuracy='excellent', first_throw=True))
'45,000'
{% endhighlight %}

Ok so 13,400 extra experience for catching fast with great accuracy. 36k for a LOT of catches with great accuracy. 45k for a LOT of catches with excellent accuracy. That still does not beat the 50k offered by becoming an Ultra Friend and is not realistic at all.

### Raid XP

Raids are usually a good source of XP. How do legendary raids compare to Ultra Friends? You get 10,000 xp for a legendary raid and _if_ placement of gyms and spawn times line up favorably you can do anywhere from 2 to 4 or more if in a very dense area.


{% highlight python %}
legendary_raid_xp = 10000
raids_in_30_min = range(2,7)
for num_raids in raids_in_30_min:
    print(f"Beating {num_raids} legendary raids yields {num_raids*legendary_raid_xp:,} xp")
{% endhighlight %}

    Beating 2 legendary raids yields 20,000 xp
    Beating 3 legendary raids yields 30,000 xp
    Beating 4 legendary raids yields 40,000 xp
    Beating 5 legendary raids yields 50,000 xp
    Beating 6 legendary raids yields 60,000 xp


Taking down 5 or more legendaries will match and then exceed ultra friend status. I've never come across that many raids close together before other than the Zapdos and Articuno days. I'll exclude those from this analysis as they are not a readily available or recurring event. Purely Legendary Raids are not going to get you near Ultra Friends levels of xp on just any day.

### Evolution XP

What about evolving? That gives 500 xp per pokemon and you can get through quite a few in 30 minutes. Let's do the math.

It takes anywhere from 18 to 22 seconds to evolve a Pokémon depending on your phone's processor and lag (whether from cell network, wifi, or Niantic's servers). There are 1,800 seconds in 30 minutes (30 * 60) which gives us anywhere from 81 to 100 Pokémon evolved during a lucky egg.


{% highlight python %}
evolve_xp = 500
time_in_egg_s = 1800
time_to_evolve_s = range(18,23)
{% endhighlight %}


{% highlight python %}
for duration_s in time_to_evolve_s:
    num_evolutions = math.floor(time_in_egg_s / duration_s)
    num_xp = num_evolutions * evolve_xp
    print(f"Evolving {num_evolutions:3} (1 per {duration_s} seconds) yields {num_xp:,} xp")
{% endhighlight %}

    Evolving 100 (1 per 18 seconds) yields 50,000 xp
    Evolving  94 (1 per 19 seconds) yields 47,000 xp
    Evolving  90 (1 per 20 seconds) yields 45,000 xp
    Evolving  85 (1 per 21 seconds) yields 42,500 xp
    Evolving  81 (1 per 22 seconds) yields 40,500 xp


Our best case scenario *exactly* matches that of an Ultra Friend at 50k xp. Any amount of lag will make this impossible.

### Multiple Sources

Ok so in reality you're not just doing ONE of these things but rather a combination of them. What would that look like at a very optimal level? We would have a friend driving us location to location as well as starting raid lobbies for us. Lures and an Incense would be used to maximize spawns.

Let's try with spending half our time (*15 minutes*) evolving Pokémon at a perfect pace. This gives us **50 total evolves**.

Next let's say we beat **3 raids**. This would take us ~10s each in lobbies (assuming a friend starts for us and we hop in at the last 10 seconds, evolving while we wait) and maybe 25s for a quick raid and running away or attempting a fast catch. This gives us 3 * 35s = _140 seconds_ (~2.5 minutes) spent raiding.

Maybe we catch **75 Pokémon** during all of this with fast catch (75 * 10s = 750s or _12.5 minutes_)

This sums up to 15 + 2.5 + 12.5 = 30 minutes. Pretty tight timeline without much room for lag or error.


{% highlight python %}
num_evolve = 50
num_catch = 75
num_legendary_raids = 3

(
 num_evolve * evolve_xp + 
 num_legendary_raids * legendary_raid_xp + 
 calculate_catch_xp(num_catch, accuracy='great', first_throw=True)
)
{% endhighlight %}

    70000

So with all of those assumptions we end up with 70k! This is great but takes a lot of effort and luck to pull off. Maybe a more realistic scenario would be 20 evolves, 30 catches and 1 legendary raid:


{% highlight python %}
num_evolve = 20
num_catch = 30
num_legendary_raids = 1

(
 num_evolve * evolve_xp + 
 num_legendary_raids * legendary_raid_xp + 
 calculate_catch_xp(num_catch, accuracy='great', first_throw=True)
)
{% endhighlight %}

    26000

26k is not quite as good.

### Conclusion

Long story short, it's very easy to coordinate a friendship level up for 50k and very difficult to come near that otherwise. Doing these other activities after a friendship level up during an egg is of course the best scenario, but if you are choosing between using an egg for becoming an Ultra Friend and something else, you are better off using it for friendship.

