---
name: sf-page-create
description: Creates Salesforce Lightning pages (FlexiPage), Custom Tabs, and Lightning Apps. Use when creating record pages, app pages, home pages, tabs, or lightning applications. Trigger on "create page", "new flexipage", "lightning page", "create tab", "new app", "record page", "app page", "home page".
license: MIT
metadata:
  version: 1.0.0
  category: salesforce-ui
  tags: [salesforce, flexipage, tab, app, lightning, ui, generic]
---

## When to Use This Skill

Use this skill when you need to:
- Create Lightning Record Pages, App Pages, or Home Pages (FlexiPage)
- Create Custom Tabs (object, web, or Visualforce tabs)
- Create Lightning Apps (CustomApplication)
- Set up complete UI navigation workflows (Page → Tab → App)
- Work with any `.flexipage-meta.xml`, `.tab-meta.xml`, or `.app-meta.xml` files

## Specification

# Salesforce UI Component Creation Guide

## Overview

This skill creates three interconnected Salesforce UI components:
1. **FlexiPage**: Lightning pages that display data and components
2. **Custom Tab**: Navigation tabs that appear in the app navigation bar
3. **Lightning App**: Application containers that organize tabs

These components work together in a dependency chain: FlexiPage → Tab → App

---

## Quick Start Workflow

### Determine What to Create

Ask yourself (or the user):
1. Do they need a **page** (to display data/components)? → Create FlexiPage
2. Do they need a **tab** (for navigation)? → Create Custom Tab
3. Do they need an **app** (to organize multiple tabs)? → Create Lightning App
4. Do they need all three? → Follow the full workflow below

---

## Part 1: Creating FlexiPages (Lightning Pages)

### Step 1: Bootstrap with CLI Template (MANDATORY)

**CRITICAL: When creating NEW FlexiPages, you MUST ALWAYS start with the CLI template command.** Never create FlexiPage XML from scratch - the CLI provides valid structure, proper regions, and correct component configuration that prevents deployment errors.

```bash
sf template generate flexipage \
  --name <PageName> \
  --template <RecordPage|AppPage|HomePage> \
  --sobject <SObject> \
  --primary-field <Field1> \
  --secondary-fields <Field2,Field3> \
  --detail-fields <Field4,Field5,Field6,Field7> \
  --output-dir force-app/main/default/flexipages
```

#### Template Requirements

**RecordPage:**
- Requires `--sobject` (e.g., Account, Custom_Object__c)
- Requires field parameters:
  - `--primary-field`: Most important identifying field (e.g., Name)
  - `--secondary-fields`: Record summary (recommended 4-6, max 12)
  - `--detail-fields`: Full record details, including required fields (e.g., Name)

**AppPage:**
- No additional requirements
- Good for dashboards and utility pages

**HomePage:**
- No additional requirements
- Good for landing pages and home screens

#### Field Selection Rules

- **Validate fields exist**: Use describe commands to discover available fields for the object before specifying them
- **Prefer compound fields**: Use `Name` (not `FirstName`/`LastName`), `BillingAddress` (not `BillingStreet`/`BillingCity`/`BillingState`)
- **Include required fields**: Always include object required fields (like `Name`) in the `--detail-fields` parameter

#### What You Get

- Valid FlexiPage XML with correct structure
- Pre-configured regions and basic components
- Proper field references and facet structure
- Ready to deploy as-is or enhance further

### Step 2: Deploy and Validate

Run a dry-run deployment to validate the page:
```bash
sf project deploy start --dry-run -d "force-app/main/default" --test-level NoTestRun --wait 10 --json
```

**Critical:** Fix any deployment errors before proceeding. The page must validate successfully.

### Step 3: STOP - No Further Modifications

**MANDATORY: Stop after Step 2. Do not add components or edit the FlexiPage XML.**

This applies even if the user requested:
- Additional components
- Page customization
- Component configuration

What you CAN do:
- Suggest what components would be useful
- Explain what enhancements are possible
- Document what would need to be added manually

What you CANNOT do:
- Modify the XML file
- Add any components
- Make any enhancements

### FlexiPage XML Structure

