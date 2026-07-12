---
title: "Structuring software as if they were recipes - Part1"
date: 2026-06-12
---

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/recipe.png?raw=true)

Recipes are universal communication structures used by many cultures, expressed in a variety of languages, across many generations.  When exchanged between individuals, recipes may not be easy to follow but they generally are easy to understand, at a high level and from a quick glance.  However, this usually isn't true for software.  Why is that?  The blog attempts to distill the secrets about recipes and how we can apply them to make software easier to understand.

## Charter
We start by discussing charter as it is the most important thing in software and what you should generally focus a majority of your time thinking about.  It is more important that what your service actually does because charter can determine whether your functionality is correct or whether it has bugs.  For example, 2 + 2 = 5 might actually be valid for a test case within a test suite.

### The dish name
"Scott, what are you making for us?"

"Well, I'm making flour mixed with water, yeast, salt, and oil, mixed for 3 minutes..."

Ummm, no.  You're making pizza (Sicilian pizza to be exact)!  And the fact that you are making pizza is important... especially to those who eat it.  There are a thousand recipes that use flour, water, and yeast but whether it is right or not is based upon the final outcome and how it differs from what the recipe was trying to do.  Too often, engineers forget that.  Perhaps it is because we are obsessed with how things are wired and start to see things by their components.  But applications are greater than the sum of their parts.  That is important because you need to measure it (via success metrics) based on how well it is doing its job.

I won't say defining charter and naming are easy tasks.  In fact, naming is one of the hardest things in software.  Charters change and it is difficult to change APIs once they are in place.  However, it doesn't mean it doesn't matter.  Therefore, avoid weasel names that don't really say anything at all (e.g. manager, handler -- after all everything manages and/or handles SOMETHING otherwise it wouldn't exist at all) and focus on intent.  It should be clear what things should be included and what things should be excluded.  If you don't have a strong opinion on it, then I promise you that the service will get misused as a garbage heap for every piece of logic that has no better home.  I don't know about you but I'm not too excited about eating a garbage pizza.

### Who is the chef
If you are an excellent chef and you promised your family to personally make a pizza but then you called Domino’s to make it and deliver it for you, chances are your family would be disappointed.  However, on the other hand, if a family member needed to order a Domino’s pizza but were super busy and you offered to help by calling Domino's for them, they would very happy.  Finally, if you promise to call Domino’s but then make the pizza yourself, there is a chance your family member may get upset with you.  ¨Company is coming over in 10 minutes and you haven’t even started cooking!  The kitchen is a mess and I needed your help to take care of the kids…¨  It doesn't matter whether you make a pizza or call Domino's; what matters is your charter and responsibilities.  In the first, you are declaring yourself as chef; while in the second you are declaring yourself as a proxy or facilitator.  Only when your actions line up with your charter and responsibilities are you fulfilling your charter.

### Anti-pattern:  How it is used
A pizza is a pizza, despite what you do with it.  While the chef may not be very happy that you use a pizza as a comedy prop (to throw in someone's face) or as a temporary tray on which to serve other food... it doesn't change the properties of a pizza.  A charter should stand on its own and should not be heavily dictated by how others use it.  A service or utility that has a strong charter could have a million of different uses and you shouldn't limit your imagination based on your original needs.  Your job, when defining the charter is to help explain the properties of your component so that when others use it, they are well aware of whether it will work for their intended purpose or not.  If you do your job well, they will be able to gauge best whether it can be used or not to fulfill their requirements.  For example, it may be important to note that a pizza will be extremely hot when it is finished... something the user might want to know before they throw it in someone's face.

### Takeaways
When examining software, try to understand what is its primary responsibility and what it is allowed to do to get it done.  Each component, from the smallest utility to the largest service, will each have their own charter.  Be very skeptical of any work that a component does that does not align with its charter.  Document the charter and responsibilities, leaving out how it is currently used.
