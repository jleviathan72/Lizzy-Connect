# Lizzy SMS Workflows

> **SMS-based customer workflow system for equipment dealers using Lizzy ERP**

Complete documentation for an SMS workflow system that enables customers to troubleshoot equipment, schedule service, order parts, and manage invoices via text message.

**Version:** 1.3.0  
**Status:** Active Development  
**Last Updated:** November 18, 2025

-----

## 📋 Quick Links

- **[Requirements v1.3](docs/requirements/Lizzy_Agentic_Commerce_CRM_Requirements_v1.3.0.md)** - Complete workflow specifications
- **[API Requirements](docs/api/Lizzy_API_Requirements_Document.md)** - API integration specs for Lizzy team
- **[CHANGELOG](CHANGELOG.md)** - Version history and changes
- **[MVP Implementation Guide](docs/implementation/Twilio_Studio_MVP_Implementation_Guide.md)** - Build guide

-----

## 🎯 Project Overview

### What Is This?

An SMS-based workflow system that allows equipment dealers' customers to:

1. **Troubleshoot Equipment** - Find resources, videos, manuals
2. **Order Parts** - Browse common parts, place orders
3. **Schedule Service** - Book equipment service
4. **Check Status** - Track service and parts orders
5. **Manage Invoices** - View and pay invoices

### Why SMS?

- ✅ **Accessible** - Works on any phone, no app required
- ✅ **Fast** - Instant responses, no login needed
- ✅ **Convenient** - Use while in garage or on job site
- ✅ **Familiar** - Customers already text their dealers

### Technology Stack

**MVP (Current):**
- Twilio Studio (workflow engine)
- Twilio Functions (custom logic)
- Lizzy API (data source)

**Production (Future):**
- Node.js/Express (workflow engine)
- DynamoDB (session caching)
- PostgreSQL (session state)
- Lizzy API (data source)

-----

## 📁 Repository Structure

```
lizzy-sms-workflows/
├── README.md                          # This file
├── CHANGELOG.md                       # Version history
│
├── docs/
│   ├── requirements/                  # Requirements documentation
│   │   ├── Lizzy_Agentic_Commerce_CRM_Requirements_v1.3.0.md  # Current
│   │   ├── Lizzy_Agentic_Commerce_CRM_Requirements_v1.2.0.md  # Archived
│   │   └── Requirements_Update_Summary_Nov18.md               # Change log
│   │
│   ├── api/                          # API documentation
│   │   ├── Lizzy_API_Requirements_Document.md                 # API specs
│   │   └── Lizzy_API_Field_Mapping_Tables.md                  # Field mapping
│   │
│   ├── implementation/               # Implementation guides
│   │   ├── Twilio_Studio_MVP_Implementation_Guide.md          # MVP guide
│   │   ├── Twilio_Studio_Quick_Reference.md                   # Quick ref
│   │   └── GitHub_Setup_Guide.md                              # This guide
│   │
│   ├── diagrams/                     # Flow diagrams
│   │   ├── Lizzy_SMS_Complete_Flow_Diagrams.md                # Mermaid
│   │   ├── Flow_Diagrams_Interactive.html                     # HTML
│   │   └── Quick_Start_View_Diagrams.md                       # How to view
│   │
│   └── supporting/                   # Supporting docs
│       ├── Redis_to_DynamoDB_Migration_Summary.md
│       ├── Version_1.2_SKU_SN_Update_Summary.md
│       ├── Documentation_Index.md
│       ├── DELIVERY_SUMMARY.md
│       └── Field_Mapping_Addition_Summary.md
│
└── archive/                          # Archived versions
    └── (old versions as needed)
```

-----

## 📖 Documentation

### Core Documents

#### 1. [Requirements v1.3.0](docs/requirements/Lizzy_Agentic_Commerce_CRM_Requirements_v1.3.0.md) ⭐

**Complete workflow specifications** for production system.

**Contents:**
- 7 workflow specifications (step-by-step)
- System architecture (Node.js engine)
- API requirements (consolidated approach)
- Session management & caching strategy
- All message templates

