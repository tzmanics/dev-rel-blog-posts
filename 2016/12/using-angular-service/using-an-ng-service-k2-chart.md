# Using an Angular Service to Deliver Data to Kendo UI Chart Visualizations
In my [blog post on creating visualizations](http://www.telerik.com/blogs/visualizing-data-on-the-web-with-kendo-ui-for-angular-2)
we took the quick and dirty route to get data into the chart component by
just adding an array of data into the main `app.component.ts` file. In this
post we'll modularize our application by separating out the data into its own file. Then we'll create a single service, a class or function that performs one task, that serves up the data to where it is needed in our application.

You can keep track of the changes in the code by looking at the corresponding
commits in [this project's repo](https://github.com/tzmanics/charts_data-binding). Each
commit will be linked at the end of the section of code changes.

Okay, let's jump in!

## Data in a Separate File
Our main app component should not have to define the data we're using and
there may come a time when we have more than one chart that uses this data.
These are two great reasons to pop this data out into its own file. We will
also make a service that provides the data wherever its needed.

*We're going to make a `data` directory to hold our data files but feel free to
leave the file in the `app` directory or create your own file structure. Just
remember to keep it easy to understand in case someone else ever uses
your code.*

Since we're moving the population data into its own file, let's first create
the file that holds the class for the model of the population data. This class
can then be referenced in multiple files, wherever we use `populationData`.

```ts
// app/data/data-point.ts

export class DataPoint {
  state: string;
  population: number;
}
```

In the `data` directory, create a new file called `data-point.ts` and export
the `DataPoint` class. We are basically taking the `Interface Model` section
out of the `app.component.ts` file and putting it into the new `DataPoint`
class.

Now create a file, cut the 'populationData' array out of the `app.component.ts`
file and paste it into the new file (`population-data.ts`).

```ts
// app/data/population-data.ts

import { DataPoint } from './data-point.ts';

export const POPULATION_DATA: DataPoint[] = [
  { state: 'Alaska', population: 738432 },
  { state: 'Arizona', population: 6828065 },
  ...
  { state: 'West Virginia', population: 1844128 },
  { state: 'Wisconsin', population: 5771337 },
  { state: 'Wyoming', population: 586107 },
  { state: 'Puerto Rico', population: 3474182 }
];
```

At the top of the file, import the `DataPoint` class. After we paste the
`populationData` we will tell the file to `export` the `populationData`. Delete
`private` and add `const` then change `Model` to our custom `DataPoint`. We're
so fancy! 💅🏻

We have our data so we can now create the service that feeds the `populationData` to any components that utilize it. Create a file named `data.service.ts` in the data directory.

```ts
// app/data/data.service.ts

import { Injectable } from '@angular/core';

import { POPULATION_DATA } from './population-data';
import { DataPoint } from './data-point';

@Injectable()

export class DataService {
  getPopulationData(): DataPoint[] {
    return POPULATION_DATA;
  }
}
```

We need to import both the `DataPoint` class and the `POPULATION_DATA`. The
`DataPoint` class is used to set the class for the information we're getting
from `getPopulationData` and we need to send `POPULATION_DATA` through to the
necessary components. Create the `getPopulationData` so that this function is
exposed when you import `DataService`. This is a simple function that returns
the array of `DataPoint`s.

Angular [recommends adding the `@Injectable`
decorator](https://angular.io/docs/ts/latest/guide/dependency-injection.html#!#injectable)
to every service class even if doesn't have dependencies. An Injectable
decorator signifies to TypeScript that there may be metadata about the service.
Angular may need that metdata to inject other dependencies into this service.
We don't need it now but it's a good habit to always import and add
`Injectable` to future proof and create consistency. Since we know when to heed
good advice, we import `Injectable` on the top line of the service and apply
the `@Injectable` decorator on line 8.

Our data is all set up, we can now edit the `app.component` file to utilize
our service.

```ts
// app/app.component.ts (top)

import { Component, OnInit } from '@angular/core';

import { DataService } from './data/data.service';
import { DataPoint } from './data/data-point';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  template: `
    <h1>2015 US Census Data Visualized</h1>
    <div class="visualization">
      <kendo-chart style="height: 1000px">
        <kendo-chart-title
...
```

At the top of the file import `DataService` and `DataPoint`. Above
this we will need to import the `OnInit` [Lifecycle
Hook](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html). We use
this hook so that Angular will call the `getPopulation` function when the `AppCompnent` implements the hook. We do this to keep the complex logic of calling this function out of the constructor (we will get to this part very soon).

```ts
// app/app.compnent.ts (bottom)

      </kendo-chart>
    </div>
    <div class="visualization"></div>
    <div class="visualization"></div>
      `,
  providers: [DataService]
})

export class AppComponent implements OnInit {
  populationData: DataPoint[];

  constructor(private dataService: DataService) {}

  getPopulationData(): void {
    this.populationData = this.dataService.getPopulationData();
  }

  ngOnInit(): void {
    this.getPopulationData();
  }
}
```

In order for Angular's injector to understand how to create a `DataService` we
have register it by adding it to the `providers` property at the bottom of the
Component metdata. This is telling Angular to make a fresh instance of the
service each time the `AppComponent` is created.To use the `OnInit` lifecycle hook we imported, we make our `AppComponent`
implement it.

Inside the `AppComponent` we have to expose the population data
for the charts. First we initialize `populationData`, then we make a
constructor define and identify the data service. So, now just as we have a new
service instance _created_ when Angular creates a new `AppComponent`, Angular
will _supply_ an instance as well.

Next add the `getPopulationData` function using the data service to assign the
data to our `populationData`. Lastly, we utilize our `OnInit` lifecycle hook to
call the `getPopulationData` function. That's it!

This is a good time to run `ng serve` to make sure everything is running just
as it was before we separated out the data. You can compare your code changes
to [the commit for this
section](https://github.com/tzmanics/charts_data-binding/commit/6bfb1135f66893cca0382627bdbdc973f31654a3).
This is what the the file structure looks like now:

```
app
  │  app.component.html
  |  app.component.scss
  |  app.component.spec.ts
  |  app.component.ts
  |  app.module.ts
  |  index.ts
  └──data
  |  |  data-point.ts
  |  |  data.service.ts
  |  |  population-data.ts
```

### Move the Template Data ###
🕺 Just for kicks 💃🏻 (and readability) let's take a minute to move the template logic to its own file. We just need to delete the contents of `app.component.html` and replace it with everything inside `template: \`...\``
in our `app.component.ts` file. Inside the `@Component` in the
`app.component.ts` file switch `template` to `templateUrl` and point it to
our `app.component.html` file. Look how clean our component file is now!

```ts
// app/app.component.ts

import { Component, OnInit } from '@angular/core';

import { DataService } from './data/data.service';
import { DataPoint } from './data/data-point';

@Component({
  selector: 'app-root',
  styleUrls: ['./app.component.scss'],
  templateUrl: 'app.component.html',
  providers: [DataService]
})

export class AppComponent implements OnInit {
  populationData: DataPoint[];

  constructor(private dataService: DataService) {}

  getPopulationData(): void {
    this.populationData = this.dataService.getPopulationData();
  }

  ngOnInit(): void {
    this.getPopulationData();
  }
}
```

The commit for this change in code can be found
[here](https://github.com/tzmanics/charts_data-binding/commit/0e9fe37abb157395c7b467c15822f585edfc2641).

We have moved a lot of the noise out of our main component file and modularized the application. Not only is this great organization for your own sanity but it helps future coders who want to learn about your code.

Let me know in the comments if you have any questions. You can also let me know what other topics you would like me to cover. Always happy to help! Have fun coding 😘
