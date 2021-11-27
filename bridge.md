# Bridge Pattern

---
> ## Connecting components together through abstraction
>
> A mechanism that decouples an interface (hierarchy) from an implementation (hierarchy).
> 
> Reminder: JS has [duck typing](https://medium.com/@eamonocallaghan/what-is-duck-typing-in-javascript-f3eb10853361), so definitions of interfaces are not strictly necessary
> 
---

## Motivation to use

- Bridge prevents a 'Cartesian product' complexity explosion
- Example:
  - Base class ```ThreadScheduler```
  - Can be preemptive or cooperative
  - Can run on Windows or Unix
  - End up with a 2x2 scebario:
    WindowsPTS, UnixPTS, WindowsCTS, UnixCTS
    
  - Before:
  - <img width="1577" alt="Screenshot 2021-11-27 at 21 00 01" src="https://user-images.githubusercontent.com/34489250/143691888-8a3edb6d-fec0-4d68-805f-2ebc4145e878.png">
  - After:
  - <img width="1292" alt="Screenshot 2021-11-27 at 21 02 33" src="https://user-images.githubusercontent.com/34489250/143691926-3afa1bde-ec0c-4c1f-9303-f2f0a6335616.png">
- Bridge pattern avoids the entity explosion

## Implementation

```javascript
class VectorRenderer
{
  renderCircle(radius)
  {
    console.log(`Drawing a circle of radius ${radius}`);
  }
}

class RasterRenderer
{
  renderCircle(radius)
  {
    console.log(`Drawing pixels for circle of radius ${radius}`);
  }
}

class Shape
{
  constructor(renderer)
  {
    this.renderer = renderer;
  }
}

class Circle extends Shape
{
  constructor(renderer, radius) {
    super(renderer);
    this.radius = radius;
  }

  draw()
  {
    this.renderer.renderCircle(this.radius);
  }

  resize(factor)
  {
    this.radius *= factor;
  }
}

// imagine Square, Triangle
// different ways of rendering: vector, raster
// we don't want a cartesian product of these

let raster = new RasterRenderer();
let vector = new VectorRenderer();
let circle = new Circle(vector, 5);
circle.draw();
circle.resize(2);
circle.draw();
```

## Bridge Coding Exercise

You are given an example of an inheritance hierarhcy which resulrs in Cartesian-Product fuplicarion.

Please refactor this hierarchy, giving the base class ```Shape``` a constructor that takes a renderer, whose expected interface is:

```javascript
class <SomeRenderer>
{
  get whatToRenderAs()
  {
    // todo
  }
}
```

There's no need to implement hte type above (due to duck typing), but I do want you to implement classes ```VectorRenderer``` and ```RasterRenderer```.
Each ingeritor of the ```Shape``` constructed object's ```toString()``` operates correctly, for example,
```new Triangle(new RasterRenderer()) // returns "Drawing Triangle as pixels"
```

```javascript
// class Shape
// {
//   constructor(name)
//   {
//     this.name = name;
//   }
// }
//
// class Triangle extends Shape
// {
//   constructor()
//   {
//     super('triangle');
//   }
// }
//
// class Square extends Shape
// {
//   constructor()
//   {
//     super('square');
//   }
// }
//
// class VectorSquare extends Square
// {
//   toString()
//   {
//     return `Drawing square as lines`;
//   }
// }
//
// class RasterSquare extends Square
// {
//   toString()
//   {
//     return `Drawing square as pixels`;
//   }
// }

// imagine VectorTriangle and RasterTriangle are here too

class Shape {
    constructor(renderer, name=null){
        this.renderer = renderer;
        this.name = name;
    }
    toString() {
        return `Drawing ${this.name} as ${this.renderer.whatToRenderAs}`;
    }
}


class Triangle extends Shape
{
  constructor(renderer)
  {
    super(renderer, 'triangle');
  }
}

class Square extends Shape
{
  constructor(renderer)
  {
    super(renderer, 'square');
  }
}

class RasterRenderer
{
  get whatToRenderAs()
  {
    return 'pixels';
  }
}

class VectorRenderer
{
  get whatToRenderAs()
  {
    return 'lines';
  }
}

```

## Conclusion

- Decouple abstraction from implementration
- Both can exist as hierarchies
- A stronger form of encapsulation
