# **Use Constructor Dependency Injection**

It's advisable to rely on dependency injection via the constructor whenever possible. Directly autowiring fields can significantly limit flexibility, as it restricts object instantiation to the Spring configuration context. This limitation becomes apparent in testing scenarios, where you must either depend on a fixed Spring test application configuration or resort to workarounds like using `@InjectMocks`.

**Benefits :**

1. **Simplified Testing:** Tests become more straightforward, and you can avoid relying on Mockito's hidden tricks or using `@InjectMocks`.
    
2. **Encapsulation Integrity:** Avoid the need for annotations like `@VisibleForTesting`, which may break encapsulation.
    
3. **Clear Object Creation:** The constructor's signature conveys the only valid way to create an instance of a class, preventing "half-created objects."
    
4. **Design Consideration:** Adding new properties remains straightforward, but an expanding constructor should prompt a reevaluation of the class's design. An overly complex constructor with many dependencies may indicate a violation of the Single Responsibility Principle, suggesting that the class should be split into smaller, more focused components.
    
5. **Transparency:** Dependencies are clearly visible to all clients of the class, enhancing code transparency and maintainability.
**Example**

1. <span style="color:red; font-weight:900">Avoid</span>
``` java
@Component
public class Car {

 @Autowired
private Engine engine;

@Autowired
private Transmission transmission;
...
}
```

2. <span style="color:green; font-weight:900">Prefer</span>
``` java
@Component
public class Car {

  private final Enigne engine;
  private final Transmission transmission;
  @Autowired
  public Car(Engine engine, Transmission transmission) {

   this.engine = engine;
    this.transmission = transmission;

}
}
///Inject With Qualifier
@Controller
public class DisposeController {

    private final Disposer fileDisposer;
    private final Disposer printerDisposer;
    @Autowired
    public DisposeController(@Qualifier("disposeFileService") Disposer

fileDisposer,

                             @Qualifier("disposePrinterService")

Disposer printerDisposer) {
        this.fileDisposer = fileDisposer;
        this.printerDisposer = printerDisposer;

} }
```

# @Transactional at Method Level

A recommended practice is to apply the `@Transactional` annotation at the method level. From a code quality perspective, this approach offers several advantages:

1. **Isolation of Transaction Logic:** By annotating methods with `@Transactional`, you ensure that only the intended methods execute within a transaction context. This prevents the unintentional inclusion of new methods that would unnecessarily consume a transaction from the connection pool, even if they have no database interactions.
    
2. **Read-Only Queries Optimization:** For read-only queries, consider adding `@Transactional(readOnly = true)`. This optimizes the process by indicating that the method only performs read operations on the database, allowing the underlying infrastructure to potentially optimize the transaction handling for improved performance.
    

By following this practice, you maintain better control over transactional behavior in your code, reduce the risk of unintentional transactions, and optimize database interactions, ultimately enhancing code quality and efficiency.

**Benefits:**

1. **Improved Performance:** Avoid the performance penalty associated with long transactions by limiting the scope of transactions to only the necessary parts of the code.
    
2. **Risk Mitigation:** Eliminate the risk of inadvertently adding new methods that unintentionally become transactional and consume resources from the connection pool, especially when they have no interactions with the database.

**Example:**
``` java
@Transactional // remove it!
public class FooService { ... }

// Step 2: Move @Transactional at method-level

public class FooService {
   ...

}

@Transactional
public int updateFoos(...) { ... }

@Transactional(readOnly = true)
public List<Foo> fetchFoos(...) { ... }

// no transaction needed
public void moveFoosAround(...) { ... }

// Alternatively, if the developer really needs to use @Transactional
at class level,
// then for a method that should not run in a transactional environment:
/*

* Use Propagation.NEVER - The NEVER behavior states that an existing
opened
transaction must not already exist. If a transaction exists an
exception will be

thrown by the container.
*/
@Transactional(propagation=Propagation.NEVER)
public void moveFoosAround(...) { ... }

/*
* Use Propagation.NOT_SUPPORTED - The NOT_SUPPORTED behavior will
execute outside of
the scope of any transaction. If an opened transaction already exists
it will be paused.
*/
@Transactional(propagation=Propagation.NOT_SUPPORTED)
public void moveFoosAround(...) { ... }
```
