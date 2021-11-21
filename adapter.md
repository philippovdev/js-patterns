# Adapter Pattern

_Getting the interface you want from the interface you have._ 

![image alt >](https://user-images.githubusercontent.com/34489250/142761730-65c29e52-c736-4f61-9c74-edf67afec735.png)

## Description

- Electric devices have different power (interface) requirements
  - Voltage (5V, 220V)
  - Socket/plug type (Europe, UK, USA)
- We cannot modify our gadgets to support every possible interface
  - Some support possible (e.g. 120/220V)
- Thus, we use a special device (an adapter) to give us the interface we require from the interface we have  

---
> A construct which adapts an existing interface X to conform to the required interface Y.
---

## Adapter Pattern Example 

```javascript
class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  toString()
  {
    return `(${this.x}, ${this.y})`;
  }
}

class Line
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
  }

  toString()
  {
    return `${this.start.toString()}→${this.end.toString()}`;
  }
}

class VectorObject extends Array {}

class VectorRectangle extends VectorObject
{
  constructor(x, y, width, height)
  {
    super();
    this.push(new Line(new Point(x,y), new Point(x+width, y) ));
    this.push(new Line(new Point(x+width,y), new Point(x+width, y+height) ));
    this.push(new Line(new Point(x,y), new Point(x, y+height) ));
    this.push(new Line(new Point(x,y+height), new Point(x+width, y+height) ));this.push
  }
}

// ↑↑↑ this is your API ↑↑↑

// ↓↓↓ this is what you have to work with ↓↓↓

let vectorObjects = [
  new VectorRectangle(1, 1, 10, 10),
  new VectorRectangle(3, 3, 6, 6)
];

let drawPoint = function(point)
{
  process.stdout.write('.');
};

// ↓↓↓ to draw our vector objects, we need an adapter ↓↓↓

class LineToPointAdapter extends Array
{
  constructor(line)
  {
    super();
    console.log(`${LineToPointAdapter.count++}: Generating ` +
      `points for line ${line.toString()} (no caching)`);

    let left = Math.min(line.start.x, line.end.x);
    let right = Math.max(line.start.x, line.end.x);
    let top = Math.min(line.start.y, line.end.y);
    let bottom = Math.max(line.start.y, line.end.y);

    if (right - left === 0)
    {
      for (let y = top; y <= bottom; ++y)
      {
        this.push(new Point(left, y));
      }
    }
    else if (line.end.y - line.start.y === 0)
    {
      for (let x = left; x <= right; ++x)
      {
        this.push(new Point(x, top));
      }
    }
  }
}
LineToPointAdapter.count = 0;

let drawPoints = function()
{
  for (let vo of vectorObjects)
    for (let line of vo)
    {
      let adapter = new LineToPointAdapter(line);
      adapter.forEach(drawPoint);
    }
};

drawPoints();
drawPoints();
```

## Adapter Caching

```javascript
class Point
{
  constructor(x, y)
  {
    this.x = x;
    this.y = y;
  }

  toString()
  {
    return `(${this.x}, ${this.y})`;
  }
}

class Line
{
  constructor(start, end)
  {
    this.start = start;
    this.end = end;
  }

  toString()
  {
    return `${this.start.toString()}→${this.end.toString()}`;
  }
}

class VectorObject extends Array {}

class VectorRectangle extends VectorObject
{
  constructor(x, y, width, height)
  {
    super();
    this.push(new Line(new Point(x,y), new Point(x+width, y) ));
    this.push(new Line(new Point(x+width,y), new Point(x+width, y+height) ));
    this.push(new Line(new Point(x,y), new Point(x, y+height) ));
    this.push(new Line(new Point(x,y+height), new Point(x+width, y+height) ));this.push
  }
}

// ↑↑↑ this is your API ↑↑↑

// ↓↓↓ this is what you have to work with ↓↓↓
String.prototype.hashCode = function(){
  if (Array.prototype.reduce){
    return this.split("").reduce(function(a,b){
      a=((a<<5)-a)+b.charCodeAt(0);return a&a},0);
  }
  let hash = 0;
  if (this.length === 0) return hash;
  for (let i = 0; i < this.length; i++) {
    const character = this.charCodeAt(i);
    hash  = ((hash<<5)-hash)+character;
    hash = hash & hash; // Convert to 32-bit integer
  }
  return hash;
};

class LineToPointAdapter extends Array
{
  constructor(line)
  {
    super();

    this.hash = JSON.stringify(line).hashCode();
    if (LineToPointAdapter.cache[this.hash])
      return; // we already have it

    console.log(`${LineToPointAdapter.count++}: Generating ` +
      `points for line ${line.toString()} (with caching)`);

    let points = [];

    let left = Math.min(line.start.x, line.end.x);
    let right = Math.max(line.start.x, line.end.x);
    let top = Math.min(line.start.y, line.end.y);
    let bottom = Math.max(line.start.y, line.end.y);

    if (right - left === 0)
    {
      for (let y = top; y <= bottom; ++y)
      {
        points.push(new Point(left, y));
      }
    }
    else if (line.end.y - line.start.y === 0)
    {
      for (let x = left; x <= right; ++x)
      {
        points.push(new Point(x, top));
      }
    }

    LineToPointAdapter.cache[this.hash] = points;
  }

  get items()
  {
    return LineToPointAdapter.cache[this.hash];
  }
}
LineToPointAdapter.count = 0;
LineToPointAdapter.cache = {};

let vectorObjects = [
  new VectorRectangle(1, 1, 10, 10),
  new VectorRectangle(3, 3, 6, 6)
];

let drawPoint = function(point)
{
  process.stdout.write('.');
};

let draw = function()
{
  for (let vo of vectorObjects)
  {
    for (let line of vo)
    {
      let adapter = new LineToPointAdapter(line);
      adapter.items.forEach(drawPoint);
    }
  }
};

draw();
draw();
```

## Adaprer Coding Excersise

You are given a class called ```Square``` and function ```calculateArea()``` which calculates the area of a given rectangle.

In order to use Square in calculate_area, instead of augmenting it with width/height members, please implement the ```SquareToRectangleAdapter``` class. This adapter takes a square and adappts it in such a way that it can be used as an argument to ```calculateArea()```.

```javascript
class Square
{
  constructor(side)
  {
    this.side = side;
  }
}

function area(rectangle)
{
  return rectangle.width * rectangle.height;
}

class SquareToRectangleAdapter
{
  constructor(square)
  {
    this.square = square;
  }

  get width() {
    return this.square.side;
  }

  get height() {
    return this.square.side;
  }
}

describe('adapter', function()
{
  it('should adapt things, duh!', function()
  {
    let sq = new Square(11);
    let adapter = new SquareToRectangleAdapter(sq);
    expect(area(adapter)).toEqual(121);
  });
});
```

## Conclusion

- implementing the Adapter is easy
- Determine the API you have and the API you need
- Create a component which aggregates (has a reference to, ...) the adaptee
- Intermediate representations can pile up: use of caching and other optimizations
