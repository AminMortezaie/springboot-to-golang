### Why many Java teams avoid deep inheritance —and what they use instead

**Joshua Bloch’s *Effective Java* (Item 18) popularised the mantra “*favor composition over inheritance*.”** Inheritance can leak implementation details (“fragile-base-class” problem), make classes hard to reason about, and lock you into a tree you can’t easily reshape later — especially in large code-bases where several teams touch the same hierarchy. Bloch’s advice remains canonical in today’s Java ecosystem and is echoed by modern tutorials and community discussions.([Oracle Blogs][1], [DEV Community][2])

---

#### 1  ·  Plain-Java delegation (“wrapper”)

Instead of `class MyList extends ArrayList`, create a field and forward the calls:

```java
class MyList<E> implements List<E> {
    private final List<E> delegate = new ArrayList<>();

    @Override public boolean add(E e) {            // compile-time contract
        return delegate.add(e);                    // runtime behaviour
    }
    // …other methods forward here (IDE can generate) …
}
```

Because `MyList` no longer inherits internal state from `ArrayList`, changes inside `ArrayList` can’t break you, and you can swap the delegate for a different `List` at runtime or test time. The only real cost is boilerplate.

**Boilerplate-reduction helpers**

* **Project Lombok `@Delegate`** generates the forwarding methods for you, letting you keep the wrapper but ditch the tedium.([projectlombok.org][3])
* Google AutoValue / Immutables can synthesise immutable value objects so you never start with inheritance at all.

---

#### 2  ·  Interfaces + default methods = mixin-style composition

Since Java 8 you can put behaviour directly on an interface:

```java
interface Timestamped {
    Instant createdAt();
    default Duration age() { return Duration.between(createdAt(), Instant.now()); }
}
```

Any class can *compose* this capability simply by implementing the interface, no superclass needed. It’s an easy way to share behaviour across otherwise unrelated types without deepening inheritance trees.([Stack Overflow][4])

---

#### 3  ·  Sealed classes: inheritance with a safety-valve

When a closed hierarchy really is the right model (e.g., an expression tree), Java 17’s **sealed classes** let you spell the exact set of permitted subclasses:

```java
public sealed interface Expr
        permits Const, Plus, Times { … }
```

The compiler prevents “God-object” sub-trees from springing up elsewhere while still letting you pattern-match exhaustively in `switch`.([The Server Side][5])

---

#### 4  ·  Records & final classes encourage “no hierarchy” by default

`record Point(int x, int y) {}` is implicitly `final` and focuses on data, not behaviour, so developers reach for inheritance less often. If you need polymorphism later you wrap the record, rather than subclass it.

---

#### 5  ·  Dependency-Injection frameworks (Spring, Dagger, …)

DI makes composition the path-of-least-resistance: instead of subclassing to specialise, you *inject* collaborators that implement an interface. Spring’s IoC container literally wires those relationships at runtime, keeping classes flat and swappable.([Medium][6], [Home][7])

---

#### 6  ·  Classic design patterns that rely on composition

| Need                         | Pattern       | Sketch                                           |
| ---------------------------- | ------------- | ------------------------------------------------ |
| Replace behaviour at runtime | **Strategy**  | `new ImageRenderer(new PngEncoder())`            |
| Layer optional features      | **Decorator** | `new CompressingOutput(new LoggingOutput(base))` |
| Represent object state       | **State**     | `context.transition(new ApprovedState())`        |

Each keeps the inheritance graph shallow while giving you the polymorphism you actually want.

---

### Practical heuristics

* **Start “final + interface”**; open up only when a real generalisation appears.
* **Wrap, don’t extend, JDK collections** unless you *really* need the full protected API (rare).
* **If the hierarchy must exist, seal it** so its surface area stays intentional.
* **Reach for DI and patterns first**; reserve inheritance for “is-a” relationships that model the domain, not for code-reuse.

Modern Java therefore *doesn’t ban* inheritance, but it arms you with language features, libraries and architectural conventions that make composition cheaper and safer—so the path of least resistance is the one that keeps your codebase decoupled.([geeksforgeeks.org][8])

[1]: https://blogs.oracle.com/javamagazine/post/java-inheritance-composition?utm_source=chatgpt.com "You should favor composition over inheritance in Java. Here's why."
[2]: https://dev.to/kylec32/effective-java-tuesday-favor-composition-over-inheritance-4ph5?utm_source=chatgpt.com "Effective Java Tuesday! Favor Composition Over Inheritance"
[3]: https://projectlombok.org/features/experimental/Delegate "@Delegate"
[4]: https://stackoverflow.com/questions/31242036/inheritance-composition-and-default-methods "java - Inheritance, composition and default methods - Stack Overflow"
[5]: https://www.theserverside.com/tip/Use-sealed-classes-in-Java-to-control-your-inheritance "Use sealed classes in Java to control your inheritance | TheServerSide"
[6]: https://firattamur.medium.com/dependency-injection-ioc-a-fun-dive-into-composition-over-inheritance-58387daacfb4?utm_source=chatgpt.com "Dependency Injection & IoC: A Fun Dive into Composition Over ..."
[7]: https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html "Dependency Injection :: Spring Framework"
[8]: https://www.geeksforgeeks.org/favoring-composition-over-inheritance-in-java-with-examples/ "Favoring Composition Over Inheritance In Java With Examples | GeeksforGeeks"
