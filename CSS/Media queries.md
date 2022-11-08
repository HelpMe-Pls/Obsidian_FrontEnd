- Rather than make completely different sites, we add CSS rules that only apply in certain cases, such as when the viewing device screen is in a specific size range. This is normally done using _media queries_, which are CSS conditions that only apply the selectors and rules inside if the condition matches. 

- The CSS rules typically choose certain screen sizes as _breakpoints_ so that the layout changes if a device is larger or smaller than that size. An example media query and breakpoint setup might look like:

```scss
/* Small devices (portrait tablets and large phones, 600px and up) */
@media only screen and (min-device-width: 600px) {
  .info {
    color: red;
  }
}

/* Medium devices (landscape tablets, 768px and up) */
@media only screen and (min-device-width: 768px) {
  .info {
    color: green;
  }
}

/* Large devices (laptops/desktops, 992px and up) */
@media only screen and (min-device-width: 992px) {
  .info {
    color: blue;
  }
}
```