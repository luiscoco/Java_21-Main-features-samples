# Java 21 Main features samples 


## 1. Pattern Matching for switch

This feature extends the capabilities of switch expressions and statements, allowing you to express more complex and sophisticated data pattern matching

```java
// Define the Shape interface
interface Shape {}

// Circle class implementing Shape interface
class Circle implements Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }
}

// Rectangle class implementing Shape interface
class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    public double getWidth() {
        return width;
    }

    public double getHeight() {
        return height;
    }
}

// Main class
public class ShapeDescriber {

    // describeShape method
    public static String describeShape(Object shape) {
        return switch (shape) {
            case Circle c -> "Circle with radius " + c.getRadius();
            case Rectangle r -> "Rectangle with width " + r.getWidth() + " and height " + r.getHeight();
            case null -> "It's null";
            default -> "Some other shape";
        };
    }

    // Main method to test the describeShape function
    public static void main(String[] args) {
        Circle circle = new Circle(5.0);
        Rectangle rectangle = new Rectangle(4.0, 6.0);
        
        System.out.println(describeShape(circle)); // Prints description of the circle
        System.out.println(describeShape(rectangle)); // Prints description of the rectangle
        System.out.println(describeShape(null)); // Prints null case
        System.out.println(describeShape("String")); // Example for default case
    }
}
```

Pattern matching in switch can be combined with '**guards**' (additional boolean conditions) to refine matching criteria

```java
enum TrafficLight { RED, YELLOW, GREEN }

String handleMessage(TrafficLight light) {
    return switch (light) {
        case RED     -> "Stop!";
        case YELLOW  -> "Prepare to stop";
        case GREEN   -> "Go!";
        default      -> "Unknown signal"; // Should rarely happen
    };
}
```

Pattern matching offers powerful data extraction mechanisms in switch expressions and statements

```java
record Rectangle(int width, int height) {}
record Circle(int radius) {}
record Triangle(int base, int height) {}

String calculateArea(Shape shape) {
    return switch (shape) {
        case Rectangle(int w, int h) -> "Rectangle area: " + (w * h);
        case Circle(int r)           -> "Circle area: " + (Math.PI * r * r);
        case Triangle(int b, int h)  -> "Triangle area: " + (0.5 * b * h);
        default                      -> "Unsupported shape";
    };
}
```

Let's craft a **more complex example** to demonstrate the expressive power of **pattern matching in switch within Java 21**

**Scenario: Parsing Geometric Shapes**

Suppose we have geometric shapes represented with a hierarchy of sealed classes and records:

```java
sealed interface Shape permits Circle, Rectangle, Triangle, ComplexShape {}

record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
record Triangle(double base, double height) implements Shape {}
record ComplexShape(List<Shape> subShapes) implements Shape {}
```

**Goal: Calculate the combined area of shapes with intricate structures**:

```java
public static double calculateTotalArea(Shape shape) {
    return switch (shape) {
        case Circle(double radius) -> Math.PI * radius * radius;
        case Rectangle(double width, double height) -> width * height;
        case Triangle(double base, double height) -> 0.5 * base * height;
        case ComplexShape(List<Shape> subShapes) -> 
            subShapes.stream()
                     .mapToDouble(ShapeUtils::calculateTotalArea) // Recursive call
                     .sum();
        default -> throw new IllegalArgumentException("Unknown shape type");
    };
}
```

**Key Points**

**Sealed Hierarchy**: The sealed keyword ensures all possible shape types are known (enhancing pattern matching safety).

**Type Patterns**: We match against specific types (Circle, Rectangle, etc.)

**Record Patterns**: Deconstruct records instantly to extract components (radius, width, height).

**Nested Patterns**: The ComplexShape case shows how patterns can be nested to handle recursive structures.

**Guarded Patterns**: We could add guards to patterns (e.g., case Triangle(double base, double height) when base > 0) for even finer-grained matching.

## 2. Record Patterns

Record patterns streamline working with record types, providing concise syntax for deconstructing them

```java
record Point(int x, int y) {}

void movePoint(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        // x and y are in scope here
        System.out.println("Coordinates: (" + x + ", " + y + ")");
    }
}
```

Record patterns can be **nested** to extract data from more complex record hierarchies:

```java
record Customer(String name, Address address) {}
record Address(String street, String city, String zipCode) {}

void processCustomer(Customer customer) {
    if (customer instanceof Customer(String name, Address(String _, String city, _))) {
        if (city.equals("New York")) {
            System.out.println("Customer " + name + " is from New York.");
        }
    }
}
```

Let's dive into more **advanced and complex** scenarios demonstrating the power of **Record Patterns in Java 21**

**Scenario 1: Deeply Nested Data Structures**

Imagine we're working with a user information structure nested several levels deep:

```java
record Address(String street, String city, String zipCode) {}
record Customer(String name, Address address) {}
record Order(int orderId, Customer customer, List<LineItem> items) {}
record LineItem(String product, int quantity) {}
```

**Task: Get Cities of Customers With Large Orders**

```java
void processOrders(List<Order> orders) {
    Set<String> citiesWithLargeOrders = orders.stream()
                                              .filter(order -> largeOrder(order)) 
                                              .flatMap(order -> extractCities(order).stream()) 
                                              .collect(Collectors.toSet());

    System.out.println(citiesWithLargeOrders);
}

private boolean largeOrder(Order order) {
    // ... logic to determine if an order is large
    return order.items().stream().mapToInt(LineItem::quantity).sum() > 100;
}

private List<String> extractCities(Order order) {
    if (order instanceof Order(_, Customer(_, Address(_, String city, _)), _)) {
        return List.of(city);
    } else {
        return List.of();
    }
}
```

**Scenario 2: Polymorphism & Record Patterns**

Consider a scenario where we have an abstract base class, with derived classes implemented as records:

```java
abstract class ApiResponse {}

record SuccessResponse(String message, Object data) extends ApiResponse {}
record ErrorResponse(int code, String message) extends ApiResponse {}
```

**Task: Selective Data Extraction**

```java
void handleApiResponse(ApiResponse response) {
    if (response instanceof SuccessResponse(String msg, Object data)) {
        // Handle successful data extraction (knowing the type of 'data')
        System.out.println("Success: " + msg); 
        // Process the 'data' object based on its expected type
    } else if (response instanceof ErrorResponse(int code, String errMsg)) {
        // Handle error scenario based on error code and message
        System.out.println("Error (" + code + "): " + errMsg);
    } 
}
```

**Key Points**:

**Deep Nesting**: Record patterns elegantly deconstruct even several layers deep

**Type Specialization**: When 'data' in SuccessResponse has a specific type, add it to the pattern for direct access (e.g., SuccessResponse(..., Customer data))

**Polymorphism**: Combine the power of inheritance and record patterns for specific handling based on concrete types
