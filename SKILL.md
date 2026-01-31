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
- **Use Generators**: Always use `plonecli` (or `make add`) to create backend components (content types, behaviors, services, etc.). **Manual creation of these files is forbidden** as it leads to registration errors.
- **Best Practices**: Follow community standards (plone.api, black, flake8, prettier, eslint).

## Backend Guidelines (Python/Plone)

### 0. Generator Usage (Strict)

- If asked to create a project scaffold, \*\*always use `uvx cookieplone` to do so. Be sure to know whether you need to create a Volto project or Classic UI project. If in doubt or no instructions are given to you, default to Volto project.
- **Always use `plonecli`** to generate boilerplate code.
- **Never** manually create Python classes, ZCML registrations, or FTI XML files from scratch.
- Use the **Automated Method** (`mrbob.ini`) whenever possible to ensure reproducibility and agent autonomy.

### 1. Content Types (Dexterity)

- Prefer **Python schemas** (plone.supermodel) over XML models.
- Use `plone.api` for all high-level interactions (create, get, search, etc.).
- Isolate business logic in **Behaviors** or **Adapters**.

When creating a content type based on user input and they request specific fields, prioritize using existing Plone behaviors over adding new fields directly to the content type's schema. Behaviors are reusable components that add fields and functionality to content types.

Here are some of the most commonly used and important behaviors provided by Plone, which you can activate in your content type's XML definition (e.g., in `profiles/default/types/MyType.xml`):

**Common Behaviors for Content (from `plone.app.contenttypes`):**

- `plone.richtext`: Provides a rich text field for the main body content.
- `plone.leadimage`: Adds a "Lead Image" field, often displayed prominently.
- `plone.collection`: Used by Collection content types to define query criteria.
- `plone.tableofcontents`: Automatically generates a table of contents from headings within a rich text field.

**General Behaviors (from `plone.app.dexterity`):**

- `plone.basic`: Provides Dublin Core title and description fields. This should only be included if `plone.dublincore` is not included.
- `plone.categorization`: Adds fields for tags (keywords) and language. This should only be included if `plone.dublincore` is not included.
- `plone.publication`: Manages publication-related fields: effective and expiration dates. This should only be included if `plone.dublincore` is not included.
- `plone.ownership`: Adds creator, contributor, and rights fields. This should only be included if `plone.dublincore` is not included.
- `plone.dublincore`: This behavior includes `plone.basic`, `plone.categorization`, and `plone.ownership`. Usually, and if the user does not say anything, this behavior should be included.
- `plone.shortname`: Gives the ability to rename an item from its edit form.
- `plone.namefromtitle`: Automatically generate short URL name for content based on its initial title

** Some other interesting behaviors **:

- `plone.versioning`: from `plone.app.versioningbehavior` that adds versioning support using Products.CMFEditions.
- `plone.relateditems`: from `plone.app.relationfield` that adds a field to relate other contents to the current content.
- `plone.locking`: from `plone.app.lockingbehavior` that provides support to lock the editing of the current content.

**Event specific behaviors (from `plone.app.event`)**: These behaviors are usually provided for Event like content-types, but they contain usefull fields, useful in other scenarios too:

- `plone.eventbasic`: Basic Event schema: `start`, `end`, `whole_day` and `open_end` fields.
- `plone.eventrecurrence`: Recurrence configuration extension for Events.
- `plone.eventlocation`: Location extension for Events: `location` field.
- `plone.eventattendees`: Attendees extension for Events: `attendees` field.
- `plone.eventcontact`: Contact extension for Events: `contact_name`, `contact_email`, `contact_phone` and `event_url` fields.

**Important Note on Default Behaviors:**
Always inspect the `.xml` file generated for your new content type after running `uvx plonecli add -b ../mrbob.ini content_type`. This file, typically found in `src/my.package/my/package/profiles/default/types/MyType.xml`, will list the behaviors that are automatically included by the template. This ensures you avoid duplicating fields or functionality when adding new behaviors based on user requests.

When a user asks for a field, consider if an existing behavior already provides that functionality before defining a new schema field.

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

_Use this when developing themes for Plone Classic UI (non-Volto)._

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

**Interactive Method:**

- Run `uvx cookieplone` in your terminal.
- **This is an interactive tool.** You will be prompted to provide:
  - **Template**: Choose "Plone 6 (Volto)".
  - **Project Title & Slug**: Name your project.
  - **Description, Author, Email**: Metadata.
  - **Python Version**: Usually defaults to system Python or recommended version.
  - **Docker Support**: "Yes" is recommended for deployment.
- Once finished, it scaffolds the backend and frontend.

