## Introduction

### Topics Covered

- Styling
- Including Graphic Elements

### Lesson Objectives

By the end of this lesson, we will:

#### Objective 1: Styling

- examine several approaches to adding styles to a React application
- author styling using css modules
- author styling using styled-components

#### Objective 2: Including Graphic Elements

- discuss techniques used to include imagery
- highlight differences between `/public` and `/assets` directories
- describe how to add images to elements

## Discussion Topics

> [!note]
> We will be working CSS as it relates to React but won't be teaching any styling basics. Remember to use to the list of references at the bottom of each lesson in case you need to brush up on any non-React topics.

### Styling

The Vite React Template provides 2 stylesheets: `App.css` and `index.css`. These are good starting points that allow us to quickly start with small projects. We don't want to rely on them if we know our application will continue to grow. Remember that React is a powerful tool and and must render each component and its children in equivalent html elements. As an app component numbers increase, the resulting html that renders our interface becomes increasingly complex.

Class and id usage alleviates this problem temporarily but keeping track of them and what they do becomes the next coding challenge for a developer. Naming conventions such as [SMACSS](https://smacss.com/) or [BEM (Block, Element, Modifier)](https://en.bem.info/methodology/quick-start/) - can help organize an increasing number of CSS selectors but only to a certain point. "Naming things is hard" has been spoken by many developers. Because this scaling problem is so common in React projects, we look for libraries that address it directly. The React ecosystem contains several libraries that help us organize styling. To examine two of these, we will be employing CSS Modules on the cart items and use Styled Components .

#### CSS Modules

[CSS Modules](https://github.com/css-modules/css-modules) allow us to scope class-based rules by working with module files that ensure defined classes only affect the the associated component. Limiting the scope of a class to the component means that we reuse class names in other modules without conflict. The benefit to this is that naming css classes becomes a much simpler matter than on a global scope.

Vite includes CSS Modules by default so we don't need to import the library into our component. We instead import a stylesheet dedicated to the component. [Vite requires](https://vite.dev/guide/features.html#css-modules) that we must end our stylesheets' file names with `.module.css`. As a best practice, we also use matching css and component file names and keep the files co-located in the project.

```terminal
# part of the directory structure for CTD-Swag
├── features/
     ├── ProductList/
   ├── ProductList.jsx
   ├── ProductList.module.css
   ├── ProductCard.jsx
   └── ProductCard.module.css
     └── Cart/
   ├── Cart.jsx
   ├── Cart.module.css
         └── CartItem.jsx (new component)
```

When we import a stylesheet into its associated component the CSS Module library parses the file contents. It then makes any classes found in the file's rules available by dot-notation on import statement object. For example, `<li className="item">` becomes `<li className={styles.cartItem}>`. Seeing in context to the component may illustrate this better:

```jsx
{/*CartItem.jsx*/}
import placeholder from '../../assets/placeholder.png';
import styles from './CartItem.module.css';

function CartItem({ item, onHandleItemUpdate }) {
  return (
    <li className={styles.item}> {/*<--- references style from import*/}
      <img src={placeholder} alt="" />
      <div>
        <h2>{item.baseName}</h2>
        {item.variantName !== 'Default' ? <p>{item.variantName}</p> : null}
      </div>
      <div className={styles.subtotal}> {/*<--- references style from import*/}
        <label>
          Count:{' '}
          <input
            type="number"
            value={item.quantity}
            onChange={(event) =>
              onHandleItemUpdate({ event, id: item.productId })
            }
          />
        </label>
        <p>Subtotal: ${(item.price * item.quantity).toFixed(2) || '0.00'}</p>
      </div>
    </li>
  );
}

export default CartItem;
```

To employ the stylesheet module, we move rules out of `App.css` that apply only to this component. Scoping gives us the opportunity to simplify names.

```css
/*CartItem.module.css*/

.item {
  font-size: 1rem;
  color: var(--dark-blue);
  border-radius: var(--small-radius);
  background-color: #fff;
  display: flex;
  justify-content: space-between;
  margin: 0;
  align-items: center;
  margin-bottom: .25rem;
  padding-left: .75rem;
}

.item h2 {
  padding: 8px;
}

.item img {
  max-height: 72px;
}

.subtotal {
  border-left: 1px solid var(--dark-blue);
  padding: .25rem;
  text-align: right;
}
```

When we go into the elements console of the browser, we can see scoping in action with the transformed class names. When Vite compiles our code, CSS Modules transforms the classnames so that they only target elements related to that component. The class names generated by CSS Modules are typically a combination of the original class name and a unique hash. For example, the class name `cartItem` might be transformed into something like `_cartItem_l8scs_1`.

![[202501_0904PM-Brave Browser.png|700]]

We continue to write normal CSS with CSS Modules which makes it an attractive tool but have to be careful with element-only selectors. Since they are not class names, any styles using only element or attribute selectors gets added to the file added to the global styles. To ensure rules are scoped, make sure all selectors include a class name. Even though it is possible to write selectors, it's discouraged since it can affect elements found in other components.

```css
/*scopes to component*/
.cartItem {
/*various styles*/
}

/*scopes to component*/
.cartItem h2 {
  /*various styles*/
}

/*scopes to component*/
.cartItem img {
  /*various styles*/
}

/*scopes to component*/
.subtotal {
  /*various styles*/
}

/*scopes globally!*/
h2 {
  background-color: green
}
```

See below how the green background of the `h2` affects not only the cart total but also the product titles on the blurry product cards.

![[202501_0824AM-Brave Browser.png|500]]

There are use cases for element-only selectors. These usually revolve applying some baseline styling globally across the app. Any such styling a React application needs to be kept in a centralized location. This can either be in `App.css` or some other centralized location. The highlighted rules below provide a few examples of how globalized styles can be used.

![[202501_1238PM-Code.png|600]]

 Like all JavaScript libraries, CSS Modules has a few other features not discussed here but they are for more advanced use cases. Even not taking advantage of them, CSS Modules is a powerful library. It helps a developer produce a maintainable app by the reducing the risk of style leakage or unintended overrides. It takes the familiar approach to writing CSS and in returns provides scoping to keep css class naming strategies manageable in a growing project.

#### Styled Components

[Styled Components](https://styled-components.com) scopes styling to a component but uses a drastically different approach. Rather than using styles from a css file, it takes a "css-in-js" approach. We write our styles using [tagged template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) with transformation functions provided by Styled Components. These functions use the styles found in the template literal to configure the styles of a component that it returns. We don't need to worry about naming style classes anymore - just new components. Let's look at an example since tagged template literals are not a frequently used JavaScript feature.

```jsx
{/*extract from ProductCard.jsx*/}
{/*...code*/}

const Details = styled.p`
  padding: 0.5rem;
`;
{/*code continues...*/}
```

The styles in the backticks are processed by the library and returns the component `Details` inside of the paragraph elements. Here, styled components gives us a `.p` transformation function. This function will return a component based on the `<p>` element and scope the classes using dynamic class names.

```jsx
{/*extract from ProductCard.jsx*/}
{/*...code*/}

<p>{product.baseDescription}</p>
<p>${product.price.toFixed(2) || '0.00'}</p>

{/*is equivalent to*/}

<Details>{product.baseDescription}</Details>
<Details>${product.price.toFixed(2) || '0.00'}</Details>

{/*code continues...*/}
```

We can also make Styled Components out of custom components. Instead of referencing any one of the transformation functions, we provide the component as an argument. We can then add styles in the same manner.

```jsx
const StyledProductVariants = styled(Product)`
/*component styles*/
`
```

##### Making Classes Readable

The class names are generated dynamically by a process internal to the library. This can be a challenge when troubleshooting styles since the class names are unreadable hashes.

![[202501_1018AM-Brave Browser.png|500]]

 We can install the [Babel plugin for styled components](https://styled-components.com/docs/tooling#babel-plugin) for readable class names to aid in troubleshooting.

```terminal
npm install --save-dev babel-plugin-styled-components
```

After installing the library, we register this plugin with Vite's React plugin inside `vite.config.js`.

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-styled-components'],
      },
    }),
  ],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './test.setup.js',
  },
});
```

This results in 2 classes for each relevant component The first class name include the its component name and its parent component. There's still some additional hashing characters but it's now much easier to match elements to a particular component.

![[202501_1025AM-Brave Browser.png|500]]

The styles are still applied through the randomly hashed class name but those can be matched to styles in your browser's elements tab:

![[202501_1057AM-Brave Browser.png|500]]

Writing styled components uses a nesting syntax similar to SCSS to keep keeps rules compact. In the example below, we add a `button` selector at the same level as the width, height, and display rules. Since "button" is not a css style, Styled Components treats this as a portion of a compound selector that targets buttons are found in the `ButtonWrapper` component. The rule applying to these buttons is then written in a nested block beside the compounded selector. We can include a `&` operator to target pseudo-selectors or optionally ensure that compound selectors apply across all instances of a component.

```scss
{/*extract from ProductCard.jsx*/}
{/*...code*/}
const ButtonWrapper = styled.div`
  width: 100%;
  height: 4rem;
  display: flex;
  {/*this selector includes a space after the "&" */}
  button {
    border: none;
    background-color: rgb(from var(--medium-blue) r g b / 0.5);
    width: 100%;
    &:hover {
      background-color: rgb(from var(--medium-blue) r g b / 0.25);
    }
    &:active {
      background-color: rgb(from var(--light-blue) r g b / 0.25);
    }
  }
{/*code continues...*/}
```

