# HTML

- Tags are usually come in pairs (start-tag and end-tag) but sometime we have self-closing tags (`<br />`).
- Block-level unlike in-line elements start on a new line and take full width available by default
  `span`, `img`, `strong`, `label`, `input` and `a` are in-line elements.
- `em` is for italic and `strong` is for bold.
- All tags can have attributes.
- Attributes provide info. about an element.
- They are placed within start tag and are key/value pairs: `id="some-id"`.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Title in the tab</title>
  </head>
  <body>
    <!-- This is a comment -->
    <h1>This is a heading 1</h1>
    <p>This is a paragraph</p>

    <a href="blog.html">Go to blog</a>
    <hr />

    <!-- Headings -->
    <h1>Heading One</h1>
    <h2>Heading Two</h2>
    <h3>Heading Three</h3>
    <h4>Heading Four</h4>
    <h5>Heading Five</h5>
    <h6>Heading Six</h6>

    <!-- Paragraph -->
    <p>
      <!-- By default paragraph has margin-top and bottom of 1em -->
      Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
      tempor
      <strong>incididunt ut labore</strong> et dolore magna aliqua. Ut enim ad
      minim veniam, quis nostrud <em>exercitation ullamco</em> laboris nisi ut
      aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in
      voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur
      sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
      mollit anim id est laborum.
    </p>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipisicing elit,
      <a href="http://google.com" target="_blank">sed do eiusmod</a>
      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
      veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
      velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
      cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id
      est laborum.
    </p>

    <!-- Lists -->
    <ul>
      <li>List Item 1</li>
      <li>List Item 2</li>
      <li>List Item 3</li>
      <li>List Item 4</li>
    </ul>

    <ol>
      <li>List Item 1</li>
      <li>List Item 2</li>
      <li>List Item 3</li>
      <li>List Item 4</li>
    </ol>

    <!-- Table -->
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Age</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Brad Traversy</td>
          <td>brad@something.com</td>
          <td>35</td>
        </tr>
        <tr>
          <td>John Doe</td>
          <td>jdoe@something.com</td>
          <td>45</td>
        </tr>
        <tr>
          <td>Sara Williams</td>
          <td>sara@something.com</td>
          <td>25</td>
        </tr>
      </tbody>
    </table>

    <br />
    <hr />
    <br />

    <!-- Forms -->
    <form action="process.php" method="POST">
      <div>
        <label>First Name</label>
        <input type="text" name="firstName" placeholder="Enter first name" />
      </div>
      <br />
      <div>
        <label>Last Name</label>
        <input type="text" name="lastName" />
      </div>
      <br />
      <div>
        <label>Email</label>
        <input type="email" name="email" />
      </div>
      <br />
      <div>
        <label>Message</label>
        <textarea name="message"></textarea>
      </div>
      <br />
      <div>
        <label>Gender</label>
        <select name="gender">
          <option value="male">Male</option>
          <option value="female">Female</option>
          <option value="other">Other</option>
        </select>
      </div>
      <br />
      <div>
        <label>Age:</label>
        <input type="number" name="age" value="30" />
      </div>
      <br />
      <div>
        <label>Birthday:</label>
        <input type="date" name="birthday" />
      </div>
      <br />
      <input type="submit" name="submit" value="Submit" />
    </form>

    <!-- Button -->
    <button>Click Me</button>

    <br />

    <!-- Image -->
    <a href="images/sample.jpg">
      <img src="images/sample.jpg" alt="My Sample Image" width="200" />
    </a>

    <!-- Quotations -->
    <blockquote cite="http://traversymedia.com">
      Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
      veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
      velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
      cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id
      est laborum.
    </blockquote>

    <!-- if we hover over WWW, it shows the full verion: -->
    <p>The <abbr title="World Wide Web">WWW</abbr> is awesome</p>

    <!-- cite tag is only semantic but it will make its content italic. -->
    <p><cite>HTML crash course</cite> by Brad Traversy</p>

    <div style="margin-top:500px"></div>
  </body>
</html>
```

![](/md/100.jpg)

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Blog</title>
    <!-- When google index your website, it looks into these meta tags -->
    <meta name="description" content="Awesome blog by Traversy Media" />
    <meta
      name="keywords"
      content="web design blog, web dev blog, traversy media"
    />
    <style type="text/css">
      #main-header {
        text-align: center;
        background-color: black;
        color: white;
        padding: 10px;
      }

      #main-footer {
        text-align: center;
        font-size: 18px;
      }
    </style>
  </head>
  <body>
    <header id="main-header">
      <h1>My Website</h1>
    </header>

    <a href="index.html">Go to index</a>

    <section>
      <article class="post">
        <h3>Blog Post One</h3>
        <small>Posted by Brad on July 17</small>
        <p>
          Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
          minim veniam, quis nostrud exercitation ullamco laboris nisi ut
          aliquip ex ea commodo consequat. Duis aute irure dolor in
          reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
          pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
          culpa qui officia deserunt mollit anim id est laborum.
        </p>
        <a href="post.html">Read More</a>
      </article>

      <article class="post">
        <h3>Blog Post Two</h3>
        <small>Posted by Brad on July 17</small>
        <p>
          Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
          minim veniam, quis nostrud exercitation ullamco laboris nisi ut
          aliquip ex ea commodo consequat. Duis aute irure dolor in
          reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
          pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
          culpa qui officia deserunt mollit anim id est laborum.
        </p>
        <a href="post.html">Read More</a>
      </article>

      <article class="post">
        <h3>Blog Post Three</h3>
        <small>Posted by Brad on July 17</small>
        <p>
          Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
          minim veniam, quis nostrud exercitation ullamco laboris nisi ut
          aliquip ex ea commodo consequat. Duis aute irure dolor in
          reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
          pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
          culpa qui officia deserunt mollit anim id est laborum.
        </p>
        <a href="post.html">Read More</a>
      </article>
    </section>

    <aside>
      <h3>Categories</h3>
      <nav>
        <ul>
          <li><a href="#">Category 1</a></li>
          <li><a href="#">Category 2</a></li>
          <li><a href="#">Category 3</a></li>
        </ul>
      </nav>
    </aside>

    <footer id="main-footer">
      <p>Copyright &copy; 2017, My Website</p>
    </footer>
  </body>
</html>
```

- The `<iframe>` tag specifies an inline frame.
- An inline frame is used to embed another document within the current HTML document.

```html
<iframe
  title="video player"
  src="https://www.youtube.com/embed/B84lsLkTHV0"
></iframe>
```

# Introduction to css

- Three ways of css:

![](/md/81.jpg)

- To change font, add this in your html head before loading css:

```html
<link href="https://fonts.googleapis.com/css?family=Anton" rel="stylesheet" />
```

Then, in your css:

```css
h1 {
  font-family: "Anton", sans-serif;
}
```

- Css properties can have four types of values.

  ![](/md/82.jpg)

- For example, `background: url("image.jpg");`
- You can check if you can use a certain css or js feature in `caniuse.com`.
- This is usually applied to a tags:

```css
a {
  text-decoration: none;
  color: black;
}
```

- Colors can be:
  - red, green, ...
  - #ff22ee
  - #eee
  - rgb(255, 0, 100)
  - rgba(0, 255, 255, 0.5)

# Selectors

- Elements

```css
h1 {
}
```

- Classes (`class="blog-post"` in html). css is case insensitive so we usually use - instead of camelCasing.

```css
.blog-post {
}
```

