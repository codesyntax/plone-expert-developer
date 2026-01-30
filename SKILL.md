---
name: plone-expert-developer
description: Expert Plone 6 and Volto development guidance. Covers backend (Python, Dexterity, plone.restapi) and frontend (React, Volto, Blocks).
---

# Plone Developer Expert

You are an expert Plone 6 and Volto developer. You assist with full-stack development involving the Plone CMS backend and the Volto React frontend.

## When to use

Use this skill when the user asks about:

- Plone 6, Zope, or Python backend development for Plone.
- Volto, React, or frontend development for Plone.
- Creating content types, behaviors, or ZCA adapters.
- Configuring `plone.restapi`.
- Developing Volto blocks, widgets, or themes.
- Deployment of Plone/Volto stacks.

## Core Principles

- **Plone 6 is the standard**: Assume Plone 6+ with Python 3.x.
- **Volto First**: Default to Volto (React) for the frontend unless specified otherwise (Classic UI).
- **Best Practices**: Follow community standards (plone.api, black, flake8, prettier, eslint).

## Backend Guidelines (Python/Plone)

### 1. Content Types (Dexterity)

- Prefer **Python schemas** (plone.supermodel) over XML models.
- Use `plone.api` for all high-level interactions (create, get, search, etc.).
- Isolate business logic in **Behaviors** or **Adapters**.

### 2. using plone.api

Use `plone.api` for all standard operations. It is the canonical API for Plone.

- **Content**:
  - Create: `api.content.create(container=portal, type='Folder', title='My Folder')`
  - Get: `api.content.get(path='/my-folder')` or `api.content.get(UID='...')`
  - Search: `api.content.find(context=context, portal_type='Document')` (returns Catalog Brains)
  - Manipulation: `api.content.move`, `api.content.rename`, `api.content.delete`, `api.content.transition`.
- **Portal**:
  - Get Portal: `api.portal.get()`
  - Tools: `api.portal.get_tool('portal_catalog')`
  - Messaging: `api.portal.show_message(message='Done', request=request)`
  - Email: `api.portal.send_email(recipient='user@example.com', subject='Hello', body='Msg')`
- **Users & Groups**:
  - Users: `api.user.create`, `api.user.get_current`, `api.user.grant_roles`.
  - Groups: `api.group.create`, `api.group.add_user`.
- **Env**:
  - `api.env.plone_version()`, `api.env.debug_mode()`.

### 3. REST API

- Extend functionalities via `plone.restapi` services.
- Never write browser views for Volto; write API endpoints/services.
- Pattern: Service Class + `configure.zcml` registration + `services` directory.

### 4. Database & ZCA

- Interact with ZODB via `plone.api` or standard accessors; avoid raw ZODB manipulation unless optimizing deep internals.
- Keep ZCML clean. Use `include package=".subpackage"` to organize large projects.

## Frontend Guidelines (Volto/React)

### 1. Architecture

- **Shadowing**: Customize core components by mirroring their path in `src/customizations`.
- **Add-ons**: Encapsulate reusable logic in Volto add-ons.

### 2. Components

- Use **Functional Components** and **Hooks**.
- Use `semantic-ui-react` for UI elements (unless using a custom design system).
- Styling: Use CSS Modules or LESS/SCSS as per project setup.

### 3. Blocks

- Creating custom blocks is the primary way to extend page layout capabilities.
- A block consists of: `View` component, `Edit` component, `schema.js`, and `icon`.

## Classic UI Guidelines (Diazo)

*Use this when developing themes for Plone Classic UI (non-Volto).*

Diazo maps a static HTML theme to dynamic Plone content using `rules.xml`.

### 1. Structure (rules.xml)

```xml
<rules
    xmlns="http://namespaces.plone.org/diazo"
    xmlns:css="http://namespaces.plone.org/diazo/css"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <theme href="index.html" />

    <!-- Replace theme content with Plone content -->
    <replace css:theme="#content" css:content="#content" />

    <!-- Drop unwanted elements -->
    <drop css:theme=".promo-banner" css:if-not-content=".section-front-page" />

    <!-- Insert content -->
    <after css:theme="#logo" css:content="#portal-searchbox" />

</rules>
```

### 2. Common Directives

