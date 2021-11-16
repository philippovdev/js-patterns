# Prototype Pattern

## When should be used

- Complicated objects (e.g. cars) aren't designed from scratch
  - They reiterate existing designs
- An existing (partially or fully constructed) design a Prototype
- We make a copy (clone) the prototype and customize it
  - Requires 'deep copy' support
 We make the cloning convenient (e.g., via a Factory)
 
---

> A partially or fully initialized object that you copy (clone) and make ise of.

---

## Prototype pattern practice

```javascript
class Address
{
  constructor(streetAddress, city, country) {
    this.streetAddress = streetAddress;
    this.city = city;
    this.country = country;
  }

  deepCopy()
  {
    return new Address(
      this.streetAddress,
      this.city,
      this.country
    );
  }

  toString()
  {
    return `Address: ${this.streetAddress}, ` +
      `${this.city}, ${this.country}`;
  }
}

class Person
{
  constructor(name, address)
  {
    this.name = name;
    this.address = address; //!
  }

  deepCopy()
  {
    return new Person(
      this.name,
      this.address.deepCopy() // needs to be recursive
    );
  }

  toString()
  {
    return `${this.name} lives at ${this.address}`;
  }
}

let john = new Person('John',
  new Address('123 London Road', 'London', 'UK'));

let jane = john.deepCopy();

jane.name = 'Jane';
jane.address.streetAddress = '321 Angel St'; // oops

console.log(john.toString()); // oops, john is called 'jane'
console.log(jane.toString());
```

## Explicite Copying

```javascript
class Address
{
  constructor(streetAddress, city, country) {
    this.streetAddress = streetAddress;
    this.city = city;
    this.country = country;
  }

  toString()
  {
    return `Address: ${this.streetAddress}, ` +
      `${this.city}, ${this.country}`;
  }
}

class Person
{
  constructor(name, address)
  {
    this.name = name;
    this.address = address; //!
  }

  toString()
  {
    return `${this.name} lives at ${this.address}`;
  }

  greet()
  {
    console.log(
      `Hi, my name is ${this.name}, ` +
      `I live at ${this.address.toString()}`
    );
  }
}

class Serializer
{
  constructor(types){
    this.types = types;
  }

  markRecursive(object)
  {
    // anoint each object with a type index
    let idx = this.types.findIndex(t => {
      return t.name === object.constructor.name;
    });
    if (idx !== -1)
    {
      object['typeIndex'] = idx;

      for (let key in object)
      {
        if (object.hasOwnProperty(key) && object[key] != null)
          this.markRecursive(object[key]);
      }
    }
  }

  reconstructRecursive(object)
  {
    if (object.hasOwnProperty('typeIndex'))
    {
      let type = this.types[object.typeIndex];
      let obj = new type();
      for (let key in object)
      {
        if (object.hasOwnProperty(key) && object[key] != null) {
          obj[key] = this.reconstructRecursive(object[key]);
        }
      }
      delete obj.typeIndex;
      return obj;
    }
    return object;
  }

  clone(object)
  {
    this.markRecursive(object);
    let copy = JSON.parse(JSON.stringify(object));
    return this.reconstructRecursive(copy);
  }
}

let john = new Person('John',
  new Address('123 London Road', 'London', 'UK'));

let jane = JSON.parse(JSON.stringify(john));

jane.name = 'Jane';
jane.address.streetAddress = '321 Angel St';

john.greet();
// this won't work
// jane.greet();

// try a dedicated serializer
let s = new Serializer([Person,Address]); // pain point
jane = s.clone(john);

jane.name = 'Jane';
jane.address.streetAddress = '321 Angel St';

console.log(john.toString());
console.log(jane.toString());
```

## Prototype Factory

```javascript
class Address
{
  constructor(suite, streetAddress, city)
  {
    this.suite = suite;
    this.streetAddress = streetAddress;
    this.city = city;
  }

  toString()
  {
    return `Suite ${this.suite}, ` +
      `${this.streetAddress}, ${this.city}`;
  }
}

class Employee // renamed
{
  constructor(name, address)
  {
    this.name = name;
    this.address = address; //!
  }

  toString()
  {
    return `${this.name} works at ${this.address}`;
  }

  greet()
  {
    console.log(
      `Hi, my name is ${this.name}, ` +
      `I work at ${this.address.toString()}` //!
    );
  }
}

class Serializer
{
  constructor(types){
    this.types = types;
  }

  markRecursive(object)
  {
    // anoint each object with a type index
    let idx = this.types.findIndex(t => {
      return t.name === object.constructor.name;
    });
    if (idx !== -1)
    {
      object['typeIndex'] = idx;

      for (let key in object)
      {
        if (object.hasOwnProperty(key) && object[key] != null)
          this.markRecursive(object[key]); // ^^^^^^^^^^ important
      }
    }
  }

  reconstructRecursive(object)
  {
    if (object.hasOwnProperty('typeIndex'))
    {
      let type = this.types[object.typeIndex];
      let obj = new type();
      for (let key in object)
      {
        if (object.hasOwnProperty(key) && object[key] != null) {
          obj[key] = this.reconstructRecursive(object[key]);
        }
      }
      delete obj.typeIndex;
      return obj;
    }
    return object;
  }

  clone(object)
  {
    this.markRecursive(object);
    let copy = JSON.parse(JSON.stringify(object));
    return this.reconstructRecursive(copy);
  }
}

class EmployeeFactory
{
  static _newEmployee(proto, name, suite)
  {
    let copy = EmployeeFactory.serializer.clone(proto);
    copy.name = name;
    copy.address.suite = suite;
    return copy;
  }

  static newMainOfficeEmployee(name, suite)
  {
    return this._newEmployee(
      EmployeeFactory.main, name, suite
    );
  }

  static newAuxOfficeEmployee(name, suite)
  {
    return this._newEmployee(
      EmployeeFactory.aux, name, suite
    );
  }
}

EmployeeFactory.serializer = new Serializer([Employee, Address]);
EmployeeFactory.main = new Employee(null,
  new Address(null, '123 East Dr', 'London'));
EmployeeFactory.aux = new Employee(null,
  new Address(null, '200 London Road', 'Oxford'));

let john = EmployeeFactory.newMainOfficeEmployee('John', 4321);
let jane = EmployeeFactory.newAuxOfficeEmployee('Jane', 222);

console.log(john.toString());
console.log(jane.toString());
```

## Exersize

Given the definitions shown in code, you are asked to implement ```Line.deepCopy()``` to perform a deep copy of the given ```Line``` object. This method should return a copy of a Line that contains copies of its start/end points.

```javascript
class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }
}

class Line
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
  }

  deepCopy()
  {
    let newStart = new Point(this.start.x, this.start.y);
    let newEnd = new Point(this.end.x, this.end.y);
    return new Line(newStart, newEnd);
  }
}

describe('prototype', function()
{
  it('test', function()
  {
    let line1 = new Line(
      new Point(3, 3),
      new Point(10, 10)
    );

    let line2 = line1.deepCopy();
    line1.start.x = line1.end.x =
      line1.end.x = line1.end.y = 0;

    expect(line2.start.x).toEqual(3);
    expect(line2.start.y).toEqual(3);
    expect(line2.end.x).toEqual(10);
    expect(line2.end.y).toEqual(10);
  });
});
```

## Conclusion

- To implement a ptototype, partially construct an object and store it somewhere
- Deep copy the prototype
- Customize the resulting instance
- A factory ptovides a convenient API for using prototypes
