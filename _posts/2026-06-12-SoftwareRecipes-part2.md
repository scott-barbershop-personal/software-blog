---
title: "Structuring software as if they were recipes - Part 2"
date: 2026-06-12
---

![Recipe]({{ site.baseurl }}/images/recipe.png)

Welcome back to Software Kitchen.  Today, in part 2, we're covering: where you get your ingredients from matters.  So, let's make a recipe today that only requires a single trip to the grocery store... which can be a lifesaver for those who own gas-guzzling cars.  Let's get started!

## Data, the ingredient shopping list
"Honey, can you drive to the store for me and get me a pinch of salt?  One pinch... no more, no less!"  If you were confused, I don't blame you.  However, this is exactly how most computer programs are written... they just don't say it as clearly.  Let's look at some code:
```java
IngredientStore ingredientStore = new FallbackCache(cupboard, groceryStore);
Dough dough = new Dough(ingredientStore);
dough.mix();
dough.knead();
...
```
This code looks pretty harmless.  However, most dangerous code usually does...  that's what makes it dangerous!  The ingredient store is a way of fetching ingredients.  It follows a typical caching strategy where we first fetch ingredients from the cupboard, if they are available;  if they aren't, we'll run to the grocery store.  However, that's not the dangerous part.  Dough's implementation is where everything goes wrong:
```java
public static class Dough {
    public Dough(IngredientStore ingredientStore) {
        this.ingredientStore = ingredientStore;
    }

    public void mix() {
        Flour flour = this.ingredientStore.fetch(Flour.class, 2, CUPS);
        Yeast yeast = this.ingredientStore.fetch(Yeast.class, 1, PACKAGE);
        Salt salt = this.ingredientStore.fetch(Salt.class, 1.5, TEASPOONS));
        ...
    }

    public void knead() {
        Flour flourForTable = this.ingredientStore.fetch(Flour.class, .5, CUPS);
        …
    }
}
```
At first glance, this code doesn't look dangerous either but it is.  If our cupboards are empty, we've made 4 separate trips to the grocery store!  The trip inside the ```knead()``` function is particularly surprising because its access to the ingredientStore is hidden via member access and it's asking for something that is small and tangential to the task it is trying to accomplish.  This kind of design could lead to us running to the store just because we needed a pinch of salt!!!   Not many people looking at the top level function would expect a trip to the store happening in these ancillary functions.  The danger here comes from access patterns being handed off between components.  Writing good code is all about being as straightforward and unsurprising as possible and this setup has completely failed us here.

This code isn't just dangerous from an operational perspective.  It is also dangerous because it facilitates extra complexity for each new feature we want to add.  Let’s say that we want to add a budget restriction to this code, since we’re tight on money.  First off, implementing budget is a pretty difficult task because we can’t do it in a single spot.  "Don't worry", you say!  "We can implement it within the IngredientStore…" However, now we have even more hidden state in our system and it has some very nasty side effects:  you may find out only after the fifth trip to the store, you don’t have enough money to buy the most important toppings at the end of the recipe (especially since some of our budget is going to gasoline to go to/from the store many times).  Even worse, by the time that we realize that we’re out of money, we’ve already started using all our money and ingredients on a recipe we can’t finish.  We’re going to have to be very clever on how to make a meal with a half completed dough.  After all, our family is hungry and we’ve promised to make a pizza.

How can we improve?  Let's go back to recipes.  Recipes are great because they tell us everything we need in advance.  I can see everything that is necessary, check what I have on stock, and go to the grocery store in just one trip to get everything I'm missing.  Let's change our software to work like we cook, by getting all the ingredients up front:
```java
Flour flourForDough = new Ingredient(Flour.class, 2, CUPS);
Flour flourForKneading = new Ingredient(Flour.class, .5, CUPS);
List<Ingredient> totalIngredientsRequired = List.of(
    flourForDough,
    flourForKneading,
    …);
List<Ingredient> lackingIngredients = subtract(totalIngredientsRequired, cupboard.getIngredients())
if (!lackingIngredients.isEmpty()) {
    groceryStore.conditionallyBuy( // either buy them all or buy none and throw
            lackingIngredients,
            new Currency(100.00, USD)); // our budget
}
// we’re ready to start the recipe with all of our ingredients required
Dough dough = new Dough()
dough.mix(flourForDough, …);
dough.knead(flourForKneading, …);
```
We'll only buy everything if we have enough money (using a single transaction).  Now, once we get back to the kitchen, we can focus on cooking.  We are confident that we won't be surprised with any trips to the grocery store because the Dough class doesn't have access to the groceryStore object.  This is a nice simplification since the Dough class can't go to the grocery store on its own (even my strongest and worst doughs couldn't achieve that!).

## Stateless operations, kneading your dough

Ever burned something in the oven and wanted to press the undo button?  Anytime I put something in the oven I get nervous -- there's no going back at that point.  Kneading is just the opposite: mess it up, add a little flour or water, try again.  No consequences.  In software, we call that stateless work, and the oven represents stateful work (more on that in a moment).

Stateless work is where your secret sauce lives:  the complex logic, the algorithms, the thing that makes your recipe yours.  And because you're working purely with what's already on the counter (with no calls to the grocery store and nothing in the oven yet), you can experiment freely until you get it just right.  Keep working until it has the right form, texture, and taste.  You want as much of your software to be in this bucket as possible.  This leaves the bulk of our software light, just like your ideal dough.  Think of it as getting all your trays prepped before you open the oven door. Shape the dough, arrange the toppings, line them all up, and get everything fully ready for the final steps.  Mixing prepping and baking into a single step is how you end up with race conditions, partial writes, and a very confused kitchen.

## Stateful operations, baking the pizza

Once everything is prepped and ready, you can focus entirely on the oven: temperature, timing, not burning the house down.  The oven is where you commit.  You set the temperature, slide in the pizza, wait, and if you are like me, you pray that all the previous steps were done correctly.  In software, these are your stateful operations:  database calls, external service calls, writes to disk.  Similar to running to the store at the beginning, you want to do these all in bulk as a single operation.  It is much easier to bake a bunch of pizzas if all we have to do is watch the oven.  Unlike kneading, you can't just try again if this goes sideways.  Good recipes know this; they tell you exactly what "done" looks like: a color, a temperature, a smell... Not just "cook for 15 minutes and hope."  Your software should do the same: clear success conditions, careful error handling, and extra monitoring.  Have your protections baked in from the start. 

One last thing: if your recipe requires pulling the pizza out halfway through, running a separate errand, and putting it back in, you probably need to rethink whether you are following one recipe or two.  Same in software: if your stateful operations can't stay at the edges because of mid-operation dependencies, you likely have a workflow made of two separate operations, each with their own grocery run and their own oven time.

## Takeaways
Top level operations should have 3 clear, separate sections:
1. Fetching all the data necessary to do its work
1. Doing its work
1. Committing the output of its work

---

That's all we have time for today.  In [Part 3]({{ site.baseurl }}{% post_url 2026-06-12-SoftwareRecipes-part3 %}), I hope you have a good knife set!  Thanks for joining us in the Software Kitchen — see you next time!
