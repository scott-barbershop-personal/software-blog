---
title: "Structuring software as if they were recipes - Part1"
date: 2026-06-12
---

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/recipe.png?raw=true)

Welcome back to the Software Kitchen.  As part of this 4 part series, we're making: a well-chartered software component, lightly seasoned with clear responsibilities, hold the scope creep.  You'll find it is surprisingly easy to make and it'll impress your guests.  Let's get started!  

But why food recipes?

First of all because you don't need a CS degree to understand them.  Everyone understands it at a primal level.  They are baked into our culture... literally!  Try saying the same for the software that you own.  You may need the smartest AI in the room to help you decipher what it is trying to do and, unlike a pizza, that's not very fun at 2am.  So, let's fix that by first starting where all good dishes start with, what you are making and who is making it.

### Charter, the dish name
"Scott, what are you making for us?"

"Well, I'm making flour mixed with water, yeast, salt, and oil, mixed for 3 minutes..."

Ummm, no.  You're making pizza (Sicilian pizza to be exact)!  And the fact that you are making pizza is important... especially to those who eat it.  There are a thousand recipes that use flour, water, and yeast but whether it is right or not is based upon the final outcome and how it differs from what the recipe was trying to do.  Too often, engineers forget that.  Perhaps it is because we are obsessed with how things are wired and start to see things by their components.  But applications are greater than the sum of their parts.  This lesson is important because your peers, leaders, and customers will judge your dish based upon whether it fulfills its expectations.  So, think carefully about what is your software's charter and don't just jump into the implementation.

Naming is one of the hardest things in software — right up there with cache invalidation and deciding if tabs vs. spaces is worth losing a friendship over.  Charters can change over time and it is difficult to change APIs once they are in place.  However, it doesn't mean it doesn't matter.  Therefore, avoid weasel names that don't really say anything at all (e.g. manager, handler -- after all everything manages and/or handles SOMETHING otherwise it wouldn't exist at all) and focus on intent.  It should be clear what things should be included and what things should be excluded.  If you don't have a strong opinion on it, then I promise you that the service will get misused as a garbage heap for every piece of logic that has no better home.  I don't know about you but I'm not too excited about eating a garbage pizza.

### Responsibilities, the chef's hat
You've been hyping your homemade pizza to your family all week. Then Saturday arrives and a Domino's delivery guy rings the doorbell. Your family is, let's say, not thrilled.  However, on the other hand, if your wife is stressed out and running around everywhere and asks you to call Domino’s for her, you're a lifesaver.  Even worse, if you tell her that you'll call Domino's but then start making pizza yourself, she might get furious with you:  ¨Company is coming over in 10 minutes and you haven’t even started cooking!  The kitchen is a mess and I needed your help to take care of the kids…¨  Nobody cares whether you made the pizza or called Domino's. They care whether you did what you said you'd do.  Make sure that your software similarly defines itself clearly as a proxy or as the heavy lifter doing the work itself.  Consult the software design patterns to help brainstorm how to advertise your components to others.

### Anti-pattern:  How it is used

A pizza is a pizza, no matter how you use it.  Pizzas don't make very good carjacks or window cleaners.  They are for solving hunger problems... especially at 2am.  You should be striving to make software with just as strong of a charter: one that stands on its own and isn't heavily dictated by how others use it.  A service or utility that has a strong charter could have a million of different uses and you shouldn't limit your imagination based on your original needs.  Your job, when defining the charter is to help explain the properties of your component so that when others use it, they are well aware of whether it will work for their intended purpose or not.  If you do your job well, your dish may just become famous.

### Takeaways
When examining software, try to understand what is its primary responsibility and what it is allowed to do to get it done.  Each component, from the smallest utility to the largest service, will each have their own charter.  Be very skeptical of any work that a component does that does not align with its charter.  Document the charter and responsibilities, leaving out how it is currently used.