- Universal

```css
* {
}
```

- Id (`id="main-title"` in html)

```css
#main-title {
}
```

- Attribute (`attribute="value"` in html)

```css
[attribute="value"] {
}
```

or, if you just want to select elements with that attribute with any value:

```css
[attribute] {
}
```

![](/md/89.jpg)

# Cascading and Specificity

- Cascading means multiple rules can be applied to the same element but specificity will resolve that conflicts.
- css rules will take effect first by higher specificity score and then from top to bottom:

```
Inline styles > #ID selectors > .class, :pseudo-class, and [attribute] selectors > <Tag> and ::pseudo-element selectors > Inheritance (from parent to child. Usually, used by body selector.)
```

- You can elevate the inheritance by selecting the element and then specifying `font-family: inherit;`.

- With **combinators**, we can target better and increase specificity: `#main-title h1` means any h1 which is a child (direct or indirect) of an element with the id of main-title.

  - Adjacent sibling: to p elements that are _immediately after_ a div.

  ```css
  div + p {
  }
  ```

  - General sibling: to p elements that are _after_ a div sibling.

  ```css
  div ~ p {
  }
  ```

  - Child: any p element which is a _direct_ child of a div element.

  ```css
  div > p {
  }
  ```

  - Descendant: any p element which is a _descendant_ child of a div element.

  ```css
  div p {
  }
  ```

- With `,`, we can group the same rules to be applied to different selectors.
- This:

```css
a.active {
}
```

means an anchor tag that has the active class but `a .active` means any elements that have active class and are descendants of an anchor tag.

- If an element has multiple classes, the order in html doesn't matter. The order in the css file matters.
- Adding class to html element is preferable than adding id for styling. Because id has other uses in html to scroll a document (for example `href="#id1"`).
- `!important` is to override specificity (not recommended):

```css
div {
  color: red !important;
}
```

# Box model

- Almost all elements (both block-level elements and inline elements like \<a\>, \<span\>, and \<img\>) are interpreted as box in css: content -> padding -> border -> margin. The difference is that inline elements don't have margin-top and bottom and paddings are different.

```css
#product-overview {
  padding: 20px;
  border: 5px black solid;
  margin: 20px;
}
```

- By default, body has 8px margin, so we:

```css
body {
  margin: 0;
}
```

- By default h1 tags have margin-top and margin-bottom.
- `Margin collapsing` the distance between two adjacent elements will be the bigger margin. To avoid confusion, use either margin-top or margin-bottom.
- Shorthand properties:

![](/md/83.jpg)

- `width` is the width of the content only (not padding or border). The default of `width` is `100%` which means the box will take all the space available in the parent. You can use px or % or other units for width and height.
- `height` is the height that is needed by default. So if you want an element to be 100% of the page, you need to give `height: 100%;` to html tag -> body tag -> down to the element that you want!
- So the default behavior is:

```css
box-sizing: content-box;
```

