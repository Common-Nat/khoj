# High Priority TODO Fixes - Completed

**Date:** 2025-12-11
**Branch:** `claude/count-todos-012QGjYMQLDLpq3qMuVvwCtB`
**Commits:** `4ef5da9`, `1dd4b53`

## Summary

Fixed 4 high-priority TODOs identified in the codebase audit. All changes have been committed and pushed.

## TODO Count Analysis

**Total matches for "TODO":** 36
**Real TODOs in Khoj code:** 14

**False positives (22):**
- Minified libraries (org.min.js, markdown-it.min.js): 8 TODOs
- Test data (org-mode entries in test files): 3 TODOs
- Org-mode parser documentation (not actual TODOs): 11 TODOs

## Fixes Implemented

### 1. Excalidraw Diagram Validation ✅
**File:** `src/khoj/routers/helpers.py:953`
**Priority:** High (Security/Data Integrity)

**Changes:**
- Added comprehensive validation for Excalidraw diagram structure
- Validates that `elements` is a non-empty list
- Checks each element has required fields: `type`, `id`, `x`, `y`
- Type-checks field values: `type`/`id` must be strings, `x`/`y` must be numbers
- Provides detailed error messages for debugging

**Why it matters:** Prevents malformed or malicious diagram data from being processed.

---

### 2. Query Images & Files Support in Operator ✅
**Files:**
- `src/khoj/processor/operator/operator_agent_base.py:40-41`
- `src/khoj/processor/operator/operator_agent_binary.py:46-47`
- `src/khoj/processor/operator/__init__.py:42,44`

**Priority:** High (Missing Core Functionality)

**Changes:**
- Extended `OperatorAgent` base class to accept `query_images` and `query_files` parameters
- Integrated with `construct_structured_message()` to create multimodal initial messages
- Updated all operator agent instantiations:
  - `AnthropicOperatorAgent`
  - `OpenAIOperatorAgent`
  - `BinaryOperatorAgent`
- Parameters now properly flow from API layer through to agent initialization
- Images and files are included in the initial user message using proper content blocks

**Why it matters:** Enables operator agents to work with image and file inputs, completing the vision-enabled workflow.

---

### 3. Multiple Tool Calls Handling in Research ✅
**File:** `src/khoj/routers/research.py:212`
**Priority:** High (Feature Limitation)

**Changes:**
- Parse all tool calls from model response (not just the first one)
- Intelligently select first non-repeated tool call from multiple options
- Notify user when multiple tool requests are detected
- Improved duplicate detection to avoid repeated tool/query combinations
- Added logging for better observability

**Implementation approach:**
- Parses entire tool call array from model
- Checks each against previously used combinations
- Selects first non-duplicate tool call
- Falls back to first tool if all are duplicates
- Logs selection for debugging

**Why it matters:** Handles cases where LLMs return multiple tool calls, preventing silent failures and improving research quality.

---

### 4. Notion Database Processing ✅
**File:** `src/khoj/processor/content/notion/notion_to_entries.py:108`
**Priority:** High (Missing Feature)

**Changes:**
- Added `process_database()` method to handle Notion databases
- Queries database to retrieve all pages within it
- Handles pagination for databases with many entries
- Reuses existing `process_page()` logic for consistency
- Includes error handling and logging
- Databases are no longer skipped during Notion sync

**Implementation:**
```python
def process_database(self, database):
    """Process a Notion database by querying all its pages and processing each one."""
    database_id = database["id"]
    database_entries = []

    # Query database with pagination
    query_params = {"page_size": 100}
    while True:
        response = self.session.post(
            f"https://api.notion.com/v1/databases/{database_id}/query",
            json=query_params,
        ).json()

        # Process each page
        for page in response.get("results", []):
            if page.get("object") == "page":
                page_entries = self.process_page(page)
                database_entries.extend(page_entries)

        # Handle pagination
        if not response.get("has_more", False):
            break
        query_params["start_cursor"] = response["next_cursor"]

    return database_entries
```

**Why it matters:** Users can now sync and search content from Notion databases, not just individual pages.

---

## Medium Priority TODOs - Completed

### 5. Replace Sync Call with Async Version ✅
**File:** `src/khoj/routers/helpers.py`
**Priority:** Medium (Code Quality)

**Changes:**
- Converted `format_automation_response` to async `aformat_automation_response`
- Converted `should_notify` to async `ashould_notify`
- Updated `scheduled_chat` to use `asyncio.run()` for calling async functions
- Updated test to use async version

**Why it matters:** Eliminates need to maintain separate sync wrapper functions.

---

