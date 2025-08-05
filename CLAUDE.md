# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the FOCUS (FinOps Open Cost and Usage Specification) repository containing the official specification for standardizing cloud, SaaS, and billing data across providers. The repository generates the specification documents from Markdown source files using a preprocessing and build pipeline.

## Key Architecture

### Document Structure

- `specification/` - Main specification content and build system
- `supporting_content/` - Additional reference materials and examples
- `guidelines/` - Development processes and editorial guidelines
- `custom_linter_rules/` - Custom Markdown linting rules

### Build System

The specification uses a multi-stage build process:

1. **Preprocessing**: MarkdownPP processes `.mdpp` template files that include other Markdown files
2. **Validation**: Python scripts validate include consistency and conformance rules
3. **Generation**: Pandoc converts to HTML and PDF formats
4. **Linting**: pymarkdownlnt checks Markdown quality

### Key Files

- `specification/spec.mdpp` - Main specification template
- `specification/Makefile` - Build configuration and commands
- `specification/validate_includes.py` - Validates that .mdpp templates include all .md files in folders
- `specification/conformance/` - Conformance rules and validation scripts

## Common Development Commands

### Building the Specification

```bash
cd specification
make                    # Build all formats (spec.md, spec.html, spec.pdf)
make spec.md           # Build Markdown only
make spec.html         # Build HTML only  
make spec.pdf          # Build PDF only
make clean             # Remove generated files
```

### Environment Setup

```bash
pip3 install -r requirements.txt    # Install Python dependencies
brew install pandoc                 # Install Pandoc (macOS)
brew install --cask wkhtmltopdf     # Install PDF engine (macOS)
```

### Linting and Validation

```bash
cd specification
pymarkdownlnt --config markdownlnt.cfg scan *.md    # Lint Markdown files
./validate_includes.py <folder_name>                 # Validate includes consistency
```

### Conformance Rules Development

```bash
cd specification/conformance
python build_cr_json.py             # Build conformance rules JSON
python build_cr_tables.py           # Generate conformance tables
python build_dag.py                 # Build dependency graph
```

## Development Workflow

### Branch Strategy

- `working_draft` - Active development branch (default)
- `candidate_recommendation` - Release candidate for IPR review
- `main` - Published releases

### Content Organization

- Each specification section has a folder with individual `.md` files for components
- Folders must have matching `.mdpp` template files (e.g., `columns/columns.mdpp`)
- Template files use `!INCLUDE "filename.md"` to include content
- The `validate_includes.py` script ensures all `.md` files are included in templates

### Issue and PR Workflow

- All work must be tracked via GitHub Issues
- Use issue templates: Feature Request [FR], Work Item [WI], Action Item [AI], Feedback, Maintenance
- PRs require formal review and approval from Working Group members
- PRs must be linked to parent Issues and meet Definition of Done criteria

## Creating Conformance Rules for Columns and Attributes

Conformance Rules (CR) provide machine-readable validation requirements for FOCUS specification compliance. See `specification/conformance/README.md` for complete documentation.

### Rule Naming Convention

Format: `<EntityName>-<EntityType>-<SequenceId>-<Status>`

- `EntityName`: The entity being validated (e.g., BilledCost, CurrencyCodeFormat)
- `EntityType`: 
  - `C`: Column (structural data elements)
  - `A`: Attribute (semantic labels or metadata)
- `SequenceId`: Three-digit sequence (001, 002, etc.)
- `Status`: M (Mandatory), O (Optional), C (Conditional)

Examples: 
- Columns: `BilledCost-C-001-M`, `ListUnitPrice-C-003-O`
- Attributes: `CurrencyCodeFormat-A-001-M`, `DateTimeFormat-A-002-M`

### Rule Structure Components

Each rule JSON contains:

```json
{
  "RuleId": {
    "Function": "Presence|Validation|Type|Format|Composite",
    "Reference": "EntityName",
    "EntityType": "Column|Attribute", 
    "Notes": "Optional explanation",
    "CRVersionIntroduced": "1.2",
    "Status": "Active",
    "ApplicabilityCriteria": [],
    "Type": "Static|Dynamic",
    "ValidationCriteria": {
      "MustSatisfy": "Human-readable requirement",
      "Keyword": "MUST|SHOULD|MAY",
      "Requirement": { /* CheckFunction specification */ },
      "Condition": { /* Optional condition logic */ },
      "Dependencies": ["List of dependent rule IDs"]
    }
  }
}
```

### Common Rule Functions

Reference `specification/conformance/check_functions.json` for available functions:

- **Presence**: `ColumnPresent` - Column must exist in dataset
- **Validation**: `CheckValue`, `CheckNotValue` - Value-based checks
- **Type**: `TypeDecimal`, `TypeString` - Data type validation
- **Format**: `FormatNumeric`, `FormatDateTime` - Format compliance
- **Composite**: `AND`, `OR` - Combine multiple rules

### Step-by-Step Workflow

1. **Create rule file**: 
   - Columns: `specification/conformance/conformance_rules/columns/<columnname>.json`
   - Attributes: `specification/conformance/conformance_rules/attributes/<attributename>.json`

2. **Define composite rule** (rule 000):
   ```json
   {
     "EntityName-C-000-M": {
       "Function": "Composite",
       "EntityType": "Column",
       "ValidationCriteria": {
         "Requirement": {
           "CheckFunction": "AND",
           "Items": [
             {"CheckFunction": "CheckConformanceRule", "ConformanceRuleId": "EntityName-C-001-M"},
             {"CheckFunction": "CheckConformanceRule", "ConformanceRuleId": "EntityName-C-002-M"}
           ]
         }
       }
     }
   }
   ```

3. **Add individual rules** (001, 002, etc.):
   - **For Columns**: Presence, null validation, type validation, format validation, business logic
   - **For Attributes**: Format requirements, semantic validation, cross-entity constraints

4. **Build and validate**:
   ```bash
   cd specification/conformance
   python build_cr_json.py  # Validates syntax and builds cr-1.2.json
   ```

5. **Common patterns**:
   - **Column presence**: `{"CheckFunction": "ColumnPresent", "ColumnName": "ColumnName"}`
   - **Not null**: `{"CheckFunction": "CheckNotValue", "ColumnName": "ColumnName", "Value": null}`
   - **Type decimal**: `{"CheckFunction": "TypeDecimal", "ColumnName": "ColumnName"}`
   - **Numeric format**: `{"CheckFunction": "FormatNumeric", "ColumnName": "ColumnName"}`
   - **Attribute validation**: See `currencycodeformat.json` for attribute-specific patterns

### Files to Update

When adding new rules:

1. **Columns**: Create `specification/conformance/conformance_rules/columns/<columnname>.json`
2. **Attributes**: Create `specification/conformance/conformance_rules/attributes/<attributename>.json`
3. Rules are automatically included in build process
4. Run `python build_cr_json.py` to validate and build final CR document

### Validation

The build process validates:
- JSON syntax and schema compliance (against `cr_schema.json`)
- CheckFunction references exist in `check_functions.json`
- Rule ID format matches pattern
- Dependencies reference valid rules

## Specification Content Guidelines

### File Naming Conventions

- Use lowercase with underscores for folders: `supported_features/`
- Use lowercase for Markdown files: `billedcost.md`
- Template files match folder names: `columns/columns.mdpp`

### Markdown Conventions

- Follow the custom linting rules in `markdownlnt.cfg`
- Custom rule MD990 enforces FOCUS-specific formatting requirements
- Tables should be properly formatted for conversion from Google Sheets
- Use consistent terminology as defined in the glossary

### Version Management

- Specification uses semantic versioning
- Changes tracked in `CHANGELOG.md`
- Version-specific content in `specification/versions/`

## Python Tools and Scripts

The repository includes several Python utilities:

- `validate_includes.py` - Ensures template/content consistency
- `conformance/build_*.py` - Generate conformance validation artifacts
- `custom_linter_rules/rule_md_990.py` - Custom Markdown linting

All Python dependencies are managed via `requirements.txt` files at both repository and conformance levels.