```xml
<FlexiPage xmlns="http://soap.sforce.com/2006/04/metadata">
   <flexiPageRegions>
      <!-- Regions and components here -->
   </flexiPageRegions>
   <masterLabel>Page Label</masterLabel>
   <template>
      <name>flexipage:recordHomeTemplateDesktop</name>
   </template>
   <type>RecordPage</type>
   <sobjectType>Object__c</sobjectType> <!-- RecordPage only -->
</FlexiPage>
```

**Page Types:**
- `RecordPage` - requires `<sobjectType>`
- `AppPage` - no sobjectType
- `HomePage` - no sobjectType

### Critical FlexiPage Rules

1. **Property Value Encoding** (MOST COMMON ERROR)
   - Any property value with HTML/XML characters MUST be manually encoded:
   ```
   1. & → &amp;   (FIRST! Encode this before others)
   2. < → &lt;
   3. > → &gt;
   4. " → &quot;
   5. ' → &apos;
   ```

2. **Field References**
   - **ALWAYS:** `Record.{FieldApiName}`
   - **NEVER:** `{ObjectName}.{FieldApiName}`

3. **Unique Identifiers**
   - Every `<identifier>` must be unique across the entire file
   - Every `<name>` in `<flexiPageRegions>` must be unique
   - If multiple components belong to same facet, combine them in ONE region with multiple `<itemInstances>`

---

## Part 2: Creating Custom Tabs

### Step 1: Determine Tab Type

Choose the appropriate tab type:
- **Object Tab**: Navigate to a custom or standard object
- **Web Tab**: Link to external website or web application
- **Visualforce Tab**: Access a custom Visualforce page

### Step 2: Generate Tab Metadata

**CRITICAL: The root element MUST always be `<CustomTab>` (NOT `<Tab>`).** The XML namespace must be `xmlns="http://soap.sforce.com/2006/04/metadata"`.

#### Object Tabs

**File name** determines the object: `{ObjectApiName}.tab-meta.xml` (e.g., `Space_Station__c.tab-meta.xml`)

**Template:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>true</customObject>
    <motif>Custom39: Telescope</motif>
</CustomTab>
```

**Allowed elements ONLY:**
- `<customObject>true</customObject>` (required)
- `<motif>` (required)
- `<description>` (optional)

**NEVER include:** `<label>`, `<sobjectName>`, `<name>`, `<fullName>`, or any other elements

#### Web Tabs

**File name**: `{TabName}.tab-meta.xml` (e.g., `Knowledge_Base.tab-meta.xml`)

**Template:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>false</customObject>
    <description>External knowledge base</description>
    <frameHeight>600</frameHeight>
    <label>Knowledge Base</label>
    <motif>Custom46: Computer</motif>
    <url>https://example.com/kb</url>
    <urlEncodingKey>UTF-8</urlEncodingKey>
</CustomTab>
```

**Allowed elements ONLY:**
- `<customObject>false</customObject>` (required)
- `<label>` (required)
- `<motif>` (required)
- `<url>` (required)
- `<urlEncodingKey>UTF-8</urlEncodingKey>` (required)
- `<description>` (optional)
- `<frameHeight>` (optional)

#### Visualforce Tabs

**File name**: `{TabName}.tab-meta.xml` (e.g., `Custom_Page_Tab.tab-meta.xml`)

**Template:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>false</customObject>
    <label>Custom Page</label>
    <motif>Custom46: Computer</motif>
    <page>CustomPage</page>
