# Factory & Abstract Factory

## When should be used

- Object creation logic becomes too convoluted
- Initializer is not descriptive
  - Name is always ```__init__```
  - Cannot overload with the same sets of arguments with different names
  - Can turn into 'optional parameter hell'
- Wholsale object creation (non-piecewise, unlike Builder) can be outsourced to
  - A separate method (Factory Method)
  - That may exist in separate class (Factory)
  - Can create hierarchy of factories with Abstract Factory

---
> A component responsible solely for the wholesale (not piecewise) creation of objects.

---


## Factory Method Practice

```javascript
CoordinateSystem = {
  CARTESIAN : 0,
  POLAR : 1
};

class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  // constructor(a, b, cs=CoordinateSystem.CARTESIAN)
  // {
  //   switch (cs)
  //   {
  //     case CoordinateSystem.CARTESIAN:
  //       this.x = a;
  //       this.y = b;
  //       break;
  //     case CoordinateSystem.POLAR:
  //       this.x = a * Math.cos(b);
  //       this.y = a * Math.sin(b);
  //       break;
  //   }
  //
  //   // steps to add a new system
  //   // 1. augment CoordinateSystem
  //   // 2. change ctor
  // }

  static newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }

  static get factory()
  {
    return new PointFactory();
  }
}

class PointFactory
{
  // not necessarily static
  newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }
}

let p1 = new Point(2, 3, CoordinateSystem.CARTESIAN);
console.log(p1);
// Point → PointFactory
let p2 = PointFactory.newPolarPoint(5, Math.PI/2);
console.log(p2);

// this line will not work if newCartesianPoint is static!
let p3 = Point.factory.newCartesianPoint(2, 3);
console.log(p3);
```


## Factory Practice

```javascript
CoordinateSystem = {
  CARTESIAN : 0,
  POLAR : 1
};

class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  // constructor(a, b, cs=CoordinateSystem.CARTESIAN)
  // {
  //   switch (cs)
  //   {
  //     case CoordinateSystem.CARTESIAN:
  //       this.x = a;
  //       this.y = b;
  //       break;
  //     case CoordinateSystem.POLAR:
  //       this.x = a * Math.cos(b);
  //       this.y = a * Math.sin(b);
  //       break;
  //   }
  //
  //   // steps to add a new system
  //   // 1. augment CoordinateSystem
  //   // 2. change ctor
  // }

  static newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }

  static get factory()
  {
    return new PointFactory();
  }
}

class PointFactory
{
  // not necessarily static
  newCartesianPoint(x, y)
  {
    return new Point(x, y);
  }

  static newPolarPoint(rho, theta)
  {
    return new Point(rho*Math.cos(theta), rho*Math.sin(theta));
  }
}

let p1 = new Point(2, 3, CoordinateSystem.CARTESIAN);
console.log(p1);
// Point → PointFactory
let p2 = PointFactory.newPolarPoint(5, Math.PI/2);
console.log(p2);

// this line will not work if newCartesianPoint is static!
let p3 = Point.factory.newCartesianPoint(2, 3);
console.log(p3);
```


## Abstract Factory Practice

```javascript
const readline = require('readline');
let rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});
const async = require('async');

class HotDrink
{
  consume() {}
}

class Tea extends HotDrink
{
  consume() {
    console.log('This tea is nice with lemon!');
  }
}

class Coffee extends HotDrink
{
  consume()
  {
    console.log(`This coffee is delicious!`);
  }
}

class HotDrinkFactory
{
  prepare(amount) { /* abstract */ }
}

class CoffeeFactory extends HotDrinkFactory
{
  prepare(amount) {
    console.log(`Grind some beans, boil water, pour ${amount}ml`);
    return new Coffee();
  }
}

class TeaFactory extends HotDrinkFactory
{
  prepare(amount) {
    console.log(`Put in tea bag, boil water, pour ${amount}ml`);
    return new Tea();
  }
}

let AvailableDrink = Object.freeze({
  coffee: CoffeeFactory,
  tea: TeaFactory
});

class HotDrinkMachine
{
  constructor()
  {
    this.factories = {};
    for (let drink in AvailableDrink)
    {
      this.factories[drink] = new AvailableDrink[drink]();
    }
  }

  makeDrink(type)
  {
    switch (type)
    {
      case 'tea':
        return new TeaFactory().prepare(200);
      case 'coffee':
        return new CoffeeFactory().prepare(50);
      default:
        throw new Error(`Don't know how to make ${type}`);
    }
  }

  interact(consumer)
  {
    rl.question('Please specify drink and amount ' +
      '(e.g., tea 50): ', answer => {
      const parts = answer.split(' ');
      const name = parts[0];
      const amount = parseInt(parts[1]);
      const d = this.factories[name].prepare(amount);
      rl.close();
      consumer(d);
    });
  }
}

let machine = new HotDrinkMachine();
// rl.question('which drink? ', function(answer)
// {
//   let drink = machine.makeDrink(answer);
//   drink.consume();
//
//   rl.close();
// });
machine.interact(
  function (drink) {
    drink.consume();
  }
);
```

## Factory Excercise

You are given a class called ```Person```. 
The person has two fields: ```id```, and ```name```.

Please implement a ```PersonFactory``` that has a non-static ```createPerson()``` method that takes a person's name and returns a person initialized with this name and an id.

The ```id``` of the person should be set as a 0-based index of the object any instance of PersonFactory has created. 

So, the first person any factory makes should have Id=0, second Id=1 and so on.

```javascript
class Person
{
  constructor(id, name)
  {
    this.id = id;
    this.name = name;
  }
}

class PersonFactory
{
  createPerson(name)
  {
      return new Person(PersonFactory.id++, name);
      
  }
}
PersonFactory.id = 0;

```

## Conclusion

- A factory method is a static method that creates objects
- A factory is any entity that can take care of object creation
- A factory can be external or reside inside the object as an inner class
- Hierarchies of factories can be used to create related objects
