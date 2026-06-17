# Module: Classic UI (Diazo & ZPT)

This module covers Plone 6 Classic UI development using Diazo theming, Browser Views, and Zope Page Templates (ZPT).

## Core Principles

- **Classic UI strictly** — Use Browser Views, Viewlets, and Portlets.
- **Bootstrap 5** — Standard for Plone 6 Classic UI.
- **Diazo** — Theming via `rules.xml`.
- **Automated Scaffolding** — Use `plonecli` (or `make add`) with `mrbob.ini` files (see procedure in Backend module).

## Diazo Theming

### rules.xml Basics
```xml
<rules
    xmlns="http://namespaces.plone.org/diazo"
    xmlns:css="http://namespaces.plone.org/diazo/css">

    <theme href="index.html" />

    <!-- Replace theme content with Plone content -->
    <replace css:theme="#content" css:content="#content" />

    <!-- Merge body classes -->
    <merge attributes="class" css:theme="body" css:content="body" />
</rules>
```

## Browser Views (ZPT/TAL)

### 1. View Registration (configure.zcml)
```xml
<browser:page
    name="my-view"
    for="*"
    class=".views.MyView"
    template="templates/my_view.pt"
    permission="zope2.View"
    />
```

### 2. View Template (ZPT)
```html
<html xmlns:metal="http://xml.zope.org/namespaces/metal"
      xmlns:tal="http://xml.zope.org/namespaces/tal"
      metal:use-macro="context/main_template/macros/master">
<body>
  <metal:content-core fill-slot="content-core">
    <h1 tal:content="context/title">Title</h1>
    <div tal:replace="structure context/text/output" />
  </metal:content-core>
</body>
</html>
```

## Viewlets

### 1. Registration
```xml
<browser:viewlet
    name="my.viewlet"
    manager="plone.app.layout.viewlets.interfaces.IBelowContentTitle"
    template="templates/my_viewlet.pt"
    permission="zope2.View"
    />
```

## Portlets

### 1. Registration
```xml
<plone:portlet
    name="my.portlet"
    interface=".portlets.IMyPortlet"
    assignment=".portlets.Assignment"
    renderer=".portlets.Renderer"
    addview=".portlets.AddForm"
    editview=".portlets.EditForm"
    />
```

## Mockup Patterns (ES6)

Create interactive behaviors with Patterns.

1. **Scaffold**: `uvx plonecli add mockup_pattern`
2. **Implementation**:
```javascript
import { BasePattern } from '@patternslib/patternslib/src/core/base';

export default BasePattern.extend({
  name: 'testpattern',
  trigger: '.pat-testpattern',
  init() {
    this.el.innerText = 'Pattern initialized';
  },
});
```

## Resource Bundles (bundles.xml)

Register modern JS/CSS bundles in `profiles/default/registry/bundles.xml`.
```xml
<registry>
  <records interface="plone.bundles.interfaces.IBundleRegistry"
           prefix="plone.bundles.my-addon">
    <value key="enabled">True</value>
    <value key="csscompilation">++plone++my.addon/my-addon.css</value>
    <value key="jscompilation">++plone++my.addon/my-addon.js</value>
    <value key="depends">plone</value>
    <value key="load_defer">True</value>
  </records>
</registry>
```

## Advanced Layouts (plone.app.mosaic)

For building advanced, user-customizable layouts in Classic UI, use `plone.app.mosaic`. It allows editors to create complex page structures using tiles and drag-and-drop.

- **Mosaic Documentation**: [https://plone.github.io/plone.app.mosaic/](https://plone.github.io/plone.app.mosaic/)

### Core Concepts
- **Layouts**: Defined in `registry.xml` or through the UI.
- **Tiles**: Small, reusable UI components. Use `plone.app.tiles` to create new ones.
- **Form Layouts**: Define which fields are available as tiles.

## Classic UI Testing

Use **Robot Framework** for integration tests in Classic UI.

Example `test_view.robot`:
```robot
*** Settings ***
Resource    plone/app/robotframework/selenium.robot
Library     Remote  ${PLONE_URL}/RobotRemote

*** Test Cases ***
Scenario: View my-view
    Given a logged in site administrator
    When I go to ${PLONE_URL}/my-document/@@my-view
    Then I should see "My Document"
```

## Documentation Retrieval Protocol (Mandatory)

Do not rely on static lists of fields or widgets. Always fetch the latest documentation:

- **Backend Fields**: `https://6.docs.plone.org/_sources/backend/fields.md`
- **Behaviors**: `https://6.docs.plone.org/_sources/backend/behaviors.md`

Use the `webfetch` tool on `https://6.docs.plone.org/llms.txt` to find the exact URLs.