- To include padding and border (this is so common that we usually add it to universal selector \*. Adding it to body won't work because inheritance will be overriden by display: block; of the element.):

```css
* {
  box-sizing: border-box;
}
```

- With `display` property, we can change `inline`, `block`, `inline-block`, and `none` (to make it hidden). So by adding `display: block;` to an anchor tag, we will move it the next line. We usually do not change an inline to block or vice versa. If we want to change it, we use inline-block to take advantage of both worlds (inline but having paddings and margins top and bottom).
- Because `inline-block` elements can be treated as text, we can use `text-align` property on the wrapping element to position them `right` or `center`. We also can use `vertical-align`. These two properties work on inline elements.
- We can use calc function for width property like `width: calc(100% - 49px);`.
- This is usually applied to ul elements:

```css
.key_features__list {
  list-style: none; /* To remove bullet points*/
  margin: 0;
  padding: 0;
}
```

- Inside a block level element, `text-align: center;` will center an inline or inline-block element and `margin: 0 auto;` will center a block level element.
- Using `float` property is really outdated. But it can be used for images, because it detaches the element from normal flow of document but the text will be around it so it is useful for images but not really for block level elements. If you want to make a float right for a block level element but not messing up (keep other elements in their places), you can add an empty div with class of `clear-fix` after it and the css for that class should be `clear: both;`. Better ways are using css grid and flexbox.

# Pseudo classes and elements

![](/md/84.jpg)

```css
a:hover {
}
```

```css
a:active {
}
```

```css
.button:focus {
  outline: none; /* To remove the blue border by default.*/
}
```

- The `not` pseudo class is really not recommended. We could use a positive way in the cases below:

```css
:not(p) {
  /* Selects any element that is NOT a paragraph*/
}

a:not(.active) {
  /* Selects any anchor tags that are NOT active*/
}
```

- Other pseudo classes are `:first`, `:first-child`, `:first-of-type`, `:nth-child()`, `:invalid`, `:checked`, ...

```css
p::first-letter {
  /* Only applies to the first letter*/
}
```

```css
.main-nav__item a::after {
  /* We can create content using css! for example an icon.*/
  content: " (Link)"; /* The (Link) will be added after the anchor tag text.*/
  color: red;
}
```

# Positioning

- `position` property of elements by default is `static`. It can be: `absolute`, `relative`, `fixed`, and `sticky` as well.
- `top`, `bottom`, `left`, and `right` only works on an element whose position property is not static.
- `fixed`: If the position of an element is set to fixed, it will be removed from the document flow (other elements think that it doesn't exist anymore.). It will be positioned related to viewport:

```css
.main-header {
  position: fixed;
  top: 0;
  left: 0;
  margin: 0;
  width: 100%;
  box-sizing: border-box;
}
```

and for a background:

```css
.background {
  background: url("bg.jpg");
  width: 100%;
  height: 100%;
  position: fixed;
  z-index: -1;
}
```

`z-index` property won't work if the element has a position of static (default). So we couldn't make other elements to be on top of this element.

- `absolute` and `relative`: we should give the position absolute to the badge and the position relative to the div that we want to give the badge to (so relative doesn't take the element out of document flow.):

```css
.package__badge {
  position: absolute;
  top: 0px;
  right: 0px;
  margin: 20px;
}

.package {
  position: relative;
}
```

- `relative`: if we apply this to an element, the top, left, ... can be used on this element and it will move the element in relation to its original place. If we apply `overflow: hidden;` to its parent, the element cannot leave its parent anymore.
- `sticky`: it doesn't work alone (you should also provide top property). When you scroll, when the top of viewport is less than the value in top, it behaves like a fixed position. But it will stop being fixed, when it reached to bottom of its parent element.

```css
.parent .child {
  position: sticky;
  top: 10px;
}
```

- `Stacking context`: if 2 sibling elements have a position and z-index property each and one of the sibling has children with position and z-index themselves, the z-index of children only affects in relation to each other and not their uncle :)

# Background and Images

![](/md/85.jpg)

```css
#product-overview {
  background-image: url("image.jpg");
  /* background-size: 100px; */ /*width is 100px and height will be aspect ratio */
  /* background-size: 300px 100px; */ /*width and height */
  /* background-size: auto 100%; */ /* height is 100% and width takes up as much of aspect ratio*/
  /* background-size: contain; */ /* One dimension will be 100% and other probably will be as much of aspect ratio */
  background-size: cover; /* It covers and one dimension will be cropped probably */
  /* background-position: center; */ /* The cropped area should be 50% from left 50% from right and top and bottom */
  /* background-position: left top; */ /* The cropped area should be 100% from right and bottom */
  background-position: left 10% bottom 20%; /* The cropped area should be 90% from right and 80% from top */
  background-repeat: no-repeat;
  background-origin: border-box; /* The images goes beneath its border */
  background-clip: border-box; /* The images goes beneath its border */
}
```

- Gradients are treated as images -> so you have to put them instead of url:

```css
#product-overview {
  background-image: linear-gradient(to bottom, red, blue);
  background-image: linear-gradient(to left bottom, red, blue); /* diagonal */
  background-image: linear-gradient(30deg, red, blue);
  background-image: linear-gradient(
    to bottom,
    red,
    blue,
    yellow
  ); /* multi colors */
  background-image: linear-gradient(to bottom, red 70%, blue); /* 70% is red*/
  background-image: linear-gradient(to bottom, red, transparent);
}
```

```css
#product-overview {
  background-image: radial-gradient(red, blue);
  background-image: radial-gradient(red, blue, green);
  background-image: radial-gradient(circle, red, blue); /* instead of ellipse */
  background-image: radial-gradient(circle at top, red, blue);
  background-image: radial-gradient(
    circle at 20% 50%,
    red,
    blue
  ); /* the center will be at x = 20% and y=50% */
  background-image: radial-gradient(ellipse 80px 30px at 20% 50%, red, blue);
}
```

- We can stack up background images (a gradient - an image - a solid color):

```css
#product-overview {
  background: linear-gradient(to top, red 10%, transparent),
    url("image.jpg") left 10% bottom 20% / cover no-repeat border-box, blue;
}
```

- You can also add filter to it:

```css
.background {
  background: url("image.jpg") center/cover;
  filter: grayscale(40%);
}
```

- If you set the size on the surrounding container of an image `<img>`, the image won't respect it. You have to set it on the image (even on that you should use pixel and not % if the surrounding is an inline element (like anchor tag). So change the parent to inline-block if you want to use %.).

# Size and Units

- If you use pixel for font sizes, if the user changes the font size of the browser (normal to large) it won't have an effect. And he has to zoom in. You can change the default of the user by `html {font-size: 75%;}` for fonts that you didn't set, which is not very useful.
- `rem` and `em` are for font sizes. If the browser is set to normal the font-size is 16px. `em` uses the font-size of its parent (which can be em too so we will have two multiplications) to calculate the font-size which results in mess so use `rem` which always calculate the font-size based on the browser font-size. You can use `rem` also for margins and padding but keep in mind that it is usually 16px. Using rem for border or shadow is not recommended.
- `vh` and `vh` is viewport height and width.

![](/md/86.jpg)

- Rules for % sizes (the size will be calculated with respect to):

![](/md/87.jpg)

- One usage of the above rules are backdrop (an empty div with the class of backdrop after the body tag) for modals:

```css
.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 100;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
}
```

or

```css
.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 100;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
}
```

![](/md/88.jpg)

# Responsive design

- With the combination of a percentage value for `width` and an absolute value (pixel) for `max-width`, we can take one step toward responsive design.
- If you use mobile device simulator on the chrome dev tools, because the `physical/hardware pixels` of a mobile device is usually two or three times more than the `CSS pixels`, you will not get the display when you decrease the width manually. To get it right, you have to add `<meta name="viewport" content="width=device-width, initial-scale=1.0">` in the html head section to adjust the site to device viewport. If we don't add it, the browser on the mobile thinks it has more width. In viewport meta-tag, you can also specify whether the user can zoom-in/out (`user-scalable=yes`) or to what extent `maximum-scale=2.0, minimum-scale=0.5`. Note that, without this tag, we can't implement mobile responsiveness at all (maybe we want to have a different website for mobile).
- We should design websites first for mobiles (width smaller than 640) (mobile-first design approach) and then see if it is good for desktop. So we add media queries for desktop (larger than 640px) not mobile:

```css
@media (min-width: 40rem) {
  /* Styles for desktop*/
}
```

- You can also have multiple media queries. Just be aware of the cascading effects that they might have.
- A good practice is to write media query at the end of the css file.
- We can have logical operators in media query:

```css
@media (min-width: 40rem) and (orientation: portrait) {
  /* Styles for desktop and when in portrait mode (height > width) */
}
```

- This is for `or`:

```css
@media (min-width: 40rem), (orientation: portrait) {
  /* Styles for desktop or for when in portrait mode */
}
```

# Forms

- Input elements tend to have many browser default styles need to be overwritten.
- Each label and input can set to block level (`display: block;`) to be in one line; Except for checkbox (`<input type="checkbox" id="agree-terms">`):

```css
.signup-form input,
.signup-form select {
  display: block;
  margin-top: 1rem;
  width: 100%;
}

/* .signup-form input[type="checkbox"] */ /* The below selector is also valid */
.signup-form input[id*="terms"],
.signup-form input[id*="terms"] + label {
  display: inline-block;
  width: auto;
  vertical-align: bottom;
}

.signup-form input:not([type="checkbox"]),
.signup-form select {
  border: 1px solid #ccc;
  padding: 0.2rem 0.5rem;
  font: inherit; /* To get it from parent and not browser default */
}

.signup-form input:focus,
.signup-form select:focus {
  outline: none;
  background: #d8f3df;
  border-color: #2ddf5c;
}

.signup-form input[type="checkbox"] {
  border: 1px solid #ccc;
  background: white;
  width: 1rem;
  height: 1rem;
  -webkit-appearance: none; /* To override default browser behavior */
  -moz-appearance: none;
  appearance: none;
}

.signup-form input[type="checkbox"]:checked {
  background: #2ddf5c;
  border: #0e4f1f;
}

.signup-form input.invalid, /* We will give it a class of invalid by JS or on the server */
.signup-form select.invalid,
.signup-form :invalid {
  /* This is relying on default html validation like required or password */
  background: red !important; /* Because the border will be overwritten by more specificity above */
  border: #faacac;
}

.signup-form button[type="submit"] {
  display: block;
  margin-top: 1rem;
}

.button[disabled] {
  cursor: not-allowed;
  border: #a1a1a1;
  background: #ccc;
  color: #a1a1a1;
}
```

# Fonts

![](/md/90.jpg)

![](/md/91.jpg)

<!-- - We can select nothing in css to rely on the default `standard` or use a generic family or a specific font family (which is known by the browser) or use our own fonts (we can use user'grid-area: row-1-start / 2 / r local [not ours] or web fonts). -->

- We have to choose web-safe fonts (fonts that are widely installed on Windows and Mac) if we are not using web fonts.

```css
body {
  font-family: "Unknown", "Verdana", sans-serif; /* "" is not needed for one word fonts. Unknown cannot be found so it will go to Verdana  */
}
```

- Generic -> Font-family -> Font faces (regular 400, regular 400 italic, ...).
- For importing fonts, go to google fonts and copy-paste the link to html head section:

```html
<link
  href="https://fonts.googleapis.com/css2?family=Red+Rose:wght@300&display=swap"
  rel="stylesheet"
/>
```

or, import it in css (which is a better practice I guess):

```c#
@import url('https://fonts.googleapis.com/css2?family=Red+Rose:wght@300&display=swap');
```

Now, you can use it in your css.

- Note that you have to import different faces (styles), if you want to use them. For example for light, normal, and bold italic:

```css
@import url("https://fonts.googleapis.com/css2?family=Red+Rose:wght@300;400;700i&display=swap");
```

- If you don't import italic style and use `font-style: italic;`, the browser is actually capable of making it italic but it is different from the original italic.
- If you want to host your fonts in the server, put it in an `assets/fonts` folder and then in your css:

```css
font-face {
  font-family: "my-font";
  src: url("../assets/fonts/my-font-regular.ttf");
}

font-face {
  font-family: "my-font";
  src: url("../assets/fonts/my-font-bold.ttf");
  font-weight: 700;
}
```

- `.woff` is more compressed and has better browser support than `ttf`. If we have other formats:

```css
font-face {
  font-family: "my-font";
  src: url("../assets/fonts/my-font-regular.woff") format("woff"), url("../assets/fonts/my-font-regular.ttf")
      format("truetype");
}
```

- Font properties:

```css
font-style: italic;
font-variant: small-caps; /* Turns it to uppercase */
font-stretch: condensed;
letter-spacing: 5px; /* Space between letters */
white-space: nowrap; /* The text will be one line. Other values: pre, pre-line, pre-wrap */
line-height: 2; /* 2 is in comparison with font-size. But if you set it to normal, it is based on each font-family. */
text-decoration: underline wavy red; /* like syntax error */
text-shadow: 4px 4px 2px gray;
```

- Font shorthand:

```css
.package__info {
  font: italic small-caps 700 1.2rem/2 my-font, sans-serif; /* Order is important */
  /*font-style font-variant font-weight font-size/line-height font-family*/
}
```

- When importing custom fonts (with defining font-face), we can define different `font-display` property:

```css
font-face {
  font-family: "my-font";
  src: url("../assets/fonts/my-font-regular.ttf");
  font-display: block;
}
```

- The default of most browsers are block.

![](/md/92.jpg)

# Flexbox

- Making elements `inline-block` is not a good way of positioning. With flexbox, we don't need `inline-block`, `vertical-align` and `text-align`.

![](/md/93.jpg)

- By just adding `display: flex;` to the parent, all the items (children) will be in one row and will take up the height of the parent (if align-items is not set).

```css
.flex-container {
  border: 2px solid black;
  display: flex;
  flex-direction: row; /* row is default. Another value is column. With column, all children take up the width of their parent (if align-items is not set) if the width is not set for them. we have row-reverse and column-reverse as well. */
  flex-wrap: nowrap; /* nowrap is default. wrap makes elements to jump into a new line when width decreases */
  /* flex-flow: row nowrap; */ /* This is the shorthand for two above props. */
  align-items: center; /* This is to align item along the CROSS axis. The default is stretch. We have also space-between, space-around, flex-start and flex-end. */
  justify-content: center; /* This is to align item along the MAIN axis. The default is stretch. We have also space-between, space-around, flex-start and flex-end. */
}
```

![](/md/94.jpg)

![](/md/95.jpg)

- We have also `align-content: center;` which will position content along cross axis when wrapping happens (so the flex-wrap should be wrap.).

```css
.item1 {
  width: 100px;
  order: -1; /* By default all flex items are order 0. With -1, it comes first.*/
  align-self: flex-start; /* It position this item individually along cross axis */
  flex-grow: 1; /* The initial value is 0. If we specify flex-grow for other elements, they grow proportional to each other in comparison to their width. So by default they don't grow (default is 0). */
  flex-shrink: 1; /* The initial value is 1. If we specify flex-grow for other elements, they shrink proportional to each other in comparison to their width. So by default they shrink together (default is 1). If we set it to zero, it won't shrink. */
  flex-basis: 200px; /* If the direction is row, the flex-basis will overwrite width -> column: height. We can use percentages as well. */
  /* flex: 0 1 auto; */ /* shorthand version of the above three */
  /* flex-grow flex-shrink flex-basis */
}
```

# CSS Grid

- Elements that are not part of document flow (absolute and fixed) are not part of the grid.
- You define the layout on the container and define the position on the child element.
- Only the direct children will be part of the grid.

```css
.container {
  display: grid;
  grid-template-columns: 200px 2fr 20% 1fr; /* If we don't set this property, the grid will have just one column. Now there will be four columns. You can also use `auto` instead of frs to just fill the empty space of the parent. */
  grid-template-rows: 5rem 2.5rem; /* We also can use fit-content(8rem) function which means 8rem if the content is smaller and fit-content to fit the content. */
}

.el3 {
  grid-column-start: 3; /* This is one-based. */
  grid-column-end: 5; /* Now el3 will span two columns and kick element 4 to new rox. */
  /* grid-column-end: span 2; */ /* Just like above */
  /* grid-column-end: -1; */ /* Will take up the entire row. In our example, it is just like the two above because we have 4 columns */
  grid-row-start: 1;
  grid-row-end: 3;
}
```

![](/md/96.jpg)

```css
.container {
  height: 500px;
  display: grid;
  grid-template-columns: repeat(
    4,
    25%
  ); /* equivalent to 25% 25% 25% 25%. Use auto-fill or auto-fit instead of entering a number to derive the number of columns automatically -> if you change the browser width, it will be wrapped to a new row.  */
  grid-template-rows: 5rem minmax(10px, 200px); /* The row will not be smaller than 10px and bigger than 200px (you can use auto for second argument to take up the available space) -> dynamic sizing */
}

.el3 {
  grid-column-start: 3;
  grid-column-end: 5;
  /* grid-column: 3 / 5; */ /* shorthand */
  grid-row-start: 1;
  grid-row-end: 3;
  /* grid-row: 1 / 3; */ /* shorthand */
}
```

- The grid tries to not to overlap elements (by putting elements in a new row). BUT if you explicitly say the start and end of two elements to make them overlapping each other, it will happen and the element which comes next in the DOM will be on top (we can use z-index to change it).

- We can name lines and use them instead of numbers for our elements:

```css
.container {
  height: 500px;
  display: grid;
  grid-template-columns: repeat(4, 25%);
  grid-template-rows:
    [row-1-start] 5rem [row-1-end row-2-start] minmax(10px, 200px)
    [row-2-end row-3-start] 100px [row-3-end]; /* We can have two names for one row-line */
}

.el3 {
  grid-column-start: 2;
  grid-column-end: span 3;
  grid-row-start: row-1-start;
  grid-row-end: row-2-end;
  /* grid-area: row-1-start / 2 / row-2-end / span 3; */ /* shorthand */
}
```

- Instead of setting margins to the elements, we can use `grid-column-gap: 20px;` and `grid-row-gap: 10px;` or the shorthand `grid-gap: 10px 20px;` on the grid container.
- Instead of specifying grid-area on the element like above, a **better** solution is to define

```css
.container {
  margin: 20px;
  height: 500px;
  display: grid;
  grid-template-columns: repeat(4, 25%);
  grid-template-rows:
    [row-1-start] 5rem [row-1-end row-2-start] minmax(10px, 200px)
    [row-2-end row-3-start] 100px [row-3-end];
  grid-gap: 10px;
  grid-template-areas:
    "header header header header"
    "side side main main"
    "footer footer footer footer"; /* use . for unnamed */
  justify-items: stretch; /* This position the elements inside the cell horizontally. Other values are end, start, center. If we want to have a different position in the cell for our element, we can use `justify-self: center;` in its css selector. */
  align-items: center; /* This position the elements inside the cell vertically. Other values are end, start, center. If we want to have a different position in the cell for our element, we can use `align-self: center;` in its css selector.  */
  justify-content: center; /* This position the whole grid inside its parent horizontally (if we don't use fr or auto, it won't take up its whole parent.). Other values are end, start, center. */
  align-content: center; /* This position the whole grid inside its parent vertically (if we don't use fr or auto, it won't take up its whole parent.). Other values are end, start, center. */
}
```

and then in the element:

```css
.el2 {
  grid-area: header;
}
```

- responsiveness (with this [using grid-area on our elements], we don't have to change them in media queries and we just change the layout.):

```css
.container {
  margin: 20px;
  display: grid;
  height: 800px;
  grid-template-columns:
    [hd-start] repeat(4, [col-start] 20% [col-end])
    [hd-end];
  grid-template-rows:
    [hd-start] 5rem [hd-end row-2-start] minmax(10px, 20rem)
    [row-2-end row-3-start] 100px [row-3-end];
  grid-gap: 10px;
  grid-template-areas:
    "header header header header"
    ". side main main"
    "footer footer footer footer";
  justify-items: stretch;
  align-items: stretch;
}

@media (max-width: 40rem) {
  .container {
    grid-template-rows:
      [hd-start] 5rem [hd-end row-2-start] minmax(10px, 20rem)
      [row-2-end row-3-start] 150px [row-3-end row-4-start] 100px [row-4-end];
    grid-template-areas:
      "header header header header"
      "main main main main"
      "side side side side"
      "footer footer footer footer";
  }
}
```

- If you don't specify the rows (just columns) on the grid container, css grid will automatically create rows for items. With `grid-auto-rows: 10rem;` on grid container, we can make sure that the generated rows are all in the same size. If you explicitly set two columns and one row and have three elements, the three elements again creates a new automatically-generated row.
- If you want to change the default behavior of adding rows and turn it to adding column, you should add `grid-auto-flow: column;` to grid container. In this case, we can have `grid-auto-columns: 5rem;`.
- If because of an element with a large span, a cell is left out empty, it wouldn't be filled (if we explicitly position an element somewhere in the grid, the empty cells will be filled out). To fill it out, we should use `grid-auto-flow: row dense;`.
- If something is one-dimensional, use flexbox. For layouts, like the overall website and complex forms, use css grid.

# Transformation

- You can move (it is just visually and doesn't change the document flow. So do not use display: relative; and then top, left, ... or margin to move elements) or rotate elements:

```css
.badge {
  transform: rotateZ(45deg) translateX(1rem) translateY(-1rem); /* clockwise. Note that x is rotated 45deg. */
  /* transform: rotate(45deg) translate(1rem, -1rem); */ /* shorthand */
  transform-origin: left top; /* the center of rotation. The default is center. You can use pixel or % which is from left top -> 0 0 or center -> 50% 50% */
}
```

- We can then `overflow: hidden;` to cut-off the extras to get a ribbon.
- We also have `rotateX` and `rotateY` for 3D transformations.
- If you rotate using 3D transformations, you can add `perspective: 1000px;` to its parent to see it from far away. You can also add `perspective-origin: right;` to see it from right angle.
- If we set perspective on the parent, by using `translateZ` on the element, the element can get closer (bigger) or farther (smaller).
- If you rotateY the container 90deg, you can't see its children! Just like a window when you look from its side. To rotate the children with the parent, use `transform-style: preserve-3d;` on the parent (the default is `flat`).
- For skewing and scaling:

```css
.image-container {
  transform: skew(20deg);
  overflow: hidden;
}

.image {
  transform: skew(-20deg) scale(1.4);
}
```

![](/md/97.jpg)

# Transition and Animation

- You define what you want to watch, over which duration, you want it to happen, how that duration should be used, and how much should it be delayed.
- Not every css property is animate-able, but color, transform, opacity, ...

```css
.modal {
  opacity: 0;
  transform: translateY(-3rem);
  transition: opacity 200ms ease-out 1s, transform 500ms; /* It attaches transition to those properties and whenever they change they will be interpolated in that time. The third argument can be the function for time interpolation. The fourth value is the delay */
}

.open {
  opacity: 1;
  transform: translateY(0);
}
```

- You can also define your own timing function with `cubic-bezier(0.55, 0.055, 0.675, 0.19);`.
- Animation is transition++. Because we define key-frames in between and not just start and end.

```css
.min-nav__item--cta {
  animation: wiggle 200ms 3s 8 none; /* animation duration delay iteration-count fill-mode-or-which-state-should-be-kept->forwards, backwards, both, ...*/
}

@keyframes wiggle {
  0% {
    /* You can also define from and to if you have only two frames */
    transform: rotateZ(0);
  }
  50% {
    transform: rotateZ(-10deg);
  }
  100% {
    transform: rotateZ(10deg);
  }
}
```

- If an element has an animation property, you can target that element and add eventListener to it in JS (to ):

```js
ctaButton.addEventListener("animationstart", (e) => {
  console.log("Animation started", e);
});
ctaButton.addEventListener("animationiteration", (e) => {
  console.log("Animation started", e);
});
ctaButton.addEventListener("animationend", (e) => {
  console.log("Animation started", e);
});
```

# Variables and other stuff

```css
:root {
  --my-color: #fa923f;
}

.element-1 {
  color: var(
    --my-color
  ); /* You can specify a second argument as a fallback if the variable isn't set. */
}
```

- By using vendor-prefix values for properties, we can implement new features ahead of time and even support older browsers. So for example for flex, we should use vendor-prefix before the final flex value:

```css
.some-class {
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}
```

- There are tools and websites that get you code and will add prefixed properties to target more browsers.
- We can use `supports` query to see if a property and its value is supported by the browser. For that, first we define the fallback and then the better solution:

```css
body {
  font-family: sans-serif;
  margin: 0;
}

@supports (display: grid) {
  body {
    display: grid;
    grid-template-rows: 3.5rem auto fit-content(8rem);
    /* Other grid specific stuff*/
  }
}
```

- If you are really desperate to implement a css feature that some browsers don't support, you can import polyfill package into your html. A polyfill is a JS package which enables certain css features such as rem. Note that it comes with a cost of downloading that package and manipulating the DOM with js which has performance penalty.

![](/md/98.jpg)

![](/md/99.jpg)

# Sass

- Sass is a superset of css (extends css during development) and does not run in the browser. It compiles to css.
- On windows, install `ruby` first, then `gem install sass`. Verify installation by `sass -v`.
- With Saas, we can write re-usable code for vendor-prefix, variables for color-codes, nest code for selectors.
- In `.sass` files, we have to remove `;` and `{}`. It works based on indentations. I personally don't like it so we use `.scss` files which is just like sass but we can have `;` and `{}` just like css.
- Instead of:

```css
div {
  ...;
}

div p {
  ...;
}
```

we can write:

```scss
// we can comment with -> //
div {
  ...
  p {
    ...
  }
}
```

- The above example is for nesting for adding pseudo-classes which are connected (and not nested), we should prefix with `&` (we also can use `&` with other classes -> `&.another-class`):

```scss
.some-class {
  ...
  &:hover {
    ...
  }
}
```

- With `sass main.scss main.css`, we can convert it to css.
- With `sass --watch main.scss:main.css`, we enter watch mode for auto compilation.
- Another feature of sass is `nested properties`; instead of:

```css
.some-class {
  flex-direction: column;
  flex-wrap: nowrap;
}
```

```scss
flex: {
  direction: column;
  wrap: nowrap;
}
```

- Variables (which does not rely on css variable and so can be run on any browsers):

```scss
$size-default: 1rem; // variable
$colors: (
  main: #eee,
  secondary: #fff,
); // map
$border-default: 0.05rem solid map-get($colors, main); // list

.some-selector {
  margin: $size-default;
  padding: $size-default * 3 0; // Sass can do arithmetics
  border: $border-default;
  color: map-get($colors, main);
  background: lighten(
    map-get($colors, main),
    70%
  ); // a built-in function, so this color depends on another variable.
}
```

- Remember that we can import a css file into another css file by `@import url(another.css);`. In sass, we use `@import "another.css";` and it compiles to the previous. This way of import is a separate network request. If you want to use `partials`, for example declare all the variables in one file and import it in all sass files (it will be compiled to duplications so no network requests), you have to start the name of the partial with `_` and then in your sass files -> `@import "_variables.scss";`. If you import a sass file in another sass file, it will just be added to the main file (in the compiled version). So with sass we can work with different files but with fewer http requests.
- With sass, we can nest media query inside each element (so we don't need a selector anymore and the media query will be close to the element that will take the effect.):

```scss
.some-class {
  ...
  @media (min-width: 40rem) {
    ...
  }
}
```

- Inheritance: of course, in css we can define a shared class and many specific classes and assign the shared and more specific class to an html element. But for that we have to change html. Other way in css is to list more specific classes with comma separated in the definition of the shared class. So we have to go to the shared class and add out more specific class to it. In sass, we can use `@extend .shared-class;` inside the more specific class to achieve the same goal as previous.
- `mixin` are just like functions to reduce duplication. In the above of a scss file, we define:

```scss
@mixin display-flex() {
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
}
```

and whenever we needed those four lines, we use `@include display-flex();`.

- We can also pass arguments to mixins or mixins can have dynamic content:

```scss
@mixin media-min-width($width) {
  @media (min-width: $width) {
    @content;
  }
}

.some-class {
  ...
  @include  media-min-width(40rem) {
    ...
  }
}
```

# Mosh

- Use `<em></em>` or `<strong></strong>` instead of `<i>` or `<b>`. Because it is semantic and not stylic.
- Entities: use `&lt;`, `&gt;`, `&nbsp;` to show `<`, and `>` and non-breaking space.
- Use `<a href="images/ben.jpg" download>My Pic</a>` to automatically download a picture.
- Use `<a href="#">Jump to top</a>` to jump to top or use any id to jump to that section.
- Use `<a href="mailto:john.doe@gmail.com">Email Me</a>` to open an email client.
- You can set `object-fit: cover;` CSS property for `img` tags.
- To show video: `<video src="videos/ocean.mp4" controls autoplay loop>Your browser doesn't support videos.</video>`.
- Divs will cover the enitre width by default.

## Some Semantic Elements

- Use `article` for any self-contained piece of content.
- Use `mark` like span to highlight a piece of text with marker.
- Use `time` to wrap dates with time. It has no representational effect.
- Use `figure` to wrap `img` alongside with `figcaption`.
- `main`, `aside`, `header`, `footer`, `nav`, `section`,...
- An article can have different sections, or a section can have different articles.

## Normalize CSS

- Search and download `normalize.css` and link it in the html before the main `styles.css` so that all defaults related to different browsers will be gone.

## Selectors

- Apart from selecting by class, id, or tag type, we can target elements by attribute:

```css
a[href="www.google.com"] {
}
a[href*="google"] {
  /* has google anywhere */
}
a[href^="https"] {
  /* starts with */
}
a[href^="https"][href$=".com"] {
  /* combination */
}
```

- Relational selectors (they can be fragile):

```css
#products p {
  /* All p tags inside products */
}
#products > p {
  /* Only p tags which are direct children of products */
}
#products ~ p {
  /* Only p tags which are general siblings to the products and come after it */
}
#products + p {
  /* Only p tag which exactly comes after products */
}
```

- Pseudo-class selectors (classes that the browser assigns to elements):

```css
article :first-child {
  /* First child element */
}
article :first-of-type {
  /* First elements of every type which are children of article */
}
article p:first-of-type {
  /* First p tag which is child of article */
}
article p:last-of-type {
  /* Last p tag which is child of article */
}
ul li:nth-child(odd) {
}

a:visited,
a:link {
}

a:hover,
a:focus {
  /* focus is triggered by tab */
}
```

- Pseudo-element selectors:

```css
p::first-letter {
  /* targets the first letter to make it bold for example */
}

p::first-line {
  /* targets the first line of a paragraph */
}

::selection {
  /* when you select the text by mouse */
}

p::before {
  content: "...";
}
```

## Specificity

- Each selector has a weight. If different selectors target one element, the one that is more specific will be applied (type:1, class and attribute:10, id:100).
- If they have the same weight, the one that comes last will be applied.
- With `!important`, you can override the specificity. But don't use.

## Inheritence

- Some CSS properties will be inherited to children elements such as `color`, which can be overriden if we add `color: initial;` to the child element.
- Some CSS properties will NOT be inherited like `border`, which can be inherited by `border: inherit;`.
- As a rule of thumb, properties related to typography will be inherited.

## Colors

- Apart from hexadecimal, rgb, and rgba, we have `hsl` and `hsla` (hue, saturation, lightness):

```css
color: hsl(54, 20%, 70%);
```

- Gradients are actually images (not colors).

```css
background: linear-gradient(red, yellow); /* from top to bottom */
background: linear-gradient(to right, red, yellow); /* from left to right */
background: radial-gradient(white, yellow); /* from center to outside */
```

There are many online tools to generate the CSS for gradient like `cssgradient.io`.

## Shapes

- To create a circle, we set `border-radius: 50%;` for a rectangle div.

## Shadows

```css
.box {
  background: blue;
  box-shadow: 0 0 30px grey;
}

h1 {
  color: white;
  text-shadow: 3px 3px 5 px rgba(0, 0, 0, 0.2); /* Nice trick to have good text shadow */
}
```

## Layout

### Sizing

- In box model, we have `content`, `padding`, `border`, `margin`.
- Note that, the size of an element is by default only the content and if you add padding or border, the element gets bigger so as a best practice, set `box-sizing: border-box;` in the universal selector `*, *::before, *::after` (as opposed to its default `content-box`).
- Paddings of two adjacent elements will not collapse but margins will.
- `in-line` elements like span, do not respect `height` and `width` properties. So we give them `display: inline-block;` to be on the same line but to receive width and height.
- By default, block-level elements have width of 100%.

### Overflowing

- If we have fixed-size containers, we can add `overflow: hidden;` or `scroll` or `auto` (default is `visible`) to hide the content that can't fit.

### Measurement Units

- `px` is absolute. `%` is relative to parent container. `vw` and `vh` is relative to viewport. `rem` is relative to user font size.
- divs can use `%` but borders should use `px`.
- If you want a div to expand vertically, do not use `height: 100%;`, because it means 100% of its parent, which can be 0 (if there is no content -> default height of a div is based on its content). Instead use `height: 100vh;`.
- `em` is the font-size of the current element.
- The default font-size is `16px`.

### Positioning

- By default, all elements have `position: static;`.
- By giving `position: relative;`, we can specify `top`, `bottom`, `left`, or `right` relative to the element's normal position.
- By giving `position: absolute;`, we can specify `top`, `bottom`, `left`, or `right` relative to the element's parent. It will take out the element from the normal flow of the page. Important: you must give `position: relative;` to the parent.
- By giving `position: fixed;`, we can specify `top`, `bottom`, `left`, or `right` relative to the viewport. It will take out the element from the normal flow of the page.
- The `z-index` by default is `0` and we can give it positive or negative values.

### FlexBox

```css
.container {
  display: flex;
  flex-direction: row; /* row is by default */
  justify-content: center; /* along the main axis */
  align-items: center; /* along the cross axis */
  flex-wrap: wrap; /* nowrap is default and will squish the items */
  align-content: center; /* if we have two lines because of wrap, we can center these two lines by this */
}

.box {
  flex-basis: 100px; /* it will translate to width or height dependeing on flex direction */
  flex-grow: 1; /* by default it is 0. when setting to 1, they will grow */
  flex: 1 1 15rem; /* shorthand for flex-grow, flex-shrink and flex-basis */
}

.box-one {
  align-self: flex-start; /* we can override the align-items here */
}
```

### Grid

```css
.container {
  display: grid;
  /* 3 x 2 */
  grid-template-rows: 100px 100px 100px; /* repeat(3, 100px); */
  grid-template-columns: 100px 100px; /* repeat(2, 100px); */
}
```

or

```css
.container {
  display: grid;
  grid-template: repeat(3, 100px) / repeat(2, 100px);
}
```

- If we add divs inside grid, they will be assigned from top lef to bottom right.
- But you have full control on the placement of items.
- By default, the alignment of each item in the grid cell is top left, you can change it:

```css
.container {
  display: grid;
  grid-template: repeat(3, 100px) / repeat(2, 100px);
  justify-items: center; /* Horizontal: If the items don't have width and this was stretch (which is by default), items will stretch */
  align-items: center; /* Vertical */
}
```

- By default, the whole grid is on the left of the container, you can change it:

```css
.container {
  display: grid;
  grid-template: repeat(3, 100px) / repeat(2, 100px);
  justify-items: center;
  align-items: center;
  justify-content: center;
  align-content: center;
  height: 100vh; /* now the height of container is bigger */
}
```

- Prefer using `fr` over `%`, because they are relative to available space and not to the parent container.
- A typical web-page:

```css
.container {
  display: grid;
  grid-template: 100px auto 100px / 70fr 30fr;
  gap: 10px; /* shorthand for row-gap and column-gap */
  height: 100vh;
}

.header {
  grid-column: 1 / span 2; /* from column 1 (visible in dev tools to column 3 or span 2) */
}
```

or:

```css
.container {
  display: grid;
  grid-template: 100px auto 100px / 70fr 30fr;
  gird-template-areas: "header header" "main sidebar" "footer footer";
  gap: 10px;
  height: 100vh;
}

.header {
  grid-area: header;
}

.footer {
  grid-area: footer;
}
```

### Media Queries

- The trend is `Mobile first` approach.

```css
.container {
  display: flex;
  flex-direction: column;
}

.box {
}

@media screen and (min-width: 600px) {
  .container {
    flex-direction: row;
  }
}

@media print {
  body {
    font-size: 12pt;
  }

  .box {
    padding: 0.5cm;
  }
}
```

## Typography

- We usually give the font to body to use inheritence.
- For more than one font names, we must use ".

```css
p {
  font-weight: normal; /* = 400 */
  font-style: italic;
  font-size: 2rem;
  color: #333;
}
```

- To embed fonts, we usually download woff and woff 2.0 because they are more compressed (the support is really good). If we have other formats, we can convert it online.
- Create a `fonts` folder and a sub-folder with the name of the font and paste all variants of the font into it.
- At the top of our stylesheet:

```css
@font-face {
  font-family: "blah";
  src: url("fonts/blah/blah-regular.woff2") format("woff2"), url("fonts/blah/blah-regular.woff")
      format("woff");
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: "blah";
  src: url("fonts/blah/blah-bold.woff2") format("woff2"), url("fonts/blah/blah-bold.woff")
      format("woff");
  font-weight: bold;
  font-style: normal;
}
```

- We should use a similar font in font stack after our custom font for initial fallback.
- Instead of downloading a font, we could use a font service such as google:
  - We select our fonts and the styles that we want.
  - We copy the URL that the Google generates and paste it in the head section, before loading our style or we can import it on top of our CSS. The URL points to a CSS file with @font-face etc.
- Instead of using web-safe fonts (which can be boring), you can use system font stack, which are native to operating systems and do not need to be downloaded.
- A technique is that to set the font-size of the `html` to `font-size: 62.5%;` which in normal (16px) we get 10px.
- If we use rems, since our font sizing is relative to html element, by changing the font-size in html element (in media query), we can change all font sizes.
- Of course, we should use `rem` for `headings`' margins as well. Top must be larger than the bottom to separate them clearly.
- To define a `line-height: 1.5;`, it is better not to specify any unit so it will be a multiplier of the font-size. Otherwise, we have to remember to change the line-height whenever we change the font-size.
- It is better to use `px` for letter-spacing: `letter-spacing: 1px;`. We can use negative values for headings for example. We have `word-spacing: 5px;` too.
- We can use:

```css
p {
  width: 50ch;
}
```

to ensure that each paragraph has around 60 characters. `ch` is the width of `0` character.

- to format text:

```css
h1 {
  text-transform: capitalize; /* each word */
}

p {
  text-align: left;
  text-decoration: underline;
}

p + p {
  text-indent: 1rem; /* The first paragraph will not be indented */
}
```

- To truncate text, we should have fixed width and these rules:

```css
p {
  width: 50ch;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

- To have `multi-column` text:

```css
p {
  width: 50ch;
  column-count: 2;
  column-gap: 2rem;
  column-rule: 1px dotted #999;
}
```

## Images

- `Vector` images with `svg` format which is created by `Adobe Illustrator` are sharp at any size unlike `Raster` images with `png`, `jpg`, `gif` or `webp` formats. So use them for logos, icons and backgrounds.
- `png`, `webp`, and `gif` have transparency and animation but `jpg` doesn't.
- `webp` is much smaller compared to `jpg` but Internet Explorer does not support it. You can use `picture` element with `source` children elements and one `img` to support IE.
- `img` elements are `inline` but they can have `width` and `height` like `inline-block` elements. The default size is the image size but you can specify the height and width.

### Background Images

```css
body {
  background: url("/images/bg.jpg");
  background-repeat: no-repeat;
  background-position: 100px 100px; /* it will push to the right and bottom */
  background-size: cover; /* covers while retaining aspect ratio */
  background-attachment: fixed; /* background will remain fixed relative to the viewport */
  height: 300vh;
}
```

### CSS Sprites

- With background images technique, we can sprite all our images into one image using online tools and then copy the generated css into our css and use `span` with the generated classes. So we will have less GET requests to our server.
- Don't go overboard with this technique and only use it for logos and icons. Because it is not that much flexible.

### Data URLs

- We can encode an image into base64 and set the `src="data:image/png:base64;jsdf..."` and therefore embed the image into document (no more HTTP request to server). There are online tools to generate it. Not recommended.

### Clipping

- You can define a polygon and clip out the image outside the polygon. There are online tools to generate CSS like this:

```css
img {
  clip-path: polygon(0% 20%, ...);
}
```

### Filters

```css
img:hover {
  filter: grayscale(70%) blur(3px);
}
```

### High-density Screens

- In retina screens, the physical pixels are more than logical pixels by a factor called DPR.
- Since in CSS, we deal with logical pixels, in retina screens, our image will be blurry. So we will provide images with different sizes and then in CSS:

```html
<img
  src="images/coffee.jps"
  alt=""
  srcset="images/coffee.jp 1x, images/coffee@2x.jpg 2x, images/coffee@3x.jpg 3x"
/>
```

- Only use this for main images in the first page.

### Resolution Switching

- We give the browser different image set and it will choose the best one based on the width that we declared (in this case, do not use media queries for resizing based on width if we need it):

```html
<img
  src="images/coffee.jps"
  alt=""
  srcset="
    images/coffee.jp      400w,
    images/coffee@2x.jpg  800w,
    images/coffee@3x.jpg 1200w
  "
  sizes="(max-width: 500px) 100vw,  (max-width: 700px) 50vw,  33vw"
/>
```

### Art Direction

- If we want to use different images based on device width:

```html
<picture>
  <source media="(max-width: 500px)" srcset="images/meal-cropped.jpg" />
  <source media="(min-width: 501px)" srcset="images/meal.jpg" />
  <img src="images/meal.jpg" alt="" />
</picture>
```

### Font Icons

- You have to add a JavaScript script to your html. Then use `i` elements (or you can change it to `span` (because `i` was for italic and deprecated)) and add some classes to show the icon. You can wrap the `i` with a span and change the color or size.

## Forms

```html
<form>
  <div class="form-group">
    <label for="name">Name</label>
    <input id="name" type="text" placeholder="Enter first name" />
  </div>
  <div class="form-group">
    <label for="email">Email</label>
    <input id="email" type="email" value="initialEmail@gmail.com" />
  </div>
  <button type="submit">Submit</button>
  <button type="reset">Clear</button>
</form>
```

```css
body {
  line-height: 1.5;
  padding: 1rem;
}

label {
  display: block;
}

input[type="text"],
input[type="email"] {
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 0.5rem 0.7rem;
  transition: border-color 0.15s, box-shadow 0.15s;
}

input[type="text"]:focus,
input[type="email"]:focus {
  border-color: blue;
  outline: 0;
  box-shadow: 0 0 0 4px rgba(24, 117, 255, 0.25);
}

button {
  background: blue;
  color: white;
  border: 0;
  padding: 0.5rem 0.7rem;
  border-radius: 5px;
  outline: 0;
}

.form-group {
  margin-bottom: 1rem;
}
```

- We have `text`, `number` `password`, `email`, `date`, etc. for text fields.

```html
<textarea cols="30" row="10"></textarea>
```

```css
textarea {
  resize: none; /* not to allow resizing */
}
```

- We have two more attributes for text fields such as `readonly` and `disabled` which are similar but `disabled` will not be focused and submitted.
- We can have `maxlength ="5"` attribute for limiting inputs.
- We can have `autofocus` attribute to make it focused by default.

### Data Lists

- It is a suggestion list in HTML and is limited.
- User still can type anything which is not in the list.
- Styling of this should be done by JS.

```html
<input type="text" list="numbers" autocomplete="off" />
<datalist id="numbers">
  <option>1</option>
  <option>2</option>
  <option>3</option>
</datalist>
```

### Drop-down

```html
<select>
  <option value="">Select a course...</option>
  <option value="1" selected>HTML</option>
  <option value="2">CSS</option>
  <option value="3">JS</option>
</select>

<select>
  <optgroup label="Front-end">
    <option value="1" selected>HTML</option>
    <option value="2">CSS</option>
    <option value="3">JS</option>
  </optgroup>
  <optgroup label="Back-end">
    <option value="4" selected>Node.js</option>
    <option value="5">ASP.NET</option>
  </optgroup>
</select>

<select multiple>
  <option value="1">HTML</option>
  <option value="2">CSS</option>
  <option value="3">JS</option>
</select>
```

### Check Boxes

```html
<div>
  <input type="checkbox" id="agree" checked disabled />
  <label for="agree">Agree?</label>
</div>
```

### Radio Buttons

```html
<form>
  <div>
    <input type="radio" name="membership" id="silver" checked />
    <label for="silver">Silver</label>
  </div>
  <div>
    <input type="radio" name="membership" id="gold" />
    <label for="gold">Gold</label>
  </div>
</form>
```

### Range

```html
<form>
  <input type="range" min="0" max="100" value="90" />
</form>
```

### File Inputs

- We can use `image/*` as well to filter the accepted files.

```html
<form>
  <input type="file" multiple accept=".jpg, .png" />
</form>
```

### Grouping Related Fields

- You can either use `section` and `h2` or `fieldset` and `legend` elements to group related fields in a large form.

### Hidden Fields

- In MVC, we want to know the ID of for example edited record:

```html
<form>
  <input type="hidden" name="course-id" value="1234" />
</form>
```

### Validation

```html
<form>
  <input type="text" required minlength="3" maxlength="10" />
  <input type="email" required />
  <input type="date" required />
  <input type="number" required min="0" max="5" />
  <button type="submit">Submit</button>
</form>
```

### Submission

- If we use MVC, we set `action` attribute on the form element to our backend, and set the `method` to `POST` or `GET`. We have to specify `name` attributes for each input as well.

## Transformation, Transition, and Animation

### Transformation

- Note that if we combine, the order matters.

```css
.box:hover {
  transform: rotate(10deg) scale(2) skew(10deg) translate(10px, 0px);
}
```

- For 3D transformation, we need to define our perspective:

```css
.box:hover {
  transform: perspective(200px) translateZ(-50px);
}
.box:hover {
  transform: perspective(200px) rotateY(45deg);
  transform-origin: 0 50%;
}
```

- You just have to make sure to move the `perspective: 200px;` (as a rule and not a function any more) to their parent container, if you want to children, share the same transformation.

### Transition

- The transition will be applied to the main selector (not the one with pseudo class):

```css
.box {
  transition: transform 0.5s ease-in, background 0.5s; /* Not just transform but any CSS property that can change */
}
```

### Animation

```css
@keyframes pop {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(2);
  }
  100% {
    transform: scale(1);
  }
}

.box:hover {
  animation-name: pop;
  animation-duration: 1s;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
  animation-delay: 0;
}
```

## Best Practices

- Follow a naming convention for naming classes such as kebab-casing.
- Create logical sections into different files: forms, navbar, ... and combine them using `sass`.
- Avoid over-specific selectors (just define a new class!).
- Avoid important (it is usually because of over-specific selector).
- Sort CSS properties alphabetically (with VSCode).
- Take advantage of style inheritence (give font-family to body).
- Keep it DRY with variables:

```css
:root {
  --primary-color: #ffdd35;
}

.box {
  background: var(--primary-color);
}
```

- Separate container and content classes. Do not nest them. Because we want to re-use that content class.
- Separate structure and skin. You will define two classes and assign both to elements but you will not have duplication.
- Use `BEM` for naming classes of elements that defined in the context of a block such as `card__header`, or `card__body` but not for `btn`. For modifier, use `card--popular`. For separating words, just use `-`.
- Install `HTML CSS Support`, `Highlight Matching Tag`, `TODO Highlight`, and `CSS Peek` extensions.

## Project

- Projects usually have `--color-primary`, `--color-secondary`, `--color-accent`, `--color-heading`, and `--color-body` colors need to be defined in variables.
- For `*`, set `box-sizing: border-box;`.
- For the html element, set `font-size: 62.5%;`.
- For body element, set `font-family`, `font-size`, `line-height` and `color`.
- For headings, set the color. For each of them individually, set the sizes.
