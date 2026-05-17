# Salesforce Reports & Dashboards — SKILL

## When to use this skill
Read this file at the start of any task involving:
- Creating Reports or Report Types
- Creating Dashboards
- Report/Dashboard Folder permissions
- Deploying reports via metadata
- "User can't see report" investigations

---

## 1. Architecture Overview

```
Report Folder
  └── Report
        └── References a Report Type (standard or custom)

Dashboard Folder
  └── Dashboard
        └── References Reports (as components/widgets)
```

> ⚠️ **Reports and Dashboards live in Folders.** Folder permissions control who can see them.
> ⚠️ A user must have access to the **Folder** AND the **underlying object** to see report data.

---

## 2. Report Types

### Standard Report Types (built-in)
Used for standard and custom objects automatically. Access via:
```
Reports → New Report → [Select Object]
```

### Custom Report Types
Required when:
- Reporting across objects with specific join logic
- Need to expose fields from related objects not available in standard types
- Combining parent + child objects

```
force-app/main/default/reportTypes/
  Employment_with_Tariff.reportType-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ReportType xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Employment_with_Tariff</fullName>
    <label>Employment with Tariff Lines</label>
    <description>Employment records with related Tariff Lines</description>
    <baseObject>Employment__c</baseObject>
    <category>other</category>
    <deployed>true</deployed>
    <join>
        <outerJoin>false</outerJoin>    <!-- false = only Employment with Tariff Lines; true = all Employment -->
        <relationship>Tariff_Lines__r</relationship>   <!-- relationshipName + __r -->
        <reportType>
            <join>
                <!-- Additional joins if needed -->
            </join>
            <label>Tariff Lines</label>
            <object>Tariff_Line__c</object>
        </reportType>
    </join>
    <sections>
        <columns>
            <field>Employment__c.NAME</field>
            <field>Employment__c.Status__c</field>
            <field>Employment__c.Start_Date__c</field>
        </columns>
        <masterLabel>Employment Fields</masterLabel>
    </sections>
</ReportType>
```

---

## 3. Report Folders

### Folder Types

| Type | Use |
|---|---|
| Public | Shared with groups/roles |
| Private | Only creator sees |
| Hidden | System use only |

### Folder Metadata

```
force-app/main/default/reports/
  HR_Reports/
    HR_Reports-meta.xml         ← folder definition
    Active_Employments.report   ← report inside folder
```

```xml
<!-- HR_Reports-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<ReportFolder xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Reports</fullName>
    <accessType>Public</accessType>   <!-- Public | Shared | Hidden -->
    <name>HR Reports</name>
    <publicFolderAccess>ReadWrite</publicFolderAccess>   <!-- Read | ReadWrite -->
    <sharedTo>
        <group>PG_HR_Managers</group>       <!-- Public Group DeveloperName -->
        <!-- OR: <role>HR_Manager</role> -->
        <!-- OR: <roleAndSubordinates>HR_Manager</roleAndSubordinates> -->
        <!-- OR: <allInternalUsers/> -->
    </sharedTo>
</ReportFolder>
```

> ⚠️ **`publicFolderAccess`:**
> - `Read` — users can view/run reports in folder but not edit/create
> - `ReadWrite` — users can view, run, edit, and create reports in folder

---

## 4. Reports — Metadata

```xml
<!-- Active_Employments.report-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Reports/Active_Employments</fullName>   <!-- Folder/ReportName -->
    <format>Tabular</format>    <!-- Tabular | Summary | Matrix | Joined -->
    <name>Active Employments</name>
    <description>All active employment records</description>
    <reportType>Employment__c</reportType>    <!-- API name of report type -->

    <!-- Columns to display -->
    <columns>
        <field>NAME</field>
    </columns>
    <columns>
        <field>Status__c</field>
    </columns>
    <columns>
        <field>Start_Date__c</field>
    </columns>
    <columns>
        <field>Employee__c</field>
    </columns>

    <!-- Filters -->
    <filter>
        <booleanFilter>1</booleanFilter>
        <criteriaItems>
            <column>Status__c</column>
            <columnToColumn>false</columnToColumn>
            <isUnlocked>true</isUnlocked>
            <operator>equals</operator>
            <value>Active</value>    <!-- API value — not Hebrew -->
        </criteriaItems>
    </filter>

    <!-- Sorting -->
    <sortColumn>Start_Date__c</sortColumn>
    <sortOrder>Desc</sortOrder>

    <!-- Row limit for Tabular -->
    <rowLimit>2000</rowLimit>
    <showDetails>true</showDetails>

    <scope>organization</scope>   <!-- mine | organization | queue | team | territory | etc. -->
</Report>
```

