Tween is a general purpose animation via script
https://popcar2.itch.io/tweens-comparison

Remember that tween object is valid one time, and you need to create a new one
If you want to interrupt the current running and start a new one, `kill` previous first, and `create_tween` then.
> do *not* override the value object of the variable or you'll loose the reference and won't be able to finish running tween, and they will race for attention animating values together.