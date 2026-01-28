# Feature: Random Natural Language Query Generator

## Feature Description
This feature adds a "Generate Random Query" button that leverages the LLM to create interesting, context-aware natural language queries based on the current database schema and table structures. When clicked, the button generates a query and automatically populates (overrides) the query input field, allowing users to manually execute it. This feature helps users discover what questions they can ask about their data and serves as an educational tool for understanding the query capabilities of the application.

## User Story
As a user
I want to click a button that generates random natural language queries based on my uploaded tables
So that I can discover interesting questions to ask about my data and learn what types of queries are possible

## Problem Statement
Users may not know what questions to ask about their data or may struggle to formulate natural language queries. Without examples or suggestions, users might underutilize the natural language to SQL conversion capabilities. A random query generator solves this by:
1. Providing concrete examples of valid queries
2. Helping users discover insights in their data they hadn't considered
3. Demonstrating the breadth of query capabilities
4. Serving as an educational tool for new users

## Solution Statement
We will add a new button labeled "Generate Random Query" positioned next to the existing "Upload Data" button using the same secondary button styling. When clicked, this button will:
1. Retrieve the current database schema (tables and columns)
2. Use the existing LLM processor to generate an interesting natural language query (limited to 2 sentences maximum)
3. Populate the query input field with the generated query, overriding any existing content
4. Allow users to manually click the "Query" button to execute the generated query

The implementation leverages existing infrastructure (llm_processor.py, schema retrieval) and integrates seamlessly into the current UI without requiring framework changes.

## Relevant Files
Use these files to implement the feature:

- **app/server/server.py** (lines 1-280) - Add new API endpoint `/api/generate-random-query` that will handle query generation requests
- **app/server/core/llm_processor.py** (lines 1-162) - Add new function `generate_random_query(schema_info)` that creates interesting queries based on database schema
- **app/server/core/data_models.py** (lines 1-82) - Add `RandomQueryRequest` and `RandomQueryResponse` models for the new endpoint
- **app/server/core/sql_processor.py** - Reference for understanding how schema information is structured and retrieved
- **app/client/src/main.ts** (lines 1-423) - Add event handler for new button and populate query input field
- **app/client/src/api/client.ts** (lines 1-79) - Add `generateRandomQuery()` method to API client
- **app/client/index.html** (lines 1-99) - Add new "Generate Random Query" button in the query-controls section
- **app/client/src/style.css** - Reference for consistent button styling (secondary-button class already exists)
- **app/client/src/types.d.ts** - Add TypeScript type definitions for `RandomQueryRequest` and `RandomQueryResponse`

### New Files
- **.claude/commands/e2e/test_random_query_generator.md** - E2E test specification for validating the random query generator feature

## Implementation Plan

### Phase 1: Foundation
First, we'll establish the data models and backend infrastructure needed to support random query generation. This includes:
- Creating Pydantic models for request/response validation
- Adding the core LLM logic to generate contextual queries based on schema
- Implementing the FastAPI endpoint to serve random queries

### Phase 2: Core Implementation
Next, we'll implement the frontend integration:
- Adding the UI button with proper styling and placement
- Connecting the button to the backend API
- Implementing the query input population logic
- Ensuring proper error handling and loading states

### Phase 3: Integration
Finally, we'll validate the complete feature works end-to-end:
- Creating comprehensive E2E tests
- Testing with various database schemas (empty, single table, multiple tables)
- Validating the generated queries are interesting and executable
- Running full regression tests to ensure no existing functionality is broken

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Create Backend Data Models
- Open `app/server/core/data_models.py`
- Add `RandomQueryRequest` model (empty request, no parameters needed)
- Add `RandomQueryResponse` model with fields:
  - `query: str` - The generated natural language query
  - `error: Optional[str]` - Error message if generation fails

### Step 2: Implement Random Query Generation Logic
- Open `app/server/core/llm_processor.py`
- Add new function `generate_random_query(schema_info: Dict[str, Any]) -> str` that:
  - Takes database schema as input
  - Formats schema information for the LLM prompt
  - Creates a prompt asking the LLM to generate an interesting query (max 2 sentences)
  - Uses the existing routing logic (OpenAI or Anthropic based on available API keys)
  - Returns the generated natural language query
  - Includes variety in query types (aggregations, filters, joins, temporal queries, etc.)
  - Ensures queries are relevant to the actual tables and columns present

### Step 3: Create Backend API Endpoint
- Open `app/server/server.py`
- Add new endpoint `@app.post("/api/generate-random-query", response_model=RandomQueryResponse)`
- Implement endpoint logic:
  - Call `get_database_schema()` to retrieve current schema
  - Check if schema has any tables (return error if empty)
  - Call `generate_random_query(schema_info)` from llm_processor
  - Return `RandomQueryResponse` with generated query
  - Include proper error handling and logging

### Step 4: Add TypeScript Type Definitions
- Open `app/client/src/types.d.ts`
- Add `RandomQueryRequest` interface (empty object)
- Add `RandomQueryResponse` interface matching the Pydantic model

### Step 5: Update API Client
- Open `app/client/src/api/client.ts`
- Add `generateRandomQuery()` method to the `api` object:
  - Make POST request to `/generate-random-query`
  - Return `Promise<RandomQueryResponse>`
  - Use proper error handling

### Step 6: Add UI Button
- Open `app/client/index.html`
- In the `.query-controls` div (line 22), add new button after the "Upload Data" button:
  - ID: `random-query-button`
  - Class: `secondary-button`
  - Text: "Generate Random Query"

