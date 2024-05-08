# learning-bot-creation
this is a small as possible project written in a "mess around" manner to learn how to make bots with python

Dispatcher is the main router, they are needed for handling incoming updates from Telegram
- Routes are a way to process events
- Routes has some specified events they can handle, as depicted below  
![image](https://github.com/KidPudel/learning-bot-creation/assets/63263301/3abed1ae-a5f5-45eb-aded-ee43a4327d29)

# event handlers
[event handlers](https://docs.aiogram.dev/en/dev-3.x/dispatcher/router.html)  
[official docs](https://core.telegram.org/bots/api#august-18-2023)




> We register endpoints at which bot will trigger, and below, what it would use to handle that request - handler
> Note: that the *request* (data with which user triggered endpoint of some type), is also captured by the endpoint
### process of handling
1. looking through current router (from outter -> nested) first is Dispatcher
2. in router looking sequentially, from top to bottom (the first that satisfy filters (non-filtered handler is always the strongest))
the order is matter

# methods
[list of methods](https://docs.aiogram.dev/en/dev-3.x/api/methods/index.html)

# filters
[filters](https://docs.aiogram.dev/en/dev-3.x/dispatcher/filters/command.html) are for events (updates) to be routed to the specific handler that match filter set.  
Searching filters is always stops on first match set of filters are passed (empty filters are always passes, so **the order is matter**) . By defauilt all updates has empty filter set, so all updates will be passed to the handler that has no filters

# state machine
set a state of the bot stage

This is done with a StatesGroup, just define a states, to determine what step is now and what user must enter
(like with enum group), just set state and then in a filters of a handler add needed state
[fine-state-machine](https://docs.aiogram.dev/en/dev-3.x/dispatcher/finite_state_machine/index.html)

# [[middleware]]
middleware reusable software that is a bridge between the functional requirement and operating system, database, internet protocols.  

It is a function that is triggered on every update from telegram bot api multiple times in a pipeline.  
It can be applied for every event type (Message, Update, etc), in two places
1. Outer scope - on each handler (before filters) `<router>.<event>.outer_middleware()`
2. Inner scope - after passing filters, but before handler `<router>.<event>.middleware()`
https://docs.aiogram.dev/en/dev-3.x/_images/basics_middleware.png

> Middleware SHOULD be a subclass of BaseMiddleware  

> To propagate event to the next middleware/handler, middleware should call `await handler(event, data)`. Otherwise it's going to stop processing event in middleware



# Callback
## Callback queries
Incoming objects from callback buttons in inline/reply keyboard


## Callback data
those callback queries carries to the handlers data that is being set in button
This data is defined from base CallbackData class
```python
class MyCallbackData(CallbackData, prefix="my"):
	more_info: str
```

this data we can `pack` into a button to transfer and `unpack` to read, and `filter` to... *filter*

> NOTE: you can also pass bot itself in a function handler