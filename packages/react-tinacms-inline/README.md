# react-tinacms-inline

This package provides the components and helpers for [Inline Editing](https://tinacms.org/docs/ui/inline-editing).

## Install

```bash
yarn add react-tinacms-inline
```

> For specific steps on setting up Inline Editing on a page, refer to the [Inline Editing](https://tinacms.org/docs/ui/inline-editing) documentation.

## Inline Form

The `InlineForm` component provides an inline editing [context](https://reactjs.org/docs/context.html). To use, wrap the component where you want to render inline fields and pass the form.

### Usage

```jsx
export function Page(props) {
  const [, form] = useForm(props.data)
  usePlugin(form)

  return (
    <InlineForm form={form}>
      <main>
        {
            //...
        }
      </main>
    </InlineForm>
  )
}
```

### Interface

```ts
interface InlineFormProps {
  form: Form
  children: React.ReactElement | React.ReactElement[] | InlineFormRenderChild
}

interface InlineFormRenderChild {
  (props: InlineFormRenderChildOptions):
    | React.ReactElement
    | React.ReactElement[]
}

type InlineFormRenderChildOptions = InlineFormState &
  Omit<FormRenderProps<any>, 'form'>

/**
 * This state is available via context
*/
interface InlineFormState {
  form: Form
  focussedField: string
  setFocussedField(field: string): void
}
```

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `form`        | A [Tina Form](/docs/plugins/forms).                                                          |
| `children`    | Child components to render — Either [React Elements](https://reactjs.org/docs/rendering-elements.html) or [Render Props](https://reactjs.org/docs/render-props.html) Children that receive the form state and other [Form Render Props](https://final-form.org/docs/react-final-form/types/FormRenderProps)                                                                                   |


### _useInlineForm_

```ts
useInlineForm(): InlineFormState
```

The `useInlineForm` [hook](https://reactjs.org/docs/hooks-intro.html) can be used to access the inline editing context. It must be used within a child component of `InlineForm`.

## Inline Field

Inline Fields should provide basic inputs for editing data on the page and account for both [enabled / disabled CMS states](https://tinacms.org/docs/cms#disabling--enabling-the-cms). All Inline Fields must be children of an `InlineForm`.

### Interface

```ts
export interface InlineFieldProps {
  name: string
  children(fieldProps: InlineFieldRenderProps): React.ReactElement
}

export interface InlineFieldRenderProps<V = any>
  extends FieldRenderProps<V>,
    InlineFormState {}
```

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `name`        | The path to the editable data.                                                          |
| `children`    | Child components to render — [React Elements](https://reactjs.org/docs/rendering-elements.html) that receive the form state and [Field Render Props](https://final-form.org/docs/react-final-form/types/FieldRenderProps).                                                                                  |

### Available Inline Fields

See the full list of [inline fields](https://tinacms.org/docs/ui/inline-editing#all-inline-fields) or learn how to make [custom inline fields](https://tinacms.org/docs/ui/inline-editing#creating-custom-inline-fields).

Below is a list of fields provided by the `react-tinacms-inline` package:
- [Inline Text](https://tinacms.org/docs/ui/inline-editing/inline-text)
- [InlineTextarea](https://tinacms.org/docs/ui/inline-editing/inline-textarea)
- [Inline Group](https://tinacms.org/docs/ui/inline-editing/inline-group)
- [Inline Image](https://tinacms.org/docs/ui/inline-editing/inline-image)
- [Inline Blocks](https://tinacms.org/docs/ui/inline-editing/inline-blocks)

## Inline Field Styles

Styles are stripped as much as possible to prevent interference with base site styling. When toggling between [enabled / disabled states](https://tinacms.org/docs/cms#disabling--enabling-the-cms), the default inline fields will switch between rendering child elements with any additional editing UI and just passing the child elements alone.

For example with `InlineText`:

```tsx
export function InlineText({
  name,
  className,
  focusRing = true,
}: InlineTextProps) {
  const cms: CMS = useCMS()

  return (
    <InlineField name={name}>
      {({ input }) => {
        /**
        * If the cms is enabled, render the input
        * with the focus ring
        */
        if (cms.enabled) {
          if (!focusRing) {
            return <Input type="text" {...input} className={className} />
          }

          return (
            <FocusRing name={name} options={focusRing}>
              <Input type="text" {...input} className={className} />
            </FocusRing>
          )
        }
        /**
        * Otherwise, pass the input value
        */
        return <>{input.value}</>
      }}
    </InlineField>
  )
}
```

`Input` is a styled-component with some [base styling](https://github.com/tinacms/tinacms/blob/master/packages/react-tinacms-inline/src/fields/inline-text-field.tsx#L64) aimed at making this component mesh well with the surrounding site. If you ever need to override default Inline Field styles, read about this workaround to [extend styles](https://tinacms.org/docs/ui/inline-editing#extending-inline-field-styles).

### Focus Ring

The common UI element on all Inline Fields is the `FocusRing`. The focus ring provides context to which field is active / available to edit.

**Interface**

```ts
interface FocusRingProps {
  name?: string
  children: React.ReactNode | ((active: boolean) => React.ReactNode)
  options?: boolean | FocusRingOptions
}

interface FocusRingOptions {
  offset?: number | { x: number; y: number }
  borderRadius?: number
  nestedFocus?: boolean
}
```

**Focus Ring Props**

You would only use these options if you were creating custom inline fields and working with the `FocusRing` directly.

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `children`    | _Optional_: Child elements to render.                                                                    |
| `name`        | _Optional_: This value is used to set the focused / active field.                            |
| `options`     | _Optional_: The `FocusRingOptions` outlined below.                                           |

<br />

> **Focus Ring Children**
>
> `FocusRing` optionally accepts [render prop](https://reactjs.org/docs/render-props.html#using-props-other-than-render) patterned children, which receive the `active` state and can be used to conditionally render elements based on whether the `FocusRing` currently has focus.
>
> ```jsx
><FocusRing>
>  {active => {
>    if (active) {
>      return <ComplicatedEditableComponent />
>    }
>    return <SimpleDisplayComponent />
>  }}
></FocusRing>
> ```

**Focus Ring Options**

These options are passed to default [inline fields](https://tinacms.org/docs/ui/inline-editing#all-inline-fields) or inline block fields via the `focusRing` prop on most default inline fields. The options are configurable by the developer setting up the inline form & fields. Refer to individual [inline field documentation](https://tinacms.org/docs/ui/inline-editing#all-inline-fields) for additional examples.

| Key           | Description                                                                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `offset`      | _Optional_: Sets the distance from the focus ring to the edge of the child element(s). It can either be a number (in pixels) for both x & y offsets, or individual x & y offset values passed in an object.|
| `borderRadius`| _Optional_: Determines (in pixels) the [rounded corners](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) of the focus ring.                                                                |
| `nestedFocus` | _Optional_:  Disables [pointer-events](https://developer.mozilla.org/en-US/docs/Web/CSS/pointer-events) (clicking) and hides blocks empty state.|

## Inline Blocks

[Inline Blocks](https://tinacms.org/docs/ui/inline-editing/inline-blocks#inlineblocks-interface) consist of an array of [Blocks](https://tinacms.org/blog/what-are-blocks) to render in an Inline Form. Refer to the Inline Blocks [Documentation](https://tinacms.org/docs/ui/inline-editing/inline-blocks) or [Guide](https://tinacms.org/guides/general/inline-blocks/overview) for code examples on creating Inline Blocks.

**Interface**

```ts
export interface InlineBlocksProps {
  name: string
  blocks: {
    [key: string]: Block
  }
  className?: string
  direction?: 'vertical' | 'horizontal'
  itemProps?: {
    [key: string]: any
  }
  min?: number
  max?: number
  components?: {
    Container?: React.FunctionComponent<BlocksContainerProps>
  }
  children?: React.ReactNode | null
}
```

| Key         |                                                                                                                                    Purpose |
| ----------- | -----------------------------------------------------------------------------------------------------------------------------------------: |
| `name`      |                                                                                            The path to the **source data** for the blocks. |
| `blocks`    |                                         An object composed of individual [Blocks](/docs/ui/inline-editing/inline-blocks#creating-a-block). |
| `className` | _Optional_ — To set styles directly on the input or extend via [styled components](/docs/ui/inline-editing#extending-inline-field-styles). |
| `direction` |                                                    _Optional_ — Sets the orientation of the `AddBlock` button position. |
| `itemProps` |                                                          _Optional_ — An object that passes additional props to every block child element. |
| `min`       |                         _Optional_ — Controls the minimum number of blocks. Once reached, blocks won't be able to be removed. _(Optional)_ |
| `max`       |                   _Optional_ — Controls the maximum number of blocks allowed. Once reached, blocks won't be able to be added. _(Optional)_ |

### Actions

The Inline Blocks Actions are used by the Inline Blocks Controls. Use these if you are building your own custom Inline Block Field Controls. These actions are avaiable in the _Inline Blocks Context_.

```ts
interface InlineBlocksActions {
  count: number
  insert(index: number, data: any): void
  move(from: number, to: number): void
  remove(index: number): void
  blocks: {
    [key: string]: Block
  }
  activeBlock: number | null
  setActiveBlock: any
  direction: 'vertical' | 'horizontal'
  min?: number
  max?: number
}
```

### _useInlineBlocks_

```ts
useInlineBlocks(): InlineBlocksActions
```

`useInlineBlocks` is a hook that can be used to access the _Inline Blocks Context_ when creating custom controls.

**Context and inline blocks children**

To conditionally render markup adjacent to your inline blocks, depending on context values, pass children to the `InlineBlocks` and call `useInlineBlocks` in your child component(s). For example, this will render some text below your inline blocks, but only if there are more than three inline blocks currently on the page:

```ts
import { InlineBlocks, useInlineBlocks } from 'react-tinacms-inline'

export function LotsOfBlocksMessage() {
  const blocks = useInlineBlocks()
  return <>
    {blocks.count > 3 &&
      <p>
        Wow, there are {blocks.count} blocks, that's rather a lot!
      </p>
    }
  </>
}

export function MyBlocksContainer() {
  return (
    <InlineBlocks name="myBlocks" blocks={MY_BLOCKS}>
      <LotsOfBlocksMessage />
    </InlineBlocks>
  )
}
```

### Inline Block Field Controls

Editors can add / delete / rearrange blocks with the [blocks controls](https://tinacms.org/docs/ui/inline-editing/inline-blocks#blocks-controls-options). They can also access additional fields in a [Settings Modal](https://tinacms.org/docs/ui/inline-editing/inline-blocks#using-the-settings-modal).

**Interface**

```ts
interface BlocksControlsProps {
  index: number
  insetControls?: boolean
  focusRing?: false | FocusRingProps
  customActions?: BlocksControlActionItem[]
  label?: boolean
  children: React.ReactChild
}

interface FocusRingProps {
  offset?: number | { x: number; y: number }
  borderRadius?: number
}

export interface BlocksControlActionItem {
  icon: React.ReactNode
  onClick: () => void
}
```

| Key             | Description                                                                                                                                                                                                                                                                                                                        |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index`         | The index of the block associated with these controls.                                                                                                                                                                                                                                                                             |
| `insetControls` | A boolean to denote whether the group controls display within or outside the group.                                                                                                                                                                                                                                                |
| `focusRing`     | Either an object to style the focus ring or `false`, which hides the focus ring entirely. For styles, `offset` (in pixels) controls the distance from the ring to the edge of the group; `borderRadius`(in pixels) controls the [rounding](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) edge of the focus ring. |
| `label`     | A boolean to control whether or not a block label is rendered.              |
| `children`      | Any child components, typically inline field(s).                                                                                                                                                                                                                                                                                   |
| `customActions` | An array of objects containing custom block action configuration. `icon` is the component to render in the toolbar. `onClick` handles the action behavior.                                                                                                                                                                                                                                                                                    |


### Block Definition

A block is made of two parts: a [component](https://tinacms.org/docs/ui/inline-editing/inline-blocks#part-1-block-component) that renders in edit mode, and a [template](https://tinacms.org/docs/ui/inline-editing/inline-blocks#part-2-block-template) to configure fields, defaults and other required data.

**Interface**

```ts
interface Block {
  Component: React.FC<BlockComponentProps>
  template: BlockTemplate
}

interface BlockComponentProps {
  name: string
  index: number
  data: any
}

interface BlockTemplate {
  label: string
  defaultItem?: object | (() => object)
  fields?: Field[]
}
```

**Block Component**

| Key           |                                                                                                                Purpose |
| ------------- | ---------------------------------------------------------------------------------------------------------------------: |
| `name`       |                                                                                                A unique identifier and pseudo-path to the block from the parent blocks array. e.g. the first child would be 'blocks.0' |
| `index`       |                                                                                                Position in the block array. |
| `data` |                                                                   The source data. |

**Block Template**

| Key           |                                                                                                                Purpose |
| ------------- | ---------------------------------------------------------------------------------------------------------------------: |
| `label`       |                                                                                                A human readable label. |
| `defaultItem` |                                                                   _Optional_ — Populates new blocks with default data. |
| `fields`      | _Optional_ — Populates fields in the [Settings Modal](/docs/ui/inline-editing/inline-blocks#using-the-settings-modal). |

### Examples

Block Components can render _Inline Fields_ to expose the data for editing. When using Inline Fields within a block, the `name` property should be _relative to the block object_ in the source data.

For example:

```js
import { useForm } from 'tinacms'
import { InlineForm, InlineBlocks, BlocksControls, InlineTextarea } from 'react-tinacms-inline'

function FeaturePage({ data }) {
  const [ , form ] = useForm({
    id: 'my-features-id',
    label: 'Edit Features',
    fields: [],
    initialValues: data,
  })

  return (
    <InlineForm form={form}>
      <div className="wrapper">
        <InlineBlocks name="features_blocks" blocks={FEATURE_BLOCKS} />
      </div>
    </InlineForm>
  )
}

function Feature({ index }) {
  return (
    <BlocksControls index={index}>
      <div className="feature">
        <h3>
        {/**
        * The `name` property is relative to individual
        * `features_blocks` array items (blocks). The full path
        * in the source file (example below) would be
        *  `features_blocks[index].heading`
        */}
          <InlineTextarea name="heading" focusRing={false} />
        </h3>
        <p>
          <InlineTextarea name="supporting_copy" focusRing={false} />
        </p>
      </div>
    </BlocksControls>
  )
}

const featureBlock = {
  Component: Feature,
  template: {
    label: 'Feature',
    defaultItem: {
      _template: 'feature',
      heading: 'Marie Skłodowska Curie',
      supporting_copy:
        'Muse about vastness.',
    },
    fields: [],
  },
}

const FEATURE_BLOCKS = {
    featureBlock
}
```

**Example JSON data**

```json
{
    "features_blocks": [
        {
            "_template": "feature",
            "heading": "Drake Equation",
            "supporting_copy": "Light years gathered by gravity Rig Veda.."
        },
        {
            "_template": "feature",
            "heading": "Jean-François Champollion",
            "supporting_copy": "Not a sunrise but a galaxyrise."
        },
        {
            "_template": "feature",
            "heading": "Sea of Tranquility",
            "supporting_copy": "Bits of moving fluff take root and flourish."
        }
    ]
}
```

Below is another example using `InlineBlocks` with multiple block definitions:

```js
import { useJsonForm } from 'next-tinacms-json'
import { InlineForm, InlineBlocks, BlocksControls, InlineTextarea } from 'react-tinacms-inline'

export default function PageBlocks({ jsonFile }) {
  const [, form] = useJsonForm(jsonFile)

  return (
    <InlineForm form={form}>
      <InlineBlocks name="my_blocks" blocks={PAGE_BLOCKS} />
    </InlineForm>
  )
}

/** Example Heading Block Definition
 * Component + template
*/
function Heading({ index }) {
  return (
    <BlocksControls index={index}>
      <InlineTextarea name="text" />
    </BlocksControls>
  )
}

const heading_template = {
  label: 'Heading',
  defaultItem: {
    text: 'At vero eos et accusamus',
  },
  fields: [],
}

const headingBlock = {
    Component: Heading,
    template: heading_template
}

/**
 * Example Paragraph Block
 * Component + template
*/

function Paragraph({ index }) {
  return (
    <BlocksControls index={index} focusRing={{ offset: 0 }} insetControls>
      <div className="paragraph__background">
        <div className="wrapper wrapper--narrow">
          <p className="paragraph__text">
            <InlineTextarea name="text" focusRing={false} />
          </p>
        </div>
      </div>
    </BlocksControls>
  );
}

const paragraphBlock = {
  Component: Paragraph,
  // template defined inline
  template: {
    label: 'Paragraph',
    defaultItem: {
      text:
        'Take root and flourish quis nostrum exercitationem ullam',
    },
    fields: [],
  },
};

/**
 * Available blocks passed to InlineBlocks to render
*/

const PAGE_BLOCKS = {
    headingBlock,
    paragraphBlock
}
```

**Configuring the Container**

`InlineBlocks` wraps your blocks with a `<div>` element by default. This can be an issue if your styles require direct inheritance, such as a flexbox grid:

```html
<div class="row">
  <div class="column">
  </div>
</div>
```

To handle this, you can pass a custom container component to `InlineBlocks`.

**Interface**

```ts
interface BlocksContainerProps {s
  className?: string
  children?: React.ReactNode
}
```

**Example**

```js
import { useJsonForm } from 'next-tinacms-json'
import { InlineForm, InlineBlocks, BlocksControls, InlineTextarea } from 'react-tinacms-inline'

const MyBlocksContainer = (children}) => (
  <div>
    {children}
  </div>
)

export default function PageBlocks({ jsonFile }) {
  const [, form] = useJsonForm(jsonFile)

  return (
    <InlineForm form={form}>
      <InlineBlocks
        name="my_blocks"
        blocks={PAGE_BLOCKS}
        components={{
          Container: MyBlocksContainer
        }}
      />
    </InlineForm>
  )
}
```

> Checkout this guide to learn more on using [Inline Blocks](https://tinacms.org/guides/general/inline-blocks/overview).

---

## _useFieldRef_: Ref-Based Inline Editing

`useFieldRef` is the first part of an experimental new API for creating an inline editing experience with Tina. With `useFieldRef`, inline editing components are defined in the form configuration, and you assign a [ref](https://reactjs.org/docs/refs-and-the-dom.html) to the component in your layout that the field should attach to.

Inline fields created in this fashion will be absolutely positioned on top of the referenced component and conform to its dimensions. This makes it possible for the field to appear as if it were replacing the layout component in the DOM without altering the markup.

### Setting up ref-based inline editing

Adding inline editing with this API is done in three steps:

#### 1. Add the `RBIEPlugin`

This feature is experimental for now, so in order to enable it, you need to add it as a plugin to your CMS config.

```js
import { TinaCMS } from 'tinacms'
import { RBIEPlugin } from 'react-tinacms-inline'

const cms = new TinaCMS({
  //...
  plugins: [
    new RBIEPlugin()
  ]
})
```

#### 2. Add an `inlineComponent` to your form field definition

Similar to how you would configure a sidebar field via that field's `component` key, you can configure an inline field via the `inlineComponent` key.

> Unlike the sidebar `component`s, `inlineComponent`s must be defined as a React component and not a string. In the future, `inlineComponent` will accept a string as well and components can be registered via the plugin system.

```jsx
import { useForm } from 'tinacms'

function MyPageComponent() {
  const [data, form] = useForm({
    //...
    fields: [
      //...
      {
        name: 'title',
        component: 'text',
        inlineComponent: ({ name }) => (
          <h1><InlineText name={name} /></h1>
        )
      }
    ]
  })
  //...
}
```

#### 3. Call `useFieldRef` and attach the ref to the component that you want to be editable

`useFieldRef` requires the ID of the form that it corresponds to, as well as the name of the field that it should be editing.

```jsx
import { useForm } from 'tinacms'
import { useFieldRef } from 'react-tinacms-inline'

function MyPageComponent() {
  const [data, form] = useForm({
    //...
  })
  const titleRef = useFieldRef(form.id, 'title')
  return (
    <InlineForm form={form}>
      <main>
        <h1 ref={titleRef}>{data.title}</h1>
      </main>
    </InlineForm>
  )
}
```

## InlineWysiwyg

The `InlineWysiwyg` is a React [inline editing component](https://tinacms.org/docs/ui/inline-editing) for Markdown and HTML.


### InlineWysiwyg Interface

```typescript
interface InlineWysiwygProps {
  name: string
  children: any
  sticky?: boolean | string
  format?: 'markdown' | 'html'
  imageProps?: ImageProps
  focusRing?: boolean | FocusRingProps
}

interface ImageProps {
  parse(media: Media): string
  uploadDir?(form: Form): string
  upload?: (files: File[]) => Promise<string[]>
  previewSrc?: (url: string) => string | Promise<string>
}

interface FocusRingProps {
  offset?: number | { x: number; y: number }
  borderRadius?: number
}
```

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `name`        | The path to some value in the data being edited.                                             |
| `children`    | Child components to render.                                                                  |
| `sticky?`     | A boolean determining whether the Wysiwyg Toolbar 'sticks' to the top of the page on scroll. |
| `format?`     | This value denotes whether Markdown or HTML will be rendered.                                |
| `imageProps?` | An object that configures how images in the Wysiwyg are uploaded and rendered. Images are disabled when undefined.|
| `focusRing?`   | Either an object to style the focus ring or a boolean to show/hide the focus ring. Defaults to `true` which displays the focus ring with default styles. For style options, `offset` (in pixels) sets the distance from the ring to the edge of the component, and `borderRadius` (in pixels) controls the [rounded corners](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) of the focus ring. |

### Basic Usage

Below is an example of how an `InlineWysiwyg` field could be defined in an [Inline Form](/docs/ui/inline-editing).

```jsx
import ReactMarkdown from 'react-markdown'
import { useForm, usePlugin } from 'tinacms'
import { InlineForm, InlineWysiwyg } from 'react-tinacms-inline'

// Example 'Page' Component
export function Page(props) {
  const [data, form] = useForm(props.data)
  usePlugin(form)
  return (
    <InlineForm form={form}>
      <InlineWysiwyg name="markdownBody" format="markdown">
        <ReactMarkdown source={data.markdownBody} />
      </InlineWysiwyg>
    </InlineForm>
  )
}
```

### With _imageProps_

To upload and manage images in the Wysiwyg, you'll need to configure `imageProps`. 

| Key           | Description                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------- |
| `parse`        | Defines how the actual front matter or data value gets populated on upload. The media object gets passed as an argument. _Defaults to `filename`_.                                           |
| `uploadDir?`    | Defines the upload directory. This function is passed the current form values.                                                                  |
| `upload?`     | An asynchronous function that handles image upload. By default, this calls the `persist` function on the [media store](https://tinacms.org/docs/media). |
| `previewSrc?`     | 	An asynchronous function that returns the path or url for the image `src` in preview or edit mode. By default, this calls the `previewSrc` function on the [media store](https://tinacms.org/docs/media)_.                             |

```jsx
<InlineWysiwyg
  name="rawMarkdownBody"
  imageProps={{
    parse: (media) => `images/about/${media.filename}`
    uploadDir: () => 'public/images/about/',
  }}
>
  <div
    dangerouslySetInnerHTML={{
      __html: props.data.markdownRemark.html,
    }}
  />
</InlineWysiwyg>
```

### Sticky Menu

When editing long content it is likely that the editor will scroll past the wysiwyg menu.

By adding the `sticky` property the menu will follow the user as they scroll through the page.

```jsx
<InlineWysiwyg name="markdownBody" format="markdown" sticky>
  <ReactMarkdown source={data.markdownBody} />
</InlineWysiwyg>
```

Alternatively you can pass a string to set the exact offset of the menu.

```jsx
<InlineWysiwyg name="markdownBody" format="markdown" sticky="2rem">
  <ReactMarkdown source={data.markdownBody} />
</InlineWysiwyg>
```

If you're using Tina's toolbar, you can pass 'var(--tina-toolbar-height)' to ensure the toolbar does not cover the WYSIWYG menu.

```jsx
<InlineWysiwyg name="markdownBody" format="markdown" sticky="var(--tina-toolbar-height)">
  <ReactMarkdown source={data.markdownBody} />
</InlineWysiwyg>
```