Pseudo-selectors in CSS are used to style a specific part of an element  
1. **`:hover`:**
   - Targets an element when the mouse pointer is over it.
   ```css
   button:hover {
     background-color: #abcdef;
   }
   ```

2. **`:active`:**
   - Targets an element when it is being activated, typically by clicking.
   ```css
   button:active {
     transform: translateY(2px);
   }
   ```

3. **`:focus`:**
   - Targets an element that has keyboard focus.
   ```css
   input:focus {
     border-color: #3498db;
   }
   ```

4. **`:first-child` and `:last-child`:**
   - Targets the first and last child of a parent element.
   ```css
   li:first-child {
     margin-left: 0;
   }

   li:last-child {
     margin-right: 0;
   }
   ```

5. **`:nth-child()`:**
   - Selects elements based on their position in a parent, using a formula.
   ```css
   tr:nth-child(even) {
     background-color: #f2f2f2;
   }
   ```

6. **`:not()`:**
   - Selects elements that do not match a given selector.
   ```css
   input:not([type="submit"]) {
     border: 1px solid #ccc;
   }
   ```

7. **`:nth-of-type()` and `:nth-last-of-type()`:**
   - Similar to `:nth-child()`, but only counting elements of the specified type.
   ```css
   p:nth-of-type(odd) {
     color: #ff0000;
   }
   ```

8. **`:first-of-type` and `:last-of-type`:**
   - Selects the first and last occurrence of a specified type of element.
   ```css
   span:first-of-type {
     font-weight: bold;
   }
   ```

9. **`:nth-last-child()`:**
   - Selects elements based on their position counting from the last child.
   ```css
   li:nth-last-child(2) {
     color: #333;
   }
   ```

10. **`:empty`:**
    - Selects elements that have no children.
    ```css
    div:empty {
      display: none;
    }
    ```
