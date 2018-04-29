# AngularJS to Angular

Reading time : 15 min

Ok so you are here because you used to code AngularJS. You are probably stuck in a maintenance project dreaming of a new framework and new paradigms. Your company have told you the next project will be built using Angular 4.x and you are happy to be part of it but suddenly you start to panic. I don't know Angular 4.x, everybody says it is a complete rewrite. I wont recognise a single concept.
There is a right and a wrong in the previous sentence. It is a complete rewrite, the concepts are quite familiar though. This post will take you through the most used concepts in AngularJS and essentially explain if and how they have changed in Angular 4.x. Let's begin.

# Core directives

In AngularJS we used core directives in the template to help us with loops, conditional logic, styling and more. They are still there in Angular 4.x but have been slightly renamed.

## ng-repeat => \*ngFor

We used ng-repeat to loop through a list of items. We used to write:

```
<div ng-repeat="item in items">
{% raw %}
{{item.name}}
{% endraw %}
</div>
```

The corresponding construct in Angular 4.x is called \*ngFor and is written like this:

```
<div *ngFor="let item of items">
{% raw %}
{{item.name}}
{% endraw %}
</div>
```

more advanced usage of the ng-repeat meant that we could access built in variables like `$index`, `$odd`, `$even`. They are still there but you need to create them when you declare the \*ngFor directive, like so:

```
<div *ngFor="let item of items; let index = index; let odd = odd; let even = even">
{% raw %}
{{item.name}}
{% endraw %}
</div>
```

Sure a tad more to write but its there.

## ng-if => \*ngIf

So this core directive was used to determine wether something should be part of the DOM tree or not. The better option over the ng-show in terms of performance. It was used like this:

```
<div ng-if="true/false">
render me if true
</div>
```

The corresponding way in Angular 4.x is:

```
<div *ngIf="true/false">
render me if true
</div>
```

# Bindings

Ok so in AngularJS we had the following bindings at our disposal:

* one way binding, interpolation
* two-way binding
* one time binding
* event binding

## one-way binding, interpolation

This type of binding meant that the only thing that could change, what was displayed on the screen, was if something in the controller changed. It took on the form of interpolation with curly brackets or using the ng-bind directive.
It looked like this:

```
<div>
{% raw %}
{{item.name}}
{% endraw %}
</div>
```

or using the ng-bind directive

```
<div ng-bind="item.name">
```

The interpolation looks exactly the same.
However there is attribute binding as a new thing in Angular 4.x. In AngularJS we used to write the following:

```
<div title="{% raw %}{{titleProperty}}{% endraw %}">
```

In Angular 4.x we can use an attribute binding, recognised by the square brackets []:

```
<div [title]="titleProperty">
```

## ng-model => [(ngModel)], aka banana in a box, two-way binding

Ok so AngularJS relied on two-way binding which looked like a great idea at first. This however meant we had a bit of ineffiecent dirty checking which made us a bit sad. So in Angular 4.x we are using a combination of event binding and attribute binding to accomplish what looks like a two-way binding. All the behaviour of a two-binding but none of the performance hit. So in AngularJS we wrote:

```
<input ng-model="product.name">
```

and in Angular 4.x we write:

```
<input [(ngModel)]="product.name">
```

and business as ususal

## one-time binding

Ok so we hade one time binding to reduce the number of watchers in AngularJS, if we had too many our app would become sluggish and nobody wants that. So we could solve that in the markup by typing :: before a property when doing interpolation:

```
<div>
{% raw %}
{{::item.name}}
{% endraw %}
</div>
```

Angular 4.x is a different story, it takes on a different approach, data isn't changeable unless you explicit state that it should be by hooking up to change events.

However if you want the exact correspondence of a one-time binding we need to look at something called ChangeDetection and set it to ChangeDetectionStrategy.OnPush. This is something you do inside of a component where you explicitly say that all my data comes from the outside - If you want to check something, check my parent component.
I havn't really explained components at this point so we will leave it here with a short explanation. A component is a unified concept of a controller and an element directive.

## event binding

In AngularJS we subscribed to events using built in directives for it, for click events there were for example ng-click:

```
<div ng-click="method()"></div>
```

for other events there were similar directives so you could assume something was called ng-<eventname>.

Angular 4.x takes a simplified approach to this, parenthesis:

```
<div (click)="method()"></div>
```

So just remember the name of the event and add parenthesis to it (event)="method". There you go, event binding

# Element directive + Controller => Component

In AngularJS we used controllers as responders to a route change. We used so called element directives to create reusable components. A change started to happen in AngularJS by the introduction of the .component() method which was essentially a syntactic sugar over .directive(). It was meant to be a bigger change though which is why a component router was added to AngularJS as well so we could finally get rid of .controller once and for all.

In Angular 4.x there are only components where there used to be element directives and controllers, easier right?

So when defining a route you now write:

```
let routes = [{
  route: 'products',
  component: ProductsComponent
}]
```

Let's show the old element directive from AngularJS and see how it maps to a component:

```
.directive('elementDir', function(){
  return {
    restrict : 'E',
    replace : true,
    controller : function($scope) {},
    link : function(scope, element, attr){
      // setting up events on the element
      // talk to attributes on the element etc
    },
    scope : {
        items: '=',
        b: '&',
        c : '@'
    },
    template : `
    <div ng-repeat="item in items">
    {% raw %}
    {{item.name}}
    {% endraw %}
    </div>
    `
  }
})
```

