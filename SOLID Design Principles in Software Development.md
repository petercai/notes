# SOLID Design Principles in Software Development
 ![](https://www.freecodecamp.org/news/content/images/size/w2000/2023/02/start-graph--1-.png) 

**SOLID** is a set of five design principles. These principles help software developers design robust, testable, extensible, and maintainable object-oriented software systems.

Each of these five design principles solves a particular problem that might arise while developing the software systems.

In this article, I’ll show you what the SOLID principles entail, what each part of the SOLID acronym means, and how to implement them in your code.

What We'll Cover
----------------

*   [What are SOLID Design Principles?](#whataresoliddesignprinciples)
    *   [The Single Responsibility Principle (SRP)](https://www.freecodecamp.org/news/solid-design-principles-in-software-development/thesingleresponsibilityprinciplesrp)
    *   [The Open-Closed Principle (OCP)](#theopenclosedprincipleocp)
    *   [The Liskov Substitution Principle (LSP)](#theliskovsubstitutionprinciplelsp)
    *   [The Interface Segregation Principle (ISP)](#theinterfacesegregationprincipleisp)
    *   [The Dependency Inversion Principle (DIP)](#thedependencyinversionprincipledip)
*   [Conclusion](#conclusion)

What are SOLID Design Principles?
---------------------------------

SOLID is an acronym that stands for:

*   Single Responsibility Principle (SRP)
*   Open-Closed Principle (OCP)
*   Liskov Substitution Principle (LSP)
*   Interface Segregation Principle (ISP)
*   Dependency Inversion Principle (DIP)

In the coming sections, we’ll look at what each of those principles means in detail.

The SOLID design principle is a subcategory of many principles introduced by the American scientist and instructor, Robert C. Martin (A.K.A Uncle Bob) in a 2000 paper.

Following these principles can result in a very large codebase for a software system. But in the long run, the main aim of the principles is never defeated. That is, helping software developers make changes to their code without causing any major issues.

### The Single Responsibility Principle (SRP)

The single responsibility principle states that a class, module, or function should have only one reason to change, meaning it should do one thing.

For example, a class that shows the name of an animal should not be the same class that displays the kind of sound it makes and how it feeds.

Here’s an example in JavaScript:

```
class Animal {
  constructor(name, feedingType, soundMade) {
    this.name = name;
    this.feedingType = feedingType;
    this.soundMade = soundMade;
  }

  nomenclature() {
    console.log(`The name of the animal is ${this.name}`);
  }

  sound() {
    console.log(`${this.name}  ${this.soundMade}s`);
  }

  feeding() {
    console.log(`${this.name} is a ${this.feedingType}`);
  }
}

let elephant = new Animal('Elephant', 'herbivore', 'trumpet');
elephant.nomenclature(); 
elephant.sound(); 
elephant.feeding(); 
```

The code above violates the single responsibility principle because the class that's responsible for printing the name of the animal also shows the sound it makes and its type of feeding.

To fix this, you have to create a separate class for the sound and feeding methods like this:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  nomenclature() {
    console.log(`The name of the animal is ${this.name}`);
  }
}

let animal1 = new Animal('Elephant');
animal1.nomenclature(); 

class Sound {
  constructor(name, soundMade) {
    this.name = name;
    this.soundMade = soundMade;
  }

  sound() {
    console.log(`${this.name}  ${this.soundMade}s`);
  }
}

let animalSound1 = new Sound('Elephant', 'trumpet');
animalSound1.sound(); 

class Feeding {
  constructor(name, feedingType) {
    this.name = name;
    this.feedingType = feedingType;
  }

  feeding() {
    console.log(`${this.name} is a/an ${this.feedingType}`);
  }
}

let animalFeeding1 = new Feeding('Elephant', 'herbivore');
animalFeeding1.feeding(); 
```

This way, each of the classes is doing only one thing:

*   the first one prints the name of the animal
*   the second prints the kind of sound it makes
*   and the third one prints its kind of feeding.

That’s more code, but better readability and maintainability. A developer who didn’t write the code can come to it and understand what’s going on quicker than having it all in one class.

### The Open-Closed Principle (OCP)

The open-closed principle states that classes, modules, and functions should be open for extension but closed for modification.

This principle might seem to contradict itself, but you can still make sense of it in code. It means you should be able to extend the functionality of a class, module, or function by adding more code without modifying the existing code.

Here’s some code that violates the open-closed principle in JavaScript:

```
class Animal {
  constructor(name, age, type) {
    this.name = name;
    this.age = age;
    this.type = type;
  }

  getSpeed() {
    switch (this.type) {
      case 'cheetah':
        console.log('Cheetah runs up to 130mph ');
        break;
      case 'lion':
        console.log('Lion runs up to 80mph');
        break;
      case 'elephant':
        console.log('Elephant runs up to 40mph');
        break;
      default:
        throw new Error(`Unsupported animal type: ${this.type}`);
    }
  }
}

const animal1 = new Animal('Lion', 4, 'lion');
animal1.getSpeed(); 
```

The code above violates the open-closed principle because if you want to add a new animal type, you have to modify the existing code by adding another case to the switch statement.

Normally, if you’re using a `switch` statement, then it’s very likely you will violate the open-closed principle.

Here’s how I refactored the code to fix the problem:

```
class Animal {
  constructor(name, age, speedRate) {
    this.name = name;
    this.age = age;
    this.speedRate = speedRate;
  }

  getSpeed() {
    return this.speedRate.getSpeed();
  }
}

class SpeedRate {
  getSpeed() {}
}

class CheetahSpeedRate extends SpeedRate {
  getSpeed() {
    return 130;
  }
}

class LionSpeedRate extends SpeedRate {
  getSpeed() {
    return 80;
  }
}

class ElephantSpeedRate extends SpeedRate {
  getSpeed() {
    return 40;
  }
}

const cheetah = new Animal('Cheetah', 4, new CheetahSpeedRate());
console.log(`${cheetah.name} runs up to ${cheetah.getSpeed()} mph`); 

const lion = new Animal('Lion', 5, new LionSpeedRate());
console.log(`${lion.name} runs up to ${lion.getSpeed()} mph`); 

const elephant = new Animal('Elephant', 10, new ElephantSpeedRate());
console.log(`${elephant.name} runs up to ${elephant.getSpeed()} mph`); 
```

This way, if you want to add a new animal type, you can create a new class that extends `SpeedRate` and pass it to the Animal constructor without modifying the existing code.

For example, I added a new `GoatSpeedRate` class like this:

```
class GoatSpeedRate extends SpeedRate {
  getSpeed() {
    return 35;
  }
}

const goat = new Animal('Goat', 5, new GoatSpeedRate());
console.log(`${goat.name} runs up to ${goat.getSpeed()} mph`); 
```

This conforms to the open-closed principle.

### The Liskov Substitution Principle (LSP)

The Liskov substitution principle is one of the most important principles to adhere to in object-oriented programming (OOP). It was introduced by the computer scientist Barbara Liskov in 1987 in a paper she co-authored with Jeannette Wing.

The principle states that child classes or subclasses must be substitutable for their parent classes or super classes. In other words, the child class must be able to replace the parent class. This has the advantage of letting you know what to expect from your code.

Here’s an example of a code that does not violate the Liskov substitution principle:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  makeSound() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  makeSound() {
    console.log(`${this.name} barks`);
  }
}

class Cat extends Animal {
  makeSound() {
    console.log(`${this.name} meows`);
  }
}

function makeAnimalSound(animal) {
  animal.makeSound();
}

const cheetah = new Animal('Cheetah');
makeAnimalSound(cheetah); 

const dog = new Dog('Jack');
makeAnimalSound(dog); 

const cat = new Cat('Khloe');
makeAnimalSound(cat); 
```

The `Dog` and `Cat` classes can successfully replace the parent `Animal` class.

On the other hand, let’s look at how the code below does violate the Liskov substitution principle:

```
class Bird extends Animal {
  fly() {
    console.log(`${this.name} flaps wings`);
  }
}

const parrot = new Bird('Titi the Parrot');
makeAnimalSound(parrot); 
parrot.fly(); 
```

The `Bird` class violates the Liskov substitution principle because it’s not implementing its own `makeSound` from the parent `Animal` class. Instead, it’s inheriting the generic sound.

To fix this, you have to make it use the `makeSound` method too:

```
class Bird extends Animal {
  makeSound() {
    console.log(`${this.name} chirps`);
  }

  fly() {
    console.log(`${this.name} flaps wings`);
  }
}

const parrot = new Bird('Titi the Parrot');
makeAnimalSound(parrot); 
parrot.fly(); 
```

### The Interface Segregation Principle (ISP)

The interface segregation principle states that clients should not be forced to implement interfaces or methods they do not use.

More specifically, the ISP suggests that software developers should break down large interfaces into smaller, more specific ones, so that clients only need to depend on the interfaces that are relevant to them. This can make the codebase easier to maintain.

This principle is fairly similar to the single responsibility principle (SRP). But it’s not just about a single interface doing only one thing – it’s about breaking the whole codebase into multiple interfaces or components.

Think about this as the same thing you do while working with frontend frameworks and libraries like React, Svelte, and Vue. You usually break down the codebase into components you only bring in when needed.

This means you create individual components that have functionality specific to them. The component responsible for implementing scroll to the top, for example, will not be the one to switch between light and dark, and so on.

Here’s an example of code that violates the interface segregation principle:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  eat() {
    console.log(`${this.name} is eating`);
  }

  swim() {
    console.log(`${this.name} is swimming`);
  }

  fly() {
    console.log(`${this.name} is flying`);
  }
}

class Fish extends Animal {
  fly() {
    console.error("ERROR! Fishes can't fly");
  }
}

class Bird extends Animal {
  swim() {
    console.error("ERROR! Birds can't swim");
  }
}

const bird = new Bird('Titi the Parrot');
bird.swim(); 

const fish = new Fish('Neo the Dolphin');
fish.fly(); 
```

The code above violates the interface segregation principle because the `Fish` class doesn’t need the `fly` method. A fish cannot fly. Birds can’t swim too, so the `Bird` class doesn’t need the `swim` method.

This is how I fixed the code to conform to the interface segregation principle:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  eat() {
    console.log(`${this.name} is eating`);
  }

  swim() {
    console.log(`${this.name} is swimming`);
  }

  fly() {
    console.log(`${this.name} is flying`);
  }
}

class Fish extends Animal {
  
}

class Bird extends Animal {
  
}

const bird = new Bird('Titi the Parrot');
bird.swim(); 

const fish = new Fish('Neo the Dolphin');
fish.fly(); 

console.log('\n');

bird.eat(); 
fish.eat(); 
```

### The Dependency Inversion Principle (DIP)

The dependency inversion principle is about decoupling software modules. That is, making them as separate from one another as possible.

The principle states that high-level modules should not depend on low-level modules. Instead, they should both depend on abstractions. Additionally, abstractions should not depend on details, but details should depend on abstractions.

In simpler terms, this means instead of writing code that relies on specific details of how lower-level code works, you should write code that depends on more general abstractions that can be implemented in different ways.

This makes it easier to change the lower-level code without having to change the higher-level code.

Here’s a code that violates the dependency inversion principle:

```
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  bark() {
    console.log('woof! woof!! woof!!');
  }
}

class Cat extends Animal {
  meow() {
    console.log('meooow!');
  }
}

function printAnimalNames(animals) {
  for (let i = 0; i < animals.length; i++) {
    const animal = animals[i];
    console.log(animal.name);
  }
}

const dog = new Dog('Jack');
const cat = new Cat('Zoey');

const animals = [dog, cat];

printAnimalNames(animals); 
```

The code above violates dependency inversion principle because the `printAnimalNames` function depends on the concrete implementations of Dog and Cat.

If you wanted to add another animal like an ape, you’d have to modify the `printAnimalNames` function to handle it.

Here’s how to fix it:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class Dog extends Animal {
  bark() {
    console.log('woof! woof!! woof!!!');
  }
}

class Cat extends Animal {
  meow() {
    console.log('meooow!');
  }
}

function printAnimalNames(animals) {
  for (let i = 0; i < animals.length; i++) {
    const animal = animals[i];
    console.log(animal.getName());
  }
}

const dog = new Dog('Jack');
const cat = new Cat('Zoey');

const animals = [dog, cat, ape];

printAnimalNames(animals); 
```

In the code above, I created a `getName` method inside the Animal class. This provides an abstraction that the `printAnimalNames` function can depend on. Now, the `printAnimalNames` function only depends on the `Animal` class, not the concrete implementations of `Dog` and `Cat`.

If you wan to add an Ape class, you can do so without modifying the `printAnimalNames` function:

```
class Animal {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

class Dog extends Animal {
  bark() {
    console.log('woof! woof!! woof!!!');
  }
}

class Cat extends Animal {
  meow() {
    console.log('meooow!');
  }
}

class Ape extends Animal {
  meow() {
    console.log('woo! woo! woo!');
  }
}

function printAnimalNames(animals) {
  for (let i = 0; i < animals.length; i++) {
    const animal = animals[i];
    console.log(animal.getName());
  }
}

const dog = new Dog('Jack'); 
const cat = new Cat('Zoey'); 

const ape = new Ape('King Kong'); 

const animals = [dog, cat, ape];

printAnimalNames(animals); 
```

Conclusion
----------

In this article, you learned what the **SOLID** design priniples are. We discussed each principle with an example, and went through ways to implement them in JavaScript.

