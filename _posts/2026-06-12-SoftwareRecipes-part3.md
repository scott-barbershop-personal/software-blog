---
title: "Structuring software as if they were recipes - Part 3"
date: 2026-06-12
---

![Recipe]({{ site.baseurl }}/images/recipe.png)

Welcome back to Software Kitchen.  Today, in part 3, we're sharpening our knives because a chef is only as good as their tools.  And the same is painfully true in software.  Let's get started!

## Utility methods, the cooking verbs
"Let's start baking our pizza by first cleaning our oven and making sure its wiring is up to governmental standards..."  I think you would quickly change the channel, unless you are into that sort of thing.  It isn't to say that those other pieces aren't important; a pizza business couldn't survive without them.  However, they aren't core to making pizzas and we're following this recipe to make a pizza!  Software engineers regularly mix up this point.

Our pizza recipe is clear and concise because it expects you have some knowledge of strong cooking verbs:  dice, simmer, sear, reduce, blanch, and poach.  It doesn't teach you how to cook.  By knowing these verbs, the recipe is clear.  For example, a chef knows the difference between a slow simmer and a rolling boil.  However, well-written software utilities are hard to find.  For example, within software, it is very common to see a utility like this:
```
Dough dough = new Dough();
...
bakePizza(dough);
```
If recipes were written like this, chances are you'd burn your house down.  Why?  Because it just happens to be that this specific bake function was programmed to cook specialty Neapolitan pizzas at 900 degrees Fahrenheit (480 degrees Celsius) for 1.5 minutes.  If you left it baking for 15 minutes, as this Siciliana pizza recipe recommends, there would be big problems.  Software engineers often create utilities like this because they see duplication and they want to minimize it.  As they say, "The road to hell is paved in good intentions."  Even if 100 recipes all used the oven to cook something at 425F, they'd never put that information into the bake utility... and neither should you.  Additionally, we shouldn't have to change all recipes just because we discovered that it is better to cook our pizza at 415F instead of 425F.

The top-level instructions within recipes always specify the important variables to a cooking verb: oven temperature, time to bake, and success criteria.  Let's do the same:
```
bake(dough, new Temperature(425, Scale.Fahrenheit), SuccessCondition.OR(TimeUnit.MINUTES.toMillis(15), Color.LightBrown));
```
Everything that is important here is explicit.  For example, the function doesn't tell you that it requires an oven or even that you must plug in the oven first; those are obvious from the word "bake".  The recipe also doesn't tell you what to do if you are having technical problems with your oven.  Software engineers like to say that in order to write good software, you should focus on what is "business logic" and what isn't.  However, I always find this definition vague and unsatisfactory because as a business owner, I'm going to have to service my oven from time to time.  I won't have a business without a working oven, so does that mean everything is "business logic"?  We know the answer is no.  

If "business logic" is not a good differentiating factor on how we should write utilities, what is?  Strong utilities are ones where the implementation is obvious, constant, and mostly unquestioned.  They are highly powerful but focused on making the tangential problems and red tape disappear.  They are important in their own rights but unimportant in the task at hand.  Therefore, if either your application or another could theoretically customize it, then it should always be passed into your utility (see section below about default values).  A best in class example of utilities are Guava's sets:
```
Set result = Sets.intersection(Set.of(1, 2, 3), Set.of(2, 4, 6));
```
Without these utilities, the code could be 40 lines and it would get in the way of what we are trying to do.  These utilities stay orthogonal to the work you are trying to get done, making your job simpler, and let you focus on your "secret sauce".

## Pure functions, no Swiss Army knives

I'm willing to wager that no kitchen regularly uses Swiss army knives, except maybe in Swiss army kitchens (I've never been in one!) -- but even then I'd doubt it.  While Swiss army knives are good in an emergency, they aren't anyone's recommended tool... for anything in particular.  You want the same out of your software utilities:  they should do a singular thing and do it well.  And just like you don't want your blender to start cooking your food, you don't want your utilities to have any unwanted side effects.  Both problems share the same root: the utility is doing something the caller didn't ask for.  No modifying inputs, you naughty developers!

## Default parameters, the temperamental oven

Recipes never say "Bake for the standard amount of time at the standard temperature".  This is because there is no standard.  Similarly, ovens don't choose their temperatures at random.  Every pizza produced by that recipe would be vastly different... and probably unpalatable.  Therefore, you should avoid default parameters in your own utilities.  Remembering that cooking a pizza requires roughly 425F might be manageable while you're cooking a pizza every day but after coming back a year later, suddenly "common knowledge" isn't so common.  Make your recipes timeless.

## Takeaways
Focus on charter when determining what should exist within a function/application's code:
1. Your top-level application should be focused on its charter and should define all variables that are fundamental to its success.  These are the application's "secret sauce"
1. Your application should defer all unimportant details into utilities.  Those utilities must:
   - Be obvious on what work they are doing (no surprises)
   - Remove the boilerplate code and details that should never change across applications
1. This process is recursive:  utilities may rely on other utilities to do the boilerplate code that are not important
1. Utility parameters should never have default values, unless they are a kind of boilerplate code that clients never will care about

---

That's all we have time for today.  In [Part 4]({{ site.baseurl }}{% post_url 2026-06-12-SoftwareRecipes-part4 %}), I hope you're ready to open a restaurant!  Thanks for joining us in the Software Kitchen — see you next time!
