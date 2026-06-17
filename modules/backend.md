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

#### REST API Functional Testing
Use the patterns from `plone.restapi.tests`.
- **RelativeSession**: Use from `plone.restapi.testing`.
- **Mandatory**: Include `Accept: application/json` and run `transaction.commit()` before the request.

**Example:**
```python
from plone.restapi.testing import RelativeSession
import transaction
import unittest

class TestMyService(unittest.TestCase):
    layer = MY_PACKAGE_FUNCTIONAL_TESTING

    def setUp(self):
        self.portal = self.layer['portal']
        self.api_session = RelativeSession(self.portal.absolute_url(), test=self)
        self.api_session.headers.update({"Accept": "application/json"})

    def test_endpoint(self):
        # Setup data and commit
        self.portal.invokeFactory("Document", id="doc1")
        transaction.commit()

        response = self.api_session.get("/doc1/@my-service")
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["@type"], "Document")
```

## Standard Generator Procedure

All backend component generation follows this same 3-step procedure:

1. **Create `mrbob.ini`** in the directory containing `pyproject.toml`.
   ```ini
   [variables]
   # See specific module for variable names
   ```
2. **Run the command**: `uvx plonecli add -b mrbob.ini <template_name>`
3. **Delete `mrbob.ini`** immediately after completion.

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

#### REST API Implementation Pattern
When implementing the `Service` class:
- **`render()`**: The primary method that returns the JSON-serializable dictionary.
- **Manual JSON Errors**: Manually set the status code and return a structured error body.
- **ISerializeToJson**: Use/implement this adapter for complex content-type serialization.

**Python Example:**
```python
from plone.restapi.services import Service

class MyService(Service):
    def render(self):
        # Perform logic
        if not self.context.get('valid'):
            self.request.response.setStatus(400)
            return {"error": {"message": "Invalid context state"}}

        # Return serializable dict
        return {
            "title": self.context.Title(),
            "custom_field": "some_value"
        }
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
- **`getCurrentUrl()`**: Returns the actual URL plus the query string.
- **`getParentObject()`**: Returns the parent (aq_inner/aq_parent) of the context.

### 2. @@plone_portal_state (IPortalState)
Global site information.
- **`portal_url()` / `navigation_root_url()`**: Site root and nav root URLs. Use `navigation_root_url` for multi-site setups where sections act as roots.
- **`navigation_root()`**: The current navigation root object. Essential for catalog searches restricted to the current site/section.
- **`portal_title()`**: The title of the Plone site.
- **`member()`**: The current authenticated member object.
- **`anonymous()`**: Boolean check for anonymous users.
- **`language()`**: The current language code (e.g., 'en').

### 3. @@plone_context_state (IContextState)
State of the current context/object.
- **`canonical_object_url()`**: URL resolving default pages to their folders.
- **`is_default_page()`**: Boolean check for folder default pages.
- **`workflow_state()`**: The current workflow state ID.
- **`is_editable()`**: Whether the user can edit the context.
- **`object_url()`**: The URL of the current object.
- **`is_folderish()`**: True if the object is folderish (structural or not).
- **`parent()`**: The direct parent of the current object.

### Codebase Discovery: Available Methods

To find all available methods for these helper views, inspect the following interfaces in the Plone source code:

- **IPlone (`@@plone`)**: `Products.CMFPlone.browser.interfaces.IPlone`
- **IPortalState (`@@plone_portal_state`)**: `plone.app.layout.globals.interfaces.IPortalState`
- **IContextState (`@@plone_context_state`)**: `plone.app.layout.globals.interfaces.IContextState`
- **ITools (`@@plone_tools`)**: `plone.app.layout.globals.interfaces.ITools`

### Usage Patterns

**Python Example:**
```python
from zope.component import getMultiAdapter

# Getting multiple states
portal_state = getMultiAdapter((self.context, self.request), name="plone_portal_state")
context_state = getMultiAdapter((self.context, self.request), name="plone_context_state")

is_anon = portal_state.anonymous()
is_folder = context_state.is_folderish()
site_title = portal_state.portal_title()

# Common pattern: Catalog search restricted to navigation root
from plone import api
results = api.content.find(
    context=portal_state.navigation_root(),
    portal_type="Document",
)
```

**ZPT Example:**
```html
<div tal:define="plone context/@@plone;
                 portal_state context/@@plone_portal_state;
                 context_state context/@@plone_context_state">
  <h1 tal:content="context_state/object_url">URL</h1>
  <p tal:condition="portal_state/anonymous">You are browsing as guest.</p>
  <span tal:replace="python:plone.toLocalizedTime(context.Date())" />
</div>
```

### Accessing the Request Globally
When the `request` object is not available in the current local context (e.g., inside a utility or a low-level event subscriber), use `getRequest` from `zope.globalrequest`.

**Python Example:**
```python
from zope.globalrequest import getRequest

