# Factory Method Pattern

## Intent
Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

## When to use
- A class can't anticipate the class of objects it must create.
- You want to localize knowledge of which concrete classes are used.
- To provide hooks for subclasses to extend or change the product type.
- When you need to decouple client code from concrete product classes for extensibility and testability.

## Participants
- Product: interface or abstract class that defines the object the factory method creates.
- ConcreteProduct: concrete implementations of Product.
- Creator: declares the factory method that returns Product. May provide default implementation that uses Product.
- ConcreteCreator: overrides the factory method to return a ConcreteProduct.

## Key Variants / Notes
- Creator can be an abstract class or interface with a factory method.
- Factory method can be an instance method (most common) or a static method (simpler but less flexible for subclassing).
- Related patterns: Simple Factory (not a GOF pattern) centralizes creation but without subclassing. Abstract Factory provides families of related products.

## Example (classic OOP style)
```java
// Product
public interface Product {
    void use();
}

// Concrete Products
public class ConcreteProductA implements Product {
    @Override
    public void use() {
        System.out.println("Using ConcreteProductA");
    }
}

public class ConcreteProductB implements Product {
    @Override
    public void use() {
        System.out.println("Using ConcreteProductB");
    }
}

// Creator
public abstract class Creator {
    // Factory method
    protected abstract Product createProduct();

    // Business logic that uses Product
    public void doSomething() {
        Product p = createProduct();
        // operate on product
        p.use();
    }
}

// Concrete Creators
public class CreatorA extends Creator {
    @Override
    protected Product createProduct() {
        return new ConcreteProductA();
    }
}

public class CreatorB extends Creator {
    @Override
    protected Product createProduct() {
        return new ConcreteProductB();
    }
}

// Client code
public class Client {
    public static void main(String[] args) {
        Creator creator = new CreatorA();
        creator.doSomething(); // Uses ConcreteProductA

        creator = new CreatorB();
        creator.doSomething(); // Uses ConcreteProductB
    }
}
```

## Example (with parameterized factory selection)
```java
// Suppose we want to create parsers for different formats
public interface Parser {
    void parse(String input);
}

public class JsonParser implements Parser {
    public void parse(String input) {
        System.out.println("Parsing JSON: " + input);
    }
}

public class XmlParser implements Parser {
    public void parse(String input) {
        System.out.println("Parsing XML: " + input);
    }
}

public abstract class ParserFactory {
    public abstract Parser createParser();

    public void handle(String input) {
        Parser parser = createParser();
        parser.parse(input);
    }
}

public class JsonParserFactory extends ParserFactory {
    @Override
    public Parser createParser() {
        return new JsonParser();
    }
}

public class XmlParserFactory extends ParserFactory {
    @Override
    public Parser createParser() {
        return new XmlParser();
    }
}
```

## Pros
- Promotes loose coupling: clients use Product interface, not concrete classes.
- Single Responsibility: creator is responsible for object creation; business logic separate.
- Open/Closed: easy to introduce new products by subclassing creators.

## Cons / Pitfalls
- Can lead to many subclasses: one creator per product variation.
- Subclassing overhead: small variations may be simpler with dependency injection or configuration.
- Overuse: if product types are known and fixed, factory method may add unnecessary indirection.

## Testing and Extensibility
- Unit tests can provide test-specific ConcreteCreator implementations that produce test doubles (mocks/stubs).
- For runtime configurability, combine Factory Method with reflection or dependency injection (e.g., supply creator via DI container).

## Exercises
1. Implement a Shape factory method: Product = Shape (draw()), ConcreteProducts = Circle, Rectangle, Triangle. Create creators for each shape and demonstrate switching creators at runtime.
2. Replace the creator-subclass approach with a registry-based factory: a map from keys to Supplier<Product>, and compare pros/cons.
3. Write unit tests that verify Creator.doSomething() uses the correct ConcreteProduct by injecting test creators that return mocks.
4. Implement a ParserFactory that reads a config file (or environment variable) to choose between JSON and XML parser factories at runtime.

## Production Checklist
- Choose Factory Method when subclasses naturally vary product creation.
- If you have many product types and don't need subclassing, consider Abstract Factory, Builder, or a registry-based factory with DI.
- Keep creator responsibilities focused: factory method should create and return product; heavy initialization can be moved to other methods.
- Document extension points so future contributors know how to add new ConcreteProducts/Creators.

---

Created by GitHub Copilot assistant — structured notes and examples for the Factory Method pattern.
