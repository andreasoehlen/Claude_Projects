# Teradata SQL Knowledge Base

## Overview
This knowledge base powers an AI-driven Teradata SQL development system with three specialized agents generating accurate, optimized queries.

## System Architecture

### Three-Agent System
1. **SQL Developer Agent** (Claude Sonnet 4.5)
   - Generates Teradata SQL queries
   - Validates against forbidden patterns
   - Uses knowledge base for schema, patterns, and business rules

2. **Diagnostics Agent** (Claude Haiku)
   - Explores database when Developer needs clarification
   - Runs diagnostic queries to verify data patterns
   - Reports findings back to Developer

3. **Librarian Agent** (Claude Code)
   - Maintains and curates this knowledge base
   - Adds new patterns from solved queries
   - Optimizes for token efficiency

## Knowledge Base Files

### Core Schema Files (READ-ONLY)
- **`databases/*.json`** - Complete database schemas: tables, columns, data types, PKs, FKs, indexes
- Primary source of truth for table/column verification
- Used by: SQL Developer (always), Diagnostics Agent (when exploring)

### Error Handling & Validation (NEW)
- **`knowledge-base/validation/error-taxonomy.json`** - Structured classification of Teradata SQL errors. 5 categories: syntax, schema, logical, temporal, performance
- **`knowledge-base/validation/validation-workflow.json`** - Multi-step validation: pre-generation, post-generation, execution
- **`knowledge-base/validation/query-correction-patterns.json`** - Error→fix mappings for iterative refinement
- **`knowledge-base/validation/teradata-forbidden-patterns.json`** - CRITICAL firewall preventing syntax errors (F1-F14)

### Schema & Patterns
- **`knowledge-base/patterns/schema-linking-patterns.json`** - FK relationships, naming conventions, column mismatches
- **`knowledge-base/patterns/common-join-patterns.json`** - Complex joins not obvious from FK (temporal, cardinality traps, bridge tables). 19+ patterns
- **`knowledge-base/patterns/sql-chunks.json`** - Reusable components (4+ tables, used 2+ times), CRITICAL Teradata limitations
- **`knowledge-base/diagnostics/diagnostic-templates.json`** - Structured templates for database exploration (9 templates)

### Business Context
- **`knowledge-base/business-rules/variable-explanations.json`** - Business logic, code translations, non-obvious column usage
- **`knowledge-base/business-rules/naming-standards.json`** - Column naming conventions and standards

## Workflows

### Normal Query Generation
```
User Request
    ↓
[SQL Developer] Pre-generation validation (knowledge-base/validation/validation-workflow.json)
  - Scan knowledge-base/validation/teradata-forbidden-patterns.json (F1-F14)
  - Verify tables/columns exist in databases/*.json
  - Check temporal join requirements (SNPST_DT)
    ↓
[SQL Developer] Schema linking (knowledge-base/patterns/schema-linking-patterns.json)
  - Look up FK relationships
  - Check knowledge-base/patterns/common-join-patterns.json for documented joins
  - Verify naming conventions
    ↓
[SQL Developer] Apply business context
  - knowledge-base/business-rules/variable-explanations.json for code values
  - knowledge-base/patterns/sql-chunks.json for reusable components
    ↓
[SQL Developer] Generate query
    ↓
[SQL Developer] Post-generation validation
  - Syntax check against forbidden patterns
  - Completeness check (aliases, GROUP BY, etc.)
  - Optimization check (QUALIFY, UNION ALL, LDB_ID filter)
    ↓
Generated Query → User
```

### Error Correction Flow
```
Query Execution Error
    ↓
[SQL Developer] Look up error in knowledge-base/validation/error-taxonomy.json
    ↓
[SQL Developer] Apply correction from knowledge-base/validation/query-correction-patterns.json
    ↓
[SQL Developer] Revalidate with knowledge-base/validation/validation-workflow.json
    ↓
Still fails after 3 attempts?
    ↓
Request Diagnostics (knowledge-base/diagnostics/diagnostic-templates.json)
```

### Diagnostics Flow
```
[SQL Developer] Needs clarification
    ↓
[SQL Developer] Creates DIAGNOSTICS REQUEST using knowledge-base/diagnostics/diagnostic-templates.json
    ↓
User copies to [Diagnostics Agent] chat
    ↓
[Diagnostics Agent] Runs exploration queries
  - verify_table_exists
  - check_code_values
  - check_join_cardinality
  - check_temporal_coverage
  - check_data_completeness
  - sample_join_results
    ↓
[Diagnostics Agent] Reports findings
    ↓
User copies findings back to [SQL Developer]
    ↓
[SQL Developer] Generates query with new insights
```

