The size of a given *element* is made up of several pieces (going inside-out):
1.  The actual content
2.  **Padding**: the spacing around the content but still *inside* the element
3.  **Border** : the lines that *wrap around* the padding
4.  **Margin**: the spacing *outside* the border

The `box-sizing` property with `content-box` value gives you the *default* CSS box-sizing behavior (i.e the size of your element is calculated by adding the **actual** measurements of *padding + border **+ content***), therefore, the final rendered result could be larger than expected.
What you need is *almost **always*** use the `box-sizing` property with `border-box` value so that the size of your element is calculated by automatically *adjusting* the measurements of *padding + border + content* to ***fit*** the given dimensions (i.e. the content could be shrinked responsively and the final rendered result includes all of its border and padding as well).