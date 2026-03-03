Rope is a string balanced binary tree data structure, to make insertions, deletions, and slicing efficient for very large strings.

Rope is binary tree 🌳.
Rope tree consists of
- 🌿 leaf node holds a small chunk of text (limited by defined threshold) (like a file 📄)
- internal node (branch) which stores (like a folder 📁):
	- A pointer to its children
	- The weight of its **whole** left node (number of character) which basically allows you to understand how many characters are on the left, which makes it efficiently locate character by O(log n)



![[Pasted image 20250420201052.png|800]]


Methods:
- insert: insert at index
- delete
	- goes through the tree
	- if index greater than current weight goes right
	- removes that weight from index that you search with
	- if is leaf, convers grapheme index to bytes index, by traversing through graphemes in data grabbing width (indeces required per grapheme) grapheme_index times, and then using bytes_index to remove specific index in bytes data (maybe store graphemes as well, to not convert data to grapheme alwasy?)
- char_at:
	- goes through the tree
	- if greater than current weight goes right
	- removes that weight from index that you search with
- cancat
- split:
	- You split the **leaf's string** into **two halves**:
	- Then you **create a new internal node** to hold them:
	- You return this internal node in place of the original leaf.
- length
- rebalance: when you insert many at one end, it will be overweighted in one side, which makes one side with too many recursive calls.
- is_balanced
- collect_leaves_in_order