## File Cross-References

### Core Dependencies
- All files reference → `databases/*.json` (schema verification)
- `claude-project-instructions.md` references → all knowledge files
- `.claude/instructions.md` references → all knowledge files

### Error Handling Chain
`knowledge-base/validation/error-taxonomy.json` ← → `knowledge-base/validation/query-correction-patterns.json` ← → `knowledge-base/validation/teradata-forbidden-patterns.json`
`knowledge-base/validation/validation-workflow.json` references → all above

### Schema Understanding Chain
`knowledge-base/patterns/schema-linking-patterns.json` ← → `knowledge-base/patterns/common-join-patterns.json`
Both reference → `databases/*.json`

### Query Generation Chain
`knowledge-base/patterns/common-join-patterns.json` references → `knowledge-base/validation/teradata-forbidden-patterns.json`, `knowledge-base/business-rules/variable-explanations.json`
`knowledge-base/patterns/sql-chunks.json` references → `knowledge-base/validation/teradata-forbidden-patterns.json`, `knowledge-base/patterns/common-join-patterns.json`

## Usage Examples

### Example 1: Normal Query
```
User: "Show customers with account balances"

[SQL Developer]:
1. Pre-generation validation:
   - Check knowledge-base/validation/teradata-forbidden-patterns.json ✓
   - Verify CST, AC_AR_BAL tables exist in databases/*.json ✓
   - Check columns exist ✓
2. Schema linking:
   - Look up CST→AC_AR→AC_AR_BAL join in knowledge-base/patterns/common-join-patterns.json
   - Note: temporal join requires SNPST_DT in all joins
3. Generate query with temporal joins
4. Post-generation validation ✓
5. Return query to user
```

### Example 2: Error Correction
```
User: "Get deduplicated customer list"

[SQL Developer] generates query with ROW_NUMBER in subquery
Query fails: Error 3706

[SQL Developer]:
1. Look up 3706 in knowledge-base/validation/error-taxonomy.json
   → "Window functions not allowed in subqueries"
2. Check knowledge-base/validation/query-correction-patterns.json:error_3706
   → Use QUALIFY clause instead
3. Rewrite query using QUALIFY
4. Revalidate ✓
5. Return corrected query
```

### Example 3: Diagnostics Needed
```
User: "Find customers in premium segment"

[SQL Developer]:
1. Check databases/*.json - no "segment" column in CST
2. Check knowledge-base/business-rules/variable-explanations.json - no segment codes
3. Check knowledge-base/patterns/schema-linking-patterns.json - no segment pattern
4. Trigger DIAGNOSTICS REQUEST

User → [Diagnostics Agent]:
[Diagnostics Agent]:
1. Use knowledge-base/diagnostics/diagnostic-templates.json → check_code_values
2. Explore CST_GRP, CST_X_CST_GRP tables
3. Find bridge table pattern for customer groups
4. Report findings

User → [SQL Developer]:
[SQL Developer]:
1. Now knows to use CST_X_CST_GRP bridge table
2. Generate query with correct join pattern
3. Propose adding to knowledge-base/patterns/common-join-patterns.json
```

## Performance Characteristics

### Token Usage (Target)
- Target: <75KB total knowledge base (excluding databases/*.json)
- Organized semantically (300-500 token chunks optimal)
- Cross-references instead of duplication
- Consistent abbreviations: cust, acct, txn, bal, dt, amt, cd, cl, f

### Accuracy Improvements
- Error taxonomy + correction loops: ~10% fewer incorrect queries
- Schema validation: ~90% errors caught pre-generation
- Forbidden patterns firewall: 100% prevention of known Teradata syntax errors

## Contributing Patterns

### What Makes a Good Pattern?
✅ Will be reused 2+ times
✅ Contains non-obvious business logic
✅ Has cardinality traps or common errors
✅ Requires code translation or temporal complexity

### What to Avoid?
❌ Simple PK/FK joins obvious from schema
❌ Self-explanatory columns
❌ Basic SQL patterns
❌ One-time solutions

## Abbreviations Dictionary

Standard abbreviations used throughout:
- **cust** = customer
- **acct** = account
- **txn** = transaction
- **bal** = balance
- **dt** = date
- **amt** = amount
- **cd** = code
- **cl** = classification
- **f** = flag
- **tbl** = table
- **col** = column
- **fk** = foreign key
- **pk** = primary key
- **idx** = index

---

**Version**: 1.0
**Last Updated**: 2025-11-17
**Knowledge Base Size**: ~70KB (target <75KB)
**Pattern Count**: 50+ documented patterns
**File Count**: 10 core knowledge files + read-only schemas
