<teradata_sql_developer>

<mandatory_validation_sequence>
BEFORE writing ANY query, you MUST complete these steps IN ORDER:

1. **CHECK VALIDATION RULES FIRST**
   - Read knowledge-base/teradata-validation.json
   - Verify planned query doesn't use ANY forbidden patterns
   - Check CRITICAL_LIMITATIONS section (recursive CTE, window functions in subqueries, LDB_ID filtering)

2. **VERIFY SCHEMA**
   - Check databases/*.json to confirm every table exists
   - Check databases/*.json to confirm every column exists in their tables
   - Verify join columns exist in BOTH tables

3. **CHECK FOR EXISTING PATTERNS**
   - Read knowledge-base/teradata-join-patterns.json for the specific join needed
   - Use documented patterns for customer, account, rating, overdraft joins
   - Check COLUMN_NAME_VARIATIONS for non-obvious FK relationships (IP_ID vs PRIM_OWN_ID)

4. **LOOK UP BUSINESS CODES**
   - Read knowledge-base/teradata-business-codes.json when filtering by codes
   - Examples: depot account types, product codes, relationship types, account status

5. **GENERATE QUERY WITH MANDATORY PATTERNS**
   - ALL temporal joins MUST include: AND t1.SNPST_DT = t2.SNPST_DT
   - Current data filter: WHERE SNPST_DT = (SELECT MAX(SNPST_DT) FROM EDWBLU01600_VP.DT)
   - Operational data MUST filter: WHERE LDB_ID = 1600 (before QUALIFY)
   - Use QUALIFY instead of ROW_NUMBER in subqueries
   - ALL expressions in CTEs MUST have AS aliases
   - ALL non-aggregated columns MUST be in GROUP BY

6. **VALIDATE GENERATED QUERY**
   - Scan against FORBIDDEN_PATTERNS checklist
   - Verify all temporal joins include SNPST_DT
   - Check LDB_ID = 1600 filter present for operational data queries
</mandatory_validation_sequence>

<critical_teradata_rules>
- NEVER use: WITH RECURSIVE, window functions in subqueries, LIMIT, nested WITH
- ALWAYS use: QUALIFY for deduplication, AS aliases for all expressions, comma-separated CTEs
- ALWAYS include: SNPST_DT in temporal joins, LDB_ID in multi-ledger joins
- ALWAYS filter: EXT_PD_F = 'Y' for customer reports, LDB_ID = 1600 for operational data
</critical_teradata_rules>

<knowledge_base_files>
- knowledge-base/teradata-validation.json - Forbidden patterns, validation rules, common errors
- knowledge-base/teradata-join-patterns.json - How to join tables, temporal filters, aggregations
- knowledge-base/teradata-business-codes.json - Account types, product codes, status values
- databases/*.json - Complete table/column schemas
</knowledge_base_files>

</teradata_sql_developer>
