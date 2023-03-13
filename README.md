# Reusable UI Component API Guide

## Goals

Define and outline API best practices for creating highly reusable UI components that work in any Web framework or library. These recommendations are ideal for stateless leaf components in UI component libraries and design systems. However, these guidelines may only apply to some UI component design use cases at the application level.

## Non-Goals

This guidance is not intended to define best practices around UI component API design in the use cases for micro-frontends or high-level application UI state. However, many of the best practices described here may still be applicable.

## Legend

- ✅ Use: best practice
- 🚫 Avoid: practice to avoid
- 🚧 Warning: risks of not following guidance
- 🏁 Performance: details of impact to performance
- 📘 Tip: details on reasoning or rationale for guidance
- 🎓 Learn: resources to learn more

## Terminology

- `Component`: Web Component defined via `Custom Elements API`
- `Framework Component`: Component defined as part of a consuming application
- `Pattern`: A combination of Web Components to create new UI without introducing new components to the library
- `Users`: Consumers or users of the Web Component library

## Topics

- [Compatibilty](#compatibilty)
- [Element Registration](#element-registration)
- [Properties & Attributes](#properties--attributes)
  - [Attributes](#attributes)
  - [Properties](#properties)
  - [Impossible States](#impossible-states)
  - [Primitive Types](#primitive-types)
  - [Boolean Types](#boolean-types)
- [Custom Events](#custom-events)
- [Slots](#slots)
  - [Slot Composition](#slot-composition)
  - [Default Slots](#default-slots)
  - [Encapsulate Slots](#encapsulate-slots)
  - [Rendering](#rendering)
- [Stateless APIs](#stateless-apis)
  - [Escape Hatch - anti-pattern](#escape-hatch---anti-pattern)
  - [State Synchornization](#state-synchornization)
  - [Stateful Events](#stateful-events)
- [Composition](#composition)
  - [API Inheritance - anti-pattern](#api-inheritance---anti-pattern)
  - [Semantic Obfuscation - anti-pattern](#semantic-obfuscation---anti-pattern)
- [Styles & CSS](#styles--css)
  - [Custom Properties](#custom-properties)
  - [Internal Host](#internal-host)
  - [State Properties](#state-properties)
  - [Custom State Pseudo Classes](#custom-state-pseudo-classes)
  - [Logical Properties](#logical-properties)
  - [Parts](#parts)
  - [Margins & Whitespace](#margins--whitespace)
  - [Responsive](#responsive)
- [Publishing](#publishing)

## Compatibilty

Consistent API design improves not only the developer experience but also the consistency and compatibility across various UI frameworks and libraries. This guide focuses on building reusable UI via Web Components to ensure maximum compatibility across the Web ecosystem. While the code examples are Web Component focused, most of the concepts also apply to framework-specific component models.

### HTML/JS

```html
<ui-alert status="success">hello world!</ui-alert>
<script type="module">
  import 'ui-alert.js';
  const alert = document.querySelector('ui-alert');
  alert.closable = true;
  alert.addEventListener('close', () => alert.remove());
</script>
```

### Angular

```html
<ui-alert status="success" *ngIf="show" [closable]="prop" (close)="fn($event)">
  hello world!
</ui-alert>
```

### Lit

```html
<ui-alert status="success" .closable=${this.prop} @close=${e => this.fn(e)}>
  hello world!
</ui-alert>
```

### Vue

```html
<ui-alert status="success" :hidden="show" :closable="prop" @close="fn">
  hello world!
</ui-alert>
```

### Preact

```html
<ui-alert status="success" hidden={show} closable={prop} onClose={this.fn}>
  hello world!
</ui-alert>
```

## Element Registration

Custom Elements are registered to a global scope. This means collisions can occur if two elements attempt to register using the same tag name. To minimize this risk, prefix the element unique to your application or library.

```html
<ui-alert></ui-alert>

<app-alert></app-alert>

<product-alert></product-alert>
```

🎓 **Learn**: A [Scoped Element Registry](https://github.com/WICG/webcomponents/blob/gh-pages/proposals/Scoped-Custom-Element-Registries.md) spec is in progress with an experimental [polyfill](https://github.com/webcomponents/polyfills/tree/master/packages/scoped-custom-element-registry). Lit also provides an experimental integration [@lit-labs/scoped-registry-mixin](https://github.com/lit/lit/tree/main/packages/labs/scoped-registry-mixin).

## Properties & Attributes

Properties and attributes should represent the visual state of a component. There are various types of states, but most fall under the following:

- types: `status: danger | success | warning`
- states: `expanded | readonly | selected | disabled`
- behavior: `draggable | closable | resizable`

Properties and attributes can represent the same value and "reflect" or keep in sync between the two.

#### Attributes

```html
<ui-alert status="success"></ui-alert>
```

#### Properties

```html
<<ui-alert></ui-alert>
<script type="module">
  import 'ui-alert.js';
  const alert = document.querySelector('ui-alert');
  alert.status = 'success';
</script>
```

### Impossible States

Avoid creating "impossible states" in your component APIs. Impossible states are typically caused by different API options describing the same visual state.

#### ✅ **Use**

```html
<ui-alert status="success"></ui-alert>
```

#### 🚫 **Avoid**

```html
<ui-alert success></ui-alert>
<ui-alert danger></ui-alert>
<ui-alert warning></ui-alert>

<ui-alert success warning></ui-alert> <!-- impossible state -->
```

By leveraging attribute or property values, we can create enum-style APIs that prevent impossible states, such as an alert being both in a success and warning state.

🎓 **Learn**: [Make Impossible States Impossible](https://kentcdodds.com/blog/make-impossible-states-impossible)

### Primitive Types

Many Web Component authoring libraries, such as [lit](https://lit.dev), can easily keep attributes and properties in sync. This allows your component APIs to accept data with HTML attributes or JavaScript properties. However, HTML attributes are always treated as a string values. Because of this behavior, only use complex types such as `Object` and `Array` when setting properties.

#### ✅ **Use**

```javascript
const element = document.querySelector('ui-element');
element.items = [1, 2, 3];
```

#### 🚫 **Avoid**

```html
<ui-element items="[1, 2, 3]"></ui-element>
```

Frameworks have property binding syntax that allows the JavaScript property to explicitly be set rather than the attribute.

```html
<!-- angular property -->
<ui-element [items]="myItemsProp"></ui-element>

<!-- vue property -->
<ui-element :items="myItemsProp"></ui-element>

<!-- lit property -->
<ui-element .items="myItemsProp"></ui-element>
```

Complex types should be avoided for highly reusable UI components as they can cause compatibility and usability issues requiring explicit JavaScript to define and render content. In addition, this can make the component challenging to use for user-generated content in CMS systems or SSR/low code applications that typically render static HTML.

🎓 **Learn**: [web.dev: Attributes and Properties](https://web.dev/custom-elements-best-practices/#attributes-and-properties)

### Boolean Types

Following the native HTML boolean attribute rules can make your components consistent and easy to use across frameworks. [Boolean style attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/disabled) in HTML are truthy when present and falsy when absent.

```html
<ui-alert closable></ui-alert> <!-- true -->
<ui-alert closable="false"></ui-alert> <!-- true -->
<ui-alert></ui-alert> <!-- false -->
```

🚧 **Warning**: Avoid double negations with APIs as it breaks common expectations with boolean behavior in HTML. (`disable-closable`, `closable="false"`)

🚧 **Warning**: [React may incorrectly](https://custom-elements-everywhere.com/#react) set boolean attribute values as strings.

🎓 **Learn**: [Reusable UI Components and Data Binding](https://coryrylan.com/blog/reusable-ui-components-and-data-binding)

## Custom Events

Events communicate user intent. Examples of custom user events can include `close`, `change`, `open`. Events should remain stateless by emitting based on user interactions with the component and not from component state changes.

```html
<ui-dialog>hello</ui-dialog>
```

```javascript
const dialog = document.querySelector('ui-dialog');
dialog.addEventListener('close', () => dialog.hidden = true);
```

Avoid using verb/action prefixes to events such as `on`. Most frameworks de-sugar event handlers or auto-prefix the event name within the syntax. For example Angular `(close)="handle()"` and Preact `onClose={this.handle}`.

🚧 **Warning**: For maximum compatibility use lower case events as [some frameworks](https://custom-elements-everywhere.com/#vue) incorrectly ignore case sensitivity with Custom Events.

🚧 **Warning**: Avoid overriding existing event names from `HTMLElement`. Overloading events can break behavior and expectations with components. Example: [eslint rule](https://github.com/stencil-community/stencil-eslint/blob/main/src/rules/reserved-event-names.ts).

## Slots

Slot or content projection enables a flexible API for consumers to provide dynamic content within a component. Slots are ideal for any content an application may render. Slots are commonly used in container-style components such as cards and tabs. Slots give complete control of the content to the host application in ways that properties and attributes would be limiting.

📘 **Tip**: If the text content is visible to the user then likely the API should use a slot

📘 **Tip**: Slots enable consumers to use the i18n solution of their choice

```html
<ui-alert status="warning">
  <p>usage limits at <strong>90%</strong>
  <a href="#">check status</a></p>
</ui-alert>
```

### Slot Composition

Slots can leverage component composition, decoupling behavior, and render/DOM order.

```html
<ui-button>
  <ui-icon name="user"></ui-icon> icon on the left
</ui-button>

<ui-button>
  icon on the right <ui-icon name="user"></ui-icon>
</ui-button>
```

Allowing composition makes API flexible to different i18n solutions and use cases that properties or attributes would prevent.

### Default Slots

Providing reasonable defaults for a component to improve the developer experience is important. However, sometimes the defaults themselves need to be customized. For example, an alert message may have different status states (e.g., success, warning, danger), each displaying a different icon within the alert. In these cases, it may be necessary to customize the component's default behavior to meet the application's specific needs.

```html
<ui-alert status="success">success message</ui-alert>
<ui-alert status="warning">warning message</ui-alert>
<ui-alert status="danger">danger message</ui-alert>
```

We can leverage default slots to provide customization hooks to avoid the risk of the alert element absorbing parts of the icon API (as discussed above). This allows the alert component to internally provide default icons for different status states while allowing developers to customize the icons as needed. Using default slots, we can mitigate the risk of tightly coupled APIs and maintain a clear separation of concerns between the alert and icon elements.

```html
<!-- ui-alert template -->
<slot name="icon">
  <ui-icon name=${this.status}></ui-icon>
</slot>
```

```html
<!-- API usage -->
<ui-alert status="warning">
  <ui-icon slot="icon" name="custom"></ui-icon>
  custom warning
</ui-alert>
```

Slots can provide default content if no content is provided. For example, in an alert element, we can set an internal icon with a status icon that matches the status of the alert. If consumers want to customize the icon, they can do so by projecting their icon into the icon slot, overriding the default. This enables complete control over the custom icon without the alert element needing to expose the icon through a series of inherited attributes and properties. As with all API design choices, tradeoffs are involved, and it is important to consider each approach's benefits and potential challenges.

🎓 **Learn**: [Reusable Component Patterns - Default Slots](https://coryrylan.com/blog/reusable-component-patterns-defult-slots)

### Encapsulate Slots

Slots can have names or specific locations where content should be rendered. Encapsulating named slots as custom elements allows for more declarative flexibility for the component.

#### ✅ **Use**

```html
<ui-dialog>
  <ui-dialog-content>
    <h3>Dialog Title</h3>
    <a href="#">learn more</a>
  </ui-dialog-content>
  <ui-dialog-footer>
    <ui-button>action</ui-button>
  </ui-dialog-footer>
</ui-dialog>
```

#### 🚫 **Avoid**

```html
<ui-dialog>
  <div slot="heading">
    <h3>Dialog Title</h3>
    <a href="#">learn more</a>
  </div>
  <div slot="footer">
    <ui-button>action</ui-button>
  </div>
</ui-dialog>
```

### Rendering

Composition-based APIs with slots allow consumers to determine the most appropriate rendering strategy. This is important for large lists of elements. For example, a treeview component theoretically may represent a dataset with hundreds or thousands of nodes.

```html
<ui-tree>
  <ui-tree-node></ui-tree-node>
  <ui-tree-node></ui-tree-node>
  <ui-tree-node></ui-tree-node>
  <ui-tree-node expanded>
    <ui-tree-node></ui-tree-node>
    <ui-tree-node></ui-tree-node>
    <ui-tree-node></ui-tree-node>
  </ui-tree-node>
</ui-tree>
```

With slots the host application can use the native rendering APIs.

#### Angular

```html
<ui-tree>
  <ui-tree-node *ngFor="let node of nodes">{{node.id}}</ui-tree-node>
</ui-tree>
```

#### Preact

```jsx
<ui-tree>
  {nodes.map(node => <ui-tree-node>{node.id}</ui-tree-node>)}
</ui-tree>
```

This allows the app to make optimizations that the treeview cannot assume, such as what nodes to render safely.

#### Angular

```html
<ui-tree>
  <ng-container *ngFor="let node of nodes">
    <ui-tree-node *ngIf="node.expanded">{{node.id}}</ui-tree-node>
  </ng-container>
</ui-tree>
```

## Stateless APIs

### Escape Hatch - anti-pattern

Reusable components should remain as stateless as possible. The visual output of a component should represent the state passed into it via properties and attributes. In this example, we will use a dialog component.

The dialog component should render conditionally based on the application state. For example, we provide a `close` event rather than defining an `open` or `show` property. The dialog will only emit the event when the user has clicked the close interaction. The event notifies and allows the host application to determine if the dialog should be hidden.

This subtle API difference prevents "escape hatch" APIs. If our dialog controls its visibility state, this can cause additional APIs to be needed. For example, suppose an application needs to validate a form before closing the dialog. In that case, we have to introduce some life cycle events such as `before-close` to prevent the dialog from closing automatically by the user.

An Angular application with a stateless dialog.

```html
<ui-dialog *ngIf="show" (close)="closeDialog()"></ui-dialog>
```

```javascript
class AppComponent {
  show = false;

  async closeDialog() {
    const valid = await someAsyncValidation();

    if (valid) {
      this.show = false;
    } else {
      this.showValidationWarning();
    }
  }
}
```

Since we pushed the state to the application, the consumer has complete control of when and how the dialog should be rendered, including dynamically showing via the HTML `hidden` attribute or conditionally rendering DOM.

📘 **Tip**: Reusable and stateless test, "Can you prototype/demo any visual state of the component with just HTML?"

### State Synchornization

Stateless components prevent the UI state from becoming out of sync. For example, suppose the dialog component controls or modifies its visibility state. In that case, it can open up situations where the dialog state is closed, but the host application state keeps its dialog visibility marked as open.

🎓 **Learn**: [Stateless vs Stateful React Demo](https://stackblitz.com/edit/http-server-kojqdg?file=src%2FApp.tsx)

🎓 **Learn**: [Stateless vs Stateful Angular Demo](https://stackblitz.com/edit/angular-ivy-cjr3hj?file=src%2Fapp%2Fapp.component.html)

### Stateful Events

Avoid emitting events in response to state changes of a component. As a consumer of a component, this creates unnecessary noise. For example, if an app sets a component's `expanded` state property, then it does not need a `expand` or `open` event to fire as it initiates the state change. Instead, events should only be triggered by user interactions.

🚧 **Warning**: Reflective state events can cause excessive rendering or infinite redner loops.

## Composition

### API Inheritance - anti-pattern

Components should default to composition whenever possible to maximize flexibility, compatibility, and simplicity. API Inheritance occurs when an API unknowingly or over time re-implements another element's API to expose additional access to its internal implementation details. For example, using a button and icon element can illustrate some of the tradeoffs involved in this approach.

```html
<ui-button icon-name="menu">
  menu
</ui-button>
```

```html
<ui-button>
  menu <ui-icon name="menu"></ui-icon>
</ui-button>
```

While it may be tempting to encapsulate other components and expose their APIs to the host component, this approach can quickly lead to API conflicts. For example, if a button has an icon, we may want to provide an icon API to set the button icon. However, this could conflict with the native button API, which already has a name attribute. To avoid these conflicts, it is crucial to consider the tradeoffs involved in exposing APIs and carefully design the API to minimize conflicts and maximize compatibility.

We may encounter layout conflicts as we continue to encapsulate other components and expose their APIs. For example, if we want to change the icon's position, we may need to provide a secondary "escape hatch" API to modify this behavior. This illustrates the tradeoffs involved in encapsulating other components and exposes the potential challenges when designing APIs.

```html
<ui-button icon-position="end" icon-name="menu">
  menu
</ui-button>
```

```html
<ui-button>
  menu <ui-icon name="menu"></ui-icon>
</ui-icon>
```

Encapsulating other components and exposing their APIs can introduce "escape hatch" APIs that the host component must support, which can complicate internationalization use cases where reading order and the elements are reversed for right-to-left languages. This approach may work in some cases, but it is important to consider the tradeoffs and potential challenges involved in exposing APIs in this way.

As the API for the icon component grows over time, there may be pressure for the host component's API (e.g., the button API) to absorb and expose additional API endpoints for the icon. This pressure, known as "API inheritance," can create tightly coupled and non-explicit APIs that only exist for certain component combinations. As a result, the API becomes more complex and verbose as more "escape hatches" are required. In this example, API inheritance leads to an API with 66 characters, while composition and slots result in an API with only 59 characters. By leveraging composition and slots, we can avoid supporting tightly coupled APIs and keep the supported API surface area smaller and easier to learn in the long term.

### Semantic Obfuscation - anti-pattern

When designing composition-based APIs, it is important to push the semantics of the HTML into the light DOM or the control of the consumer. This helps to create a correct DOM structure and improved accessibility. For example, if a card element includes an `h1` heading, this makes an incorrect DOM structure, as there should only be one `h1` element within the page. By pushing the semantics of the heading up into the light DOM, the consumer can control the heading level and ensure that the page structure is correct.

```html
<!-- ui-card template -->
<h1><slot name="header"></slot></h1>
<slot></slot>
<slot name="footer"></slot>

<!-- API usage -->
<ui-card>
  <div slot="header">card header</div>
  <p>card content</p>
</ui-card>
```

It is important to push the semantic meaning of elements up to the consumer rather than obfuscating it internally. This enables the consumer to control which heading level is appropriate for their page, which is important for accessibility and features like aria-describedby, used to associate content. Furthermore, by exposing the semantic meaning of elements to the consumer, we can enable more accessible and flexible behaviors in our components and libraries.

```html
<!-- ui-card template -->
<slot name="header"></slot>
<slot></slot>
<slot name="footer"></slot>

<!-- API usage -->
<ui-card>
  <h2 slot="header">card header</h2>
  <p>card content</p>
</ui-card>
```

Composition-based APIs may be more verbose than other approaches. However, they have the advantage of lowering the overall API surface area of the system and ensuring that there is only one way to use any component. This makes it easier for consumers to learn and use the APIs, as the usage remains predictable and reliable throughout the system.

Adding a layer of abstraction on top of the components is possible to provide more opinionated, terse APIs while enabling consumers to access the reusable components directly as needed. However, it is essential to carefully consider the tradeoffs involved in using abstractions. It can be easier to add new abstractions but much more difficult to fix or remove the wrong abstraction.

🎓 **Learn**: [Reusable Component Anti-Patterns - Semantic Obfuscation](https://coryrylan.com/blog/reusable-component-anti-patterns-semantic-obfuscation)

## Styles & CSS

### Custom Properties

It is better to use shorthand values instead of specific properties. This gives the user more flexibility without making the public API surface bigger. When it can be done, keep the names of CSS Custom Properties simple and match them one-to-one with the native CSS property. This makes the API easier to learn and consistent with other components in the project.

#### ✅ **Use**

```css
:host {
  --border: ...
  --padding: ...
  --background: ...
}
```

#### 🚫 **Avoid**

```css
:host {
  --border-color: ...
  --border-width: ...
  --padding-left: ...
  --background-color: ...
}
```

### Internal Host

The internal host element pattern offers protection for the element styles in the API. When customizing the look of a custom element, try not to add styles to the host element beyond simple display properties and custom properties. The more styles that are added to the host, the more likely it is for the user to modify them in unexpected ways, potentially affecting the desired appearance.

#### 🚫 **Avoid**

```css
/* component styles */
:host {
  --background: #fcfcfc;
  --color: #2d2d2d;
  background: var(--background);
  color: var(--color);
  display: flex;
}
```

```css
/* consumer styles */
ui-card {
  /* bypassing defined custom property API */
  background: #fff;

  /* display override could break layouts in unexpected ways */
  display: inline;
}
```

This component attaches styles directly to the `:host` element. Sometimes this is necessary for certain styles like display types. However, it also makes it easy for consumers to override your component styles unexpectedly. For example, the component is set to display flex but is being overridden to block by the consumer.

Style overrides like this are possible because the component allows its styles to leak into the public API. However, this API leak makes future changes or migrations of those components more problematic as it could have an unintended impact on the consumer application.

Applying an internal host element for styles prevents or mitigates style leakage. This will allow more control over explicitly available styles on the public API.

#### ✅ **Use**

```html
<!-- internal component template -->
<div internal></div>
```

```css
/* internal component styles */
:host {
  --background: #fcfcfc;
  --color: #2d2d2d;
}

[internal] {
  background: var(--background);
  color: var(--color);
  display: flex;
}
```

Adding an internal containing element prevents accidental style API leakage for the component. Now the consumer must use the exposed CSS Custom Properties in a predictable way to customize the component.

### State Properties

Leverage the `:host` selector with CSS Custom Properties to customize the visual states of a component. Using the host avoids expanding the public API of the component and provides a single consistent CSS API of the component. Each visual variant is responsible for modifying the existing public API to reflect the component's visual state.

```css
:host {
  --color: #fff;
  --background: #6d6f74;
  --padding: 16px;
  --border-radius: 4px;
  --font-size: 16px;
}

[internal] {
  border-radius: var(--border-radius);
  background: var(--background);
  padding: var(--padding);
  color: var(--color);
  font-size: var(--font-size);
  display: flex;
  align-items: center;
}
```

#### ✅ **Use**

```css
:host([status=success]) {
  --background: #298338;
  --color: #fff;
}

:host([status=danger]) {
  --background: #c21919;
  --color: #fff;
}

:host([size=compact]) {
  --padding: 8px 12px;
  --font-size: 14px;
}
```

#### 🚫 **Avoid**

```css
:host([status=success]) {
  background: #298338;
  color: #fff;
}

:host([status=danger]) {
  background: #c21919;
  color: #fff;
}

:host([size=compact]) {
  padding: 8px 12px;
  font-size: 14px;
}
```

With this approach, we define the look and feel of our component once. Then leveraging CSS Custom Properties, we "theme" the component for the new visual state. Also, by modifying the CSS Custom Properties, we keep the CSS specificity low and easy to maintain.

With this pattern, we also enable future customizations for consumers. By leveraging the state-based attributes, consumers can create their own custom-style states with the same API.

```html
<ui-alert status="promotion">product alert</ui-alert>
```

```css
ui-alert[status=promotion] {
  --background: purple;
  --color: gray;
}
```

🎓 **Learn**: [Style States with Web Components and CSS Custom Properties](https://coryrylan.com/blog/style-states-with-web-components-and-css-custom-properties)

### Custom State Pseudo Classes

When using custom elements, use the [Custom States API](https://developer.mozilla.org/en-US/docs/Web/API/ElementInternals/states) to style visual states of your component while avoiding the need to attach attributes or classes to the DOM and host element.

```javascript
class Checkbox extends HTMLElement {
  #internals = this.attachInternals();
  #checked = false;

  set checked(value) {
    this.#checked = value;

    if (this.#checked) {
      this.#internals.states.add('--checked');
    } else {
      this.#internals.states.delete('--checked');
    }
  }

  get checked() {
    return this.#checked;
  }
}
```

By leveraging the Custom States API, we can add custom CSS pseudo-classes to represent visual states without modifying DOM. This selector can be used internally in a component or externally as part of the public API.

```css
:host(:--checked) {
  --background: ...
}
```

```css
ui-checkbox:--checked {
  --background: ...
}
```

### Logical Properties

Logical Properties can ensure the component styles will support various RTL languages. Logical Properties apply styles based on block and inline axis that can be inverted based on the browser language settings.

#### ✅ **Use**

```css
[internal] {
  padding-inline: 12px 24px;
}
```

#### 🚫 **Avoid**

```css
[internal] {
  padding-left: 12px;
  padding-right: 24px;
}
```

🎓 **Learn**: https://web.dev/logical-property-shorthands/

### Parts

CSS Parts enable components to expose internal elements as a public API that usually are encapsulated via Shadow DOM.

```html
<!-- internal template ui-alert -->
<button part="close-button">
  close
</button>
```

```css
/* consumer CSS */
ui-alert::part(close-button) {
  ...
}
```

🚧 **Warning**: CSS Parts expose a significant amount of components' internal details, including DOM structure and style properties.
Be cautious in using CSS Parts as it increases the maintenance and risk of visual breaking changes of your component.

CSS Parts enable complete control of a DOM element to the consuming developer. However, this has a significant tradeoff. Exposing additional elements creates a more extensive public API surface. Over time this increases the difficulty of maintaining the API and changes to the internal template causing unexpected and brittle changes.

Exposing a component that is a versioned and controlled API can reduce this risk of API overexposure.

```html
<!-- internal template ui-alert -->
<ui-button part="close-button">
  close
</ui-button>
```

```css
/* consumer CSS */
ui-alert::part(close-button) {
  ...
}
```

The `ui-button` has a controlled and versioned API, so exposing the `ui-button` is slightly less risky since consumers can only use it via the public API defined by `ui-button`.

### Margins and Whitespace

Well-encapsulated components should avoid projecting outer margins or whitespace outside the visible containment of the component. Margins on any reusable component make assumptions about the host layout and can tightly couple layout and component responsibilities. The layout or white space between components should be managed separately via layout components or utilities. If working in a Design System layout, utilities should be driven by consistent design tokens that manage both size and spacing values.

Avoiding margins also can add a performance boost using the CSS Containment API. This API enables your components to provide hints to the browser about how it renders its layout. By avoiding margins, we can tell the browser that the layout will remain within our component's host element. This allows the browser to make performance optimizations when rendering.

- 🏁 Performance: [CSS Containment API](developer.chrome.com/bhlog/css-containment)

Fonts can have a significant impact on white space and the accuracy of layouts. New CSS proposals like [CSS Leading Trim](https://medium.com/microsoft-design/leading-trim-the-future-of-digital-typesetting-d082d84b202) can ensure typography is precise and adds no additional excess whitespace to layouts. Tools like [Capsize CSS](seek-oss.github.io/capsize/) allow components to use leading trim-like features today.

### Responsive

Components should be responsive by default, enabling them to be used in various contexts and devices. Leveraging APIs such as [CSS Container Queries](https://web.dev/cq-stable/) and [Resize Observers](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) can enable a component to be responsive relative to its container size and not just the viewport.

## Publishing

### API Documentation

Tools like the [Custom Elements Analyzer](https://custom-elements-manifest.open-wc.org/) and [Custom Elements Manefest](https://github.com/webcomponents/custom-elements-manifest) provides a standard way to document and generate metadata for our components.

Using JSDoc syntax, we can add additional information to document the public API of the component that tools like TypeScript cannot.

```javascript
/**
 * @element ui-alert
 *
 * a alert box component
 *
 * @slot - default content
 * @event close
 * @cssprop --background
 * @cssprop --color
 * @csspart close-button
 */
class UIAlert {

}
```

🎓 **Learn**: [Custom Elements Analyzer](https://custom-elements-manifest.open-wc.org/)

🎓 **Learn**: [Custom Elements Manefest](https://github.com/webcomponents/custom-elements-manifest)

### Dependencies

For highly reusable components, be cautious about what dependencies are added. Consider the following when deciding if a dependency should be used:

- Is this dependency tree shakable?
- Is this something that is built into the Web Platform?
- Is this something that could be polyfilled until available in most browsers?

List dependencies within the `package.json` rather than bundling them into the component. This enables the consuming application to de-dupe dependencies and properly tree shake/remove unused code.

Optional features or polyfills can be listed under the `optionalDependencies`.

```json
{
  "dependencies": {
    "lit": "..."
  },
  "optionalDependencies": {
    "element-internals-polyfill": "..."
  }
}
```

### Entrypoints

Each component should have a single entry point. This help ensures each component is isolated and can be imported independently from other components or dependencies within the same package.

```javascript
import { UIButton } from 'ui-library/button';
import { UIAlert } from 'ui-library/alert';
```

### Side Effects

Custom Elements must register via the `customElements` API. This associates the Class definition to the tag instance used in the HTML. This registration step is a global side effect. The side effect means the tag name is globally available to the HTML document once registered.

To ensure global side effects are not unexpected such as cases like unit testing or [Scoped Element Registries](https://github.com/webcomponents/polyfills/tree/master/packages/scoped-custom-element-registry), they should be isolated from the component implementation.

```javascript
// ui-library/includes/alert.js
import 'ui-library/includes/button.js';
import 'ui-library/includes/icon.js';
import { UIAlert } from 'ui-library/alert';

customElements.get('ui-alert') || customElements.define('ui-alert', UIAlert);
```

The isolated registration and imported dependencies ensure that the global registration only happens when explicitly included by the consumer.

```javascript
import 'ui-library/includes/button.js';
import 'ui-library/includes/alert.js';
```

The `package.json` needs to have a `sideEffects` property that explicitly lists all side effect files provided by the library package. This entry enables build tooling like Rollup and Webpack to optimize tree shaking while ensuring the side effects are included correctly in the final application bundle.

🚧 **Warning**: Side effect isolation is required for [Scoped Element Registries](https://github.com/webcomponents/polyfills/tree/master/packages/scoped-custom-element-registry)

🎓 **Learn**: [High-Performance Web UI with Web Components](https://www.youtube.com/watch?v=QmDToR6mLhk)
