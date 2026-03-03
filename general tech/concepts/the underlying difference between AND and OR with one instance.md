~~AND is *composing*, *tightly bounding* , meaning, combining two statements with &&, makes it one dependable statement~~
~~OR is an *alternative* case, meaning it is separate from the first one~~

~~That's why checking if `status` != `created` || `status` != `success`, will **always** return true, because always one of the cases is true (because  one of them is not, because inverse), it is a "*rush*" case, ***obviously X cannot be Y and Z at the same time***, and it just jumps to body at one of them, but here it is `!=`, so it doesn't make any sense~~
~~So the *sane* case would be if `status` != `created` && `status` != `success`, now we are really checking if status is not one of them.~~

~~But if we are, not looking for the rest (discarding one of them), but rather looking for the specific to go to the case, then if `status` == `success` || `status` == `paid` => `cool`, would be the one. it is inverse.~~
- ~~looking for broad, discarding some -> **combine** what you don't want~~
- ~~looking for specific (any one of them)  -> **separate** with parts/**alternatives**~~

with OR we want to take **at least one** of the cases
with AND we want to take **all** of the cases