Now this looks like the following in Angular 4.x.

```
@Component({
    selector : 'element-dir',
    template : `
    <div *ngFor="let item of items">
    {% raw %}
    {{item.name}}
    {% endraw %}
    </div>
    `
})
export class ElementDirComponent {
    @Input() items:Item[];
    @Input() c;
    @Output() b = new EventEmitter();

}
```

Like with everything, you get used to it, if using it a couple or 100+ times. That was probably the case for me with directives but they arn't very nice creatures. You had to know a bunch of stuff like:

* Casing in the name mattered, if you wanted to call <element-dir> you had to call it elementDir when declaring it
* input and outputs where declared on a scope object and =, &, @ gave us different binding types instead of the much more clear @Input and @Output decorator. And there were link function and controller function instead of just a class as in Angular 4.x

For me an Angular 4.x component is a lot less noise and to be frank so is the .component method in AngularJS which is one of the reasons you should use that instead.

# Filters => Pipes

Filters are a thing that can act on both a property as well as a list. The declaration and usage looked like this:

```
.filter('banana', function(){
    return function(data, param1, param2..){
      if(param1) {
        return 'banana' + data
      } else { return 'minion' + data }
    }
})
```

and to use it:

```
<div>
{% raw %}
{{ 'bob' | banana:true }}
{% endraw %}
</div>
```

The corresponding thing in Angular 4.x is this:

```
@Pipe({
    name : 'banana'
})
export class BananaPipe implements PipeTransform {
  transform(data, arg1, arg2) {
      if(arg1) {
        return 'banana' + data;
      } else { return 'minion' + data }
  }
}
```

## modules

In AngularJS we used modules as way to group our constructs under one module like so:

```
angular.module('products', [])
.controller('productsController', productsController)
.service('productsService', productsService)
.filter('productsFilter', productsFilter)
```

You get the idea. We did this to make these constructs available to the dependency injection system. This was however a global container which meant we could inject a productsFilter inside of a user controller. That was a bit of a leak.

Modules exist in Angular 4.x but they are now able to contain what should be in the module and what should be exposed to other modules. A module is declared by using the decorator @NgModule like so:

```
@NgModule({
    declarations : [ProductsComponent, ProductsFilter],
    exports : [ ProductsComponent ]
})
export class ProductModule{}
```

We assume thast ProductsComponent is the only construct that should be public facing which is why we input it into the `exports` array. The `declarations` is the new way of saying that something belongs to a module.

We have furthermore assumed that ProductsService is only for ProductsComponent to use. So how does the DI system know how to inject that to the component? the answer is the `providers` array on component level:

```
@Component({
    selector : 'products',
    providers : [ ProductsService ]
})
export class ProductsComponent{}
```

This will ensure that the ProductsService is contained within the ProductsComponent and its view component children.

Should we change our minds in the future and make the ProductsService available to outside modules we just need to add to the `providers` array of the module instead, like so:

```
@NgModule({
  declarations : [ProductsComponent, ProductsFilter],
    exports : [ ProductsComponent ],
    providers : [ ProductsService ]
})
export class ProductModule{}
```

And of course the last but important bit, how to use things from other modules? In AngularJS we did it like this:

```
angular.module('featuremodule',[]);
angular.module('coremodule',[])

angular.module('app', [ 'featuremodule','coremodule'... ])
```

In Angular 4.x we use the `imports` array on module level, like so:

```
@NgModule({
    imports : [ FeatureModule, CoreModule ]
})
export class AppModule{}
```

## .service + .factory + .provider => ES2015 module + Injectable

Ok we are almost done, we just need to explain what happened to services, you know the superclear concept ;)

In AngularJS we could use .provider to create a service like this:

```
.provider('productServiceProvider', function(){

   this.init = function(){
     // configure service prior to usage
   }

   function Service(){
     this.getData = function(){
         return [];
     }
   }

    return {
        $get : Service
    }
})
```

then we could inject it the config() lifecycle by doing this:

```
.config(function(productServiceProvider){
  productServiceProvider.init();
})
```

and later on like this:

```
.controller('ctrl', function(productService){

})
```

Of course in most cases you just wanted a simple service so you could use .service or .factory for pretty much the same thing.

In Angular 4.x a service is just an ES2015 module. A separate file product.service.ts, looking something like this:

```
export class ProductService() {
    getData(){}
}
```

and to use it from a consumer we just write:

```
import { ProductService } from './product.service';

@Component({

})
export class ProductComponent {
    constructor(service: ProductService) {
      service.getData();
    }
}
```

or if our ProductService had dependencies we needed to decorate it with an @Injectable decorator like so:

```
@Injectable()
export class ProductsService {
    constructor(logger:Logger, dataService:DataService){

    }
    getData(){}
}
```

This would ensure that the DI mechanism would be able to resolve what Logger and DataService is and just give you a nice instantiated ProductService when you ask for it.

## Summary

This was a bit lengthy bit I hope I have given you a crash course in AngularJS to Angular 4.x mental remapping. The concepts you know are there, in some cases they are renamed but mostly they work the same way on the surface. More changes have happened under the surface of the framework than on it. So my tip from here is have a look at [Code lab](https://codelab.fun) to be given a quick zeroconfig view on Angular 4.x then download @angular-cli and off you go to conquer the world.

So before you even think of trying out things like lazy loading, AOT compilation, server side rendering etc, learn the fundamentals first and know them well.
