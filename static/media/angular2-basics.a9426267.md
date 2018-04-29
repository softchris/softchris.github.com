# Angular 4.x

Angular is component centric framework that is a complete rewrite of AngularJS. It uses Typescript as a default language to create apps in. It also utilises something called ES2015 imports to refer to external files or modules as they are also called.

In this post we will learn the following:

* bootstrapping and what the core parts are
* introductory knowledge on angular-cli
* components and modules
* services
* dealing with HTTP

## Scaffolding and Bootstrapping Angular 2

Because I want you to get going as soon as possible the recommended approach for bootstrapping is to scaffold a new project with @angular-cli.

To do that we need to ensure we have NodeJs installed. You can download it from here [NodeJs download](https://nodejs.org/en/download/)

After that is installed it is time to install the @angular-cli. This is done through the Node Package Manager, NPM that comes with your NodeJs install. Let's type the following to download @angular-cli

    npm install -g @angular-cli

Now it is time to scaffold our project. I.e create a stub for a project that we can keep adding things to. So let's now use the @angular-cli for this:

    ng new <project name>

To find your created project type the following:

    cd <project name>

To start the project and have it shown in a browser you type:

    ng serve

If you now navigate to localhost:4200 in your browser it will display 'app works'.

That was simple. Now let's talk about some important parts that makes up your application.

### The core parts

* main.ts
* app.module.ts
* app.component.ts

These files have different responsibilities.
The main.ts file is our bootstrapper the thing that bootstraps the application and points out our entry point. Let's look at the content of main.ts :

**main.ts**

```
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppModule);
```

The crucial part here is calling the function bootstrapModule() with a module as an argument. As you can see this points to AppModule which resides in the file app.module.ts, looking like this:

**app.module.ts**

    import { BrowserModule } from '@angular/platform-browser';

    import { AppComponent } from './app.component';

    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
      ],
      providers: [ ],
        bootstrap: [AppComponent]
      })
      export class AppModule { }

This file creates our root module that we use to bootstrap the entire application. They two keywords we need to keep track of right now are `bootstrap` and `declaration`.
The `bootstrap` keyword is an array expecting the component that will be the entry point for the application. As you can see it points to AppComponent. The `declarations` keyword contains an array. Here you put all the components the module should consist of. This is were you make components aware of each other. A component put in the declarations keyword can be used in the template of another component. A component needs to belong to exactly one module. Let's have a look at our entry point, the AppComponent:

**app.component.ts**

```
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.css']
  })
  export class AppComponent {
    title = 'app lives'
  }
```

Our component consists of two parts, the decorator @Component and the backing class AppComponent. By decorating the class Angular is able to make out what it is. As you can see the decorator takes an object that has different properties on it. The properties mentioned so far are:

* selector, what your component will be called if inside of a template
* template/templateUrl, this is where you define the HTML for the app. It's templating where you get to mix and match between HTML and data properties you want to display
* styles/styleUrls, this is where you define the CSS

## Building our app

So far we have introduced some bootstrapping, that is basically producing a 'hello world' of our app. To start adding more things to our app we need to understand what we are doing. The first thing we need to understand is that there are different types of binding that allows us to show data but also act when data is changed.

### Bindings

These are the bindings that exist:

    - interpolation
    - one way binding
    - event binding
    - two way binding

#### interpolation

Interpolation is about interpreting what is there and output it to the screen. For this we use the curly brackets {{}}. Inside of our curly brackets is an expression that is evaluated and then output to the screen. It can look like the following

```
{{movie.title}}
```

Here we try to render a variable movie that has a title property on it. Let us extend our app.component.ts class to ensure that the above code works:

```
//imports omitted

@Component({
  selector: 'app-root',
  template: `
    {{ movie.title }}
  `,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app lives';
  movie = { title : 'Star Wars' };
}
```

As you can see here we needed to add the movie property to our class. Which meant it was available for our template to render.

#### one-way, property binding

In this type of binding we can bind to attributes on an element. Imagine the following code:

```
<div [title]="titleProperty">
```

The component will therefore need the field titleProperty like so:

```
@Component({})
export class Component {
  titleProperty='title'
}
```

The square bracket [] is what creates the property binding and it needs to point to a property on your backing component class.

#### event binding

There are tons of events that an element can respond to. The one most people use is the click event. Use it like so:

```
<div (click)="save()">
```

This will point to a method save() on the backing class:

```
@Component({})
export class Component {
  save(){}
}
```

#### Two-way binding

The last is two-way binding, it's not really a two-way binding though, more of a shorthand that makes it look like it is a two-way binding. It is essentially an event binding and an attribute binding, looking like this :

```
<input [ngModel]="name" (ngModelChange)="name=$event">
```

Our attribute binding is bound to the name property and the event binding is bound to the event ngModelChange. We can write this in one motion though, like this:

```
<input [(ngModel)]="name"/>
```

Everytime you time in the input it will be changed both in the view and in the backing class.

### Core directives

Ok so we learned to extend our AppComponent class with a property. Now it's time to understand what core functionality we have at our disposal so we can add logic in the template. The most common core directives used for this is:

* \*ngIf
* \*ngFor

There are other ones like ngStyle, ngClass and ngSwitch but we will concentrate mainly on the two first mentioned as they are heavily used.

\*ngIf is used in the following way:

```
<div *ngIf="title"> title is set</div>
<div *ngIf="!title"> no title</div>
```

This directive investigates wether a property is true/false and renders a piece of HTML based on it.

\*ngFor is what used to be known as ng-repeat in AngularJS. It is used in the following way:

    <div *ngFor="let item of items">
    {{ item.title }}
    </div>

