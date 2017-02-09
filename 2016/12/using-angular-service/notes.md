interpolation (a kind of one-way data binding)
We can both display and change the hero’s name after adding a two-way data
binding to the <input> element using the built-in ngModel directive.

The ngModel directive also propagates changes to every other binding of the
hero.name.


Feeding data to visualizations...nom nom nom

In my last post we walked through creating a visualization of US Census data. To keep it simple we just left the data in the app component. This is a total no-no because the component should not be having to deal with our data. Today we'll walkthrough the different ways to feed data to our Kendo UI Charts using angular services,