In the example above, we see that we can perform nesting on multiple levels. The resulting rules are then added to the application's styles.

```css
/*extract from the browser's elements console*/
.kocZSi {
    width: 100%;
    height: 4rem;
    display: flex;
}

.kocZSi button {
    border: none;
    background-color: rgb(from var(--medium-blue) r g b / 0.5);
    width: 100%;

.kocZSi button:hover {
    background-color: rgb(from var(--medium-blue) r g b / 0.25);
}

.kocZSi button:active {
    background-color: rgb(from var(--light-blue) r g b / 0.25);
}
/*code continues...*/
```

CSS Modules provides far more features than we use in CTD Swag and it's important to include one more in our conversation. Since this library returns components, it also has the ability to work with props. Can use these props to configure dynamic selectors and rules to meet a range of use cases. In the example below, we pass in "blue" to the `color` props in `SuccessModal`. That prop can then be referenced as an input for a function that sets the `color` property's value.

```jsx
import styled from 'styled-components';
//...component code

<SuccessModalTitle color="blue">Success Message</SuccessModalTitle>;

//continued component code

const SuccessModalTitle = styled.h1`
    color: ${(props) => props.color || 'green'};
    font-size: 24px;
    font-weight: bold;
`;
```

### Including Graphic Elements

CTD Swag usage imagery for the product previews, title and the shopping cart. Each of these can be categorized into content or UI element.