request = getRequest()
if request:
    # Use request.form, request.SESSION, etc.
    pass
```

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

## GenericSetup & Configuration

Plone configuration belongs in GenericSetup XML files.
- **`registry.xml`**: Central configuration (standard settings, component overrides).
- **`catalog.xml`**: For defining new indexes and metadata columns.
- **`controlpanel.xml`**: For custom control panels.
- **`types/`**: Content type definitions (FTI).

### catalog.xml Pattern
Use this to make fields searchable or available in catalog brains.
```xml
<?xml version="1.0"?>
<object name="portal_catalog">
  <!-- New Indexes -->
  <index name="my_field" meta_type="FieldIndex">
    <indexed_attr value="my_field"/>
  </index>
  <index name="my_tags" meta_type="KeywordIndex">
    <indexed_attr value="my_tags"/>
  </index>
  <index name="is_featured" meta_type="BooleanIndex">
    <indexed_attr value="is_featured"/>
  </index>

  <!-- Metadata Columns -->
  <column value="my_field"/>
</object>
```

### Managing Configuration Changes
- **Upgrade Steps**: Use `uvx plonecli add upgrade_step` for any changes on live sites. Never rely on re-installing the product.

## Zope Component Architecture (ZCA) Patterns

### 1. Named Utility Lookup
```python
from zope.component import getUtility
from plone.registry.interfaces import IRegistry

registry = getUtility(IRegistry)
```

### 2. Multi-Adapter Lookup
Used when an adapter depends on both the context and the request.
```python
from zope.component import getMultiAdapter

# e.g., looking up a view or a specific state view
view = getMultiAdapter((context, request), name="my-view")
```

## Security & CSRF Protection

### 1. Write Protection (CSRF)
For any view that performs data modification, you MUST protect it against CSRF.
```python
from plone.protect.utils import addTokenToUrl
from plone.protect.authenticator import check_authenticator
from plone.protect import PostOnly

# In a browser view:
def __call__(self):
    # Ensure it's a POST request
    PostOnly(self.request)
    # Check the CSRF authenticator token
    check_authenticator(self.request)
    # Perform modification...
```

### 2. Elevated Privileges (unrestrictedSearchResults)
Use `unrestrictedSearchResults` only when you need to find objects the current user doesn't have permission to see (use with extreme caution).
```python
from plone import api
catalog = api.portal.get_tool("portal_catalog")
results = catalog.unrestrictedSearchResults(portal_type="Document")
```

## Performance & Caching

### 1. Memoization (plone.memoize)
Use `plone.memoize` to cache expensive method calls. Choose the correct decorator based on the scope:

- **BrowserViews**: Use `plone.memoize.view.memoize`. It uses the request to memoize results, ensuring the cache is cleared at the end of the request.
- **Content-type classes**: Use `plone.memoize.instance.memoize`. It memoizes results in an attribute of the context/instance, persisting for the lifetime of that object in memory.
- **Cross-request (RAM)**: Use `plone.memoize.ram.cache` for global caching across requests. You MUST provide a cache key function.

**RAM Cache Example:**
```python
from plone.memoize import ram

def _render_cachekey(method, self, param1, param2):
    # Key MUST include function name (self.__name__) and all relevant parameters
    return (method.__name__, param1, param2)

class MyView(BrowserView):
    @ram.cache(_render_cachekey)
    def expensive_calculation(self, param1, param2):
        return ...
```

### 2. plone.app.caching
Standard Plone 6 caching involves defining rules in `registry.xml`. Always prefer using the `With caching proxy` profile in production to support Varnish/CDN.

## In-Place Migration Patterns

When migrating from Plone 5.x to 6.0:
1. **Update FTI**: Ensure `icon_expr` uses the new `${portal_url}/++plone++...` pattern.
2. **Behaviors**: Replace legacy behaviors (e.g., `plone.app.contenttypes.interfaces.ILeadImage`) with the canonical Plone 6 equivalents (`plone.leadimage`).
3. **Registry**: Use `registry.xml` GenericSetup steps to migrate settings instead of Python scripts whenever possible.

## Best Practices

- **Avoid raw ZODB manipulation**: Always use `plone.api`.
- **Clean ZCML**: Organize with `<include package=".subpackage" />`.
- **Searchable Fields**: Use `plone.app.dexterity.textindexer.searchable(field)`.

## Documentation Retrieval Protocol (Mandatory)

Do not rely on static lists of fields or behaviors. Always fetch the latest documentation:

- **Backend Fields**: `https://6.docs.plone.org/_sources/backend/fields.md`
- **Behaviors**: `https://6.docs.plone.org/_sources/backend/behaviors.md`

Use the `webfetch` tool on `https://6.docs.plone.org/llms.txt` to find the exact URLs.