### Report Formats

| Format | Use |
|---|---|
| `Tabular` | Simple list — rows and columns |
| `Summary` | Grouped by field — subtotals |
| `Matrix` | Grouped by rows AND columns |
| `Joined` | Multiple report blocks |

---

## 5. Dashboard Folders

```
force-app/main/default/dashboards/
  HR_Dashboards/
    HR_Dashboards-meta.xml
    Employment_Overview.dashboard
```

```xml
<!-- HR_Dashboards-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<DashboardFolder xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Dashboards</fullName>
    <accessType>Public</accessType>
    <name>HR Dashboards</name>
    <publicFolderAccess>ReadWrite</publicFolderAccess>
    <sharedTo>
        <roleAndSubordinates>HR_Manager</roleAndSubordinates>
    </sharedTo>
</DashboardFolder>
```

---

## 6. Dashboards — Metadata

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Dashboard xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Dashboards/Employment_Overview</fullName>
    <title>Employment Overview</title>
    <description>Overview of active employments</description>
    <backgroundEndColor>#FFFFFF</backgroundEndColor>
    <backgroundStartColor>#FFFFFF</backgroundStartColor>
    <dashboardType>SpecifiedUser</dashboardType>   <!-- LoggedInUser | SpecifiedUser | MyTeamUser -->
    <runningUser>admin@moh.gov.il</runningUser>     <!-- required for SpecifiedUser -->
    <leftSection>
        <columnSize>Medium</columnSize>
        <components>
            <chartAxisRange>Auto</chartAxisRange>
            <componentType>Metric</componentType>
            <displayUnits>Auto</displayUnits>
            <footer>Active Employments</footer>
            <header>Active</header>
            <indicatorBreakpointValue1>50</indicatorBreakpointValue1>
            <indicatorHighColor>#54C254</indicatorHighColor>
            <indicatorLowColor>#F54B4B</indicatorLowColor>
            <indicatorMiddleColor>#FFD700</indicatorMiddleColor>
            <reportName>HR_Reports/Active_Employments</reportName>
        </components>
    </leftSection>
</Dashboard>
```

### dashboardType Options

| Type | Running User | Data Shown |
|---|---|---|
| `LoggedInUser` | Current viewer | Viewer's own data |
| `SpecifiedUser` | Fixed admin user | Admin sees — consistent for all |
| `MyTeamUser` | Manager's perspective | Manager's team data |

> ⚠️ **`SpecifiedUser` requires `runningUser` email.** The specified user must have access to all relevant objects.

---

## 7. Profile Permissions for Reports & Dashboards

```xml
<!-- In Profile XML -->
<tabVisibilities>
    <tab>standard-report</tab>
    <visibility>DefaultOn</visibility>
</tabVisibilities>
<tabVisibilities>
    <tab>standard-dashboard</tab>
    <visibility>DefaultOn</visibility>
