1. **Single Responsibility Principle (SRP):**
   A type should have only one reason to change.

   ```go
   package main

   import "fmt"

   // Printer has the responsibility of printing messages.
   type Printer struct{}

   func (p *Printer) Print(message string) {
       fmt.Println(message)
   }

   // User has the responsibility of managing user-related operations.
   type User struct {
       Name string
   }

   func (u *User) Greet(p *Printer) {
       p.Print("Hello, " + u.Name)
   }

   func main() {
       printer := &Printer{}
       user := &User{Name: "John"}
       user.Greet(printer)
   }
   ```

2. **Open/Closed Principle (OCP):**
   Software entities should be open for extension but closed for modification.

   ```go
   package main

   import "fmt"

   // Greeter defines the interface for greeting.
   type Greeter interface {
       Greet() string
   }

   // EnglishGreeter implements the Greeter interface for English greetings.
   type EnglishGreeter struct{}

   func (eg *EnglishGreeter) Greet() string {
       return "Hello"
   }

   // GreetPrinter prints the greeting.
   type GreetPrinter struct{}

   func (gp *GreetPrinter) PrintGreeting(greeter Greeter) {
       fmt.Println(greeter.Greet())
   }

   func main() {
       englishGreeter := &EnglishGreeter{}
       greetPrinter := &GreetPrinter{}

       greetPrinter.PrintGreeting(englishGreeter)
   }
   ```

3. **Liskov Substitution Principle (LSP):**
   Subtypes must be substitutable for their base types without altering the correctness of the program.

   ```go
   package main

   import "fmt"

   // Shape defines the interface for shapes.
   type Shape interface {
       Area() float64
   }

   // Rectangle implements the Shape interface for rectangles.
   type Rectangle struct {
       Width  float64
       Height float64
   }

   func (r *Rectangle) Area() float64 {
       return r.Width * r.Height
   }

   // Circle implements the Shape interface for circles.
   type Circle struct {
       Radius float64
   }

   func (c *Circle) Area() float64 {
       return 3.14 * c.Radius * c.Radius
   }

   // CalculateArea prints the area of a shape.
   func CalculateArea(shape Shape) {
       fmt.Println("Area:", shape.Area())
   }

   func main() {
       rectangle := &Rectangle{Width: 5, Height: 3}
       circle := &Circle{Radius: 2}

       CalculateArea(rectangle)
       CalculateArea(circle)
   }
   ```

4. **Interface Segregation Principle (ISP):**
   Clients should not be forced to depend on interfaces they do not use.

   ```go
   package main

   import "fmt"

   // Worker interface defines methods for a worker.
   type Worker interface {
       Work()
   }

   // Eater interface defines methods for an eater.
   type Eater interface {
       Eat()
   }

   // Robot implements both Worker and Eater interfaces.
   type Robot struct{}

   func (r *Robot) Work() {
       fmt.Println("Robot is working.")
   }

   func (r *Robot) Eat() {
       fmt.Println("Robot is eating.")
   }

   func main() {
       robot := &Robot{}
       // The client can use only the methods it needs.
       robot.Work()
   }
   ```

5. **Dependency Inversion Principle (DIP):**
   High-level modules should not depend on low-level modules; both should depend on abstractions.

   ```go
   package main

   import "fmt"

   // Messenger defines the interface for sending messages.
   type Messenger interface {
       Send(message string)
   }

   // Email implements the Messenger interface for sending emails.
   type Email struct{}

   func (e *Email) Send(message string) {
       fmt.Println("Sending email:", message)
   }

   // SMS implements the Messenger interface for sending SMS.
   type SMS struct{}

   func (s *SMS) Send(message string) {
       fmt.Println("Sending SMS:", message)
   }

   // NotificationManager depends on the Messenger interface.
   type NotificationManager struct {
       Messenger
   }

   func main() {
       email := &Email{}
       sms := &SMS{}

       emailNotifier := &NotificationManager{Messenger: email}
       smsNotifier := &NotificationManager{Messenger: sms}

       emailNotifier.Send("Hello, via email")
       smsNotifier.Send("Hi, via SMS")
   }
   ```

These examples demonstrate how Go (Golang) can adhere to the SOLID principles to create maintainable and flexible code. Keep in mind that the implementation of these principles may vary based on the specific requirements of your application.