- `<theme>`: Specifies the static HTML file.
- `<replace>`: Replaces the target node in the theme with the source node from content.
- `<drop>`: Removes the target node from the output.
- `<before>` / `<after>`: Inserts content before or after the target theme node.
- `<merge>`: Merges attributes (e.g., class names) from content to theme.

### 3. Conditions

Use conditions to apply rules only on specific pages.

- `css:if-content="body.section-front-page"`: Only on front page.
- `css:if-path="/news"`: Only on paths starting with /news.

## Reference: Common Field Types & Widgets

### Backend Fields (zope.schema)

Common fields used in Dexterity content types and behaviors.

```python
from zope import schema
from plone.app.textfield import RichText
from plone.namedfile.field import NamedBlobImage, NamedBlobFile
from z3c.relationfield.schema import RelationChoice, RelationList
from plone.app.vocabularies.catalog import CatalogSource

# Text
title = schema.TextLine(title=u"Title", required=True)
description = schema.Text(title=u"Description", required=False)
details = RichText(title=u"Details", required=False)

# Numbers & Logic
count = schema.Int(title=u"Count", default=0)
enabled = schema.Bool(title=u"Enabled", default=True)

# Dates
start = schema.Datetime(title=u"Start Date")

# Files
image = NamedBlobImage(title=u"Image", required=False)
file = NamedBlobFile(title=u"File", required=False)

# Relations
related_items = RelationList(
    title=u"Related Items",
    default=[],
    value_type=RelationChoice(title=u"Target", source=CatalogSource())
)
```

### Volto Schema Widgets

Common widgets used in Block schemas (`schema.js`).

```javascript
properties: {
  // Text
  title: { title: 'Title', widget: 'text' },
  description: { title: 'Description', widget: 'textarea' },
  text: { title: 'Body Text', widget: 'richtext' },

  // Numbers & Logic
  count: { title: 'Count', type: 'number' },
  visible: { title: 'Visible', type: 'boolean' }, // Renders as checkbox

  // Choices
  color: {
    title: 'Color',
    widget: 'select', // Also: 'radio', 'simple_color_picker'
    choices: [['red', 'Red'], ['blue', 'Blue']],
  },

  // Relations & Links
  internal_link: {
    title: 'Internal Link',
    widget: 'object_browser',
    mode: 'link', // 'image', 'multiple'
  },
  external_url: { title: 'External URL', widget: 'url' },

  // Special
  align: { title: 'Alignment', widget: 'align' },
  date: { title: 'Date', widget: 'datetime' },
}
```

## Project Structure

- **Backend**: Standard python package structure (`src/my.package`).
- **Frontend**: Standard Volto project (`/frontend` or separate repo).

## Common Scenarios (How-To)

### Creating a New Project

To start a new Plone 6 project with Volto, use **Cookieplone**.

- Use `uvx cookieplone` to generate the project structure.
- Select the "Plone 6 (Volto)" template.
- This scaffolds both backend (buildout/pip) and frontend (Volto) in a monorepo-like structure or separate folders depending on options.

### Creating a new Content Type

Use **plonecli** to generate the boilerplate code.

- **Inside a project** (created with cookieplone): Run `make add content_type`.
- **Standalone**: Run `uvx plonecli add content_type`.
- Follow the prompts to define the class name and fields.
- Review the generated schema in `content/yourtype.py` and adjust fields using `zope.schema`.

### Creating a Custom API Endpoint

Use **plonecli** to generate the service boilerplate.

- **Inside a project**: Run `make add restapi_service`.
- **Standalone**: Run `uvx plonecli add restapi_service`.
- This creates the service class, registers it in `configure.zcml`, and sets up the interface.

### Creating a Behavior

Use behaviors to add reusable functionality or fields to content types.

- **Command**: `uvx plonecli add behavior` (or `make add behavior`).
- This creates an interface, a factory (optional), and registers it in `configure.zcml`.
- You can then enable this behavior on content types via the Control Panel or FTI XML.

### Creating a Control Panel

Use this to create custom configuration forms in the backend site setup.

- **Command**: `uvx plonecli add controlpanel` (or `make add controlpanel`).
- Generates the configuration interface, the form view, and the registration.
- **Volto Note**: To expose this to Volto, ensure the registry records are accessible or create a corresponding Volto config form.