</tabVisibilities>
```

> ⚠️ Without `standard-report` and `standard-dashboard` tab visibility, users cannot access the Reports and Dashboards tabs at all.

### Object Permissions Required

Users must have at minimum `allowRead=true` on the objects referenced in a report to see data in that report. No special report permission is needed — it follows standard object/FLS permissions.

---

## 8. Deploy Order

```
1. Custom Report Types (if needed)
2. Report Folders
3. Dashboard Folders
4. Reports (inside their folders)
5. Dashboards (after their source Reports exist)
6. Profile tab visibility (standard-report, standard-dashboard)
```

> ⚠️ **"Silent Failures" בפריסה כוללת (`--source-dir force-app`):**
> Salesforce יכול לדווח "Succeeded" על קומפוננטה שבפועל לא נוצרה בארגון.
> זה קורה כאשר:
> - Dashboard Folder מצביע על Role שלא קיים בזמן הפריסה → נוצר בשקט ללא `sharedTo`
> - Report מצביע על Report Type שלא הספיק להיות זמין → Report מדולג בשקט
> - **תמיד לאמת** אחרי פריסה: `sf data query --query "SELECT Id, Title FROM Dashboard WHERE FolderName = 'X'"` 
> - **פתרון:** הפרד פריסת Reports/Dashboards לפריסה נפרדת אחרי שכל שאר המטאדאטה קיים

---

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Report tab not visible | Tab not in Profile | Add `standard-report` tab visibility |
| Report shows no data | User lacks object Read | Add object permissions to Profile |
| Report shows partial data | OWD limits visibility | User sees only records they can access |
| Folder not visible | Folder not shared with user | Share folder with user's role/group |
| Dashboard shows "no access" | Running user lacks object access | Change running user or fix their permissions |
| "לא ניתן להשתמש בדוח זה כמקור עבור רכיב זה" | Report format incompatible with dashboard component | See rules below |
| "Report type not found" | Custom report type not deployed | Deploy report type first |

---

## 9b. Report Format vs. Dashboard Component — Compatibility Rules

> שגיאה: **"לא ניתן להשתמש בדוח זה כמקור עבור רכיב זה"**
> ("Cannot use this report as a source for this component")

| פורמט דוח | תנאי לשימוש בדשבורד | פתרון אם לא עובד |
|---|---|---|
| **Summary** | חייב לפחות קיבוץ אחד (`<groupingsDown>` או `<groupingsAcross>`) | הוסף קיבוץ לפי שדה רלוונטי |
| **Matrix** | תמיד תקין | — |
| **Tabular** | חייב Row Limit + Dashboard Settings מוגדרים בדוח | הוסף `<rowLimit>` + `<showGrandTotal>` + `<dashboardFilterColumns>` |
| **Joined** | לא נתמך כרכיב בדשבורד | שנה פורמט |

### Tabular Report — הגדרת Row Limit ל-Dashboard (metadata)

```xml
<Report>
    <format>Tabular</format>
    <reportType>JOB_CATALOG_RT__c</reportType>
    <!-- Row Limit for dashboard use: -->
    <rowLimit>10</rowLimit>
    <showGrandTotal>false</showGrandTotal>
    <showSubTotals>false</showSubTotals>
    <!-- columns... -->
</Report>
```

> ⚠️ גם אחרי הוספת `<rowLimit>` בmetadata, ייתכן שב-UI עדיין תצטרך להגדיר "Dashboard Settings" פעם אחת ידנית בReport Builder.
> **כלל מעשי:** העדף **Summary format** עם קיבוץ — עובד תמיד בדשבורד ללא הגדרות נוספות.

---

## 10. Common Mistakes

| Mistake | What Happens | Fix |
|---|---|---|
| Report filter uses Hebrew picklist label | Filter matches nothing | Use API value (e.g. `Active` not `פעיל`) |
| Dashboard `SpecifiedUser` without `runningUser` | Deploy error | Add `runningUser` email |
| Folder `accessType=Public` but no `sharedTo` | No one can see the folder | Add `sharedTo` |
| Report deployed before its folder | Deploy error | Folder must exist first |
| Dashboard deployed before source reports | Deploy error | Reports must exist first |
| `standard-report` tab not in Profile | Users can't see Reports tab | Add tab visibility |
| Object not `enableReports=true` | Object not available in report builder | Set `enableReports=true` in object XML |
| Custom Report Type developer name without `__c` | "invalid report type" deploy error | Add `__c` suffix: `EMP_RT__c` not `EMP_RT` |
| Field names without object prefix in CRT report | "invalid report type" or missing fields | Use `ObjectName$FieldName` format |
| Dashboard uses `<reportName>` instead of `<report>` | Deploy error | Use `<report>` element in DashboardComponent |
| Dashboard missing `backgroundFadeDirection` | Deploy error — required field | Add `<backgroundFadeDirection>Diagonal</backgroundFadeDirection>` |
| Dashboard missing `textColor`, `titleColor`, `titleSize` | Deploy error — required at Dashboard root | Add all three at root level, not inside component |
| Dashboard `indicatorBreakpointValue1` in component | Deploy error — invalid element | Remove it entirely |
| Folder metadata file placed INSIDE the folder dir | fullName computed as `HR_Reports/HR_Reports` | Move metadata file to SIBLING of folder directory |
| `dashboardType=LoggedInUser` when org limit reached | Deploy error — org quota exceeded | Change to `SpecifiedUser` with `runningUser` |

---

## 11. CRT Reports — Critical Rules

> ⚠️ These rules apply ONLY when using Custom Report Types (CRT). Standard object reports work differently.

### reportType — MUST include `__c` suffix

In the Report XML `<reportType>` element:
- ✅ `<reportType>EMP_RT__c</reportType>`
- ❌ `<reportType>EMP_RT</reportType>` — causes "invalid report type" deploy error

This is different from other metadata references where you use the developer name without `__c`.

### Field References — Object Prefix Required

In CRT-based reports, ALL field references must use `ObjectName$FieldName` format:

```xml
<!-- ✅ CRT report — correct -->
<columns>
    <field>Employment__c$Name</field>
