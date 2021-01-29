---
title: Lists & repeating templates
eleventyNavigation:
  key: Lists & repeating templates
  parent: Templates
  order: 4
---

{% todo %}

-  Check all import paths.
-  Add interactive examples.

{% endtodo %}

You can use standard JavaScript constructs to create repeating templates.

Lit also provides a `repeat` directive to build certain kinds of dynamic lists more efficiently.

## Rendering arrays

When an expression in the child position in returns an array or iterable, Lit renders all of the items in the array:

```js
render() {
  const colors = ['red', 'green', 'blue'];
  return html`<p>Colors: ${colors}</p>`;
}
```
This renders the following HTML:

```
<p>Colors: redgreenblue</p>
```

In most cases, you'll want to transform the array items into a more useful form.

##  Repeating templates with map

To render lists, you can use `map` to transform a list of data into a list of templates:

```js
render() {
  return html`
    <ul>
      ${this.items.map((item) =>
        html`<li>${item}</li>`
      )}
    </ul>
  `;
}
```

Note that this expression returns an array of `TemplateResult` objects. Lit will render an array or iterable of subtemplates and other values.

## Repeating templates with looping statements

You can also build an array of templates and pass it into a template expression.

```js
const itemTemplates = [];
for (const i of items) {
  itemTemplates.push(html`<li>${i}</li>`);
}

html`
  <ul>
    ${itemTemplates}
  </ul>
`;
```

## The repeat directive

In most cases, using loops or `map` is an efficient way to build repeating templates. However, if you want to reorder a large list, or mutate it by adding and removing individual entries, this approach can involve updating a large number of DOM nodes.

The `repeat` directive can help here.

The repeat directive performs efficient updates of lists based on user-supplied keys:

`repeat(items, keyFunction, itemTemplate)`

Where:

*   `items` is an array or iterable.
*   `keyFunction` is a function that takes a single item as an argument and returns a guaranteed unique key for that item.
*   `itemTemplate` is a template function that takes the item and its current index as arguments, and returns a `TemplateResult`.

For example:

```js
import {html} from 'lit-html';
import {repeat} from 'lit-html/directives/repeat.js';

const employeeList = (employees) => html`
  <ul>
    ${repeat(employees, (employee) => employee.id, (employee, index) => html`
      <li>${index}: ${employee.familyName}, ${employee.givenName}</li>
    `)}
  </ul>
`;
```

If you re-sort the `employees` array, the `repeat` directive reorders the existing DOM nodes.

To compare this to Lit's default handling for lists, consider reversing a large list of names:

*   For a list created using `map`, Lit maintains the DOM nodes for the list items, but reassigns the values.
*   For a list created using `repeat`, the `repeat` directive reorders the _existing_ DOM nodes, so the nodes representing the first list item move to the last position.


### When to use map or repeat

Which repeat is more efficient depends on your use case:

*   If updating the DOM nodes is more expensive than moving them, use the `repeat` directive.

*   If the DOM nodes have state that _isn't_ controlled by a template expression, use the `repeat` directive.

    For example, consider this list:

    ```js
    html`${this.users.map((user) =>
      html`
      <div><input type="checkbox"> ${user.name}</div>
    }`
    ```

    The checkbox has a checked or unchecked state, but it isn't controlled by a template expression.

    If  you reorder the list after the user has checked one or more checkboxes, Lit would update the names associated with the checkboxes, but not the state of the checkboxes.

 If neither of these situations apply, use `map` or looping statements.
