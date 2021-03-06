# Builder pattern

- Some objects are simple and can be created in a single initializer call
- Other objects require a lot of ceremony to create
- Having an object with 10 initializer arguments is not productive
- Instead, opt for piecewise construction
- Builder provides an API for constructing an object step-by-step

## DEFINITION
When piecewise object construction is complicated, provide an API for doing it succictly.

## Practice

```javascript
class Tag
{
  static get indentSize() { return 2; }

  constructor(name='', text='')
  {
    this.name = name;
    this.text = text;
    this.children = [];
  }

  toStringImpl(indent)
  {
    let html = [];
    let i = ' '.repeat(indent * Tag.indentSize);
    html.push(`${i}<${this.name}>\n`);
    if (this.text.length > 0)
    {
      html.push(' '.repeat(Tag.indentSize * (indent+1)));
      html.push(this.text);
      html.push('\n');
    }

    for (let child of this.children)
      html.push(child.toStringImpl(indent+1));

    html.push(`${i}</${this.name}>\n`);
    return html.join();
  }

  toString()
  {
    return this.toStringImpl(0);
  }

  static create(name)
  {
    return new HtmlBuilder(name);
  }
}

class HtmlBuilder
{
  constructor(rootName)
  {
    this.root = new Tag(rootName);
    this.rootName = rootName;
  }

  // non-fluent
  addChild(childName, childText)
  {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
  }

  // fluent
  addChildFluent(childName, childText)
  {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
    return this;
  }

  toString()
  {
    return this.root.toString();
  }

  clear()
  {
    this.root = new Tag(this.rootName);
  }

  build()
  {
    return this.root;
  }
}

// just a single paragraph using string concatenation
const hello = 'hello';
let html = [];
html.push('<p>');
html.push(hello);
html.push('</p>');
console.log(html.join());

// a list with 2 words in it
const words = ['hello', 'world'];
html = [];
html.push('<ul>\n');
for (let word of words)
  html.push(`  <li>${word}</li>\n`);
html.push('</ul>');
console.log(html.join());

// ordinary non-fluent builder
//let builder = new HtmlBuilder('ul');
let builder = Tag.create('ul');
for (let word of words)
  builder.addChild('li', word);
//console.log(builder.toString());
let tag = builder.build();
console.log(tag.toString());

// fluent builder
builder.clear();
builder
  .addChildFluent('li', 'foo')
  .addChildFluent('li', 'bar')
  .addChildFluent('li', 'baz');
console.log(builder.toString());
```

## Person builder

```javascript
class Person
{
  constructor()
  {
    // address info
    this.streetAddress = this.postcode = this.city = '';

    // employment info
    this.companyName = this.position = '';
    this.annualIncome = 0;
  }

  toString()
  {
    return `Person lives at ${this.streetAddress}, ${this.city}, ${this.postcode}\n`
      + `and works at ${this.companyName} as a ${this.position} earning ${this.annualIncome}`;
  }
}

class PersonBuilder
{
  constructor(person=new Person())
  {
    this.person = person;
  }

  get lives()
  {
    return new PersonAddressBuilder(this.person);
  }

  get works()
  {
    return new PersonJobBuilder(this.person);
  }

  build()
  {
    return this.person;
  }
}

class PersonJobBuilder extends PersonBuilder
{
  constructor(person)
  {
    super(person);
  }

  at(companyName)
  {
    this.person.companyName = companyName;
    return this;
  }

  asA(position)
  {
    this.person.position = position;
    return this;
  }

  earning(annualIncome)
  {
    this.person.annualIncome = annualIncome;
    return this;
  }
}

class PersonAddressBuilder extends PersonBuilder
{
  constructor(person)
  {
    super(person);
  }

  at(streetAddress)
  {
    this.person.streetAddress = streetAddress;
    return this;
  }

  withPostcode(postcode)
  {
    this.person.postcode = postcode;
    return this;
  }

  in(city)
  {
    this.person.city = city;
    return this;
  }
}

let pb = new PersonBuilder();
let person = pb
  .lives.at('123 London Road').in('London').withPostcode('SW12BC')
  .works.at('Fabrikam').asA('Engineer').earning(123000)
  .build();
console.log(person.toString());
```

## Object builder

```javascript
class CodeBuilder
{
  constructor(className='Foo')
  {
    // todo
    this.className = className;
    this.fields = [];
    
  }

  addField(name)
  {
    if (!name) {
        return this;
    }
    // todo
    // reminder: we want a fluent interface
    this.fields.push(name);
    return this;
  }

  toString()
  {
      let buffer = [];
      
      buffer.push(`class ${this.className} {\n`);
      
      if (this.fields.length) {
          buffer.push(`  constructor(${this.fields.join(', ')}) {\n`);
          
          for (const field of this.fields) {
              if (field) {
                  buffer.push(`    this.${field} = ${field};\n`);
              }
          }
          
          buffer.push(`  }\n`);
      }
      
      buffer.push(`}`);

      return buffer.join('');
  }
}
```

## Conclusion 

- A builder is a separate component fo building an object
- Can eather give builder an initializer or return it via a static function
- To make builder fluent, return self
- Different facets of an object can be built with different builders working in tandem via a base class