</CustomTab>
```

**Allowed elements ONLY:**
- `<customObject>false</customObject>` (required)
- `<label>` (required)
- `<motif>` (required)
- `<page>` (required - Visualforce page name)
- `<description>` (optional)

### Step 3: Choose Motif (Icon)

**CRITICAL: Choose a unique, contextually relevant motif for each tab.** Do not reuse the same motif.

**Example motifs:**
- `Custom39: Telescope` - for astronomy/observation objects
- `Custom98: Truck` - for logistics/supply objects
- `Custom46: Computer` - for technical/IT objects
- `Custom57: Desert` - for location-based objects
- `Custom12: Plane` - for travel/transport objects
- `Custom85: Wrench` - for maintenance/repair objects

### Step 4: Deploy and Validate

```bash
sf project deploy start --dry-run -d "force-app/main/default/tabs" --test-level NoTestRun --wait 10 --json
```

---

## Part 3: Creating Lightning Apps

### Step 1: Define App Properties

Determine:
- **App Name**: API name (e.g., `Space_Station_Management`)
- **App Label**: Display name (e.g., "Space Station Management")
- **App Description**: Purpose of the app
- **Tabs to Include**: List of tab API names (object tabs, web tabs, Visualforce tabs)

### Step 2: Generate App Metadata

**File name**: `{AppName}.app-meta.xml` (e.g., `Space_Station_Management.app-meta.xml`)

**Template:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <brand>
        <headerColor>#0070D2</headerColor>
        <shouldOverrideOrgTheme>false</shouldOverrideOrgTheme>
    </brand>
    <description>Manage space stations, supplies, and operations</description>
    <formFactors>Small</formFactors>
    <formFactors>Large</formFactors>
    <isNavAutoTempTabsDisabled>false</isNavAutoTempTabsDisabled>
    <isNavPersonalizationDisabled>false</isNavPersonalizationDisabled>
    <label>Space Station Management</label>
    <navType>Standard</navType>
    <tabs>Space_Station__c</tabs>
    <tabs>Supply__c</tabs>
    <tabs>Knowledge_Base</tabs>
    <uiType>Lightning</uiType>
    <utilityBar>Space_Station_Management_UtilityBar</utilityBar>
</CustomApplication>
```

### Step 3: Configure Navigation Items

**Required elements:**
- `<label>`: Display name
- `<navType>`: Usually `Standard` for Lightning apps
- `<tabs>`: One per tab to include (in order of appearance)
- `<uiType>`: Must be `Lightning` for Lightning Experience apps

**Optional elements:**
- `<brand>`: Custom branding (header color, theme)
- `<description>`: App purpose
- `<formFactors>`: `Small` (mobile), `Large` (desktop)
- `<utilityBar>`: Utility bar configuration

### Step 4: Deploy and Validate

```bash
sf project deploy start --dry-run -d "force-app/main/default/applications" --test-level NoTestRun --wait 10 --json
```

---

## Complete Workflow: Page → Tab → App

### Scenario: Create a complete UI for a custom object

**User Request:** "Create a Lightning page, tab, and app for my Space_Station__c object"

**Execution Steps:**

#### 1. Create FlexiPage (RecordPage)
```bash
sf template generate flexipage \
  --name Space_Station_Record_Page \
  --template RecordPage \
  --sobject Space_Station__c \
  --primary-field Name \
  --secondary-fields Status__c,Capacity__c,Commander__c \
  --detail-fields Name,Status__c,Capacity__c,Commander__c,Launch_Date__c \
  --output-dir force-app/main/default/flexipages
```

Validate:
```bash
sf project deploy start --dry-run -d "force-app/main/default/flexipages" --test-level NoTestRun --wait 10 --json
```

#### 2. Create Custom Tab (Object Tab)

Create file: `force-app/main/default/tabs/Space_Station__c.tab-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>true</customObject>
    <motif>Custom39: Telescope</motif>
</CustomTab>
```

Validate:
```bash
sf project deploy start --dry-run -d "force-app/main/default/tabs" --test-level NoTestRun --wait 10 --json
```

#### 3. Create Lightning App

Create file: `force-app/main/default/applications/Space_Station_Management.app-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <brand>
        <headerColor>#0070D2</headerColor>
        <shouldOverrideOrgTheme>false</shouldOverrideOrgTheme>
    </brand>
    <description>Manage space stations and operations</description>
    <formFactors>Small</formFactors>
    <formFactors>Large</formFactors>
    <isNavAutoTempTabsDisabled>false</isNavAutoTempTabsDisabled>
    <isNavPersonalizationDisabled>false</isNavPersonalizationDisabled>
    <label>Space Station Management</label>
    <navType>Standard</navType>
    <tabs>Space_Station__c</tabs>
    <uiType>Lightning</uiType>
</CustomApplication>
```

