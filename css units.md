In CSS, there are different types of units that can be used to specify values for properties such as width, height, font size, and margin. Here are some of the most commonly used CSS units:

1. Pixels (`px`): Pixels are a fixed unit of measurement that are used to define exact sizes. One pixel represents one dot on the screen. The size of a pixel may vary depending on the screen resolution.

2. Percentages (`%`): Percentages are a **_relative unit of measurement that are based on the size of the parent element_**. For example, if a parent element is 200px wide and a child element is set to be 50% wide, it will be 100px wide. **like Flexible and FractionallySizedBox in Flutter**

3. Viewport Width (`vw`): Viewport width is a **_relative unit of measurement that is based on the width of the browser window_ (and not on a parent like `%`)**. 1vw is equal to 1% of the viewport width. For example, if the viewport is 1000px wide, 1vw would be equal to 10px. **Like MediaQuiery in Flutter width**

4. Viewport Height (`vh`): Viewport height is similar to viewport width, but it's based on the height of the browser window instead. **Like MediaQuiery in Flutter height**

5. Em (`em`): Em is a relative unit of measurement **_that is based on the font size of the parent element_**. For example, if the font size of a parent element is 16px, and a child element is set to 1em, it will be 16px. If the font size of the parent element is changed to 20px, the child element will be 20px. **Just like px but accounting font size**

6. Rem (`rem`): Rem is similar to Em, but it's based on the **_font size of the root element (usually the <html> tag)_**. This can be useful for creating more consistent layouts across multiple pages or components. **root em**

7. Absolute (`in`, `cm`, `mm`): Absolute units are based on physical measurements, such as inches, centimeters, and millimeters. These units are rarely used in web design, as they are not very flexible or adaptable to different screen sizes.

8. Flexible (fractional) (`fr`): Flex units are a flexible unit of measurement that are used in CSS Grid layouts. 1fr represents one fraction of the available space in the grid container. **Like Flexible in Flutter or fractionallysizedbox**

# Showcase
![image](https://user-images.githubusercontent.com/63263301/229217559-8c140797-6285-4e52-997b-5c938ede6c9b.png)
![image](https://user-images.githubusercontent.com/63263301/229218927-53bd8b3b-e39d-4a87-b0a4-f6e72d2fb043.png)




# Rule of thumb
- font-size: `rem`
- width: `%` in combination with a `max-width (stretch)`, `ch`
- height: question urself "do i rly need to set height" if yes -> use a `min-height`
- padding/margin: `rem` or `em`, kevin often uses `em` for padding of buttons
- media queries: `em`
- `px` only for little things like shadows, borders etc.