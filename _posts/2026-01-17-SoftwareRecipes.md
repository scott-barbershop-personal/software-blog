---
title: "Structuring software as if they were recipes"
date: 2026-01-17
---

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/recipe.png?raw=true)

Recipes are universal communication structures used by many cultures, expressed in a variety of languages, across many generations.  When exchanged between individuals, recipes may not be easy to follow but they generally are easy to understand, at a high level even from a quick glance.  However, this usually isn't true for software.  Why is that?  The blog attempts to distill the secrets about recipes and how we can apply them to make software easier to understand.

## Data, the ingredient shopping list

One of the most useful things about a recipe is that it tells you everything that is needed in advance.  This is useful because it is a fairly heavyweight operation going to the grocery store to pick up new supplies.  If you do that fetching lazily, you may find yourself going to the grocery store multiple times for individual products.  However, this is exactly how most computer programs are written (with data accesses multiple layers deep within the call hierarchy):
```java
IngredientStore ingredientStore = new FallbackCache(cubbord, groceryStore); // we try the cubbord first but go to the grocery store, if necessary
Dough dough = new Dough(ingredientStore);
Sauce sauce = new Sauce(ingredientStore);
dough.knead(); // Note: this looks harmless enough since we don’t take in a groceryStore object but beware below!
…
public static class Dough {
    public Dough(IngredientStore ingredientStore) {
        this.ingredientStore = ingredientStore;
        mix(
            this.ingredientStore.fetch(Flour.class, 2, CUPS), // first trip to the store.  This may be expected
            this.ingredientStore.fetch(Yeast.class, 1, PACKAGE),  // oops unexpected second trip to the store
            this.ingredientStore.fetch(Salt.class, 1.5, TEASPOONS));  // oops unexpected third trip to the store
        …
    }

    public void knead() {
        // ooh, sneaky:  Since we are a member class of Dough, we have hidden access to the groceryStore here!
        Flour flourForTable = this.ingredientStore.fetch(Flour.class, .5, CUPS);  // oops REALLY unexpected fourth trip to the store (who can 
                                                                                  // blame us for not knowing we needed to buy more the first time)!
        …
    }
}
```
Let’s say that we want to add a budget to this code, since we’re tight on money.  First off, implementing budget is a pretty difficult task because we can’t do it in a single spot.  No worry, you say!  We can implement it within the IngredientStore… However, now we have even more hidden state in our system and it has some very nasty side effects:  you may find out only after the fifth trip to the store, you don’t have enough money to buy the most important toppings at the end of the recipe (especially since some of our budget is going to gasoline to go to/from the store many times).  Even worse, by the time that we realize that we’re out of money, we’ve already used all our money and ingredients on a recipe we can’t finish.  We’re going to have to be very clever on how to make a meal with a half completed dough.  After all, our family is hungry and we’ve promised to make a meal of some kind for them.

Instead, consider how much cleaner the solution is with all of the data access taken care of up front (maybe not perfect… but better than before):
```java
Flour flourForDough = new Ingredient(Flour.class, 2, CUPS);
Flour flourForKneading = new Ingredient(Flour.class, .5, CUPS);
List<Ingredient> totalIngredientsRequired = List.of(
    flourForDough,
    flourForKneading, 
    …);
List<Ingredient> missingIngredients = ... // we’ll talk about this later.  Basically, we need to know how much we need to buy from the store
if (!missingIngredients.isEmpty()) {
    groceryStore.conditionallyBuy( // either buy them all or buy none and throw
            missingIngredients,
            new Currency(100.00, USD)); // our budget
}
// we’re ready to start the recipe with all of our ingredients required
…
Dough dough = new Dough()
dough.mix(flourForDough, …);
dough.knead(flourForKneading, …);
```
Here, we do our best to make sure we only have to make a single successful trip to the store so that when we get back to the kitchen, we can focus on cooking.  This same rule applies when programming.  Your application code should have 3 clear, separate sections:
1. Fetching all the data necessary to do your work
1. Doing your work
1. Sending the output of your work

#1 and #3 heavyweight operations which we'll call "stateful operations" because they usually focus on transferring state in and out of the application (e.g. via services, databases, hard disks).  We'll call #2 "stateless" since it is focused on tranformations on that state.  There are some pretty significant differences between stateful and stateless work:
1. Authority:  Stateful operations leverage external authorities (e.g. using a DB schema or a external service's API); whereas stateless work is the secret sauce of the application and uses its own authoritative mechanisms.
1. Bottlenecks:  Stateful operations tend to be I/O bound (either network or disk); whereas stateless operations tend to be CPU bound.  
1. Intent:  Stateful operations are usually simple in concept (e.g. gets/puts);  whereas stateless operations usually have complex algorithms
1. Exception handling:  Stateful operations usually require complex handshakes (e.g. distributed transactions with partial commits); whereas stateless operations are usually fairly straightforward in and of themselves.

For each and every one of these differences, there are significant gains by keeping them colocated.  For example, as we saw with the grocery store, we can batch calls, parallelize, and optimize payloads.

## Utility methods, the cooking verbs

## Metrics, the serving portions
