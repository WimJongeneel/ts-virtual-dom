# A virtual DOM in typescript

This project contains a virtual DOM implementation in TypeScript.

## Virtual DOM explained

Dealing with the DOM in a SPA is a huge task. Because the DOM is reeeaaaaly slow we need to make as few updates as possible. When updating the DOM we should also be carefull with replacing elements because this will make the user lose his tabfocus etc. The idea of a virtual DOM is that the application is not going to borther at all with those challenges and outputs a completely new DOM everytime it wants to update something. The virtual DOM will then compare the new DOM with the existing one and create a diff. This diff contains all the changes that should be made to the dom. After this the virtual DOM will apply those changes while trying to update as little as possible.

## The library

This library contains tree main modules: the virtual DOM, the diffing and the render. Those are split between the files. The virtual DOM is a tree based type that describes the HTML. This module also contains factory methods for easier use. The diffing function creates a diff object from two DOMs. It will create it's own tree shaped datastructure that contains all the updates. At last, the render module will render a virtual DOM in the real DOM and apply the diffs genereted by the diffing module.

## Example

Create a DOM:

```ts
const app = createElement(
  'div',
  { 'id': 'root' },
  new Map().set(
    'h1': createElement(
      'h1',
      { 'class': 'header' },
      new Map().set('txt': createText('hello world'))
    )
  )
)

const rootElem = renderDOM('root', app)
```

Create second version of this DOM with some changes:

```ts
const app1 = createElement(
  'div',
  { 'id': 'root-updated' },
  new Map().set(
    'h1', createElement(
      'h1',
      { 'class': 'header-updated', id: 'id-new' },
      new Map().set( createText('hello world update') )
    )
  )
)
```

Create and apply the diff:

```ts
const diff = createDiff(app, app1)

applyUpdate(rootElem, diff)
```

This example will create and apply the following update. Note that it will update the `div` and the `h1` and only replace the textnode instead of replacing the entire tree.

```json
{
  "kind": "update",
  "attributes": {
    "delete": [],
    "set": {
      "id": "root-updated"
    }
  },
  "childeren": [
    {
      "kind": "update",
      "attributes": {
        "delete": [],
        "set": {
          "class": "header-updated",
          "id": "id-new"
        }
      },
      "childeren": [
        {
          "kind": "replace",
          "newNode": {
            "kind": "text",
            "text": "hello world update"
          }
        }
      ]
    }
  ]
}
```

## Updating childs

Childnodes do have keys to identify previous versions of a node between different VDOM trees. We need to set manual keys because the index of a child in the collection of childs is no garanty for being the same node, their could have been childs inserted in between or childs could have been deleted. This concept of childs being deleted or inserted in the collection of nodes while still wanting to preserve versions between virtual DOM instances is the source for a lot of the complexity of the child diffing alogratim.

The rules for child are as follows:

- If new keys occure, the will be added on their index without affecting existing nodes.
- If keys don't occur in the new node they will be removed (**WIP**).
- childs with the same key will always be updated with the smallest diff, unless the keys are in a different order then in the old tree. Adding new nodes inbetween and deleting existing ones doesn't affect this. If the keys are in a different order, all the child from the first child that is out of order will be completely rerendered.