### 6. Get OS Info from Environment ✅
**Files:**
- `src/khoj/processor/operator/operator_environment_base.py`
- `src/khoj/processor/operator/operator_environment_computer.py`
- `src/khoj/processor/operator/operator_environment_browser.py`
- `src/khoj/processor/operator/operator_agent_openai.py`

**Priority:** Medium (Environment Accuracy)

**Changes:**
- Added `os` field to `EnvState` model
- `ComputerEnvironment` now detects OS using `platform.system()`
- `BrowserEnvironment` returns "browser" as OS
- `OpenAIOperatorAgent.get_tools()` uses OS from environment state

**Why it matters:** Correct OS detection for operator computer use tools.

---

### 7. Replace Hacky Provider Name Filtering ✅
**Files:**
- `src/khoj/database/models/__init__.py`
- `src/khoj/utils/initialization.py`
- `src/khoj/database/admin.py`

**Priority:** Medium (Code Quality)

**Changes:**
- Added `ApiType` enum to `AiModelApi` model with `STANDARD` and `OPENAI_COMPATIBLE` choices
- Added `api_type` field to `AiModelApi` model
- Updated initialization to filter by `api_type` instead of name
- Added `api_type` to admin display and filters

**Why it matters:** Properly identifies OpenAI-compatible APIs like Ollama for model listing.

---

### 8. Update Public Agent Filtering Logic ✅
**Files:**
- `src/khoj/database/models/__init__.py`
- `src/khoj/database/adapters/__init__.py`
- `src/khoj/database/admin.py`

**Priority:** Medium (Feature Enhancement)

**Changes:**
- Added `officially_approved` field to `Agent` model
- Updated `get_all_accessible_agents` to allow public agents that are either admin-managed OR officially approved
- Added field to admin display and filters

**Why it matters:** Enables approving community-created public agents for listing.

---

### 9. Only Show Available Chat Modes ✅
**File:** `src/interface/obsidian/src/chat_view.ts`
**Priority:** Medium (User Experience)

**Changes:**
- Added `input_tools` and `output_modes` to Agent interface
- Created `allChatModes` static constant with all possible modes
- Added `serverAvailableModes` set to track server-enabled modes
- Added `fetchChatOptions()` to get enabled modes from server
- Added `updateFilteredChatModes()` to filter based on server and agent capabilities
- Modes now dynamically filtered on view open and agent change

**Why it matters:** Only shows relevant chat modes to users based on server config and agent capabilities.

---

## Remaining TODOs (4 - Low Priority)

### Low Priority
1. `src/khoj/processor/operator/__init__.py:82` - Remove dead OpenAI operator code
2. `src/khoj/processor/operator/__init__.py:94` - Remove dead Binary operator code
3. `src/interface/web/app/components/excalidraw/excalidrawWrapper.tsx:149` - Create common theme detection function
4. `src/interface/web/app/components/chatHistory/chatHistory.tsx:357` - IntersectionObserver delay optimization

---

## Testing Recommendations

### For Async Automation Functions
- Run automated tasks (scheduled jobs)
- Verify notifications still work correctly
- Check that async calls don't block

### For OS Detection in Operator
- Run operator on Linux, Mac, and Windows hosts
- Verify correct OS is reported in tools
- Test browser environment returns "browser"

### For API Type Filtering
- Configure an Ollama instance with `api_type=openai_compatible`
- Verify models are automatically discovered
- Test standard providers still work

### For Public Agent Approval
- Create a user agent with `officially_approved=True`
- Verify it appears in public listings
- Test admin-managed agents still work

### For Chat Mode Filtering (Obsidian)
- Open Obsidian chat with operator disabled on server
- Verify operator mode doesn't appear
- Switch agents and verify modes update
- Test agent with restricted input_tools

### For Excalidraw Validation
- Test with valid diagrams
- Test with missing required fields
- Test with wrong field types
- Test with empty elements array

### For Operator Images/Files
- Test operator with image attachments
- Test operator with file attachments
- Test operator with both images and files
- Verify vision-enabled model is used

### For Multiple Tool Calls
- Monitor research mode when multiple tools are suggested
- Verify duplicate detection works correctly
- Check that user is notified appropriately

### For Notion Databases
- Sync workspace with databases
- Verify database pages are indexed
- Search for content within database pages
- Test with large databases (pagination)

---

## Related Documentation

- Operator documentation: [Agent SDK docs]
- Research mode: `src/khoj/routers/research.py`
- Notion integration: `src/khoj/processor/content/notion/`
- Excalidraw integration: `src/khoj/routers/helpers.py`

---

## Notes

- All changes follow existing code patterns and conventions
- Error handling added where appropriate
- Logging included for debugging and monitoring
- No breaking changes to existing APIs
- Changes are backward compatible
