# sf-page-create Skill

Create Salesforce Lightning pages (FlexiPage), Custom Tabs, and Lightning Apps with proper structure. Combines three related UI components into one skill. Works across any Salesforce project.

## Quick Start

### Create a Lightning Record Page
```
Create a Lightning record page for Account with Name as primary field
```

### Create a Custom Tab
```
Create a custom tab for my Space_Station__c object
```

### Create a Lightning App
```
Create a Lightning app called "Fleet Management" with Vehicle__c and Maintenance__c tabs
```

### Complete Workflow (Page + Tab + App)
```
Create a complete UI for my Vehicle__c object with page, tab, and app
```

## What This Skill Does

Creates three types of Salesforce UI components:

1. **FlexiPage** (Lightning Pages)
   - RecordPage - displays record details
   - AppPage - utility pages and dashboards
   - HomePage - landing pages

2. **Custom Tab** (Navigation)
   - Object tabs - navigate to custom/standard objects
   - Web tabs - link to external websites
   - Visualforce tabs - access Visualforce pages

3. **Lightning App** (Application Container)
   - Organizes tabs into applications
   - Configures navigation
   - Enables Lightning Experience

## Key Features

- **CLI-First**: Uses `sf template generate flexipage` for FlexiPages (mandatory)
- **Strict Validation**: Follows element allowlists for tabs (prevents deployment errors)
- **Unique Motifs**: Selects contextually relevant tab icons
- **Dry-Run First**: Validates before actual deployment
- **Generic**: Works across any Salesforce project
- **Integration-Ready**: Works with sf-lwc-create and generating-custom-object skills

## Workflow

The skill follows dependency order:

```
FlexiPage → Custom Tab → Lightning App
```

1. Create FlexiPage using CLI template
2. Create Custom Tab for navigation
3. Create Lightning App to organize tabs

## When to Use

Use this skill when you need to:
- Create Lightning pages (record, app, home)
- Add navigation tabs to objects
- Build Lightning applications
- Set up complete UI workflows

Trigger phrases:
- "create page", "new flexipage", "lightning page"
- "create tab", "object tab", "web tab"
- "create app", "lightning app", "new application"
- "record page", "app page", "home page"

## Key Requirements

### FlexiPage
- MUST use CLI template command (never create XML from scratch)
- Must validate with dry-run deployment
- Must STOP after validation (no manual XML edits)

### Custom Tab
- MUST follow strict element allowlist
- Object tabs: ONLY `<customObject>true` and `<motif>`
- Web tabs: ONLY allowed elements (see SKILL.md)
- Must choose unique, contextually relevant motifs

### Lightning App
- Must specify `<uiType>Lightning</uiType>`
- Must reference existing tabs
- Must include form factors (Small, Large)

## Examples

See [SKILL.md](./SKILL.md) for detailed examples including:
- RecordPage with field configuration
- Object tab with motif selection
- Web tab with external URL
- Lightning app with multiple tabs
- Complete workflow (Page + Tab + App)

## Testing

See [tests/README.md](./tests/README.md) for test cases:
- Test 1: FlexiPage (RecordPage) using CLI
- Test 2: Custom Tab (Object tab)
- Test 3: Lightning App with tabs

## Troubleshooting

Common issues and solutions:

### FlexiPage Issues
- **CLI command not found** → Install templates plugin: `sf plugins install templates`
- **Invalid field reference** → Use `Record.FieldName` format (not `Object.FieldName`)
- **XML parsing error** → Encode HTML/XML characters in values

### Tab Issues
- **Duplicate external id** → Remove `<label>` from object tabs
- **Unknown element type** → Use only allowed elements from templates
- **Missing required field: label** → Add `<label>` to web/Visualforce tabs

### App Issues
- **Invalid tab reference** → Ensure tabs exist and are deployed first
- **Must reference Lightning Page Tabs only** → Don't include Visualforce tabs

See [SKILL.md](./SKILL.md) for complete troubleshooting guide.

## Integration

### With sf-lwc-create
1. Create LWC using sf-lwc-create skill
2. Create FlexiPage using this skill
3. Add LWC to FlexiPage (manual XML edit)

### With generating-custom-object
1. Create custom object using generating-custom-object skill
2. Create object tab using this skill
3. Include tab in Lightning App using this skill

## Best Practices

1. Always use CLI template for FlexiPages (never manual XML)
2. Choose unique, meaningful motifs for tabs
3. Follow strict element allowlists (prevents deployment errors)
4. Validate with dry-run before actual deployment
5. Follow dependency order: Page → Tab → App
6. Work generically (don't hardcode project paths)

## Metadata

- **Author**: Claude Code
- **Version**: 1.0.0
- **Category**: salesforce-ui
- **License**: MIT
- **Tags**: salesforce, flexipage, tab, app, lightning, ui, generic

## Documentation

- [SKILL.md](./SKILL.md) - Complete skill specification
- [tests/](./tests/) - Test cases and validation

## Contributing

When modifying this skill:
1. Update SKILL.md with any new patterns
2. Add test cases for new functionality
3. Update troubleshooting section
4. Maintain generic approach (no project-specific code)
5. Follow Claude Code attribution

## Support

For issues or questions:
1. Check troubleshooting section in SKILL.md
2. Review test cases in tests/ directory
3. Verify CLI plugins installed: `sf plugins`
4. Run dry-run deployment to catch errors early