**When to Use:** Understanding complete system design, planning custom engine

---

#### 2. [API Requirements](docs/api/Lizzy_API_Requirements_Document.md) ⭐

**API integration specifications** for Lizzy technical team.

**Contents:**
- 9 API endpoint specifications
- Request/response examples
- 48 questions for Lizzy team
- Error handling requirements
- Testing requirements

**When to Use:** API integration planning, Lizzy team handoff

---

#### 3. [API Field Mapping Tables](docs/api/Lizzy_API_Field_Mapping_Tables.md) ⭐

**Detailed field-by-field mapping** for Lizzy team to complete.

**Contents:**
- 35 mapping tables
- 183 fields to map
- 5-column format (our name → their DB)
- 78 specific questions

**When to Use:** Data model alignment, field name verification

---

#### 4. [MVP Implementation Guide](docs/implementation/Twilio_Studio_MVP_Implementation_Guide.md)

**Build guide** for Twilio Studio MVP (no-code approach).

**Contents:**
- All 7 workflows (widget-by-widget)
- 5 Twilio Functions (complete code)
- Cross-workflow context solution
- 6-week timeline
- Cost estimates (~$57/month)

**When to Use:** Building MVP, rapid prototyping

---

#### 5. [Flow Diagrams](docs/diagrams/Lizzy_SMS_Complete_Flow_Diagrams.md)

**Visual Mermaid diagrams** for all workflows.

**Contents:**
- System architecture diagram
- 7 detailed workflow diagrams
- Cross-workflow context diagram
- Widget mapping guide

**When to Use:** Visual understanding, building in Twilio Studio

**Viewing:** See [Quick Start Guide](docs/diagrams/Quick_Start_View_Diagrams.md) for viewing options

---

### Supporting Documents

- **[CHANGELOG.md](CHANGELOG.md)** - Complete version history
- **[Documentation Index](docs/supporting/Documentation_Index.md)** - Guide to all docs
- **[Quick Reference](docs/implementation/Twilio_Studio_Quick_Reference.md)** - Fast lookup
- **[GitHub Setup Guide](docs/implementation/GitHub_Setup_Guide.md)** - This repository setup

-----

## 🚀 Getting Started

### For New Team Members

1. **Read:** [Documentation Index](docs/supporting/Documentation_Index.md)
2. **Review:** [Requirements v1.3](docs/requirements/Lizzy_Agentic_Commerce_CRM_Requirements_v1.3.0.md) (pages 1-30)
3. **View:** [Flow Diagrams](docs/diagrams/Flow_Diagrams_Interactive.html) (download & open)
4. **Understand:** [CHANGELOG](CHANGELOG.md) for recent changes

### For Lizzy API Team

1. **Read:** [API Requirements Document](docs/api/Lizzy_API_Requirements_Document.md)
2. **Complete:** [Field Mapping Tables](docs/api/Lizzy_API_Field_Mapping_Tables.md)
3. **Return:** Completed tables within 1 week
4. **Schedule:** Technical kickoff call

### For Implementation

1. **Follow:** [MVP Implementation Guide](docs/implementation/Twilio_Studio_MVP_Implementation_Guide.md)
2. **Reference:** [Flow Diagrams](docs/diagrams/Lizzy_SMS_Complete_Flow_Diagrams.md) constantly
3. **Use:** [Quick Reference](docs/implementation/Twilio_Studio_Quick_Reference.md) while building

-----

## 📊 Project Status

### Current Version: 1.3.0

**Released:** November 18, 2025

**Status:** Requirements Complete, API Integration In Progress

