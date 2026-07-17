---
title: "Structuring software as if they were recipes - Part 4"
date: 2026-06-12
---

![Recipe]({{ site.baseurl }}/images/recipe.png)

Welcome back to the Software Kitchen. Today we're taking our pizza making skills to the big leagues by opening our own pizza restaurant.  And then, if we're not careful, watching it become a victim of its own success. Let's get started!

## Middle managers, the bureaucratic pizza makers

You've just opened a small pizza restaurant. Two pizzas on the menu: Sicilian and Neapolitan. Business is good, customers are happy, and the kitchen is simple. Life is beautiful.  

Then a food blogger shows up. The review goes viral and suddenly you have more customers than you can handle.  You need to scale and quick.  You think to yourself: "I need help.  I need a Pizza Manager!"  Once hired, the Pizza Manager promptly hires a Dough Manager, a Kneading Manager, and an Oven Manager.  Each has their own processes and opinions but they all need to know the full menu, as there are subtle differences with how each style impacts each of their spaces.  Now your kitchen looks like this:
```
@API
public Pizza makeSicilianPizza(PizzaParameters params) {
    return new PizzaManager().makePizza(Type.Sicilian, params);
}

@API
public Pizza makeNeapolitanPizza(PizzaParameters params) {
    return new PizzaManager().makePizza(Type.Neapolitan, params);
}

public class PizzaManager {
    public Pizza makePizza(Type pizzaType, PizzaParameters params) {
        ...
        Dough dough = doughManager.prepareDough(pizzaType, params);
        dough = kneadingManager.kneadDough(pizzaType, params, dough);
        return ovenManager.bake(pizzaType, params, dough);
    }
}
```
At first this seems fine... until you start getting customers with dietary requirements:  no lactose, no gluten.  The problem is that each of these different ingredients affects each of the steps of the process.  Gluten-free bread doesn't cook at the same temperature or time as the others.  The Oven Manager changes the temperature for gluten-free, doesn't tell the Dough Manager, so the dough is wrong, and now the Kneading Manager is blaming the Dough Manager for sending bad dough.  You try to gain control by walking through how your managers create a Sicilian pizza.  You start by looking at the dough manager's space:
```
public prepareDough(PizzaType pizzaType, PizzaParameters params) {
      Flour flour;
      if (params.isGlutenFree()) {
          flour = new RiceFlour(params.size == Medium ? 2 : 3, CUPS);
      } else if (pizzaType == NEAPOLITAN) {
          flour = new DoubleZeroFlour(params.size == Medium ? 2 : 3, CUPS);
      } else {
          flour = new BreadFlour(params.size == Medium ? 2.5 : 3.5, CUPS);
      }
      Milk milk;
      if (params.isGlutenFree() && pizzaType == NEAPOLITAN) {
          milk = new Milk(params.size == Medium ? 1.7 : 2.2, CUPS);  // GF Neapolitan needs extra hydration
      } else if (params.isGlutenFree()) {
          milk = new Milk(params.size == Medium ? 1.5 : 1.9, CUPS);
      } else if (pizzaType == NEAPOLITAN) {
          milk = new Milk(params.size == Medium ? 0.65 : 0.85, CUPS);
      } else {
          milk = new Milk(params.size == Medium ? 0.70 : 0.80, CUPS);
      }
      if (params.isLactoseFree()) {
          milk = new Water(milk.quantity()*.75);
      }

      // Does lactose-free affect the dough? Sicilian uses olive oil but
      // sometimes butter is added for richness... let's be safe:
      if (params.isLactoseFree() && pizzaType == SICILIAN) {
          return new Dough(flour, milk, OIL_ONLY);
      }
      return new Dough(flour, milk, OIL_AND_BUTTER);
}
```
The other managers' spaces echo the same chaos.  Your eyes water.  You've been making pizzas for years but none of this makes any sense to you.  As the managers are arguing with each other, they have no idea what a Sicilian pizza even looks like!

This is the core problem with middle managers in software and kitchens alike: they don't make decisions, they route them.  The API layer above has no real charter — it's supposed to make a Sicilian pizza but immediately hands the job to someone else (sound familiar from Part 1?).  The PizzaManager becomes the single chokepoint for every pizza decision, growing more complex with every new type, topping, and customization that gets forwarded to it.  Once developers learn this pattern, it multiplies: PizzaManager spawns CheeseManager, DoughManager, SauceManager, each with their own expanding fiefdoms:  

![Middle Manager Graph]({{ site.baseurl }}/images/MiddleManagerGraphs.png)

The fix is simple in concept: give each pizza its own station. A chef who owns their recipe from start to finish, using strong shared utilities (the cooking verbs from Part 3) for the common heavy lifting:
```
@API
public Pizza makeSicilianPizza(PizzaParameters params) {
    PizzaIngredientAmounts amounts = SicilianPizza.getAmounts(params.size);
    Flour flourForDough = amounts.getFlourForDough();
    Flour flourForKneading = amounts.getFlourForKneading();
    // ... fetch ingredients (see Part 2)
    Dough dough = new Dough();
    dough.mix(flourForDough, water, yeast);
    dough.knead(flourForKneading, …);
    pan = press(dough, pan);
    wait(1, Hour); // let the yeast rise, this is what gives Sicilian its thick, airy crust
    pan = pour(pan, makeTomatoSauce(tomatoes, garlic, basil));
    pan = add(pan, amounts.getToppings());
    return bake(pan, new Temperature(425, Fahrenheit), Duration.ofMinutes(15));
}

@API
public Pizza makeNeapolitanPizza(PizzaParameters params) {
    PizzaIngredientAmounts amounts = NeapolitanPizza.getAmounts(params.size);
    Flour flourForDough = amounts.getFlourForDough();
    // ... fetch ingredients (see Part 2)
    Dough dough = knead(flourForDough, water, yeast);
    dough.stretchByHand(); // never rolled or pressed
    dough = pour(dough, makeTomatoSauce(tomatoes, basil));
    dough = add(dough, amounts.getToppings());
    // dough goes directly into the oven
    return bake(dough, new Temperature(900, Fahrenheit), Duration.ofMinutes(2));
}
```
Notice how easy it is to see the differences; and what differences they are... even though they are both pizzas!  Instead of managers spread across vague, overlapping territories, we have a helping config factory that resolves all the quantities upfront.  This is like a recipe that has a table for different measurements depending on the size... or even better, like one of those professional TV chefs, who have all their ingredients already measured and put into individual bowls ready for use.  All these side details let us focus on making the perfect pizza.  

## Takeaways

Keep your business logic at the top layer and your utilities sharp at the bottom. Middle-manager classes that group by shared implementation rather than by purpose add complexity without adding value. If you find yourself writing a class whose job is to route decisions to other classes, that's your cue to rethink the organization — not add another manager.  If you need assistants, keep them narrow -- one job/responsibility, not the whole menu or restaurant.

---

And that's the final dish!  We've come a long way in the Software Kitchen.  We went from figuring out what we were going to cook to opening our own restaurant.  Who knows what we'll have in store next.  Thanks for joining us in the Software Kitchen — bon appétit!