</columns>
<columns>
    <field>Employment__c$Status__c</field>
</columns>
<groupingsDown>
    <field>Employment__c$Status__c</field>
</groupingsDown>
<filter>
    <criteriaItems>
        <column>Employment__c$Status__c</column>
    </criteriaItems>
</filter>
<sortColumn>Employment__c$Effective_From__c</sortColumn>

<!-- ❌ Wrong — bare field name (works for standard reports, NOT for CRT) -->
<columns>
    <field>Status__c</field>
</columns>
```

### dateGranularity for Non-Date Groupings

When grouping by a Picklist or Text field in `<groupingsDown>` or `<groupingsAcross>`:
```xml
<groupingsDown>
    <dateGranularity>Day</dateGranularity>   <!-- Use "Day" for non-date fields -->
    <field>Employment__c$Status__c</field>
    <sortOrder>Asc</sortOrder>
</groupingsDown>
```

### Complete CRT Report Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <format>Summary</format>
    <name>Active Employments</name>
    <reportType>EMP_RT__c</reportType>          <!-- __c suffix required! -->
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>Employment__c$Status__c</field>   <!-- ObjectName$Field format -->
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <columns>
        <field>Employment__c$Name</field>
    </columns>
    <columns>
        <field>Employment__c$Employee__c</field>
    </columns>
    <filter>
        <booleanFilter>1</booleanFilter>
        <criteriaItems>
            <column>Employment__c$Status__c</column>
            <columnToColumn>false</columnToColumn>
            <isUnlocked>true</isUnlocked>
            <operator>equals</operator>
            <value>Active</value>
        </criteriaItems>
    </filter>
    <scope>organization</scope>
    <showDetails>true</showDetails>
</Report>
```

### Folder Metadata Placement — Critical

> ⚠️ The folder `.meta.xml` file must be a **SIBLING** of the folder directory, NOT inside it.

```
force-app/main/default/reports/
  HR_Reports-meta.xml        ← ✅ SIBLING of HR_Reports/ dir
  HR_Reports/
    Active_Employments.report-meta.xml
    All_Job_Positions.report-meta.xml
```

```
force-app/main/default/reports/
  HR_Reports/
    HR_Reports-meta.xml          ← ❌ INSIDE — Salesforce computes fullName as HR_Reports/HR_Reports
    Active_Employments.report-meta.xml
```

---

## Cross-Domain Interactions (Critical — Often Missed)

### Sharing Rules and Report Data
- Reports show only records the **running user** can access (respects OWD + Sharing Rules)
- If OWD = Private and no Sharing Rule → report returns only records the user owns
- Dashboard "Run as specified user": the dashboard reflects that user's access level, not the viewer's
- Dashboard "Run as logged-in user": each viewer sees only their own accessible records
- If a report seems to return fewer records than expected → check OWD and Sharing Rules, not the report itself

### FLS and Report Field Visibility
- If user lacks FLS `readable=true` on a field → that **column shows blank** in the report (no error, no warning)
- The report runs successfully; the field simply displays empty for all rows
- This is silent and can cause confusion — always verify FLS when a report column is unexpectedly empty

### Encrypted Fields in Reports
- Encrypted field values display normally to users with FLS `readable=true`
- **Filter behavior**: can filter on encrypted field using `=` (equals) only
  - CONTAINS, STARTS WITH, ENDS WITH, NOT EQUAL TO (string partial) → not supported → runtime error or no results
  - `!=` (not equals, exact) → supported
- If report filter uses CONTAINS on an encrypted field → report shows 0 results or runtime error
- Cannot sort (ORDER BY) by an encrypted field in a summary/grouped report

### Cascade: Deleting a Report Type
- If a Custom Report Type is deleted → all reports built on that CRT are deleted with it (no warning)
- Before deleting a CRT: identify all dependent reports via Setup → Report Types → [CRT] → Used By
- Reports that are deleted cannot be recovered (no recycle bin for reports)

### Cascade: Deleting or Renaming an Object Field
- If a field referenced in a CRT is deleted → that field disappears from the CRT silently; reports referencing it show blank column
- If a field is renamed (API name change) → report column reference breaks; must rebuild the report column
- Always audit dependent Reports before deleting or renaming fields
