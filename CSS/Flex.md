- **Flexbox** is a CSS *layout algorithm* that allows aligning items into rows or columns, with rules for aligning items and wrapping them as they overflow.

- A _flex parent_ (or the **flex container**) is defined by adding a `display: flex` rule to an element. All of the *immediate children* are then laid out automatically as _flex items_.

- Key properties (for the flex container):
1.  `flex-direction`: establishes the main layout axis (`row` / `column` / `row-reverse` / `column-reverse`)
2.  `justify-content`: attempts to distribute extra space on the main layout axis (`flex-start` /  `center` / `flex-end` / `space-between` / `space-around` / `space-evenly` / `stretch`)
3.  `align-items`: determines how flex items are laid out on the cross axis (`flex-start` / `center` / `flex-end` / `stretch` /`baseline`)

- Centering items both vertically and horizontally had been an exceptionally difficult task on the web for many years, but Flexbox allows centering things fairly easily, such as `display: flex; justify-content: center; align-items: center;`.