# Changelog

All notable changes to the Lizzy SMS Workflows project documentation.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

-----

## [Unreleased]

Nothing currently in development.

-----

## [1.3.0] - 2025-11-18

### Changed
- **In-service equipment menu simplified** - Reduced from 7 options to 5 options for cleaner UX
- **Main Menu re-sequenced** - Order Parts moved to position #2 for prioritization
- **Service Status workflow** - Now has single entry point (Main Menu only)
- **Key Principles simplified** - Removed obsolete four-action API framework
- **Order Parts menu message** - Clarified purpose: "Select Equipment to see list of common parts"
- **Service Status label** - Changed from "Service Status" to "Check Service Status"

### Added
- **SKU field to equipment lists** - Added to 6 equipment list displays (format: "SKU: ###")
- **BACK navigation** - Added to Filter by Unit Type menu (returns to previous menu)
- **BACK navigation** - Added to Filter by Model menu in 3 workflows (Troubleshoot, Service, Parts)

### Removed
- **Schedule Service option** - Removed from in-service equipment menu (redundant)
- **Check Service Status option** - Removed from in-service equipment menu (redundant)
- **Workflow transition** - Removed Troubleshoot Equipment → Service Status direct path
- **Four-action framework** - Removed from Key Principles (obsolete with consolidated API)

### Technical
- Updated all routing tables to reflect new option numbers
- Updated 13 message templates with new text
- Updated 6 equipment display nodes with SKU field
- Simplified Key Principles from 6 to 5 bullet points

-----

## [1.2.0] - 2025-11-13

### Added
- **SKU search capability** - Customers can select equipment by SKU
- **Serial Number search capability** - Customers can select equipment by Serial Number
- **Two-layer filtering** - Filter by Unit Type, then by Model
- **Equipment resources display** - Tutorial videos and owner's manuals
- **Consolidated API approach** - Single customer profile endpoint with includes
- **Session caching strategy** - DynamoDB for 15-minute customer data cache
- **Cross-workflow equipment context** - Equipment selection carries between workflows

### Changed
- **API strategy** - Consolidated from 16 potential APIs to 7-9 endpoints
- **Troubleshoot Equipment workflow** - Added 4 equipment selection methods
- **Schedule Service workflow** - Added 4 equipment selection methods
- **Order Parts workflow** - Added 4 equipment selection methods
- **Equipment display format** - Now shows Make, Model, SN, and Status

### Technical
- Defined Redis → DynamoDB migration path
- Specified caching TTLs (customer: 15min, transactions: 5min)
- Documented all 24 Lizzy API endpoints with request/response formats
- Created API consolidation strategy reducing calls by 60-75%

-----

## [1.1.0] - 2025-11-10

### Added
- **Initial complete workflow specifications** for 7 workflows:
  1. Main Menu
  2. Troubleshoot Equipment
  3. Schedule Service
  4. Service Status
  5. Order Parts
  6. Parts Order Status
  7. View or Pay Invoice
- **System architecture** - Components, message flow, caching strategy
- **Session management** - 15-minute timeout, state persistence
- **Dealer handoff** - Text Shop and Call Shop options
- **Error handling** - Invalid input, timeouts, API failures
- **Message templates** - All customer-facing messages defined

### Technical
- Node.js/Express workflow engine specification
- Twilio SMS integration requirements
- PostgreSQL session storage design
- Redis caching layer design (later changed to DynamoDB)

-----

## Versioning Strategy

### Version Number Format: MAJOR.MINOR.PATCH

**MAJOR version** (1.x.x):
- Breaking changes to workflow structure
- Complete redesign or fundamental approach changes
- Incompatible with previous implementations

**MINOR version** (x.3.x):
- New features or workflows added
- Significant UX improvements
- New API endpoints required
- Backward compatible changes

**PATCH version** (x.x.1):
- Bug fixes
- Typo corrections
- Documentation clarifications
- No functional changes

-----

## Document Versions by Type

### Requirements Document
- **Current:** v1.3.0
- **Previous:** v1.2.0, v1.1.0
- **File:** `Lizzy_Agentic_Commerce_CRM_Requirements_v1.3.0.md`

### API Requirements Document
- **Current:** v1.0.0 (first stable release)
- **File:** `Lizzy_API_Requirements_Document.md`

### API Field Mapping Document
- **Current:** v1.0.0 (first stable release)
- **File:** `Lizzy_API_Field_Mapping_Tables.md`

### Implementation Guides
- **MVP Guide:** v1.2 (matches requirements v1.2)
- **Flow Diagrams:** v1.2 (needs update to v1.3)

-----

## Links

- **Repository:** [GitHub - lizzy-sms-workflows](https://github.com/YOUR-USERNAME/lizzy-sms-workflows)
- **Issues:** [GitHub Issues](https://github.com/YOUR-USERNAME/lizzy-sms-workflows/issues)
- **Latest Release:** [v1.3.0](https://github.com/YOUR-USERNAME/lizzy-sms-workflows/releases/tag/v1.3.0)

-----

## Contributors

- **Jonathan** - Project Lead, Requirements, Documentation

-----

## Notes

### Shared with Stakeholders

**v1.2.0 (Nov 13):**
- Shared with Lizzy API Team
- Shared with Business Stakeholder

**v1.3.0 (Nov 18):**
- Ready to share as updated version
- Includes 8 improvements based on initial feedback

### Migration Notes

Moving from v1.2 to v1.3:
- No breaking API changes
- UX improvements only
- All existing API specs remain valid
- Twilio Studio flows need minor message updates
- Flow diagrams need update for new menu structure

-----

**For detailed technical changes, see individual document changelogs within each file.**
