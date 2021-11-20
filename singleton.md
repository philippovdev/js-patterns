# Singleton Pattern

---
> When discussing which patterns to drop, we found that we still love them all. (Not really - i'm in favor of dropping Singleton. Its use is almost always a design smell.)
>
> -- <cite>Erich Gamma</cite>
---


## What is Singleton

---
> A component which is instantiated only once.
---

## When would we use Singleton

- For some components it only makes sense to have one in the system
  - Database repository
  - Object factory
- E.g. the constuctor call is expensive
  - We want initialization to only happen once
  - We provide everyone with the same instance
- Want to prevent anyone creating additional copies

## Singleton implementation

```javascript
class Singleton
{
  constuctor()
  {
    cinst instance = this. constyctor.instance;
    if (instance) return instance;
    
    this.constyctor.instance = this;
  }
}

let s1 = new Sigleton();
let s2 = new Singleton();

console.log('Are they identical? ', + (s1 === s2)));
s1.foo();
```

## Monostate Example

```javascript
class ChiefExecutiveOfficer
{
  get name() { return ChiefExecutiveOfficer._name; }
  set name(value) { ChiefExecutiveOfficer._name = value; }

  get age() { return ChiefExecutiveOfficer._age; }
  set age(value) { ChiefExecutiveOfficer._age = value; }

  toString()
  {
    return `CEO's name is ${this.name} ` +
      `and he is ${this.age} years old.`;
  }
}
ChiefExecutiveOfficer._age = undefined;
ChiefExecutiveOfficer._name = undefined;

let ceo = new ChiefExecutiveOfficer();
ceo.name = 'Adam Smith';
ceo.age = 55;

let ceo2 = new ChiefExecutiveOfficer();
ceo2.name = 'John Gold';
ceo2.age = 66;

console.log(ceo.toString()); // CEO's name is John Gold and he is 66 years old.
console.log(ceo2.toString()); // CEO's name is John Gold and he is 66 years old.
```

## Singleton problems

## Data to consume from ```capitals.txt```

```
Tokyo
332200000
New York
17800000
Sao Paulo
17700000
Seoul
17500000
Mexico City
17400000
Osaka
16425000
Manila
14750000
Mumbai
14350000
Deli
18980000
```

## JS code

```javascript
const fs = require('fs');
const path = require('path');

class MyDatabase
{
  constructor()
  {
    const instance = this.constructor.instance;
    if (instance) {
      return instance;
    }

    this.constructor.instance = this;


    console.log(`Initializing database`);
    this.capitals = {};

    let lines = fs.readFileSync(
      path.join(__dirname, 'capitals.txt')
    ).toString().split('\r\n');

    for (let i = 0; i < lines.length/2; ++i)
    {
      this.capitals[lines[2*i]] = parseInt(lines[2*i+1]);
    }
  }

  getPopulation(city)
  {
    // possible error handling here
    return this.capitals[city];
  }
}

/* 
* ↑↑↑ low-level module
* Up to Dependency Inversion Principle ("D" from "SOLID" )
* ↓↓↓ high-level module
*/

class SingletonRecordFinder
{
  totalPopulation(cities)
  {
    return cities.map(
      city => new MyDatabase().getPopulation(city)
    ).reduce((x,y) => x+y);
  }
}

class ConfigurableRecordFinder
{
  constructor(database)
  {
    this.database = database;
  }

  totalPopulation(cities)
  {
    return cities.map(
      city => this.database.getPopulation(city)
    ).reduce((x,y) => x+y);
  }
}

class DummyDatabase
{
  constructor()
  {
    this.capitals = {
      'alpha': 1,
      'beta': 2,
      'gamma': 3
    };
  }

  getPopulation(city)
  {
    // possible error handling here
    return this.capitals[city];
  }
}

describe('singleton database', function()
{
  it('is a singleton', function()
  {
    const db1 = new MyDatabase();
    const db2 = new MyDatabase();
    expect(db1).toBe(db2);
  });

  it('calculates total population', function()
  {
    let rf = new SingletonRecordFinder();
    let cities = ['Seoul', 'Mexico City'];
    let tp = rf.totalPopulation(cities);
    expect(tp).toEqual(17400000+17500000);
  });

  it('calculates total population better', function()
  {
    let db = new DummyDatabase();
    let rf = new ConfigurableRecordFinder(db);
    expect(rf.totalPopulation(['alpha', 'gamma'])).toEqual(4);
  });
});
```

## Singleton Exercise

Since implementing a singleton is easy, you have a different challenge: write a function called ```isSingleton()```. This method takes a factory (i.e., a function that returns an object); it's up to you to determine whether or not that object is a singleton instance or not.

```javascript
class SingletonTester
{
  static isSingleton(generator)
  {
    const obj1 = generator();
    const obj2 = generator();
    
    return obj1 === obj2;
  }
}
```

## Conclusion

- A constructor can chose what to return; we can keep returning same instance
- Monostate: many instances, shared data
- Directly depending on the Singleton is a bad idea; introduce a dependency instead
