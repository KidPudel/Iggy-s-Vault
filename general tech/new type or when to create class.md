Type is the **classification** of the data, that tells the [[compiler]] or [[interpreter]] how data should be represented, how it should look, what have, what to do, and what it is at all.

Act of defining a new type, is the **urge for a new classification**, therefore - **urge for the ability of creation of new type of entity**. 

Reasons:
- Organization: class lets you organize data and procedure in one *idiomatic* way, as well as *reusability* in case of structured methods
- State: class lets you keep track of the state, as long as the object lives and is easily accessible in contrast to functional programming.
- Inheritance


For example, we writing an API, were we constantly access the user (it's existence) and it have couple of repetitive functionality, makes sense to create a class.
Why class and not just an arbitrary function? because it is *arbitrary*, class will help with organization

When to avoid:
- When we just need to store data. use *dataclass* or *named tuple*
- When it has one method. Use just function instead

# when to use function and class
- function -> code action-focused
- class -> code state-focused
- combination -> in class a lot of action
[idea from](https://youtube.com/shorts/oIyq0q5Q7eo?si=O5L6g_oSah6UsvDt)