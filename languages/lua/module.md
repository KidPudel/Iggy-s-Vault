Module is the file containing [[table]] (typically named after a module) for organization, containing utility code that other code can depend on.

- **Characteristics:**
    - **Single instance:** When a module is `require`d, it is loaded once, and subsequent `require` calls for the same module will return the same table.
    - **Encapsulation:** Modules provide a way to hide internal implementation details by only exposing a specific interface (the returned table).
    - **Global or Local Use:** Modules can be used globally or assigned to local variables for more controlled access.