### Step 7: Implement Frontend Logic
- Open `app/client/src/main.ts`
- Add `initializeRandomQueryButton()` function that:
  - Gets reference to `#random-query-button`
  - Adds click event listener
  - Shows loading state while generating query
  - Calls `api.generateRandomQuery()`
  - Populates `#query-input` textarea with generated query (overriding existing content)
  - Handles errors gracefully with `displayError()`
  - Resets button state after completion
- Call `initializeRandomQueryButton()` from the DOMContentLoaded event listener

### Step 8: Manual Testing
- Start the server and client
- Upload sample data (users, products, events)
- Click "Generate Random Query" button multiple times
- Verify generated queries:
  - Are relevant to uploaded tables
  - Vary in complexity and type
  - Are limited to 2 sentences max
  - Populate the input field correctly
  - Can be executed successfully
- Test edge cases:
  - Click button with no tables uploaded (should show error)
  - Click button while a query is already in the input field (should override)
  - Click button multiple times rapidly (should handle gracefully)

### Step 9: Create E2E Test Specification
- Create new file `.claude/commands/e2e/test_random_query_generator.md`
- Define test steps to validate:
  1. Navigate to application
  2. Take screenshot of initial state
  3. Upload sample data (users.json)
  4. Verify "Generate Random Query" button is present
  5. Click "Generate Random Query" button
  6. Take screenshot of loading state
  7. **Verify** query input field is populated with text
  8. **Verify** generated query mentions existing tables/columns
  9. Take screenshot of populated query
  10. Click "Query" button to execute
  11. **Verify** query executes successfully with results
  12. Take screenshot of results
  13. Click "Generate Random Query" again
  14. **Verify** input field content changes to new query
  15. Take screenshot of second generated query
- Define success criteria:
  - Button is visible and clickable
  - Query is generated and populates input field
  - Generated query is relevant to database schema
  - Generated query is executable
  - Multiple clicks generate different queries
  - 5 screenshots are taken

### Step 10: Run Validation Commands
- Execute all validation commands listed below to ensure zero regressions
- Fix any issues that arise
- Verify the E2E test passes completely

## Testing Strategy

### Unit Tests
- **Backend Tests** (add to `app/server/tests/`):
  - Test `generate_random_query()` with various schema configurations:
    - Single table with few columns
    - Multiple tables with relationships
    - Tables with different data types (text, numeric, dates)
    - Empty schema (should handle gracefully)
  - Test `/api/generate-random-query` endpoint:
    - Success case with valid schema
    - Error case with no tables
    - Error case when LLM API fails
  - Verify response format matches `RandomQueryResponse` model
  - Verify generated queries are limited to 2 sentences

- **Frontend Tests** (manual verification):
  - Button renders correctly with secondary-button styling
  - Button is positioned next to "Upload Data" button
  - Click handler populates input field
  - Loading state displays during API call
  - Error messages display appropriately

### Edge Cases
1. **No tables uploaded**: Should display friendly error message
2. **API key missing**: Should return error from backend
3. **LLM returns invalid format**: Should handle gracefully with fallback error
4. **Network failure**: Should show error and allow retry
5. **Empty query input vs existing query**: Both should be overridden correctly
6. **Multiple rapid clicks**: Should debounce or queue requests appropriately
7. **Very large schema (many tables)**: Should still generate relevant query within limits
8. **Tables with no rows**: Should generate valid query even if tables are empty

## Acceptance Criteria
1. ✅ A "Generate Random Query" button is visible next to the "Upload Data" button
2. ✅ Button uses the same styling as "Upload Data" button (secondary-button class)
3. ✅ Clicking the button generates a natural language query based on current database schema
4. ✅ Generated queries are limited to maximum 2 sentences
5. ✅ Generated query automatically populates (overrides) the query input field
6. ✅ Generated queries are contextually relevant to existing tables and columns
7. ✅ Generated queries are executable (produce valid SQL when processed)
8. ✅ Multiple clicks produce different/varied queries
9. ✅ Button shows loading state during query generation
10. ✅ Appropriate error message shown when no tables exist
11. ✅ Feature works with both OpenAI and Anthropic LLM providers
12. ✅ All existing tests pass with zero regressions
13. ✅ E2E test validates complete user flow

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

1. Read `.claude/commands/test_e2e.md` for E2E testing framework understanding
2. Execute the E2E test: Read and execute `.claude/commands/e2e/test_random_query_generator.md` following the test_e2e.md instructions to validate the random query generator feature works end-to-end
3. `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
4. `cd app/client && bun run tsc --noEmit` - Run TypeScript type checking to validate no type errors
5. `cd app/client && bun run build` - Run frontend build to validate the feature builds successfully

## Notes

### LLM Prompt Engineering
The prompt for generating random queries should encourage variety and relevance:
- Include instructions to vary query complexity (simple filters, aggregations, multi-table joins)
- Ask for queries that demonstrate different SQL capabilities
- Emphasize using actual table and column names from the schema
- Request creative but realistic queries users might actually want to run
- Limit to 2 sentences maximum for clarity and brevity

### Future Enhancements (Out of Scope)
The following enhancements could be considered in future iterations but are NOT part of this implementation:
- Query difficulty selector (simple/medium/complex)
- Query history to avoid repeating recent generations
- Favorite/save generated queries
- Query categories (analytical, operational, exploratory)
- Multiple query suggestions at once
- Auto-execute generated query option

### Dependencies
No new packages required. This feature uses existing dependencies:
- FastAPI (already installed)
- OpenAI SDK (already installed)
- Anthropic SDK (already installed)
- Pydantic (already installed)

### Security Considerations
- Generated queries go through the same security validation as user-entered queries
- No direct SQL execution without going through `execute_sql_safely()`
- Schema information is already sanitized through existing security module
- No user input required for generation, reducing injection attack surface
