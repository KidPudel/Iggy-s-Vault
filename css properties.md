
# Important CSS properties of selectors to know !!

- `display`: specifies how an element should be displayed (e.g. block, inline-block).
  - `bloc`, `inline` and `inline-bloc`: defines how to treat element, as an inline or a bloc that takes up whole width
  - `flexbox` and `grid`: layout systems for creating flexible, responsive designs.\
    - `flex-direction` set column-based or row-based
    - `flex-shring` in child, we can set it to 0, so it won't shrink at all
    - `flex-grow` in child, we can set it to 1(weight), so it will scale as our screen stretches
    - `flex` for both
    - `align-self` in child, we can override cross-axis for each item
    - `order` in child, we can change the order in which element is positioned
- `background-color`: sets the background color of an element.
- `color`: sets the text color of an element.
- `border-color`: color of a border
- `font-size`: sets the size of the text.
- `font-style`: to set a style for example, italic
- `font-family`: sets the font family for the text.
- `font-weight`: sets the weight (boldness) of the text.
- `line-height`: sets the height of each line of text.
- `text-align`: sets the horizontal alignment of text within an element.
- `width` and `height`: sets the width and height of an element.
- `margin` and `padding`: controls the spacing around an element.
- `border`: sets the border around an element.
- `box-sizing`: specifies how the **_total_** width and height of an element should be calculated.
  - `content-box`: This is the default value. When you set the width and height of an element, you're setting the size of the content area inside the element. The padding and border are added to the outside of the content area, which means they increase the overall size of the element.
  - `border-box`: With this value, the width and height of an element are calculated to include the padding and border. This means that if you set the width and height of an element, the total size of the element will be that size, including any padding and border. The content area will then be calculated to fit inside that total size.
- `overflow`: controls how content exceeded of the box dimentions should be handles
  - `visible` (default) - The content is not clipped and may be rendered outside the box.
  - `hidden` - The content is clipped and any content that exceeds the dimensions of the box is hidden.
  - `scroll` - The content is clipped and a scrollbar is added to see the rest of the content.
  - `auto` - The content is clipped and a scrollbar is added only when necessary.
- `position`: specifies the positioning method for an element (e.g. relative, absolute, fixed).
- `z-index`: sets the stacking order of elements.
- `float`: positions an element to the left or right of its container.
- `padding, padding-left, etc`
- `margin, margin-left, etc`
- `box-shadow`: shadow

# Alignment
- `text-align` - align text on the main axis
- `justify-center` IN FLEXBOX align main axis
- `align-items` IN FLEXBOX align cross axis

# Flexbox

```css
.App {
    display: flex;
}
```  
**_will create flexbox container_**

in flexbox there is a main and cross access represented by `justify-center` and `align-items` respectively

<img src="https://user-images.githubusercontent.com/63263301/229141149-51268235-fb6a-44bc-9309-ba8ae5f6477b.png" width=500px alt="image">
<img src="https://user-images.githubusercontent.com/63263301/229141218-9420920e-e0a7-4afb-acfa-d3f20e7c9220.png" width=500px alt="image">

# Make it shine!
```css
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.react:hover {
  filter: drop-shadow(0 0 2em #61dafbaa);
}
```
![shine_vite_react](https://user-images.githubusercontent.com/63263301/229606711-de23da93-ba8f-437a-b265-017aef01194e.gif)