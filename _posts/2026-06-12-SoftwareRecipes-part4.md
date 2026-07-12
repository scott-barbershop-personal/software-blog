---
title: "Structuring software as if they were recipes: Part 4"
date: 2026-06-12
---

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/recipe.png?raw=true)

If you promised your family to personally make a pizza and then you called Domino’s to make it and deliver it for you, chances are your family would be disappointed (unless you are a horrible cook, hahaha).  However, on the other hand, if a family member was super busy and wanted you to order a Domino’s pizza to help them, they would be very happy.  Furthermore, if you had promised to call Domino’s but then started making the pizza yourself, your family member may get upset with you.  ¨Our friends are arriving in 10 minutes and you haven’t even started cooking!  The kitchen is a mess and I needed your help to take care of the kids…¨.  The fact the pizza is from Domino's or not is not the problem.  The difference between these situations is charter and responsibilities.  In the first, you are declaring yourself as chef; while in the other you are declaring yourself as a proxy or facilitator.  Only in the second case are you fulfilling your declared responsibilities;  in the first and third you drastically changed what you promised you were going to do.

Most enterprise software (and organization structure too) eventually morphs into painful "middle-manager" layers, which is similar to the first example explained above.  This instance will talk about why these are poorly abstracted software layers.

## The dreaded middle-manager layers

As we start learning multiple recipes, we start noticing a lot of commonality between them.  A Sicilian pizza and a Neopolitan pizza both need dough, tomato sauce, and toppings.  We might be tempted to so this:
```
@API
public makeSicilianStylePizzia() {
    return new PizzaManager().makePizza(Type.Sicilian);
}

@API
public makeNeopolitanStylePizza() {
    return new PizzaManager().makePizza(Type.Neopolitan);
}

public class PizzaManager {
    public Pizza makePizza(Type pizzaType) {
        // do common setup
        if (pizzaType == Sicilian) {
            ....
        } else if (pizzaType == Neopolitan) {
            ...
        } else {
            throw new Exception("Unexpected code path: pizzaType("+ pizzaType +"));
        }
        // do common functionality
    }
}
```

There are a number of reasons this pattern falls over at scale.  The first is that the controller API layer has no charter.  Rather, it has charter but then defers all of it to another (just like when you said you'd make pizza but called Domino's).  The second is that the PizzaManager becomes a middle manager:  if either of the controller APIs want to add specializations, such as styles of pizza with different combinations of toppings, those have to get forwarded to the PizzaManager.  This forwarding and the managing of all the commonalities across pizzas becomes problematic.  This may start out simple, with a simple if/else structure like the one above but multiply every time each of the individual pizza recipes needs customization.  Once developers learn this pattern, then it itself multiples, as the PizzaManager has other managers, CheeseManager, DoughManager, and others that all expand the pattern further.

When this happens, it is harder and harder to understand what a Sicilian pizza looks like (you have to deep dive into all the code to understand anything).  This makes testing and quality control harder.  Furthermore, changing any mid-level manager is difficult because it is unclear how it could affect all the pizza recipes (e.g. is it ok to freeze the dough?).  "But!", you say, "I know how to solve this:  Class encapsulation cleans up these problems!"  The short answer is that is slightly better but misses the bigger point:  the charter of the API is already owns that charter.

Generally, you want your code to focus on utilities and simple APIs and minimize the number of middle-managers:

![Recipe](https://github.com/scott-barbershop-personal/software-blog/blob/main/images/MiddleManagerGraphs.png?raw=true)

## Takeaways

Middle managers are death for careers and software designs.  Avoid them whenever possible.  Instead, focus on very strong utilities that make your recipes easy to manage and keep all of your business logic at the top layer.