Validate:
```bash
sf project deploy start --dry-run -d "force-app/main/default/applications" --test-level NoTestRun --wait 10 --json
```

#### 4. Deploy All Components

```bash
sf project deploy start -d "force-app/main/default" --test-level NoTestRun --wait 10
```

---

## Integration with Other Skills

### With sf-lwc-create

FlexiPages can include Lightning Web Components (LWC). The workflow is:
1. Create LWC using `sf-lwc-create` skill
2. Create FlexiPage using this skill
3. Add LWC to FlexiPage by editing the XML (manual step - see FlexiPage documentation)

### With generating-custom-object

Custom objects need tabs for navigation:
1. Create custom object using `generating-custom-object` skill
2. Create object tab using this skill
3. Include tab in Lightning App using this skill

---

## Troubleshooting

### FlexiPage Issues

**Error: "sf template generate flexipage command not found"**
- **Cause**: Templates plugin not installed
- **Fix**: Install the plugin:
  ```bash
  sf plugins install templates
  ```

**Error: "We couldn't retrieve or load the information on the field"**
- **Cause**: Invalid field API name - field doesn't exist on the object
- **Fix**: Use describe commands to discover valid fields, then update the command

**Error: "Invalid field reference"**
- **Cause**: Used `ObjectName.Field` instead of `Record.Field` in XML
- **Fix**: Change to `Record.{FieldApiName}` format

**Error: "XML parsing error"**
- **Cause**: Unencoded HTML/XML in property values
- **Fix**: Manually encode `<`, `>`, `&`, `"`, `'` in all `<value>` tags

