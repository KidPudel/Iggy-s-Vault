Rule of thumb.
It is fine to call functions on the nodes below (children) the caller (parent), but not wise versa, **instead** you should use signals of the children and emit it on a children and that way connected listener in a parent will get the information
If you need to communicate between 2 siblings, then it is a good practice to connect the "wire" between the parent.