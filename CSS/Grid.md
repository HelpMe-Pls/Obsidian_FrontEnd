- _CSS **Grid**_ is a newer CSS layout algorithm designed for 2D layouts with various numbers of rows and columns.

- A _grid container_ is defined by adding a `display: grid` rule to an element. All of the immediate children are then laid out automatically as _grid items_.

- Grid syntax is extensive, and allows defining sets of columns and rows with varying repeated patterns:
  - For the grid *container*:
	  1. `grid-gap`:  specifies the size of column and row gutters
	  2. `justify-items`: places the grid items along the *row* axis. Its common values are `start` / `center` / `end` / `stretch`
	  3. `justify-content`: set the alignment *of the grid* within the grid container on row axis (when total grid size is ***smaller*** than container; this could happen if all of your grid items are sized with non-flexible units like `px`). Its common values are `start`/ `center` / `end` / `stretch` / `space-between` / `space-around` / `space-evenly` 
	  4. `align-content`: same as `justify-content` but for column axis (block axis)
	  5. `place-content`: sets both the `align-content` and `justify-content` properties in a single declaration (the first value sets `align-content`, the second value `justify-content`. If the second value is omitted, the first value is assigned to both properties)
	  6. `align-items`: same as `justify-items` but for column axis.

	- For the grid *items*:
		1. `justify-self`: places *the content* inside a cell along the _inline (row)_ axis. Its common values are `start` / `center` / `end` / `stretch`
		2. `align-self`: same as `justify-self` but for the _block (column)_ axis.
