# Module: Volto (TypeScript & React)

This module covers Volto development using TypeScript, React, and the Blocks engine.

## Core Principles

- **TypeScript strictly** — Use `.ts` and `.tsx` for all new components.
- **Functional Components** — Use Hooks (`useState`, `useEffect`, `useSelector`).
- **Component Extensibility** — Prefer `config.registerComponent` or `blocksConfig` overrides over shadowing to reduce maintenance debt.
- **Shadowing (Caution)** — Use `src/customizations/` only when a component cannot be extended via the registry. Mirror paths exactly.

## Volto Component Patterns (TypeScript)

### 1. Custom Block Schema
```typescript
import { IntlShape } from 'react-intl';

export interface MyBlockData {
  title: string;
  description?: string;
}

export const myBlockSchema = (intl: IntlShape) => ({
  title: 'My Block',
  fieldsets: [
    {
      id: 'default',
      title: 'Default',
      fields: ['title', 'description'],
    },
  ],
  properties: {
    title: { title: 'Title', widget: 'text' },
    description: { title: 'Description', widget: 'textarea' },
  },
  required: [],
});
```

### 2. Block View Component
```tsx
import React from 'react';
import { MyBlockData } from './schema';

interface Props {
  data: MyBlockData;
}

const MyBlockView: React.FC<Props> = ({ data }) => (
  <div className="my-block">
    <h2>{data.title}</h2>
    {data.description && <p>{data.description}</p>}
  </div>
);

export default MyBlockView;
```

### 3. Block Edit Component
```tsx
import React from 'react';
import { SidebarPortal, BlockDataForm } from '@plone/volto/components';
import { useIntl } from 'react-intl';
import MyBlockView from './View';
import { myBlockSchema, MyBlockData } from './schema';

interface Props {
  data: MyBlockData;
  block: string;
  selected: boolean;
  onChangeBlock: (block: string, data: MyBlockData) => void;
}

const MyBlockEdit: React.FC<Props> = (props) => {
  const { data, block, onChangeBlock, selected } = props;
  const intl = useIntl();
  const schema = myBlockSchema(intl);

  return (
    <>
      <MyBlockView data={data} />
      <SidebarPortal selected={selected}>
        <BlockDataForm
          schema={schema}
          title={schema.title}
          onChangeField={(id, value) =>
            onChangeBlock(block, { ...data, [id]: value })
          }
          formData={data}
        />
      </SidebarPortal>
    </>
  );
};

export default MyBlockEdit;
```

## Testing (Cypress & TypeScript)

Use Cypress for End-to-End tests of Volto blocks.

Example `block.spec.ts`:
```typescript
describe('MyBlock', () => {
  beforeEach(() => {
    cy.autologin();
    cy.createContent({
      contentType: 'Document',
      contentId: 'my-page',
      title: 'My Page',
    });
    cy.visit('/my-page/edit');
  });

  it('should add the custom block', () => {
    cy.get('.block-add-button').click();
    cy.get('.blocks-chooser .common .button.myBlock').click();
    cy.get('.block.myBlock .view h2').should('exist');
  });
});
```

## Configuration

Register components in your add-on's `index.ts`:
```typescript
import MyBlockView from './components/Blocks/MyBlock/View';
import MyBlockEdit from './components/Blocks/MyBlock/Edit';
import icon from '@plone/volto/icons/block.svg';
import { ConfigRegistry } from '@plone/volto/types';

const applyConfig = (config: ConfigRegistry) => {
  config.blocks.blocksConfig.myBlock = {
    id: 'myBlock',
    title: 'My Block',
    icon: icon,
    group: 'common',
    view: MyBlockView,
    edit: MyBlockEdit,
  };
  return config;
};

export default applyConfig;

## Documentation Retrieval Protocol (Mandatory)

Do not rely on static lists of components or widgets. Always fetch the latest documentation:

- **Volto Widgets**: `https://6.docs.plone.org/_sources/volto/reference/widgets.md`
- **Blocks Anatomy**: `https://6.docs.plone.org/_sources/volto/blocks/anatomy.md`

Use the `webfetch` tool on `https://6.docs.plone.org/llms.txt` to find the exact URLs.
```
