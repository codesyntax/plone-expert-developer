# Module: Backend (Python & Testing)

This module covers Plone 6 backend development using Python, Dexterity, and Zope Component Architecture (ZCA).

## Core Principles

- **Use `plone.api`** for all standard operations (content, portal, users, groups).
- **REST API First**: For Volto projects, never write browser views; write REST API services instead.
- **Modern Testing**: Use `pytest-plone` as the test runner.

## Testing Strategy

### 1. Pytest-Plone (Preferred)
Modern projects use `pytest` with the `pytest-plone` plugin.
- **Fixtures**: Use `plone_app_testing` provided fixtures.
- **Structure**: Tests live in the `tests/` directory.

Example `test_example.py`:
```python
import pytest
from plone.api import content

def test_content_creation(portal):
    doc = content.create(
        container=portal,
        type="Document",
        title="My Document",
    )
    assert doc.title == "My Document"
```

### 2. plone.app.testing (Compatibility)
Core Plone and `plonecli` generators rely on `plone.app.testing` layers.
- **Layers**: Define a `Fixture` class and `Integration_Testing` / `Functional_Testing` layers in `testing.py`.
- **TestCase**: Inherit from `unittest.TestCase`.

Example:
```python
from my.package.testing import MY_PACKAGE_INTEGRATION_TESTING
import unittest

class TestSetup(unittest.TestCase):
    layer = MY_PACKAGE_INTEGRATION_TESTING

    def test_product_installed(self):
        """Test if the product is installed."""
        self.assertTrue(self.layer['portal'].portal_setup.isProductInstalled('my.package'))
```

## Backend Scenario Catalog

### Creating an Add-on Package
**Template**: `addon` | **Command**: `uvx plonecli create -b mrbob.ini addon my.addon`
```ini
[variables]
author.name = Plone Developer
plone.version = 6.0.0
python.version = python3
```

### Creating a Content Type
**Template**: `content_type` | **Command**: `uvx plonecli add -b mrbob.ini content_type`
```ini
[variables]
dexterity_type_name = My Type
dexterity_type_base_class = Container
dexterity_type_create_class = y
dexterity_type_activate_default_behaviors = y
```

### Creating a REST API Endpoint
**Template**: `restapi_service` | **Command**: `uvx plonecli add -b mrbob.ini restapi_service`
```ini
[variables]
service_class_name = MyService
```

### Creating a Behavior
**Template**: `behavior` | **Command**: `uvx plonecli add -b mrbob.ini behavior`
```ini
[variables]
behavior_name = MyBehavior
```

### Creating a Control Panel
**Template**: `controlpanel` | **Command**: `uvx plonecli add -b mrbob.ini controlpanel`
```ini
[variables]
controlpanel_python_class_name = MyControlPanel
```

### Creating an Indexer
**Template**: `indexer` | **Command**: `uvx plonecli add -b mrbob.ini indexer`
```ini
[variables]
indexer_name = my_index
```

### Creating a Subscriber
**Template**: `subscriber` | **Command**: `uvx plonecli add -b mrbob.ini subscriber`
```ini
[variables]
subscriber_handler_name = my_handler
```

### Creating an Upgrade Step
**Template**: `upgrade_step` | **Command**: `uvx plonecli add -b mrbob.ini upgrade_step`
```ini
[variables]
upgrade_step_title = Upgrade to new version
```

### Creating a Vocabulary
**Template**: `vocabulary` | **Command**: `uvx plonecli add -b mrbob.ini vocabulary`
```ini
[variables]
vocabulary_name = MyVocabulary
```

## Common Helper Views & Patterns

In Plone Backend and Classic UI development, helper views provide essential state and formatting logic. Access them via `getMultiAdapter` in Python or `context/@@view_name` in ZPT.

### 1. @@plone (IPlone)
Primary helper for formatting and structural checks.
- **`toLocalizedTime(time, long_format=None)`**: Localizes dates/times.
- **`toLocalizedSize(size)`**: Formats byte sizes (e.g., 3KB).
- **`normalizeString(text)`**: Converts text to a valid ID.
- **`cropText(text, length, ellipsis)`**: Safely crops text on word boundaries.
- **`isStructuralFolder()`**: Boolean check for folderish items.

### 2. @@plone_portal_state (IPortalState)
Global site information.
- **`portal_url()` / `navigation_root_url()`**: Site root and nav root URLs.
- **`member()`**: The current authenticated member object.
- **`anonymous()`**: Boolean check for anonymous users.

### 3. @@plone_context_state (IContextState)
State of the current context/object.
- **`canonical_object_url()`**: URL resolving default pages to their folders.
- **`is_default_page()`**: Boolean check for folder default pages.
- **`workflow_state()`**: The current workflow state ID.
- **`is_editable()`**: Whether the user can edit the context.

### 4. @@images (Modern Image Rendering)
Use the `@@images` view for responsive images. **Note**: In ZPT, use `python:` expressions to call methods with parameters.

**ZPT Examples:**
```html
<tal:block define="images context/@@images">
  <!-- Render img tag with srcset -->
  <div tal:replace="structure python:images.srcset('image', scale_in_src='teaser', sizes='(max-width: 768px) 100vw, 400px')" />

  <!-- Render picture tag using a defined variant -->
  <div tal:replace="structure python:images.picture('image', picture_variant='banner')" />
</tal:block>
```

**Python Example:**
```python
images = getMultiAdapter((self.context, self.request), name="images")
img_tag = images.srcset("image", scale_in_src="teaser")
```

## Internationalization (i18n) & Localization (l10n)

For manual translation of months or weekdays, use `plone.base.i18nl10n`. These methods return `msgid` strings intended for the `plonelocales` domain.

**Python Example:**
```python
from plone.base import i18nl10n
from zope.i18n import translate

# Get msgid for January (1)
msgid = i18nl10n.monthname_msgid(1) 
# Result: 'month_jan'
translated = translate(msgid, domain='plonelocales', context=self.request)
```

## Best Practices

- **Avoid raw ZODB manipulation**: Always use `plone.api`.
- **Clean ZCML**: Organize with `<include package=".subpackage" />`.
- **Searchable Fields**: Use `plone.app.dexterity.textindexer.searchable(field)`.

## Documentation Retrieval Protocol (Mandatory)

Do not rely on static lists of fields or behaviors. Always fetch the latest documentation:

- **Backend Fields**: `https://6.docs.plone.org/_sources/backend/fields.md`
- **Behaviors**: `https://6.docs.plone.org/_sources/backend/behaviors.md`

Use the `webfetch` tool on `https://6.docs.plone.org/llms.txt` to find the exact URLs.
