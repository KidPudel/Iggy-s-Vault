technique used to rename class attribute names with 2 leading underscores (`__`) at the front, to a different name to avoid naming conflicts with attributes in a subclass.

with `__private_attribute` Python automatically renames it by prepending the class name and adding an additional underscore before the name: `_ClassName__private_attribute`.