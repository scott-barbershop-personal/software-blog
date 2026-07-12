---
title: "Structuring software as if they were recipes: Part 3"
date: 2026-06-12
---

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/recipe.png?raw=true)

There is a saying:  A good chef is only as good as his best knife.  The same is true in software.  Generally, you can tell the quality of a software application by looking at their utilities.  Most utilities are poorly structured because developers view them as secondary citizens.  In this chapter, we'll talk about how inverting this mental model will make your application code orders of magnitude simpler and bug-free.

## Utility methods, the cooking verbs
Recipes don't tell you how to do everything.  They generally don't tell you what sort of kitchen tools you should have.  They are not cooking classes.  They expect you have some knowledge of basic cooking verbs:  dice, simmer, sear, reduce, blanch, poach.  However, once you learn the verbs, it is immediately clear what is needed.  For example, a chef knows the difference between a slow simmer and a rolling boil.   These terms apply a set of well-defined steps that make your recipe short and compact but still let you know exactly what needs to be done.  Utilities provide the same benefits within software.  However, well-written utilities are hard to find.  For example, within software, it is very common to see a utility like this:
```
Dough dough = new Dough();
...
bakePizza(dough);
```
If recipes were written like this, chances are you'd burn your house down.  Why?  Because it just happens to be that this specific bake function was programmed to cook specialty [Neapolitan pizzas at 900 degrees Fahrenheit (480 degrees Celsius) for 1.5 minutes](https://www.tastingtable.com/1494921/neapolitan-pie-anthony-mangieri-una-pizza-napoletana-interview/).  If I leave for 15 minutes, like my Siciliana pizza recipe recommended, there would be big problems.  Software engineers often create utilities like this because they see duplication and they want to minimize it.  As they say, "The road to hell is paved in good intentions."  Even if 100 recipes all used the oven to cook something at 425F, they'd never put that information into the bake utility... and neither should you.  Additionally, we shouldn't have to change all recipes just because we discovered that it is better to cook our pizza at 415F instead of 425F.

Recipes know how to balance what information is important and what can be deferred.  In this case, the crucial "business logic", which is oven temperature, time to bake, and success criteria are always defined at the topmost level of the recipe:

```
bake(dough, new Temperature(425, Scale.Fahrenheit), SuccessCondition.OR(TimeUnit.MINUTES.toMillis(15), Color.LightBrown));
```

This code is super clear because the important information is passed in explicitly and everything else is left out.  For example, the function doesn't tell you that it requires an oven or even that you must plug in the oven first; those are obvious from the word "bake".  The recipe also doesn't tell you what to do if you are having technical problems with your oven.  Software engineers like to say that in order to write good software, you should focus on what is "business logic" and what isn't.  However, I always find this definition vague and unsatisfactory because as a business owner, I'm going to have to service my oven from time to time.  I won't have a business without a working oven, so does that mean everything is "business logic"?  We know the answer is no.  

If "business logic" is not a good differentiating factor on how we should write utilities, what is?  Strong utilities are ones where the implementation is obvious, constant, and mostly unquestioned.  They are highly leveragable but are not related to what you are trying to do.  It is all the logic that your recipe doesn't care about and views as "heavy-lifting" boilerplate code.  Therefore, if either your application or another could theoretically customize it, then it should always be passed into your utility (no using default parameters either).  A best in class example of utilities are Guava's sets:
```
Set result = Sets.intersection(Set.of(1, 2, 3), Set.of(2, 4, 6));
```

This is such a magical piece of code in a very compact space, all thanks to the 2 utilities:
* Set.of - Simplifies the creation of a set of data (using the appropriate subclass)
* Sets.intersection - Creates a pointer over other data structures, so data isn't duplicated

Without these utilities, the code could be 40 lines and it would get in the way of what we are trying to do.  These utilities stay orthogonal to the work you are trying to get done, making your job simpler, and let you focus on your "secret sauce".

One last note:  You should avoid default parameters, especially those chosen for historical reasons, within your utilities.  Developers are highly biased on how they think about their application today and let that confuse others later when those defaults no longer make sense.  Developers are lazy (myself included) and we don't want to update all the clients when a previously iron-clad certainty because something that can vary in the future.  Often times, they let bias and laziness prevent them from inlining those defaults into their clients but that is just as dangerous as having the values defined within the body of the utility.  Clients should never have to look at implementations to understand what is happening and that includes default values.

## Takeaways
Focus on charter when determining what should exist within a function/application's code:
1. Your top-level application should be focused on its charter and should define all variables that are fundamental to its success.  These are the application's "secret sauce"
1. Your application should defer all unimportant details into utilities.  Those utilities must:
   - Be obvious on what work they are doing (no surprises)
   - Remove the boilerplate code and details that should never change across applications
1. This process is recursive:  utilities may rely on other utilities to do the boilerplate code that are not important
1. Utility parameters should never have default values, unless they are a kind of boilerplate code that clients never will care about