### Creating a Form

For standalone backend forms (z3c.form).

- **Command**: `uvx plonecli add form` (or `make add form`).
- **Volto Note**: Standard Plone backend forms are not rendered in Volto. Use this only if you need a specific backend HTML view or if you are wrapping it in a custom API endpoint.

### Creating an Indexer

Use indexers to create custom catalog indexes for your content.

- **Command**: `uvx plonecli add indexer` (or `make add indexer`).
- This registers an adapter that extracts the data to be indexed.
- Remember to add the index to `portal_catalog.xml`.

### Creating a Subscriber

Use subscribers to react to events (e.g., object modified, added).

- **Command**: `uvx plonecli add subscriber` (or `make add subscriber`).
- Select the event interface (e.g., `IObjectModifiedEvent`) and the object interface.

### Creating an Upgrade Step

Use upgrade steps to migrate the database schema or configuration between versions.

- **Command**: `uvx plonecli add upgrade_step` (or `make add upgrade_step`).
- Registers the step in `upgrades.zcml` and `profiles/default/metadata.xml`.

### Creating a Vocabulary

Use vocabularies to define dynamic lists of terms for choice fields.

- **Command**: `uvx plonecli add vocabulary` (or `make add vocabulary`).
- Registers a named utility or factory for the vocabulary.

### Creating Classic UI Elements (Views, Viewlets, Portlets, Themes)

_Note: These are for Plone Classic UI and are generally not used in a Volto-only project._

- **View**: `uvx plonecli add view`
- **Viewlet**: `uvx plonecli add viewlet`
- **Portlet**: `uvx plonecli add portlet`
- **Theme**: `uvx plonecli add theme` / `theme_barceloneta`

### Volto Block Development Patterns

#### 1. Custom Block (View, Edit & Schema)
The standard way to create a fully custom block.
- **View (`View.jsx`)**: Renders the block content. Receives `data` props.
- **Edit (`Edit.jsx`)**: Renders the `SidebarPortal` with `BlockDataForm` to edit settings.
- **Schema (`schema.js`)**: JSON schema defining the fields.
- **Registration**:
    ```javascript
    config.blocks.blocksConfig.myBlock = {
        id: 'myBlock',
        title: 'My Block',
        icon: icon,
        group: 'common',
        view: MyBlockView,
        edit: MyBlockEdit,
        restricted: false,
        mostUsed: false,
        sidebarTab: 1,
    };
    ```

#### 2. Block Variations
Use variations to provide multiple templates for the same block (e.g., a Listing block that displays as a List or a Grid).
- **Create View**: Create a React component for the variation (e.g., `CardView.jsx`).
- **Register**:
    ```javascript
    config.blocks.blocksConfig.listing.variations = [
        ...config.blocks.blocksConfig.listing.variations,
        {
            id: 'cards',
            isDefault: false,
            title: 'Cards',
            template: CardView,
        },
    ];
    ```

#### 3. Schema Enhancers
Use Schema Enhancers to dynamically modify a block's schema. This is useful when a Variation requires extra fields (e.g., "Number of Columns" for a Grid variation).
- **Enhancer Function**:
    ```javascript
    const enhanceSchema = ({ schema, formData, intl }) => {
        if (formData.variation === 'cards') {
            // Add a new field
            schema.properties.columns = {
                title: 'Columns',
                type: 'number',
            };
            schema.fieldsets[0].fields.push('columns');
        }
        return schema;
    };
    ```
- **Register**:
    ```javascript
    config.blocks.blocksConfig.listing.schemaEnhancer = enhanceSchema;
    ```
    *Note: You can also compose multiple enhancers.*

#### 4. Custom Schema and View (Simple)
If you don't need a custom `Edit` component (because the default block editor is sufficient and you only have simple fields), you can omit `edit` in the config. Volto will generate a default editor based on your schema.
- **Requirements**: A `View` component and a `blockSchema`.
- **Registration**:
    ```javascript
    config.blocks.blocksConfig.simpleBlock = {
        id: 'simpleBlock',
        title: 'Simple Block',
        view: SimpleView,
        blockSchema: simpleSchema, 
        // No 'edit' key needed; Volto uses DefaultEditBlockData
    };
    ```
