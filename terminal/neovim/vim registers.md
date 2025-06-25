#### **1. Unnamed & Yank Registers**

- `"` – Default register (last yank/delete).
    
- `0` – Last yanked text (does not store deletions).
    
- `_` – Black hole register (deletes without storing).
    

#### **2. Clipboard & Selection Registers**

- `+` – System clipboard (`"+y` to copy, `"+p` to paste).
    
- `*` – X11 primary selection clipboard (`"*y`, `"*p` on Linux).
    

#### **3. Delete Registers (`1` to `9`)**

- `1` – Most recent deletion.
    
- `2-9` – Older deletions (shifted down).
    

#### **4. Named Registers (`a` to `z`)**

- `a-z` – Manually store text (`"ay` to yank, `"ap` to paste).
    
- `A-Z` – Append to lowercase registers (`"Ay` appends to `a`).
    

#### **5. Expression & File Registers**

- `=` – Evaluate expression (`<C-r>=5+5<CR>` inserts `10`).
    
- `%` – Current file name.
    
- `#` – Alternate file name.
    
- `.` – Last inserted text.
    
- `/` – Last searched pattern.
    

#### **6. Read-Only Registers**

- `:` – Last command entered.
    
- `.` – Last inserted text.
    
- `%` – Current file name.