**Automated Method (Preferred):**

You can pass parameters directly to `cookiecutter-plone` using the `--no-input` flag and `--extra-context` arguments to avoid interactive prompts. This is useful for scripting and automation.

Here's an example:

```bash
uvx cookieplone \
  --no-input \
  --extra-context project_title="My Awesome Plone Project" \
  --extra-context project_slug="my-awesome-plone-project" \
  --extra-context description="A new Plone 6 project generated automatically." \
  --extra-context author="AI Assistant" \
  --extra-context email="ai@example.com" \
  --extra-context language_code="en" \
  --extra-context container_registry="GitHub" \
  --extra-context devops_cache="1" \
  --extra-context devops_ansible="1" \
  --extra-context devops_gha_deploy="1"
```

Refer to the `cookiecutter.json` file of the `cookiecutter-plone-starter` template (usually located in `~/.cookiecutters/cookiecutter-plone-starter/cookiecutter.json`) for a full list of available parameters and their default values. The `__prompts__` section in `cookiecutter.json` provides user-friendly descriptions of the variables.

### Creating an Add-on Package

**MANDATORY**: Use `plonecli` (which wraps `mr.bob`) to generate the boilerplate.

**Automated Method (Preferred)**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    author.name = Plone Developer
    author.email = dev@plone.org
    author.github.user = plone
    package.description = An add-on for Plone
    package.git.init = n
    plone.version = 6.0.0
    python.version = python3
    vscode_support = n
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini addon my.addon`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a New Content Type

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    dexterity_type_name = My Type
    dexterity_type_desc = Description of the type
    dexterity_type_icon_expr = puzzle
    dexterity_type_supermodel = n
    dexterity_type_base_class = Container
    dexterity_type_global_allow = y
    dexterity_type_filter_content_types = n
    dexterity_type_create_class = y
    dexterity_type_activate_default_behaviors = y
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini content_type`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Custom API Endpoint

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    service_class_name = MyService
    service_name = my-service
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini restapi_service`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Behavior

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    behavior_name = MyBehavior
    behavior_description = Description of the behavior
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini behavior`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Control Panel

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    controlpanel_python_class_name = MyControlPanel
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini controlpanel`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Form

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    form_name = MyForm
    form_title = My Form
    ```
    _(Note: Check `bobtemplates.plone` source if uncertain about variable names, as they change occasionally)._
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini form`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating an Indexer

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    indexer_name = my_index
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini indexer`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Subscriber

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    subscriber_handler_name = my_handler
    ```
    _Note: The template creates a subscriber for `IObjectModifiedEvent` on `IDexterityContent`. Edit `configure.zcml` manually to change the event or interface._
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini subscriber`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating an Upgrade Step

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    upgrade_step_title = Upgrade to new version
    upgrade_step_description = Description of what this step does
    ```
    _Note: Source and destination versions are automatically calculated from `metadata.xml`._
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini upgrade_step`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating a Vocabulary

**MANDATORY**: Use `plonecli` to generate the boilerplate.

**Automated Method**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    vocabulary_name = MyVocabulary
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini vocabulary`
3.  **Cleanup**: Delete `mrbob.ini`.

### Creating Classic UI Elements (Views, Viewlets, Portlets, Themes)

**MANDATORY**: Use `plonecli` to generate the boilerplate. Do not manually create files.
_Note: These are for Plone Classic UI and are generally not used in a Volto-only project._

**Automated Method - View**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    view_python_class = y
    view_python_class_name = MyView
    view_base_class = BrowserView
    view_name = my-view
    view_template = y
    view_template_name = my_view
    view_register_for = *
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini view`
3.  **Cleanup**: Delete `mrbob.ini`.

**Automated Method - Viewlet**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    viewlet_python_class_name = MyViewlet
    viewlet_name = myviewlet
    viewlet_template = y
    viewlet_template_name = viewlet
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini viewlet`
3.  **Cleanup**: Delete `mrbob.ini`.

**Automated Method - Portlet**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    portlet_name = Weather
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini portlet`
3.  **Cleanup**: Delete `mrbob.ini`.

**Automated Method - Theme**:

1.  **Create `mrbob.ini`**:
    ```ini
    [variables]
    theme.name = My Theme
    ```
2.  **Run Command**:
    - `uvx plonecli add -b ../mrbob.ini theme`
    - Or for specific starting points: `theme_barceloneta`, `theme_basic`.
3.  **Cleanup**: Delete `mrbob.ini`.

**Interactive Method**: `uvx plonecli add view`, `viewlet`, etc.

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
  _Note: You can also compose multiple enhancers._

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
