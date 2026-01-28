# E2E Test: Random Query Generator Feature

Test the random query generator button that generates interesting natural language queries based on the database schema.

## Test Steps

1. **Navigate to application**
   - Open the application in the browser
   - Verify page loads successfully

2. **Take initial screenshot**
   - Capture the initial state of the application
   - Screenshot: `initial-state.png`

3. **Upload sample data**
   - Click the "Upload Data" button
   - Select and upload `users.json` sample data
   - Wait for upload to complete
   - Verify tables section shows the uploaded table

4. **Verify "Generate Random Query" button is present**
   - **VERIFY**: Button with text "Generate Random Query" is visible
   - **VERIFY**: Button is positioned next to "Upload Data" button in query-controls section

5. **Click "Generate Random Query" button**
   - Click the "Generate Random Query" button
   - Wait for the API call to complete

6. **Take screenshot of loading state** (if visible)
   - Capture the button in loading state (if timing allows)
   - Screenshot: `loading-state.png` (optional)

7. **Verify query input field is populated**
   - **VERIFY**: The `#query-input` textarea contains generated text
   - **VERIFY**: The generated query is not empty
   - **VERIFY**: The generated query mentions table names or column names from the schema (e.g., "users", "name", "email", etc.)

8. **Take screenshot of populated query**
   - Capture the state with the generated query in the input field
   - Screenshot: `generated-query-1.png`

9. **Click "Query" button to execute**
   - Click the "Query" button to execute the generated query
   - Wait for results to appear

10. **Verify query executes successfully**
    - **VERIFY**: Results section is visible
    - **VERIFY**: SQL query is displayed
    - **VERIFY**: Results table contains data or shows appropriate message
    - **VERIFY**: No error messages are displayed

11. **Take screenshot of results**
    - Capture the query results
    - Screenshot: `query-results.png`

12. **Click "Generate Random Query" again**
    - Click the "Generate Random Query" button a second time
    - Wait for the API call to complete

13. **Verify input field content changes to new query**
    - **VERIFY**: The query input field now contains different text than before
    - **VERIFY**: The new query is also relevant to the database schema

14. **Take screenshot of second generated query**
    - Capture the state with the second generated query
    - Screenshot: `generated-query-2.png`

15. **Test edge case: No tables uploaded**
    - Remove all tables from the database (if possible via UI)
    - Click "Generate Random Query" button
    - **VERIFY**: An error message is displayed indicating no tables are available
    - Screenshot: `error-no-tables.png` (optional)

## Success Criteria

- All VERIFY assertions pass
- "Generate Random Query" button is visible and styled correctly
- Generated queries are populated in the input field
- Generated queries reference actual tables and columns from the schema
- Generated queries are executable (produce valid SQL)
- Multiple clicks generate varied queries
- At least 4 screenshots are captured successfully
- No unexpected errors occur during the test

## Expected Behavior

1. Button should be visible and clickable
2. Clicking generates a contextually relevant natural language query
3. Query should be limited to 2 sentences maximum
4. Query should reference actual table/column names
5. Generated query should populate and override the input field
6. Multiple clicks should produce different queries
7. Error handling works when no tables exist

## Notes

- The generated queries will vary due to the LLM's creative nature
- Focus on verifying that queries are relevant rather than specific content
- Ensure queries can be successfully executed (valid SQL generation)
