**Single responsibility** → class should has ***1 reason to change and method 1 purpose.*** just clear and right organisation of the code
    
**Open-Closed Principle** → must be opened for extension and closed for modifications. we can have abstract classes and we can inherit and also extension keyword, so we can use parents methods and add ours

**Liskov Substitution** → subclass should be able to replace objects of a superclass without errors in a program
    
**Interface Segregation** → interfaces should not force clients to implement something they don’t need, ***interfaces should not be monolithic*** (cleaner and more maintainable code)
    
**Dependency Inversion** → high-level modules (application logic) should not depend on low-level modules (some specific implementations of some part like repositories or db calls) but both should depend on abstract (interface of abstract class) (lose coupling and dependency injection)