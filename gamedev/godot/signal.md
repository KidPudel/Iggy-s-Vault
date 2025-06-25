Signal in Godot is basically a pub/sub patter like [[event bus]], where you have a signal, that you can emit, and subscribed/connected nodes via functions gets notified (connected function gets called).

Perfect for side effect and cascade calls.

Exposes, makes it global and common