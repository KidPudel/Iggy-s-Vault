Resource is the base class for all Godot-specific resource types, serving primary as **data containers**, it allows for modeling [[static]] data.
They inherit from RefCounter [[reference counter]], meaning we can reference count it and free when not needed.
By extending class with `Resource`, we add the ability to export a data.


Example would be a `PackedScene` which is a resource able to store and instantiate a [[scene]] it contains as many times as needed.
So basically `PackedScene` is a built-in type representing a scene resource

In GDScript, resources can loaded from disk by their [resource_path](https://docs.godotengine.org/en/stable/classes/class_resource.html#class-resource-property-resource-path) using [@GDScript.load()](https://docs.godotengine.org/en/stable/classes/class_%40gdscript.html#class-gdscript-method-load) or [@GDScript.preload()](https://docs.godotengine.org/en/stable/classes/class_%40gdscript.html#class-gdscript-method-preload).
But consider `load_threaded_request

> The engine keeps a global cache for all loaded resources, referenced by paths (see [ResourceLoader.has_cached()](https://docs.godotengine.org/en/stable/classes/class_resourceloader.html#class-resourceloader-method-has-cached))
A resource will be cached to the disk for the first time it is loaded, and will be in the cache as long as the last referenced is released. all subsequent load calls will return cached result.

> To avoid unnecessary delays when loading something multiple times, either store the resource in a variable or use [preload()](https://docs.godotengine.org/en/stable/classes/class_%40gdscript.html#class-gdscript-method-preload).
This method is equivalent of using [ResourceLoader.load()](https://docs.godotengine.org/en/stable/classes/class_resourceloader.html#class-resourceloader-method-load) with [ResourceLoader.CACHE_MODE_REUSE](https://docs.godotengine.org/en/stable/classes/class_resourceloader.html#class-resourceloader-constant-cache-mode-reuse).


load or preload literally loads a resource and then you can **instantiate new** instance of a node/scene
> NOTE: we can use ResourceLoader to use something like `ResourceLoader.load_threaded_request("path")`, `ResourceLoader.load_threaded_get_status("path")` and `ResourceLoader.load_threaded_get("path")` and we can save using `ResourceSaver.save()`

`preload()` when you require resources to be available immediately and consistently throughout your game or when optimizing performance is crucial.
For dynamic or occasional resource needs, opt for `load()` or the resource loader methods.


Examples of resource:
- [[scene]] `PackedScene`
- any sort of assets
- animation
- collision shape
- *custom resources* for something like [[data-driven design in godot]]

Note: -  it’s _generally better to store the “owns many” relationship on the parent_ (e.g. Column has an array of Cards) rather than modeling “many-to-one” from the children’s side (e.g. each Card storing its own column_index).
Because: Godot's resource system is hierarchical and tree-oriented, and data structures underneath benefits that approach. It is better to make owner to "own" its sub-resources, it makes it clear, who manages them, simplifies serialization, avoids dangling pointers.

Note 3: static data like the template, definition, during design time is stored in `res://` is immutable, while saving state or just mutable data should be in the `user://` 