**Error: "Cannot create component with namespace"**
- **Cause**: Invalid page name (don't use `__c` suffix in page names)
- **Fix**: Use "Volunteer_Record_Page" not "Volunteer__c_Record_Page"

### Custom Tab Issues

**Error: "Duplicate external id specified for CustomTab"**
- **Cause**: Including `<label>` on object tabs (object tabs inherit label from object)
- **Fix**: Remove `<label>` element from object tab XML

**Error: "In field: sobjectName - no such column 'sobjectName' on entity 'CustomTab'"**
- **Cause**: Including `<sobjectName>` element (not allowed)
- **Fix**: Remove `<sobjectName>` - the file name determines the object

**Error: "Unknown element type"**
- **Cause**: Including forbidden elements (`<name>`, `<fullName>`, `<isHidden>`, `<tabVisibility>`, etc.)
- **Fix**: Use only allowed elements from the templates above

**Error: "Missing required field: label"**
- **Cause**: Web or Visualforce tab missing `<label>` element
- **Fix**: Add `<label>` element (required for non-object tabs)

### Lightning App Issues

**Error: "A Lightning Experience app must reference Lightning Page Tabs only"**
- **Cause**: Including Visualforce tabs in Lightning app
- **Fix**: Only include object tabs, web tabs, or Lightning component tabs

**Error: "In field: uiType - no CustomApplication named X found"**
- **Cause**: Missing or incorrect `<uiType>Lightning</uiType>`
- **Fix**: Ensure `<uiType>Lightning</uiType>` is present

**Error: "Invalid tab reference"**
- **Cause**: Tab doesn't exist or hasn't been deployed yet
- **Fix**: Ensure all referenced tabs are created and deployed before creating the app

### Deployment Order Issues

**Error: "Component not found"**
- **Cause**: Trying to deploy in wrong order (e.g., app before tabs)
- **Fix**: Follow dependency order: FlexiPage → Tab → App

---

## Best Practices

### FlexiPages
1. Always use CLI template - never create from scratch
2. Keep field lists focused - don't overload with too many fields
3. Use compound fields (Name, Address) when available
4. Validate with dry-run before actual deployment
5. Stop after validation - don't modify generated XML manually

### Custom Tabs
1. Choose unique, semantically meaningful motifs
2. Use descriptive names for web and Visualforce tabs
3. Follow strict element allowlist - don't add extra elements
4. Object tabs: only `<customObject>` and `<motif>`
5. Web tabs: include `<urlEncodingKey>UTF-8</urlEncodingKey>`

### Lightning Apps
1. Organize tabs in logical order
2. Include both mobile and desktop form factors
3. Use descriptive labels and descriptions
4. Test navigation flow after deployment
5. Consider user profiles and permissions

### General
1. Work generically - don't hardcode project-specific paths
2. Use dry-run deployment to catch errors early
3. Follow dependency order: Page → Tab → App
4. Document any manual steps required
5. Provide clear error messages and remediation steps

---

## Validation Checklist

Before completing:
- [ ] CLI template command used for FlexiPages (if creating new page)
- [ ] Dry-run deployment successful for each component
- [ ] Tab motifs are unique and semantically meaningful
- [ ] Only allowed elements used in tab XML
- [ ] Lightning App references all created tabs
- [ ] All components follow naming conventions
- [ ] Deployment order respected (Page → Tab → App)
- [ ] User informed of any manual steps required

---

## Examples

### Example 1: RecordPage Only

**Request:** "Create a Lightning record page for my Custom_Object__c"

**Response:**
```bash
sf template generate flexipage \
  --name Custom_Object_Record_Page \
  --template RecordPage \
  --sobject Custom_Object__c \
  --primary-field Name \
  --secondary-fields Status__c,Type__c,Amount__c \
  --detail-fields Name,Status__c,Type__c,Amount__c,Created_Date__c \
  --output-dir force-app/main/default/flexipages

# Validate
sf project deploy start --dry-run -d "force-app/main/default/flexipages" --test-level NoTestRun --wait 10 --json
```

### Example 2: Object Tab Only

**Request:** "Create a tab for my Inventory__c object"

**Response:**

Create file: `force-app/main/default/tabs/Inventory__c.tab-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>true</customObject>
    <motif>Custom98: Truck</motif>
</CustomTab>
```

Validate:
```bash
sf project deploy start --dry-run -d "force-app/main/default/tabs" --test-level NoTestRun --wait 10 --json
```

### Example 3: Complete Workflow (Page + Tab + App)

**Request:** "Create a complete UI for my Vehicle__c object with page, tab, and app"

**Response:**

**1. Create FlexiPage:**
```bash
sf template generate flexipage \
  --name Vehicle_Record_Page \
  --template RecordPage \
  --sobject Vehicle__c \
  --primary-field Name \
  --secondary-fields Make__c,Model__c,Year__c,Status__c \
  --detail-fields Name,Make__c,Model__c,Year__c,Status__c,VIN__c,License_Plate__c \
  --output-dir force-app/main/default/flexipages
```

**2. Create Tab:**

File: `force-app/main/default/tabs/Vehicle__c.tab-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>true</customObject>
    <motif>Custom12: Plane</motif>
</CustomTab>
```

**3. Create App:**

File: `force-app/main/default/applications/Vehicle_Management.app-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <brand>
        <headerColor>#0070D2</headerColor>
        <shouldOverrideOrgTheme>false</shouldOverrideOrgTheme>
    </brand>
    <description>Manage vehicle fleet and maintenance</description>
    <formFactors>Small</formFactors>
    <formFactors>Large</formFactors>
    <isNavAutoTempTabsDisabled>false</isNavAutoTempTabsDisabled>
    <isNavPersonalizationDisabled>false</isNavPersonalizationDisabled>
    <label>Vehicle Management</label>
    <navType>Standard</navType>
    <tabs>Vehicle__c</tabs>
    <uiType>Lightning</uiType>
</CustomApplication>
```

**4. Validate All:**
```bash
sf project deploy start --dry-run -d "force-app/main/default" --test-level NoTestRun --wait 10 --json
```

**5. Deploy:**
```bash
sf project deploy start -d "force-app/main/default" --test-level NoTestRun --wait 10
```

---

## Summary

This skill creates three interconnected Salesforce UI components:
- **FlexiPage**: Use CLI template (mandatory), validate, stop
- **Custom Tab**: Follow strict element allowlist, choose unique motif
- **Lightning App**: Reference tabs, configure navigation

Follow the dependency order: Page → Tab → App

Always validate with dry-run before actual deployment.

Work generically across any Salesforce project.

For integration with LWC components, use the `sf-lwc-create` skill first.