![[202501_0248PM-Firefox Developer Edition.png|500]]

- content
  - product preview image
  - product placeholder image
- ui element
  - favicon
  - CTD logo in the title
  - shopping cart icon

Pictures are ubiquitous across the web and most React applications reflect this. When we think about imagery for an application, it is usually in terms of how it's used on a page. We can distill most uses down to 2 major use cases: page content or visual UI elements. We have several approaches to bringing imagery into our application to support our needs.

#### The `/public` Directory

The first and easiest to embed the images into the project's code base. The React template that Vite uses provides two locations where we can place them: `/public` and `/assets`.

The `/public` directory is used for managing static assets. Generally, these files are used outside of the application such as the page's favicon or for any imagery that, for whatever reason, cannot be imported directly into a component. Vite will not make changes in any way to these files and serves them from the relative url, `/`, when the application is running locally. We can use sub-folders to keep assets organized.

CTD-Swag currently uses this directory for the favicon for the site and all the project previews. CTD's imagery is here because they are only temporarily housed in the project. The images will eventually come from a live server so it makes more sense to access them via a relative path rather than import them.

![[202501_0246PM-Brave Browser.png|700]]
![[202501_0250PM-Code.png|400]]

#### Static Assets

Vite uses `/assets` to house asset files that can be imported and provides us with some handy things under the hood. CTD Swag's product placeholder, logo, and shopping cart icon reside in this directory.

The placeholder image that we have been using is a fallback for the product image if it fails to load. Since it is not associated with a specific product, we keep it colocated with the rest of the codebase instead of fetching it from the back end's API.

Whenever the image is imported from `assets`, Vite gives us a url that we can then use in our app.

![[202501_0330PM-Code.png|600]]
![[202501_0333PM-Brave Browser.png|600]]

In production, Vite adds a short hash to each asset's file name which is useful for [cache invalidation](https://www.keycdn.com/support/what-is-cache-busting). Caching assets in the browser saves the user time and bandwidth but it makes updating them challenging. The easiest way to get an asset to load outside of cache is to change its name. Every time our application is re-built during production, our assets' filenames include a random hash which fools the browser into thinking it is a new file. Rather than serving `placeholder.png` each time the application is updated Vite may serve `placeholder.79c5eb7.png` and then update it to `placeholder.ed84551.png` when developer deploys a new version of the application to production.