**Latest Changes:**
- In-service equipment menu simplified
- Main Menu re-sequenced (Order Parts #2)
- SKU added to equipment lists
- BACK navigation added
- Key Principles simplified

### Milestones

- [x] Requirements v1.1 - Initial workflows defined
- [x] Requirements v1.2 - SKU/SN search, consolidated APIs
- [x] Requirements v1.3 - 8 UX improvements
- [ ] API Integration - Lizzy team field mapping
- [ ] MVP Development - Twilio Studio implementation (6 weeks)
- [ ] UAT - Test with 2-3 dealers
- [ ] Production Launch - Soft launch
- [ ] Custom Engine - Node.js implementation (future)

-----

## 🔄 Version History

| Version | Date | Status | Key Changes |
|---------|------|--------|-------------|
| **1.3.0** | 2025-11-18 | Current | 8 UX improvements, API consolidation |
| 1.2.0 | 2025-11-13 | Archived | SKU/SN search, two-layer filtering |
| 1.1.0 | 2025-11-10 | Archived | Initial complete specifications |

**See [CHANGELOG.md](CHANGELOG.md) for detailed version history.**

-----

## 👥 Team

### Project Lead
- **Jonathan** - Requirements, Documentation, Project Management

### Stakeholders
- **Lizzy API Team** - API integration
- **Business Stakeholder** - Requirements validation, business logic

### Contributors
- See [CHANGELOG.md](CHANGELOG.md) for contribution history

-----

## 🤝 Contributing

### Making Changes

1. **Clone repository** (if not already done)
2. **Create branch** for your changes
3. **Make changes** to relevant documents
4. **Update CHANGELOG.md** with your changes
5. **Commit** with clear message
6. **Push** and create pull request

### Commit Message Format

```
type(scope): brief description

- Detailed change 1
- Detailed change 2

Closes #issue-number
```

**Types:**
- `feat`: New feature or workflow
- `fix`: Bug fix or correction
- `docs`: Documentation updates
- `refactor`: Restructuring without functional changes
- `test`: Testing additions
- `chore`: Maintenance tasks

**Example:**
```
feat(workflows): Add BACK navigation to filter menus

- Added BACK option to Filter by Unit Type
- Added BACK option to Filter by Model (3 workflows)
- Updated routing tables

Improves UX by allowing customers to correct mistakes
```

### Version Bumping

**When updating documents:**

1. **Update version number** in document header
2. **Update CHANGELOG.md** with changes
3. **Update README.md** if structure changes
4. **Commit with version tag**

**Version increment rules:**
- **MAJOR (1.x.x):** Breaking changes
- **MINOR (x.3.x):** New features, significant changes
- **PATCH (x.x.1):** Bug fixes, clarifications

-----

## 📞 Contact & Support

### Questions?

- **Requirements:** Review [Documentation Index](docs/supporting/Documentation_Index.md)
- **API Integration:** See [API Requirements](docs/api/Lizzy_API_Requirements_Document.md)
- **Implementation:** Follow [MVP Guide](docs/implementation/Twilio_Studio_MVP_Implementation_Guide.md)

### Issues

[Open an issue](https://github.com/YOUR-USERNAME/lizzy-sms-workflows/issues) for:
- Bug reports
- Feature requests
- Documentation improvements
- Questions

-----

## 📄 License

**Internal Project** - Proprietary  
Copyright © 2025 [Your Company Name]

All rights reserved. This documentation is for internal use only.

-----

## 🎯 Success Metrics

### MVP Goals
- ✅ Complete requirements documentation
- ✅ API integration specifications
- ✅ Implementation guide ready
- ⏳ Lizzy API integration complete
- ⏳ MVP built in Twilio Studio
- ⏳ Tested with 2-3 dealers
- ⏳ Launch with initial dealers

### Production Goals (Future)
- Custom Node.js engine built
- DynamoDB caching implemented
- 1,000+ sessions/month
- 95%+ customer satisfaction
- <2 second response times

-----

## 📚 Additional Resources

### External Links
- [Twilio Studio Docs](https://www.twilio.com/docs/studio)
- [Twilio Functions Docs](https://www.twilio.com/docs/serverless/functions-assets)
- [Mermaid Diagram Syntax](https://mermaid.js.org/)

### Internal Links
- [Company Wiki](#) (if applicable)
- [Project Slack Channel](#) (if applicable)

-----

**Last Updated:** November 18, 2025  
**Maintained By:** Jonathan  
**Repository:** [lizzy-sms-workflows](https://github.com/YOUR-USERNAME/lizzy-sms-workflows)

---

**⭐ Star this repository if you find it useful!**