This produces a piece of repeating HTML. The backing component class would have to be extended with the items array for this to work, like so:

```
//imports omitted

@Component({
  selector: 'app-root',
  template: `
    <div *ngFor="let item of items">
    {{ item.title }}
    </div>
  `,
   styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app lives';
  movie = { title : 'Star Wars' };
  items = [{
    title : 'Star Wars IV'
  },
  {
    title : 'Star Wars V'
  },
  {
    title : 'Star Wars VI'
  }]
}
```

## Extending it with services

Ok so we learned a bit about bootstrapping and a little bit on core directives but all our data lives in the component. That's a lot of noise. It is better if it resides somewhere else, like a service. A service is just a class in a separate file that we tell Angular about. Let's first define our service:

**movie.service.ts**

```
export class MovieService {
  getMovies() {
    return [{
      title : 'Star Wars IV'
    },
    {
      title : 'Star Wars V'
    },
    {
      title : 'Star Wars VI'
    }];
  })
}
```

This is just a simple class. As you can see we have moved the data from the component into the service. Our component now should look like this:

**app.component.ts**

```
//imports omitted

@Component({
  selector: 'app-root',
  template: `
    <div *ngFor="let item of items">
    {{ item.title }}
    </div>
    `,
    styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app lives';
  movie = { title : 'Star Wars' };
  items = [];

  constructor(privat srv: Service) {
    this.items = srv.getMovies();
  }
}
```

However this doesn't work quite yet. We need to tell Angular that the service exists. We have provided a service instance to the components constructor above but we have not told Angular on how to find and construct an instance of the Service class. We can do that on component level and module level. We choose to tell the module about this service though :

**app.module.ts**

```
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';

import { Service } from './service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
  ],
  providers: [ Service ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Note how we added the Service to the providers array:

```
  providers : [ Service ]
```

## Create a new component

Ok so we learned how to move data from component into a service. But we feel that the app.component.ts shouldn't really be the one rendering movies. It is our root component. It would be better if a more dedicated component could do the job, a movies component, so let's create that:

**movies.component.ts**

```
@Component({
  selector : 'movies',
  template : `
    <div *ngFor="let movie of movies">
    {{ movie.title }}
    </div>
`
})
export class MoviesComponent {
  constructor(private srv:Service){
    this.movies = srv.getMovies();
  }
}
```

Which means we can clean up our app.component.ts to look like this:

**app.component.ts**

```
//imports omitted

@Component({
  selector: 'app-root',
  template: `
  <movies></movies>
  `,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app lives';
}
```

We are almost done. We need to tell the AppModule about this new component so it is actually allowed to place the movies component inside of the template for AppComponent, so let's do that:

```
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { MoviesComponent } from './movies.component'

import { Service } from './service';

@NgModule({
  declarations: [
    AppComponent, MoviesComponent
  ],
  imports: [
    BrowserModule,
  ],
  providers: [ Service ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

We added the import for our new component like so:

```
import { MoviesComponent } from './movies.component'
```

And we also added it to the declarations list like so:

```
declarations: [
	AppComponent, MoviesComponent
]
```

## Communicate over HTTP

So far we have been displaying static data that lived inside of a service. In a more realistic scenario that data would be accessible on an endpoint somewhere that we would be able to fetch using Ajax. Angular has a Http service just for this. To use it we need to add the HttpModule and simply inject the Http service where we want to use it. So let's start with the import:

**app.module.ts**

```
import { BrowserModule } from '@angular/platform-browser';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { MoviesComponent } from './movies.component'

import { Service } from './service';

@NgModule({
  declarations: [
    AppComponent, MoviesComponent
  ],
  imports: [
    BrowserModule,
    HttpModule
  ],
  providers: [ Service ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

We did two things here, importing the HttpModule :

```
  import { HttpModule } from '@angular/http';
```

Adding to the imports keyword:

```
  imports: [
    BrowserModule,
    HttpModule
]
```

Now we are ready to use it. We simply need to inject in a suitable place, lets create a person.service.ts for this:

**person.service.ts**

```
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/map';

export class PersonService {
  constructor(private http:Http)
  getPeople(): Observable<any> {
    return this.http
    .get('/people')
    .map( r => r.json() );
  })
}
```

Worth mentioning here is Observable and .map operator. The Observable is an async concept much like a Promise but with a lot more power. The .map operator filters the data so we are able to transform the data into the exact shape we want.

To consume said service we use dependecy injection which involve:

* telling a module it exists
* inject it into a component
* recive the data from the service call

Telling the module, we do that by the following code:

```
import { BrowserModule } from '@angular/platform-browser';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { MoviesComponent } from './movies.component'

import { Service } from './service';
import { PersonService } from './person.service';

@NgModule({
  declarations: [
    AppComponent, MoviesComponent
  ],
  imports: [
    BrowserModule,
    HttpModule
  ],
  providers: [ Service, PersonService ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

This involves importing the construct like so:

```
import { PersonService } from './person.service';
```

and adding it to the providers array:

```
providers: [ Service, PersonService ]
```

Now for injecting it into a component:

**people.component**

```
import { Component } from '@nagular/core';

@Component({
  selector : 'people',
  template : `
  <div *ngFor="let person of people">
  {{ person.title }}
  </div>
	`
})
export class PeopleComponent {
  constructor(
    private personService: PersonService
  ) {
    this.personService
    .getPeople()
    .subscribe( people => {
      this.people = people;
    })
  }
}
```

Ok so this is all the code we need. Worth noting is the call to .subscribe. .subscribe method is called when the data arrives which could be anything from 2ms to 2 minutes. It works much like the .then for a Promise.

In the next post I will cover more advanced topics like routing and custom component with in and output data.
