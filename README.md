openai
sentence-transformers
chromadb
python-dotenv

requirements.txt

search_knowledge.py
-----------------------------
import hashlib
import chromadb

chroma_client = chromadb.PersistentClient(path="./chroma_db")
collection = chroma_client.get_collection(name="learner_knowledge")

def get_fake_embedding(text, dimension=384):
    vector = []
    for i in range(dimension):
        hash_value = hashlib.md5((text + str(i)).encode()).hexdigest()
        number = int(hash_value[:8], 16)
        vector.append(number / 0xFFFFFFFF)
    return vector

query = "Which ETF method should I use to click a button?"

results = collection.query(
    query_embeddings=[get_fake_embedding(query)],
    n_results=5
)

for i, doc in enumerate(results["documents"][0], start=1):
    print(f"\nResult {i}:")
    print(doc)
    print("Metadata:", results["metadatas"][0][i - 1])


    -----------------------------

    cd..
cd .\WorkSpaces\
mkdir learner-agent-poc
cd .\learner-agent-poc\
cls
New-Item requirements.txt -ItemType File
python -m venv venv
(Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned) ; (& c:\WorkSpaces\learner-agent-poc\venv\Scripts\Activate.ps1)
venv\Scripts\activate
pip install -r requirements.txt
python -m pip install --upgrade pip certifi
python -m pip install -r requirements.txt --trusted-host pypi.org --trusted-host files.pythonhosted.org
python.exe -m pip install --upgrade pip
python -m pip config set global.trusted-host "pypi.org files.pythonhosted.org"
python.exe -m pip install --upgrade pip
pip list
pip show openai
pip show chromadb
New-Item .env -ItemType File
New-Item learner_agent.py -ItemType File
python learner_agent.py
pip install sentence-transformers
pip install chromadb sentence-transformers
python learner_agent.py
pip install chromadb python-dotenv
python learner_agent.py
(Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned) ; (& c:\WorkSpaces\learner-agent-poc\venv\Scripts\Activate.ps1)
python learner_agent.py
mkdir knowledge
cd .\knowledge\
dir
dor
dir
cd..
New-Item ingest_knowledge.py -ItemType File
python ingest_knowledge.py
New-Item search_knowledge.py -ItemType File
python search_knowledge.py


---------------------------------
learner_agent.py

import hashlib
import chromadb

chroma_client = chromadb.PersistentClient(path="./chroma_db")

collection = chroma_client.get_or_create_collection(
    name="learner_knowledge"
)

def get_fake_embedding(text, dimension=384):
    vector = []
    for i in range(dimension):
        hash_value = hashlib.md5((text + str(i)).encode()).hexdigest()
        number = int(hash_value[:8], 16)
        vector.append(number / 0xFFFFFFFF)
    return vector

def store_knowledge(doc_id, content, metadata):
    collection.upsert(
        ids=[doc_id],
        documents=[content],
        embeddings=[get_fake_embedding(content)],
        metadatas=[metadata]
    )

def search_knowledge(query, top_k=3):
    return collection.query(
        query_embeddings=[get_fake_embedding(query)],
        n_results=top_k
    )

if __name__ == "__main__":
    store_knowledge(
        "method_click_element",
        "Framework Method: clickElement(locator). Use this method to click buttons and links.",
        {"type": "framework_method", "module": "common"}
    )

    store_knowledge(
        "method_enter_text",
        "Framework Method: enterText(locator, value). Use this method to enter username, password, or form data.",
        {"type": "framework_method", "module": "common"}
    )

    store_knowledge(
        "app_flow_login",
        "Application Flow: Login. User enters username and password, clicks login, and lands on dashboard.",
        {"type": "application_flow", "module": "login"}
    )

    query = "How to automate login functionality?"
    results = search_knowledge(query)

    print("Search Query:", query)
    print("\nRetrieved Knowledge:")

    for i, doc in enumerate(results["documents"][0], start=1):
        print(f"\nResult {i}:")
        print(doc)
        print("Metadata:", results["metadatas"][0][i - 1])


        --------------------------


        ingest_knowledge.py


        import json
import hashlib
import os
import chromadb

file_path = os.path.join("knowledge", "ETF_Framework_v1.json")

print("File exists:", os.path.exists(file_path))

chroma_client = chromadb.PersistentClient(path="./chroma_db")
collection = chroma_client.get_or_create_collection(name="learner_knowledge")


def get_fake_embedding(text, dimension=384):
    vector = []
    for i in range(dimension):
        hash_value = hashlib.md5((text + str(i)).encode()).hexdigest()
        number = int(hash_value[:8], 16)
        vector.append(number / 0xFFFFFFFF)
    return vector


def add_knowledge_item(knowledge_items, item_id, content, metadata):
    if not content:
        return

    knowledge_items.append({
        "id": item_id,
        "content": content,
        "metadata": metadata
    })


with open(file_path, "r", encoding="utf-8") as file:
    data = json.load(file)

knowledge_items = []

# 1. Domain context
add_knowledge_item(
    knowledge_items,
    "domain_context_001",
    json.dumps(data.get("domain_context", {}), indent=2),
    {
        "type": "domain_context",
        "source": "ETF_Framework_v1"
    }
)

# 2. Application flow model
add_knowledge_item(
    knowledge_items,
    "application_flow_model_001",
    json.dumps(data.get("application_flow_model", {}), indent=2),
    {
        "type": "application_flow",
        "source": "ETF_Framework_v1"
    }
)

# 3. Project structure
add_knowledge_item(
    knowledge_items,
    "project_structure_001",
    json.dumps(data.get("project_structure", {}), indent=2),
    {
        "type": "project_structure",
        "source": "ETF_Framework_v1"
    }
)

# 4. Coding standards
add_knowledge_item(
    knowledge_items,
    "coding_standards_001",
    json.dumps(data.get("coding_standards", {}), indent=2),
    {
        "type": "coding_standards",
        "source": "ETF_Framework_v1"
    }
)

# 5. Utility classes and methods
for utility in data.get("utility_classes", []):
    class_name = utility.get("class_name", "unknown_class")

    # Store utility class overview
    utility_content = f"""
Class Name: {class_name}
Package: {utility.get("package")}
Import Statement: {utility.get("import_statement")}
Purpose: {utility.get("purpose")}
Source Slide: {utility.get("source_slide")}
"""

    add_knowledge_item(
        knowledge_items,
        f"utility_class_{class_name}",
        utility_content,
        {
            "type": "utility_class",
            "class_name": class_name,
            "source": "ETF_Framework_v1"
        }
    )

    # Store each method separately
    for method in utility.get("methods", []):
        method_id = method.get("id", method.get("name", "unknown_method"))

        method_content = f"""
Framework Class: {class_name}
Method Name: {method.get("name")}
Signature: {method.get("signature")}
Return Type: {method.get("return_type")}
Parameters: {method.get("parameters_text")}
Description: {method.get("description")}
Capability Category: {method.get("capability_category", "")}
Source Slide: {method.get("source_slide")}
"""

        safe_method_id = method_id.replace(".", "_")

        add_knowledge_item(
            knowledge_items,
            f"{class_name}_{safe_method_id}",
            method_content,
            {
                "type": "framework_method",
                "class_name": class_name,
                "method_name": method.get("name", ""),
                "capability_category": method.get("capability_category", ""),
                "source": "ETF_Framework_v1"
            }
        )

# 6. Method selection rules
for index, rule in enumerate(data.get("method_selection_rules", []), start=1):
    intent = rule.get("intent", rule.get("situation", f"rule_{index}"))

    add_knowledge_item(
        knowledge_items,
        f"method_selection_rule_{index}",
        json.dumps(rule, indent=2),
        {
            "type": "method_selection_rule",
            "intent": intent,
            "source": "ETF_Framework_v1"
        }
    )

# 7. Web automation patterns
add_knowledge_item(
    knowledge_items,
    "web_automation_patterns_001",
    json.dumps(data.get("web_automation_patterns", {}), indent=2),
    {
        "type": "web_automation_patterns",
        "source": "ETF_Framework_v1"
    }
)

# 8. Database patterns
add_knowledge_item(
    knowledge_items,
    "database_patterns_001",
    json.dumps(data.get("database_patterns", {}), indent=2),
    {
        "type": "database_patterns",
        "source": "ETF_Framework_v1"
    }
)

# 9. Reporter patterns
add_knowledge_item(
    knowledge_items,
    "reporter_patterns_001",
    json.dumps(data.get("reporter_patterns", {}), indent=2),
    {
        "type": "reporter_patterns",
        "source": "ETF_Framework_v1"
    }
)

# 10. Maven and execution
add_knowledge_item(
    knowledge_items,
    "maven_and_execution_001",
    json.dumps(data.get("maven_and_execution", {}), indent=2),
    {
        "type": "maven_and_execution",
        "source": "ETF_Framework_v1"
    }
)

# 11. Git workflow
add_knowledge_item(
    knowledge_items,
    "git_workflow_001",
    json.dumps(data.get("git_workflow", {}), indent=2),
    {
        "type": "git_workflow",
        "source": "ETF_Framework_v1"
    }
)

# 12. Common failure modes and fixes
for index, failure in enumerate(data.get("common_failure_modes_and_fixes", []), start=1):
    add_knowledge_item(
        knowledge_items,
        f"common_failure_fix_{index}",
        json.dumps(failure, indent=2),
        {
            "type": "failure_fix",
            "source": "ETF_Framework_v1"
        }
    )

# Store all items into ChromaDB
for item in knowledge_items:
    collection.upsert(
        ids=[item["id"]],
        documents=[item["content"]],
        embeddings=[get_fake_embedding(item["content"])],
        metadatas=[item["metadata"]]
    )

print(f"Stored {len(knowledge_items)} knowledge items into ChromaDB.")



------------------

ETF_Framework_v1.json

------------------------------
{
  "schema_version": "2.0.0",
  "knowledge_base_id": "empower_etf_imported_framework_working_knowledge_v2",
  "title": "ETF Imported Framework - Working Knowledge Base for Learner Agent",
  "generated_on": "2026-06-17",
  "intended_consumer": {
    "agent_type": "Learner Agent",
    "primary_use": "Teach, guide, validate, and generate UI automation implementation steps for Empower Test Framework Imported Framework project.",
    "learner_tasks_supported": [
      "Explain ETF framework concepts",
      "Map feature files to step definitions",
      "Generate Page Object Model classes",
      "Select correct ETF Web, DB, Reporter and config methods",
      "Guide GitLab branch/commit/merge workflow",
      "Guide Maven/test runner/runbdd.xml execution flow",
      "Review code for ETF standards",
      "Generate learner exercises and assessment rubrics"
    ]
  },
  "source_manifest": [
    {
      "id": "ppt_framework_deck",
      "type": "presentation",
      "file_name": "Automation Framework For learner agent.pptx",
      "role": "Base framework architecture, utility class catalog, Maven, Git, and TestNG reference"
    },
    {
      "id": "transcript_session_4_8_4",
      "type": "meeting_transcript",
      "file_name": "20) UI Automation Session 4 - ETF - 8_4-en-US.docx",
      "role": "Operational end-to-end framework flow, folders, feature/step/page object/test runner/reporting/config usage"
    },
    {
      "id": "transcript_workshop_2_8_5",
      "type": "meeting_transcript",
      "file_name": "21) UI Automation ETF Workshop 2 - 8_5-en-US.docx",
      "role": "Template import from GitLab, participant web automation, locators, waits, report steps, and Git push/merge workflow"
    },
    {
      "id": "transcript_workshop_3_8_6",
      "type": "meeting_transcript",
      "file_name": "21) UI Automation Workshop 3 & Script Generation using AI Tool - 8_6-en-US.docx",
      "role": "GitHub Copilot/AI assisted script generation, MFA handling, POM standards, waits, assertions, loadable components and assessment guidance"
    }
  ],
  "domain_context": {
    "framework_name": "Empower Test Framework",
    "short_name": "ETF",
    "project_name": "ETF Imported Framework",
    "automation_type": "UI automation using Selenium + Java + Cucumber/BDD + Maven + TestNG-style execution",
    "core_rule": "Do not write raw Selenium code as first choice in project implementation. Use ETF utility methods such as Web.*, DB.*, Reporter.*, Stock.*, and ReadProperties where applicable.",
    "mental_model": "Feature file expresses business requirement. Step definition binds Gherkin steps to Java. Page object owns page locators and reusable page actions. ETF utility classes perform browser, database, reporting and configuration operations. runbdd.xml/test runner starts execution and report generation.",
    "expert_positioning": "Treat this as a 15-year QE automation expert knowledge base: emphasize maintainability, reuse, framework standards, clean naming, reporting, diagnostics, and team Git workflow."
  },
  "learning_ontology": {
    "entities": [
      "Feature File",
      "Scenario",
      "Scenario Outline",
      "Examples Table",
      "Tag",
      "Step Definition",
      "Page Object",
      "Page Element",
      "Page Method",
      "Test Runner",
      "runbdd.xml",
      "testConfig/testExecutionConfig.properties",
      "Maven pom.xml",
      "ETF Jar Dependency",
      "Web Utility",
      "DB Utility",
      "Reporter Utility",
      "MasterStepDefinitions",
      "GitLab Branch",
      "Merge Request",
      "Cucumber HTML Report",
      "Screenshot",
      "Loadable Component",
      "GitHub Copilot Prompt"
    ],
    "relationships": [
      {
        "from": "JIRA Story",
        "to": "Feature File",
        "relation": "converted_into"
      },
      {
        "from": "Feature File",
        "to": "Step Definition",
        "relation": "generates_or_maps_to"
      },
      {
        "from": "Step Definition",
        "to": "Page Object",
        "relation": "calls_page_methods_from"
      },
      {
        "from": "Page Object",
        "to": "Web Utility",
        "relation": "uses_for_ui_actions"
      },
      {
        "from": "Page Object",
        "to": "testConfig.properties",
        "relation": "reads_dynamic_values_from"
      },
      {
        "from": "Step Definition",
        "to": "Reporter Utility",
        "relation": "logs_business_steps_using"
      },
      {
        "from": "Test Runner",
        "to": "Feature File",
        "relation": "filters_by_tags_and_feature_path"
      },
      {
        "from": "runbdd.xml",
        "to": "Test Runner",
        "relation": "executes_entry_point"
      },
      {
        "from": "Maven pom.xml",
        "to": "ETF Jar Dependency",
        "relation": "declares_framework_version"
      },
      {
        "from": "Execution",
        "to": "Cucumber HTML Report",
        "relation": "produces"
      }
    ]
  },
  "application_flow_model": {
    "canonical_ui_automation_flow": [
      {
        "step": 1,
        "name": "Receive requirement",
        "details": "Start from JIRA story or requirement and express expected behavior as Gherkin."
      },
      {
        "step": 2,
        "name": "Create feature file",
        "details": "Place .feature under features/<application_or_module>. Use clear tags such as @participantWebLogin or @titleVerification."
      },
      {
        "step": 3,
        "name": "Configure TestRunner",
        "details": "Set features path, glue path for step definitions, plugin/report configuration, and tags to execute."
      },
      {
        "step": 4,
        "name": "Run runbdd.xml once",
        "details": "Execute to let Cucumber print undefined step snippets when new steps are not implemented."
      },
      {
        "step": 5,
        "name": "Create step definition class",
        "details": "Place under src/test/java/<application_or_module>. Paste generated snippets and import Given/When/Then."
      },
      {
        "step": 6,
        "name": "Create page object classes",
        "details": "Place under src/main/java/page_objects or a module package. Create one class per application page."
      },
      {
        "step": 7,
        "name": "Identify locators",
        "details": "Use @FindBy(id=...), @FindBy(xpath=...), etc. Keep elements private and access through page methods."
      },
      {
        "step": 8,
        "name": "Implement page methods",
        "details": "Use ETF Web methods for click, text entry, waits, dropdowns, title/text verification and screenshots."
      },
      {
        "step": 9,
        "name": "Call page methods from step definitions",
        "details": "Step definitions should orchestrate business flow, not contain raw locator/action code."
      },
      {
        "step": 10,
        "name": "Log reporter steps",
        "details": "Use Reporter.logEvent with status, step name, details and screenshot flag. Prefer step definitions for business-level report text."
      },
      {
        "step": 11,
        "name": "Execute and debug",
        "details": "Run runbdd.xml. Check console, browser behavior, screenshots, and Cucumber HTML report."
      },
      {
        "step": 12,
        "name": "Commit and push",
        "details": "Commit only relevant source/config changes, push to a working branch, raise merge request to master/sprint branch based on team process."
      }
    ],
    "example_participant_web_flow": {
      "pages": [
        "Login Page",
        "MFA / Two-step Authentication Page",
        "React / Layout Home Page"
      ],
      "feature_steps": [
        "Given user is on Participant Web application",
        "When user logs in with valid username and password",
        "When user submits MFA code for verification",
        "Then React/Layout page is displayed"
      ],
      "page_object_mapping": [
        {
          "page": "ParticipantWebLoginPage",
          "responsibilities": [
            "launchParticipantWebUrl",
            "acceptCookiesIfDisplayed",
            "loginToApplication(username,password)"
          ]
        },
        {
          "page": "ParticipantWebMFAPage",
          "responsibilities": [
            "clickAlreadyHaveCode",
            "enterMfaCode(code)",
            "clickSignIn"
          ]
        },
        {
          "page": "ParticipantWebLayoutPage",
          "responsibilities": [
            "verifyLayoutPageDisplayed",
            "verifyExpectedHomePageContent"
          ]
        }
      ],
      "special_handling": [
        "MFA through external channels such as phone, email, Okta push, text-me, or call-me is not reliable for automation.",
        "Use static/test MFA code when the application team provides one for automation.",
        "If MFA code is split into multiple text boxes, enter one character per field or implement a helper method that distributes characters."
      ]
    },
    "example_title_verification_flow": {
      "feature_steps": [
        "Given user is on Plan Service Center login page",
        "Then title should contain Empower"
      ],
      "implementation_pattern": "Page object launches URL using Web.getWebDriver().get(Stock.getConfigParameter(\"PSCLoginURL\")) and returns boolean from verifyTitleContainsEmpower. Step definition logs pass/fail with Reporter.logEvent."
    }
  },
  "project_structure": {
    "features": {
      "path": "features/<module>",
      "purpose": "BDD feature files derived from JIRA stories or requirements."
    },
    "step_definitions": {
      "path": "src/test/java/<module>",
      "purpose": "Generated/implemented Java methods mapped to Given/When/Then."
    },
    "page_objects": {
      "path": "src/main/java/<page_object_package>",
      "purpose": "Page-wise locators, page initialization, and reusable page actions."
    },
    "test_data": {
      "path": "testdata or test data folder",
      "purpose": "External test data files."
    },
    "reports": {
      "path": "test report / cucumber HTML reports",
      "purpose": "Generated execution report and screenshots; refreshed each run unless backed up."
    },
    "pom_xml": {
      "path": "pom.xml",
      "purpose": "Maven configuration and ETF framework dependency/version."
    },
    "run_bdd_xml": {
      "path": "runbdd.xml or run BDD XML",
      "purpose": "Execution launcher that points to TestRunner package/class."
    },
    "test_config": {
      "path": "testExecutionConfig.properties / testConfig.properties",
      "purpose": "Environment, URL, browser, object sync timeout, usernames, passwords, DB and framework parameters."
    },
    "ignored_or_generated": [
      ".idea",
      ".gitignore",
      "generated report folders",
      "target/build output"
    ]
  },
  "coding_standards": {
    "naming": [
      "Use names that explain application/module/page/action, for example ParticipantWebLoginStepDefinitions and ParticipantWebMFAPage.",
      "Avoid Method1, Test2, David1-style names.",
      "Tags should describe scope, application, module or purpose, for example @participantWebLogin or @pscTitleVerification."
    ],
    "page_object_rules": [
      "Create one page object class per web page or logical page section.",
      "Keep WebElement fields private.",
      "Initialize page objects using PageFactory.initElements(Web.getDriver() or Web.getWebDriver(...), this) depending on project template usage.",
      "Page methods should perform actions and return boolean/value where needed; avoid embedding business report sentences inside page objects.",
      "Reports should usually be added at step definition level so reused page methods do not create noisy or misleading reports."
    ],
    "step_definition_rules": [
      "Step definitions should call page object methods and coordinate business flow.",
      "Do not place large raw Selenium locator/action logic directly inside step definitions.",
      "Use Reporter.logEvent for pass/fail/info evidence and screenshot capture."
    ],
    "config_rules": [
      "Do not hard-code URLs, browsers, usernames, passwords, or frequently changing environment values in page objects.",
      "Read dynamic values through Stock.getConfigParameter or relevant properties reader.",
      "Treat config file updates carefully in Git; communicate changes and avoid pushing config unless required and approved."
    ],
    "comments_and_javadoc": [
      "Add meaningful comments/Javadoc for page methods and step definitions.",
      "Comments should explain purpose, input parameters, returned value and important framework behavior."
    ]
  },
  "utility_classes": [
    {
      "class_name": "CommonLib.java",
      "package": null,
      "purpose": "Common reusable methods such as date formatters, password decryption, property handling, and email formats.",
      "source_slide": 7
    },
    {
      "class_name": "DB.java",
      "package": "/src/main/java/bdd_lib",
      "import_statement": "import bdd_lib.DB;",
      "purpose": "Database connection, query, update, result extraction, and DB-vs-web validation operations.",
      "source_slide": 9,
      "methods": [
        {
          "id": "method.executeQuery.1",
          "name": "executeQuery",
          "signature": "executeQuery(String dbName,String query,String... queryParameterValues)",
          "return_type": "ResultSet",
          "parameters_text": "String dbName,String query,String... queryParameterValues",
          "description": "Method to execute the query and return ResultSet",
          "source_slide": 9
        },
        {
          "id": "method.executeUpdate.2",
          "name": "executeUpdate",
          "signature": "executeUpdate(String dbName,String query,String... queryParameterValues)",
          "return_type": "int",
          "parameters_text": "String dbName,String query,String... queryParameterValues",
          "description": "Method to update the records in Data Base",
          "source_slide": 9
        },
        {
          "id": "method.getPreparedStatement.3",
          "name": "getPreparedStatement",
          "signature": "getPreparedStatement(String dbName,String query,String[] queryParameterValues)",
          "return_type": "PreparedStatement",
          "parameters_text": "String dbName,String query,String[] queryParameterValues",
          "description": "Method to establish data base connection and prepare Sql statement",
          "source_slide": 9
        },
        {
          "id": "method.getDBConnection.4",
          "name": "getDBConnection",
          "signature": "getDBConnection(String DBName)",
          "return_type": "Connection",
          "parameters_text": "String DBName",
          "description": "Method to establish data base connection",
          "source_slide": 9
        },
        {
          "id": "method.closeDBConnections.5",
          "name": "closeDBConnections",
          "signature": "closeDBConnections()",
          "return_type": "Void",
          "parameters_text": "",
          "description": "Method to close data base connection",
          "source_slide": 9
        },
        {
          "id": "method.compareDB_Date_With_Web_Date.6",
          "name": "compareDB_Date_With_Web_Date",
          "signature": "compareDB_Date_With_Web_Date(String dbDate,String webDate)",
          "return_type": "boolean",
          "parameters_text": "String dbDate,String webDate",
          "description": "Method to compare data base date and web date",
          "source_slide": 9
        },
        {
          "id": "method.getFilteredRowSetCount.7",
          "name": "getFilteredRowSetCount",
          "signature": "getFilteredRowSetCount(FilteredRowSet frs)",
          "return_type": "int",
          "parameters_text": "FilteredRowSet frs",
          "description": "Method to get Filtered RowSet Count",
          "source_slide": 9
        },
        {
          "id": "method.getRecordFromRecordSet.8",
          "name": "getRecordFromRecordSet",
          "signature": "getRecordFromRecordSet(FilteredRowSet recordSet,String paramTobeFetched)",
          "return_type": "String",
          "parameters_text": "FilteredRowSet recordSet,String paramTobeFetched",
          "description": "",
          "source_slide": 9
        }
      ]
    },
    {
      "class_name": "Log.java",
      "package": null,
      "purpose": "Logging-related framework functions.",
      "source_slide": 7
    },
    {
      "class_name": "ReadProperties.java",
      "package": null,
      "purpose": "Reads test config property files.",
      "source_slide": 7
    },
    {
      "class_name": "Stock.java",
      "package": null,
      "purpose": "Reads and manages test data.",
      "source_slide": 7
    },
    {
      "class_name": "Web.java",
      "package": "/src/main/java/bdd_lib",
      "import_statement": "import bdd_lib.web;",
      "purpose": "UI and Selenium browser automation functions.",
      "source_slide": 8,
      "methods": [
        {
          "id": "method.actionsClickOnElement.1",
          "name": "actionsClickOnElement",
          "signature": "actionsClickOnElement(org.openqa.selenium.WebElement webElementName)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement webElementName",
          "description": "Method to click on a specified web element using Actions click.",
          "source_slide": 8,
          "capability_category": "user_actions"
        },
        {
          "id": "method.assertTrue.2",
          "name": "assertTrue",
          "signature": "assertTrue(org.openqa.selenium.WebElement ele, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement ele, java.lang.String sMsg",
          "description": "Confirm that the target element is present somewhere on the page.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.captureScreenshot.3",
          "name": "captureScreenshot",
          "signature": "captureScreenshot()",
          "return_type": "static java.lang.String",
          "parameters_text": "",
          "description": "Method to capture screenshots from active window handle Files are placed under folder named specific to test case Base folder location is provided in property \"ScreenshotFolder\" File name: TestCaseName_I_IterationNumber_TotalNumberOfFilesInFolder.png Example: MyTestCase_I1_6.png",
          "source_slide": 8,
          "capability_category": "diagnostics_and_artifacts"
        },
        {
          "id": "method.clickOnElement.4",
          "name": "clickOnElement",
          "signature": "clickOnElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
          "description": "Clicks on a target element (e.g., a link, button, checkbox, or radio button).",
          "source_slide": 8,
          "capability_category": "user_actions"
        },
        {
          "id": "method.clickOnElement.5",
          "name": "clickOnElement",
          "signature": "clickOnElement(org.openqa.selenium.WebElement clickableElement)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement clickableElement",
          "description": "Clicks on a target element (e.g., a link, button, checkbox, or radio button).",
          "source_slide": 8,
          "capability_category": "user_actions"
        },
        {
          "id": "method.executeJavascript.6",
          "name": "executeJavascript",
          "signature": "executeJavascript(org.openqa.selenium.WebDriver driver, java.lang.String className)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebDriver driver, java.lang.String className",
          "description": "TO verify Java Script JS content is loaded successfully",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.figlet.7",
          "name": "figlet",
          "signature": "figlet(java.lang.String text)",
          "return_type": "static void",
          "parameters_text": "java.lang.String text",
          "description": "Method which will print Figlet fort in Console",
          "source_slide": 8,
          "capability_category": "diagnostics_and_artifacts"
        },
        {
          "id": "method.getDriver.8",
          "name": "getDriver",
          "signature": "getDriver()",
          "return_type": "static org.openqa.selenium.WebDriver",
          "parameters_text": "",
          "description": "Method will return WebDriver driver instance.",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.getDropDownOptionAsText.9",
          "name": "getDropDownOptionAsText",
          "signature": "getDropDownOptionAsText(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
          "return_type": "static java.lang.String",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
          "description": "Method to return the complete text of specified item in Dropdown box if item exists",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.getDropDownOptionAsText.10",
          "name": "getDropDownOptionAsText",
          "signature": "getDropDownOptionAsText(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
          "return_type": "static java.lang.String",
          "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
          "description": "Method to return the complete text of specified item in Dropdown box if item exists",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.getDropDownOptionCount.11",
          "name": "getDropDownOptionCount",
          "signature": "getDropDownOptionCount(java.lang.Object pageClassObj, java.lang.String dropDownElementName)",
          "return_type": "static int",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName",
          "description": "Method to return the Option Count of the dropdown",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.getElementAttribute.12",
          "name": "getElementAttribute",
          "signature": "getElementAttribute(java.lang.Object pageClassObj, java.lang.String webElementName, java.lang.String elementAttribute)",
          "return_type": "static java.lang.String",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName, java.lang.String elementAttribute",
          "description": "Method to return the attribute of an element",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.getIntegerCurrency.13",
          "name": "getIntegerCurrency",
          "signature": "getIntegerCurrency(java.lang.String curValue)",
          "return_type": "static float",
          "parameters_text": "java.lang.String curValue",
          "description": "Method to convert amount displayed as String with $ symbol and returns a float value.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.jsClick.14",
          "name": "jsClick",
          "signature": "jsClick(org.openqa.selenium.WebElement element)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement element",
          "description": "Alternate to clickOnElement Method If clickOnElement does not work, use jsClick method You can face issue in IE browser with clickOnElement method",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.mouseHover.15",
          "name": "mouseHover",
          "signature": "mouseHover(org.openqa.selenium.WebElement webElement)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement webElement",
          "description": "Simulates a user hovering a mouse over the specified element.",
          "source_slide": 8,
          "capability_category": "user_actions"
        },
        {
          "id": "method.removeWebDriverInstance.16",
          "name": "removeWebDriverInstance",
          "signature": "removeWebDriverInstance()",
          "return_type": "static void",
          "parameters_text": "",
          "description": "",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.returnElement.17",
          "name": "returnElement",
          "signature": "returnElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
          "return_type": "static org.openqa.selenium.WebElement",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
          "description": "Method to return WebElement using Page Object as reference",
          "source_slide": 8,
          "capability_category": "element_lookup"
        },
        {
          "id": "method.returnElements.18",
          "name": "returnElements",
          "signature": "returnElements(java.lang.Object pageClassObj, java.lang.String webElementName)",
          "return_type": "static java.util.List<org.openqa.selenium.WebElement>",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
          "description": "Method to return List of WebElement using Page Object as reference",
          "source_slide": 8,
          "capability_category": "element_lookup"
        },
        {
          "id": "method.selectDropDownOption.19",
          "name": "selectDropDownOption",
          "signature": "selectDropDownOption(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
          "description": "Method to select a specified item in Dropdown box",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.selectDropDownOption.20",
          "name": "selectDropDownOption",
          "signature": "selectDropDownOption(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
          "description": "Method to select a specified item in Dropdown box",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.selectDropDownOptionVarArgs.21",
          "name": "selectDropDownOptionVarArgs",
          "signature": "selectDropDownOptionVarArgs(org.openqa.selenium.WebElement dropDownElement, java.lang.String... selValue)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String... selValue",
          "description": "Method to select a specified item in Dropdown box on Multiple options basis",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.selectDropnDownOptionAsIndex.22",
          "name": "selectDropnDownOptionAsIndex",
          "signature": "selectDropnDownOptionAsIndex(org.openqa.selenium.WebElement dropDownElement, java.lang.String dropDownIndex)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String dropDownIndex",
          "description": "",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.setLastIteration.23",
          "name": "setLastIteration",
          "signature": "setLastIteration(boolean isLastIteration)",
          "return_type": "static void",
          "parameters_text": "boolean isLastIteration",
          "description": "Framework will set number of iteration for each Test cases",
          "source_slide": 8,
          "capability_category": "miscellaneous"
        },
        {
          "id": "method.setPassWordToTextBox.24",
          "name": "setPassWordToTextBox",
          "signature": "setPassWordToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet)",
          "return_type": "static java.lang.String",
          "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet",
          "description": "Method to set password to specified text box element.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.setTextToAngularTextBox.25",
          "name": "setTextToAngularTextBox",
          "signature": "setTextToAngularTextBox(org.openqa.selenium.WebElement textBox, java.lang.String textToBeEntered)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement textBox, java.lang.String textToBeEntered",
          "description": "Sets the value of an input field, as though you typed it in.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.setTextToTextBox.26",
          "name": "setTextToTextBox",
          "signature": "setTextToTextBox(java.lang.String textBoxFieldName, java.lang.Object pageClassObj, java.lang.CharSequence... valueToSet)",
          "return_type": "static java.lang.String",
          "parameters_text": "java.lang.String textBoxFieldName, java.lang.Object pageClassObj, java.lang.CharSequence... valueToSet",
          "description": "Sets the value of an input field, as though you typed it in.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.setTextToTextBox.27",
          "name": "setTextToTextBox",
          "signature": "setTextToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.CharSequence... valueToSet)",
          "return_type": "static java.lang.String",
          "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.CharSequence... valueToSet",
          "description": "Sets the value of an input field, as though you typed it in.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.setTextToTextBox.28",
          "name": "setTextToTextBox",
          "signature": "setTextToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet",
          "description": "On Angular pages some times sendkeys sends only partial text to handle this issue, method sends the string in character only if set value does not match inputed string.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.verifyChecked.29",
          "name": "verifyChecked",
          "signature": "verifyChecked(org.openqa.selenium.WebElement sObj, java.lang.Boolean bExpflag, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.Boolean bExpflag, java.lang.String sMsg",
          "description": "Soft assert that a toggle-button (checkbox/radio) has been checked.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.getParent.30",
          "name": "getParent",
          "signature": "getParent(org.openqa.selenium.WebElement element)",
          "return_type": "static org.openqa.selenium.WebElement",
          "parameters_text": "org.openqa.selenium.WebElement element",
          "description": "Method to return parent element of the specified element",
          "source_slide": 8,
          "capability_category": "element_lookup"
        },
        {
          "id": "method.getRemoteWebDriver.31",
          "name": "getRemoteWebDriver",
          "signature": "getRemoteWebDriver(java.lang.String webBrowser, java.lang.String nodeUrl)",
          "return_type": "static org.openqa.selenium.WebDriver",
          "parameters_text": "java.lang.String webBrowser, java.lang.String nodeUrl",
          "description": "Method to initiate remote webDriver object based on used specified browser type and node url when execution type is set to grid Browser Types: 1.",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.getSelectedDropDownOption.32",
          "name": "getSelectedDropDownOption",
          "signature": "getSelectedDropDownOption(java.lang.Object pageClassObj, java.lang.String dropDownElementName)",
          "return_type": "static java.lang.String",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName",
          "description": "Method to return the Selected Option of the dropdown",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.getWebDriver.33",
          "name": "getWebDriver",
          "signature": "getWebDriver(java.lang.String webBrowser)",
          "return_type": "static org.openqa.selenium.WebDriver",
          "parameters_text": "java.lang.String webBrowser",
          "description": "Framework will initiate webDriver object based on used specified browser type Browser Types: 1.",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.getWebElementText.34",
          "name": "getWebElementText",
          "signature": "getWebElementText(org.openqa.selenium.WebElement element)",
          "return_type": "static java.lang.String",
          "parameters_text": "org.openqa.selenium.WebElement element",
          "description": "Get Web element Text",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.isCheckBoxChecked.35",
          "name": "isCheckBoxChecked",
          "signature": "isCheckBoxChecked(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement",
          "description": "Method to return the True or false Status if checkbox is check or not",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.isCheckBoxChecked.36",
          "name": "isCheckBoxChecked",
          "signature": "isCheckBoxChecked(org.openqa.selenium.WebElement fieldName, boolean... waitForElement)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement fieldName, boolean... waitForElement",
          "description": "Method to return the True or false Status if checkbox is check or not",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.isLastIteration.37",
          "name": "isLastIteration",
          "signature": "isLastIteration()",
          "return_type": "static boolean",
          "parameters_text": "",
          "description": "Framework will check last iteration for each Test cases.",
          "source_slide": 8,
          "capability_category": "miscellaneous"
        },
        {
          "id": "method.ispageloaded.38",
          "name": "ispageloaded",
          "signature": "ispageloaded(java.lang.String document)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.String document",
          "description": "Method to verify is page Loaded successfully",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.isWebEementEnabled.39",
          "name": "isWebEementEnabled",
          "signature": "isWebEementEnabled(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement",
          "description": "Method to check if element enabled on the page.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.isWebElementDisplayed.40",
          "name": "isWebElementDisplayed",
          "signature": "isWebElementDisplayed(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForEle)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForEle",
          "description": "Method to check if element exists on the page and is displayed",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.isWebElementDisplayed.41",
          "name": "isWebElementDisplayed",
          "signature": "isWebElementDisplayed(org.openqa.selenium.WebElement element, boolean... waitForElement)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement element, boolean... waitForElement",
          "description": "Method to check if element exists on the page and is displayed",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.isWebElementsDisplayed.42",
          "name": "isWebElementsDisplayed",
          "signature": "isWebElementsDisplayed(java.util.List<org.openqa.selenium.WebElement> elements, boolean... waitForElement)",
          "return_type": "static boolean",
          "parameters_text": "java.util.List<org.openqa.selenium.WebElement> elements, boolean... waitForElement",
          "description": "Method to check if list of elements exists on the page and is displayed",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyDropDownOptionExists.43",
          "name": "verifyDropDownOptionExists",
          "signature": "verifyDropDownOptionExists(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
          "description": "Method to verify if the specified item is found in Dropdown box",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.verifyDropDownOptionExists.44",
          "name": "verifyDropDownOptionExists",
          "signature": "verifyDropDownOptionExists(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
          "return_type": "static boolean",
          "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
          "description": "Method to verify if the specified item is found in Dropdown box",
          "source_slide": 8,
          "capability_category": "dropdown_handling"
        },
        {
          "id": "method.verifyEditable.45",
          "name": "verifyEditable",
          "signature": "verifyEditable(org.openqa.selenium.WebElement locator, java.lang.Boolean bExpflag, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement locator, java.lang.Boolean bExpflag, java.lang.String sMsg",
          "description": "Soft assert whether the specified input element is editable (e.g., hasn't been disabled).",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyElementIsDisplayed.46",
          "name": "verifyElementIsDisplayed",
          "signature": "verifyElementIsDisplayed(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
          "description": "Soft assert that the specified element is displayed somewhere on the page.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyElementIsEnabled.47",
          "name": "verifyElementIsEnabled",
          "signature": "verifyElementIsEnabled(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
          "description": "Soft assert that the specified element is enable somewhere on the page he test will continue even if the verify fails.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyElementIsNotDisplayed.48",
          "name": "verifyElementIsNotDisplayed",
          "signature": "verifyElementIsNotDisplayed(org.openqa.selenium.WebElement ele, java.lang.String details, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement ele, java.lang.String details, java.lang.String sMsg",
          "description": "Soft assert that the specified element is not displayed on the page.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyElementIsNotEnabled.49",
          "name": "verifyElementIsNotEnabled",
          "signature": "verifyElementIsNotEnabled(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
          "description": "Soft assert that the specified element is not enable on the page he test will continue even if the verify fails.",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.verifyErrorMessage.50",
          "name": "verifyErrorMessage",
          "signature": "verifyErrorMessage(org.openqa.selenium.WebElement sObj, java.lang.String sExpText, java.lang.Boolean bPartialText)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sExpText, java.lang.Boolean bPartialText",
          "description": "Method will verify error message with expected Value and report the status",
          "source_slide": 8,
          "capability_category": "verification_and_assertions"
        },
        {
          "id": "method.VerifyPartialText.51",
          "name": "VerifyPartialText",
          "signature": "VerifyPartialText(java.lang.String inPartialText, java.lang.String inActualText, boolean ignoreCase)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.String inPartialText, java.lang.String inActualText, boolean ignoreCase",
          "description": "Method to verify partial text of two strings.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.VerifyText.52",
          "name": "VerifyText",
          "signature": "VerifyText(java.lang.String inExpectedText, java.lang.String inActualText, boolean... ignoreCase)",
          "return_type": "static boolean",
          "parameters_text": "java.lang.String inExpectedText, java.lang.String inActualText, boolean... ignoreCase",
          "description": "Method to verify two strings.",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.verifyText.53",
          "name": "verifyText",
          "signature": "verifyText(org.openqa.selenium.WebElement Obj, java.lang.String sExpText, java.lang.String sMsg)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement Obj, java.lang.String sExpText, java.lang.String sMsg",
          "description": "Method to verify element text is as expected value and report the status",
          "source_slide": 8,
          "capability_category": "text_attribute_and_value_handling"
        },
        {
          "id": "method.verifyTitlePage.54",
          "name": "verifyTitlePage",
          "signature": "verifyTitlePage(java.lang.String sExpText)",
          "return_type": "static void",
          "parameters_text": "java.lang.String sExpText",
          "description": "Soft assert the title of the current page contains the provided text.",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.waitForElement.55",
          "name": "waitForElement",
          "signature": "waitForElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
          "return_type": "static void",
          "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
          "description": "Method to wait for the specified element's presence",
          "source_slide": 8,
          "capability_category": "synchronization"
        },
        {
          "id": "method.waitForElement.56",
          "name": "waitForElement",
          "signature": "waitForElement(org.openqa.selenium.WebElement element)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement element",
          "description": "Method to wait for the specified element's presence",
          "source_slide": 8,
          "capability_category": "synchronization"
        },
        {
          "id": "method.waitForElements.57",
          "name": "waitForElements",
          "signature": "waitForElements(java.util.List<org.openqa.selenium.WebElement> elements)",
          "return_type": "static void",
          "parameters_text": "java.util.List<org.openqa.selenium.WebElement> elements",
          "description": "Method to wait for the list element's presence",
          "source_slide": 8,
          "capability_category": "synchronization"
        },
        {
          "id": "method.waitForPageToLoad.58",
          "name": "waitForPageToLoad",
          "signature": "waitForPageToLoad(org.openqa.selenium.WebDriver driver)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebDriver driver",
          "description": "Method to wait until page loads",
          "source_slide": 8,
          "capability_category": "browser_and_page_lifecycle"
        },
        {
          "id": "method.waitInMillSec.59",
          "name": "waitInMillSec",
          "signature": "waitInMillSec(int inmillSec)",
          "return_type": "static void",
          "parameters_text": "int inmillSec",
          "description": "Hard coded wait in millisecond 1 sec = 1000 millsec",
          "source_slide": 8,
          "capability_category": "synchronization"
        },
        {
          "id": "method.waitTillElementIsNotPresent.60",
          "name": "waitTillElementIsNotPresent",
          "signature": "waitTillElementIsNotPresent(org.openqa.selenium.WebElement sElement, int sec)",
          "return_type": "static void",
          "parameters_text": "org.openqa.selenium.WebElement sElement, int sec",
          "description": "Method will wait for given sec till element is not displayed.",
          "source_slide": 8,
          "capability_category": "synchronization"
        }
      ]
    },
    {
      "class_name": "Reporter.java",
      "package": "/src/main/java/bdd_reporter",
      "import_statement": "import bdd_reporter.Reporter;",
      "purpose": "Extent/HTML report generation and test-step event logging.",
      "source_slide": 10,
      "methods": [
        {
          "id": "method.logEvent.1",
          "name": "logEvent",
          "signature": "logEvent(Status logStatus, String Step,String Details, boolean attachScreenshot)",
          "return_type": "void",
          "parameters_text": "Status logStatus, String Step,String Details, boolean attachScreenshot",
          "description": "",
          "source_slide": 10
        }
      ]
    }
  ],
  "method_selection_rules": [
    {
      "intent": "Click a normal button/link/checkbox/radio",
      "preferred_methods": [
        "Web.clickOnElement(WebElement)",
        "Web.clickOnElement(pageClassObj, webElementName)"
      ],
      "fallback_methods": [
        "Web.actionsClickOnElement(WebElement)",
        "Web.jsClick(WebElement)"
      ],
      "expert_guidance": "Start with clickOnElement. Use Actions click for complex hover/action chains. Use jsClick only when normal click fails due to browser or DOM interaction issues."
    },
    {
      "intent": "Enter text",
      "preferred_methods": [
        "Web.setTextToTextBox(WebElement, CharSequence...)",
        "Web.setTextToTextBox(textBoxFieldName, pageClassObj, CharSequence...)"
      ],
      "fallback_methods": [
        "Web.setTextToAngularTextBox(WebElement, String)"
      ],
      "expert_guidance": "For Angular pages or partial sendKeys issues, use the Angular/character-by-character text method and verify the final value."
    },
    {
      "intent": "Select or verify dropdown values",
      "preferred_methods": [
        "Web.selectDropDownOption",
        "Web.getSelectedDropDownOption",
        "Web.verifyDropDownOptionExists",
        "Web.getDropDownOptionAsText",
        "Web.getDropDownOptionCount"
      ],
      "expert_guidance": "Use get/verify methods before selection when test stability depends on dynamic dropdown loading."
    },
    {
      "intent": "Wait for dynamic UI state",
      "preferred_methods": [
        "Web.waitForElement",
        "Web.waitForElements",
        "Web.waitForPageToLoad",
        "Web.waitTillElementIsNotPresent"
      ],
      "avoid": [
        "Web.waitInMillSec unless there is no stable condition to wait for"
      ],
      "expert_guidance": "Prefer condition-based waits over hard-coded waits."
    },
    {
      "intent": "Validate displayed UI state",
      "preferred_methods": [
        "Web.verifyElementIsDisplayed",
        "Web.verifyElementIsEnabled",
        "Web.verifyText",
        "Web.verifyErrorMessage",
        "Web.VerifyText",
        "Web.VerifyPartialText"
      ],
      "expert_guidance": "Use soft verification methods when test should continue and report all validation issues."
    },
    {
      "intent": "Validate backend data",
      "preferred_methods": [
        "DB.executeQuery",
        "DB.getRecordFromRecordSet",
        "DB.getFilteredRowSetCount",
        "DB.compareDB_Date_With_Web_Date"
      ],
      "expert_guidance": "Use parameterized query methods to avoid brittle SQL and to support reusable validation."
    },
    {
      "intent": "Update backend data for setup/cleanup",
      "preferred_methods": [
        "DB.executeUpdate"
      ],
      "expert_guidance": "Use carefully for controlled test data setup or cleanup only; log the action and close connections."
    },
    {
      "intent": "Report a test step",
      "preferred_methods": [
        "Reporter.logEvent"
      ],
      "expert_guidance": "Log status, step name, details, and screenshot choice. For FAIL/WARNING, include stack trace when available."
    },
    {
      "situation": "Launch browser/application in ETF UI project",
      "preferred_method": "Web.getWebDriver(browser).get(url) or Web.getDriver().get(url) based on template",
      "avoid": "new ChromeDriver() inside business code"
    },
    {
      "situation": "Read URL or environment value",
      "preferred_method": "Stock.getConfigParameter(\"KeyName\") or ReadProperties equivalent",
      "avoid": "Hard-coded URL in page object"
    },
    {
      "situation": "Click a standard button/link",
      "preferred_method": "Web.clickOnElement(webElement)",
      "fallback": "Web.jsClick(webElement) only when normal click fails due to browser/page issue"
    },
    {
      "situation": "Enter text",
      "preferred_method": "Web.setTextToTextBox(webElement, value)",
      "special_case": "Use angular-specific method if normal sendKeys causes partial text entry on Angular pages"
    },
    {
      "situation": "Wait for element",
      "preferred_method": "Web.waitForElement or Web.isWebElementDisplayed(element, true)",
      "avoid": "Thread.sleep except as last-resort troubleshooting"
    },
    {
      "situation": "Validate text/title",
      "preferred_method": "Web.VerifyText / Web.VerifyPartialText / Web.verifyTitlePage or boolean page method + Assert/Reporter",
      "note": "Use assertions where required and Reporter for readable execution evidence"
    },
    {
      "situation": "DB validation",
      "preferred_method": "DB.executeQuery, DB.getRecordFromRecordSet, compareDB_Date_With_Web_Date",
      "note": "Close connections after use"
    },
    {
      "situation": "Report a step",
      "preferred_method": "Reporter.logEvent(Status.PASS/FAIL/INFO/WARNING, step, details, attachScreenshot)",
      "note": "Set screenshot=true when visual evidence is useful"
    }
  ],
  "web_automation_patterns": {
    "locator_strategy": [
      "Prefer stable id/name when available.",
      "Use XPath/CSS when id/name is unavailable or insufficient.",
      "Do not assume XPath is only for clickable elements; any stable locator strategy can be used for input, button, label, or validation elements.",
      "When dynamic popups such as Accept Cookies appear, create locator and handle conditionally before interacting with covered elements."
    ],
    "wait_strategy": [
      "ETF methods already wrap many Selenium wait/exception handling patterns.",
      "Use Web.isWebElementDisplayed(element, true) or Web.waitForElement before interacting with slow-loading elements.",
      "Object sync timeout may be controlled in config properties, for example 60 seconds.",
      "Know the theory: implicit wait applies broadly, explicit wait targets a condition, fluent wait polls at configured intervals. Use framework-level methods unless a custom wait is necessary."
    ],
    "reporting_strategy": [
      "Add business-readable report lines in step definitions.",
      "Use Status.PASS for successful validations, Status.FAIL for failed validations, Status.INFO for neutral progress logs, and Status.WARNING for warning states.",
      "When failed or warning, reporter may include stack trace depending on framework behavior.",
      "Generated reports are refreshed on each run; copy/rename report folder when evidence must be preserved locally. Jenkins may publish reports centrally."
    ],
    "loadable_components": {
      "purpose": "Avoid repeating full login/navigation when already on a later page; check current page state and navigate only from the nearest loaded previous page.",
      "concept": "Each page implements isLoaded and load. Calling get() first checks isLoaded; if not loaded, load() navigates from the previous page or starts from URL/login as needed.",
      "pattern": [
        "SearchPage.isLoaded checks for a stable search page element.",
        "SearchPage.load calls HomePage.get(), then performs navigation from home to search.",
        "HomePage.load calls MFAPage.get(), then performs MFA-to-home transition.",
        "MFAPage.load calls LoginPage.get(), then performs login-to-MFA transition.",
        "LoginPage.load launches URL and performs base login preparation."
      ],
      "benefit": "Saves execution time during release/regression runs by not replaying unnecessary navigation from scratch."
    }
  },
  "database_patterns": {
    "when_to_use": [
      "Validate backend state after UI action",
      "Fetch expected data for comparison",
      "Update/setup test data when approved"
    ],
    "methods": [
      "DB.getDBConnection",
      "DB.getPreparedStatement",
      "DB.executeQuery",
      "DB.executeUpdate",
      "DB.getFilteredRowSetCount",
      "DB.getRecordFromRecordSet",
      "DB.compareDB_Date_With_Web_Date",
      "DB.closeDBConnections"
    ],
    "best_practices": [
      "Parameterize SQL queries instead of string concatenation",
      "Do not leave open connections",
      "Keep DB validation logic reusable",
      "Separate data setup from assertion when possible"
    ]
  },
  "reporter_patterns": {
    "method": "Reporter.logEvent(Status logStatus, String Step, String Details, boolean attachScreenshot)",
    "usage_modes": [
      "Log step status, step name, details and optional screenshot",
      "Log step status, name and details without screenshot",
      "Attach only screenshot when required by framework overloads/templates",
      "Include stack trace for FAIL or WARNING when framework supports it"
    ],
    "example_decision": "If page object verifyTitleContainsEmpower returns true, step definition logs PASS with screenshot; otherwise logs FAIL with screenshot."
  },
  "maven_and_execution": {
    "pom_xml": "Declares ETF framework dependency/version and related Selenium/Cucumber/TestNG/plugin dependencies.",
    "commands": [
      "mvn clean",
      "mvn test",
      "mvn install"
    ],
    "execution_entry_points": [
      "TestRunner class for Cucumber options",
      "runbdd.xml for actual project execution"
    ],
    "test_runner_keys": {
      "features": "Path to features folder or specific feature subfolder",
      "glue": "Package path where step definitions exist",
      "plugin": "Report output options",
      "tags": "Tag expression to filter scenarios"
    },
    "runbdd_xml_role": "Initializes framework and points execution to the selected test runner package/class."
  },
  "git_workflow": {
    "repository_role": "GitLab stores team source code and protects against local machine loss.",
    "clone_vs_pull": {
      "clone": "First-time copy from GitLab to local",
      "pull_or_update": "Refresh an existing local project from GitLab"
    },
    "daily_flow": [
      "Pull/update from master or sprint/application branch before work when appropriate.",
      "Implement feature, step definitions, page objects, config updates if approved, and reports.",
      "Use Commit panel in IntelliJ to review changed/unversioned files.",
      "Commit only relevant files; avoid generated report folders and unrelated IDE files.",
      "Push local commit to user/sprint/application branch, not directly to protected master.",
      "Create merge request to master/sprint branch and assign reviewer/lead.",
      "Resolve merge conflicts before pushing or merging.",
      "After merge, update local project so team changes are available."
    ],
    "branching_strategy": {
      "application_repository": [
        "Master branch updated once at sprint end",
        "Create one sprint branch from master at sprint beginning",
        "Team pulls from sprint branch for test development",
        "Users merge back only to sprint branch during sprint",
        "Resolve conflicts before pushing to sprint branch",
        "Merge sprint branch to master at sprint end via merge request",
        "Delete user and sprint branches after master merge",
        "Jenkins jobs pull only from master"
      ],
      "page_object_repository": [
        "Master branch updated once at sprint end",
        "Create one sprint branch from master at sprint beginning",
        "Create one branch per application from sprint branch, such as PSC or PartWeb",
        "Users pull application branch for development",
        "Users merge back only to application branch during sprint",
        "Resolve conflicts at application branch to avoid leaking conflicts to sprint branch",
        "Merge sprint branch to master at sprint end via merge request",
        "Delete application, user, and sprint branches after master merge",
        "Jenkins jobs pull only from master"
      ]
    },
    "config_file_caution": "Config files should usually be controlled and reviewed. Push config changes only with clear reason and approval because shared URLs/browser/environment values can impact the full team."
  },
  "ai_assisted_generation": {
    "tools": [
      "GitHub Copilot",
      "ChatGPT",
      "other coding agents"
    ],
    "important_warning": "AI agents do not inherently know Empower Test Framework methods. Prompt must provide ETF method patterns, locator format, coding rules, and the current file context.",
    "safe_usage": [
      "Use AI to draft locators, page methods, comments and repetitive code.",
      "Never blindly accept generated code; review methods, imports, element order and framework compliance.",
      "Do not paste sensitive credentials, real tokens, or confidential user data into external AI prompts.",
      "Keep existing code safe by saying: Do not delete existing code.",
      "Ask AI to use ETF methods such as Web.clickOnElement, Web.setTextToTextBox, Web.waitForElement, Reporter.logEvent instead of raw driver.findElement."
    ],
    "prompt_template": {
      "name": "Generate ETF Page Object Methods from HTML",
      "template": "Below is my web element format: [paste @FindBy + private WebElement example]. Below are ETF framework methods I use: Web.clickOnElement(element), Web.setTextToTextBox(element,value), Web.waitForElement(element), Web.isWebElementDisplayed(element,true). Below is HTML for the target web elements: [paste sanitized HTML]. Identify web elements and create page methods for each action. Use ETF methods only. Do not delete existing code. Add JavaDoc comments.",
      "expected_output": "Private @FindBy elements plus public reusable page methods."
    },
    "review_checklist": [
      "Are element names clear?",
      "Are locators stable?",
      "Are ETF methods used instead of raw Selenium?",
      "Did it preserve existing code?",
      "Are steps in correct order?",
      "Are parameters handled correctly, such as MFA code characters?",
      "Are imports correct?",
      "Are comments/Javadoc useful?"
    ]
  },
  "assessment_and_review_rubric": {
    "evaluation_dimensions": [
      "Requirement understanding and feature/step mapping",
      "Correct step definition implementation",
      "Correct Page Object Model separation",
      "ETF method usage",
      "Validation/assertion quality",
      "Reporter usage and screenshots",
      "Comments/Javadocs and naming standards",
      "Execution result/debugging quality",
      "Git/source hygiene where applicable"
    ],
    "important_note": "A script failing during execution does not automatically mean zero credit. The process, structure, page objects, reports, ETF method usage, comments and logic are also evaluated.",
    "learner_practice_guidance": [
      "Start from an empty/template project and automate one simple flow end-to-end.",
      "Practice feature -> step definition -> page object -> report -> run -> debug repeatedly.",
      "Do not be intimidated by large existing projects; focus on the files and flow relevant to your story.",
      "Use README.txt in submitted projects to tell reviewers where feature files, step definitions, page objects and reports are located."
    ]
  },
  "common_failure_modes_and_fixes": [
    {
      "symptom": "Feature step is pale/yellow or undefined",
      "likely_cause": "Step definition not generated or regex/text mismatch",
      "fix": "Run runbdd.xml, copy generated snippets, import Given/When/Then, ensure glue path is correct."
    },
    {
      "symptom": "NoSuchElementException on username/password",
      "likely_cause": "Page not loaded, popup covers element, locator changed, or missing wait",
      "fix": "Debug browser state, handle Accept Cookies popup, add Web.waitForElement/Web.isWebElementDisplayed, verify locator."
    },
    {
      "symptom": "Scenario not executing",
      "likely_cause": "Wrong tag or TestRunner feature/glue path",
      "fix": "Verify tag in feature file and TestRunner tags option; verify runbdd.xml points to correct test runner."
    },
    {
      "symptom": "Report has only default steps/no screenshots",
      "likely_cause": "Reporter.logEvent not added or MasterStepDefinitions not copied",
      "fix": "Add Reporter.logEvent in step definitions and ensure reporting support class/package exists."
    },
    {
      "symptom": "AI generated driver.findElement code",
      "likely_cause": "Prompt did not provide ETF methods",
      "fix": "Re-prompt with ETF method examples and coding rules."
    },
    {
      "symptom": "Merge conflicts",
      "likely_cause": "Multiple people changed same file/package",
      "fix": "Pull latest, compare changes, resolve locally, communicate with team before merge request."
    }
  ],
  "learner_agent_behaviors": {
    "when_user_asks_how_to_automate_flow": "Return a step-by-step implementation plan, identify feature steps, step definitions, page object classes, ETF methods, reports and validation points.",
    "when_user_pastes_html": "Generate @FindBy locators and page methods using ETF Web utility methods; include comments and avoid raw Selenium unless explicitly requested.",
    "when_user_pastes_error": "Classify error by stage: feature mapping, runner config, locator/wait, framework config, report, Maven, or Git. Then provide targeted fix.",
    "when_user_requests_code_review": "Check POM separation, framework method usage, reporting, assertions, waits, config usage, naming, comments, and Git hygiene.",
    "when_user_requests_assessment_help": "Guide without overcomplicating: satisfy structure, ETF methods, reports, comments, and basic execution; remind that process points matter."
  },
  "sample_agent_queries_supported": [
    "Create feature/step/page-object structure for Participant Web login with MFA.",
    "Which ETF Web method should I use to enter text into a field?",
    "How do I fix undefined Cucumber steps?",
    "How should I handle Accept Cookies popup in ETF?",
    "Generate Copilot prompt for a new page object using ETF methods.",
    "Explain loadable components with login, MFA, home, and search pages.",
    "Review my step definition for ETF standards.",
    "What files should I commit and push to GitLab?"
  ],
  "base_presentation_knowledge": {
    "schema_version": "1.0.0",
    "knowledge_base_id": "empower_etf_automation_framework_learner_knowledge",
    "title": "Empower Test Framework Learner Knowledge Base",
    "source": {
      "file_name": "Automation Framework For learner agent.pptx",
      "source_type": "presentation",
      "confidentiality": "FOR INTERNAL USE ONLY",
      "slide_count": 28,
      "generated_utc": "2026-06-17T05:53:00.515929+00:00",
      "content_hash_sha256": "6cf18d2e3294f9ddc8c8d0f93ba765368610f00b92a3ddfa6b3fbd503d7a6794"
    },
    "intended_consumer": {
      "agent_type": "learner_agent",
      "use_cases": [
        "teach ETF framework fundamentals",
        "answer framework usage questions",
        "recommend correct utility method for web/database/reporting scenarios",
        "explain project setup, Maven, Git branching, and TestNG flow",
        "support QA automation onboarding"
      ],
      "expected_outputs": [
        "concept explanations",
        "method selection guidance",
        "step-by-step application flow",
        "quiz generation",
        "troubleshooting recommendations"
      ]
    },
    "domain_context": {
      "persona": "15+ years quality engineering test automation subject matter expert",
      "primary_stack": [
        "Java",
        "Selenium WebDriver",
        "Maven",
        "TestNG",
        "Cucumber",
        "Extent Reports",
        "Jenkins/GitLab CI/CD",
        "Git",
        "Database validation"
      ],
      "framework_name": "Empower Test Framework",
      "framework_acronym": "ETF",
      "definition": "A structured combination of guidelines, coding standards, tools, and practices designed to efficiently create, maintain, and scale automated tests.",
      "architecture_summary": "ETF acts as an abstraction layer over automation tools. It standardizes common practices and exposes reusable libraries/functions so testers can focus on fast, efficient automated test development without needing deep tool-specific implementation knowledge.",
      "value_proposition": [
        "consistent test scripts",
        "reusable code",
        "maintainable tests",
        "scalable automation for growing projects",
        "reporting and CI/CD integration",
        "faster test development",
        "improved coverage",
        "detailed reporting"
      ]
    },
    "application_flow_model": {
      "summary": "A typical ETF automation flow starts with project import and Maven build, reads test configuration, initializes browser/database dependencies, executes TestNG/Cucumber test methods using Page Objects and ETF utility classes, validates UI and database state, logs each step through Reporter, captures screenshots when needed, and produces reports that can be triggered locally or through CI/CD.",
      "ordered_steps": [
        {
          "step": 1,
          "name": "Project setup",
          "description": "Import automation project into workspace. ETF dependency is already available as a Maven dependency in the M2 repository. Run Maven build before using framework methods.",
          "related_slides": [
            12,
            16,
            17
          ]
        },
        {
          "step": 2,
          "name": "Configure execution",
          "description": "Populate testConfig.properties with environment, application URL, database details, framework parameters, browser driver paths, screenshot folder, and other runtime values.",
          "related_slides": [
            11
          ]
        },
        {
          "step": 3,
          "name": "Dependency resolution and build",
          "description": "Use pom.xml to pull dependencies, configure plugins, compile, and prepare test execution. Common commands are mvn clean, mvn test, and mvn install.",
          "related_slides": [
            17,
            18,
            19
          ]
        },
        {
          "step": 4,
          "name": "Test execution orchestration",
          "description": "TestNG organizes execution as Suite -> Tests -> TestNG Class -> Test Method and supports annotations, parameters, data providers, parallel execution, and multiple test categories.",
          "related_slides": [
            25,
            26
          ]
        },
        {
          "step": 5,
          "name": "Web interaction layer",
          "description": "Use Web.java methods for browser driver creation, page loading, element lookup, click/input actions, dropdown handling, waits, screenshots, and UI verification.",
          "related_slides": [
            8
          ]
        },
        {
          "step": 6,
          "name": "Database validation layer",
          "description": "Use DB.java methods to open DB connections, run parameterized queries/updates, fetch result values, count rowsets, compare database and web dates, and close DB connections.",
          "related_slides": [
            9
          ]
        },
        {
          "step": 7,
          "name": "Reporting layer",
          "description": "Use Reporter.logEvent to record step status, step name, details, screenshots, and stack trace for FAIL or WARNING conditions in the HTML/Extent report.",
          "related_slides": [
            10
          ]
        },
        {
          "step": 8,
          "name": "Version control and delivery",
          "description": "Develop changes in sprint or application branches, resolve conflicts before pushing, merge to master at sprint end through merge request, delete temporary branches, and have Jenkins jobs pull from master.",
          "related_slides": [
            23,
            24
          ]
        }
      ],
      "execution_pseudocode": [
        "Read testConfig.properties",
        "Resolve dependencies from pom.xml",
        "Initialize WebDriver using Web.getWebDriver or Web.getRemoteWebDriver",
        "Navigate to App URL",
        "Use Page Objects plus Web.returnElement/returnElements to locate elements",
        "Use Web.waitForElement/waitForPageToLoad before interacting",
        "Use Web.clickOnElement, jsClick, setTextToTextBox, selectDropDownOption, etc. for UI actions",
        "Use Web.verifyText/verifyElementIsDisplayed/VerifyText for UI assertions",
        "Use DB.getDBConnection and DB.executeQuery for backend validation where needed",
        "Use Reporter.logEvent for every meaningful test step",
        "Capture screenshots for failures or key proof points",
        "Close DB connections and remove WebDriver instance",
        "Review HTML/Extent report and CI/CD results"
      ]
    },
    "concepts": [
      {
        "id": "concept.framework",
        "name": "Test Automation Framework",
        "description": "Guidelines, coding standards, tools, and reusable practices that help create, maintain, and scale automated tests efficiently.",
        "source_slides": [
          3,
          4
        ]
      },
      {
        "id": "concept.etf_architecture",
        "name": "ETF Architecture",
        "description": "ETF provides reusable common libraries and functions such as date formatting, password decryption, property file reading, email formatting, web UI utilities, database access, logging, and reporting.",
        "source_slides": [
          5,
          7
        ]
      },
      {
        "id": "concept.utility_classes",
        "name": "Framework Utility Classes",
        "description": "Central framework classes expose reusable methods. Access pattern is ClassName.MethodName().",
        "source_slides": [
          7
        ]
      },
      {
        "id": "concept.maven",
        "name": "Maven in Automation",
        "description": "Maven standardizes Java project structure, dependency management, builds, plugins, execution, and reporting.",
        "source_slides": [
          14,
          15,
          16,
          17,
          18,
          19,
          20
        ]
      },
      {
        "id": "concept.git_strategy",
        "name": "Git Branching Strategy",
        "description": "Sprint-based branch strategy with master updated at sprint end, one sprint branch per sprint, optional application branches for page objects, conflict resolution before merge, and Jenkins pulling from master.",
        "source_slides": [
          23,
          24
        ]
      },
      {
        "id": "concept.testng",
        "name": "TestNG",
        "description": "Java test framework that supports annotations, parameters, data-driven testing, parallelism, flexible configuration, and Suite -> Tests -> Class -> Method execution hierarchy.",
        "source_slides": [
          25,
          26
        ]
      }
    ],
    "utility_classes": [
      {
        "class_name": "CommonLib.java",
        "package": null,
        "purpose": "Common reusable methods such as date formatters, password decryption, property handling, and email formats.",
        "source_slide": 7
      },
      {
        "class_name": "DB.java",
        "package": "/src/main/java/bdd_lib",
        "import_statement": "import bdd_lib.DB;",
        "purpose": "Database connection, query, update, result extraction, and DB-vs-web validation operations.",
        "source_slide": 9,
        "methods": [
          {
            "id": "method.executeQuery.1",
            "name": "executeQuery",
            "signature": "executeQuery(String dbName,String query,String... queryParameterValues)",
            "return_type": "ResultSet",
            "parameters_text": "String dbName,String query,String... queryParameterValues",
            "description": "Method to execute the query and return ResultSet",
            "source_slide": 9
          },
          {
            "id": "method.executeUpdate.2",
            "name": "executeUpdate",
            "signature": "executeUpdate(String dbName,String query,String... queryParameterValues)",
            "return_type": "int",
            "parameters_text": "String dbName,String query,String... queryParameterValues",
            "description": "Method to update the records in Data Base",
            "source_slide": 9
          },
          {
            "id": "method.getPreparedStatement.3",
            "name": "getPreparedStatement",
            "signature": "getPreparedStatement(String dbName,String query,String[] queryParameterValues)",
            "return_type": "PreparedStatement",
            "parameters_text": "String dbName,String query,String[] queryParameterValues",
            "description": "Method to establish data base connection and prepare Sql statement",
            "source_slide": 9
          },
          {
            "id": "method.getDBConnection.4",
            "name": "getDBConnection",
            "signature": "getDBConnection(String DBName)",
            "return_type": "Connection",
            "parameters_text": "String DBName",
            "description": "Method to establish data base connection",
            "source_slide": 9
          },
          {
            "id": "method.closeDBConnections.5",
            "name": "closeDBConnections",
            "signature": "closeDBConnections()",
            "return_type": "Void",
            "parameters_text": "",
            "description": "Method to close data base connection",
            "source_slide": 9
          },
          {
            "id": "method.compareDB_Date_With_Web_Date.6",
            "name": "compareDB_Date_With_Web_Date",
            "signature": "compareDB_Date_With_Web_Date(String dbDate,String webDate)",
            "return_type": "boolean",
            "parameters_text": "String dbDate,String webDate",
            "description": "Method to compare data base date and web date",
            "source_slide": 9
          },
          {
            "id": "method.getFilteredRowSetCount.7",
            "name": "getFilteredRowSetCount",
            "signature": "getFilteredRowSetCount(FilteredRowSet frs)",
            "return_type": "int",
            "parameters_text": "FilteredRowSet frs",
            "description": "Method to get Filtered RowSet Count",
            "source_slide": 9
          },
          {
            "id": "method.getRecordFromRecordSet.8",
            "name": "getRecordFromRecordSet",
            "signature": "getRecordFromRecordSet(FilteredRowSet recordSet,String paramTobeFetched)",
            "return_type": "String",
            "parameters_text": "FilteredRowSet recordSet,String paramTobeFetched",
            "description": "",
            "source_slide": 9
          }
        ]
      },
      {
        "class_name": "Log.java",
        "package": null,
        "purpose": "Logging-related framework functions.",
        "source_slide": 7
      },
      {
        "class_name": "ReadProperties.java",
        "package": null,
        "purpose": "Reads test config property files.",
        "source_slide": 7
      },
      {
        "class_name": "Stock.java",
        "package": null,
        "purpose": "Reads and manages test data.",
        "source_slide": 7
      },
      {
        "class_name": "Web.java",
        "package": "/src/main/java/bdd_lib",
        "import_statement": "import bdd_lib.web;",
        "purpose": "UI and Selenium browser automation functions.",
        "source_slide": 8,
        "methods": [
          {
            "id": "method.actionsClickOnElement.1",
            "name": "actionsClickOnElement",
            "signature": "actionsClickOnElement(org.openqa.selenium.WebElement webElementName)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement webElementName",
            "description": "Method to click on a specified web element using Actions click.",
            "source_slide": 8,
            "capability_category": "user_actions"
          },
          {
            "id": "method.assertTrue.2",
            "name": "assertTrue",
            "signature": "assertTrue(org.openqa.selenium.WebElement ele, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement ele, java.lang.String sMsg",
            "description": "Confirm that the target element is present somewhere on the page.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.captureScreenshot.3",
            "name": "captureScreenshot",
            "signature": "captureScreenshot()",
            "return_type": "static java.lang.String",
            "parameters_text": "",
            "description": "Method to capture screenshots from active window handle Files are placed under folder named specific to test case Base folder location is provided in property \"ScreenshotFolder\" File name: TestCaseName_I_IterationNumber_TotalNumberOfFilesInFolder.png Example: MyTestCase_I1_6.png",
            "source_slide": 8,
            "capability_category": "diagnostics_and_artifacts"
          },
          {
            "id": "method.clickOnElement.4",
            "name": "clickOnElement",
            "signature": "clickOnElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
            "description": "Clicks on a target element (e.g., a link, button, checkbox, or radio button).",
            "source_slide": 8,
            "capability_category": "user_actions"
          },
          {
            "id": "method.clickOnElement.5",
            "name": "clickOnElement",
            "signature": "clickOnElement(org.openqa.selenium.WebElement clickableElement)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement clickableElement",
            "description": "Clicks on a target element (e.g., a link, button, checkbox, or radio button).",
            "source_slide": 8,
            "capability_category": "user_actions"
          },
          {
            "id": "method.executeJavascript.6",
            "name": "executeJavascript",
            "signature": "executeJavascript(org.openqa.selenium.WebDriver driver, java.lang.String className)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebDriver driver, java.lang.String className",
            "description": "TO verify Java Script JS content is loaded successfully",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.figlet.7",
            "name": "figlet",
            "signature": "figlet(java.lang.String text)",
            "return_type": "static void",
            "parameters_text": "java.lang.String text",
            "description": "Method which will print Figlet fort in Console",
            "source_slide": 8,
            "capability_category": "diagnostics_and_artifacts"
          },
          {
            "id": "method.getDriver.8",
            "name": "getDriver",
            "signature": "getDriver()",
            "return_type": "static org.openqa.selenium.WebDriver",
            "parameters_text": "",
            "description": "Method will return WebDriver driver instance.",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.getDropDownOptionAsText.9",
            "name": "getDropDownOptionAsText",
            "signature": "getDropDownOptionAsText(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
            "return_type": "static java.lang.String",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
            "description": "Method to return the complete text of specified item in Dropdown box if item exists",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.getDropDownOptionAsText.10",
            "name": "getDropDownOptionAsText",
            "signature": "getDropDownOptionAsText(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
            "return_type": "static java.lang.String",
            "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
            "description": "Method to return the complete text of specified item in Dropdown box if item exists",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.getDropDownOptionCount.11",
            "name": "getDropDownOptionCount",
            "signature": "getDropDownOptionCount(java.lang.Object pageClassObj, java.lang.String dropDownElementName)",
            "return_type": "static int",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName",
            "description": "Method to return the Option Count of the dropdown",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.getElementAttribute.12",
            "name": "getElementAttribute",
            "signature": "getElementAttribute(java.lang.Object pageClassObj, java.lang.String webElementName, java.lang.String elementAttribute)",
            "return_type": "static java.lang.String",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName, java.lang.String elementAttribute",
            "description": "Method to return the attribute of an element",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.getIntegerCurrency.13",
            "name": "getIntegerCurrency",
            "signature": "getIntegerCurrency(java.lang.String curValue)",
            "return_type": "static float",
            "parameters_text": "java.lang.String curValue",
            "description": "Method to convert amount displayed as String with $ symbol and returns a float value.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.jsClick.14",
            "name": "jsClick",
            "signature": "jsClick(org.openqa.selenium.WebElement element)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement element",
            "description": "Alternate to clickOnElement Method If clickOnElement does not work, use jsClick method You can face issue in IE browser with clickOnElement method",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.mouseHover.15",
            "name": "mouseHover",
            "signature": "mouseHover(org.openqa.selenium.WebElement webElement)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement webElement",
            "description": "Simulates a user hovering a mouse over the specified element.",
            "source_slide": 8,
            "capability_category": "user_actions"
          },
          {
            "id": "method.removeWebDriverInstance.16",
            "name": "removeWebDriverInstance",
            "signature": "removeWebDriverInstance()",
            "return_type": "static void",
            "parameters_text": "",
            "description": "",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.returnElement.17",
            "name": "returnElement",
            "signature": "returnElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
            "return_type": "static org.openqa.selenium.WebElement",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
            "description": "Method to return WebElement using Page Object as reference",
            "source_slide": 8,
            "capability_category": "element_lookup"
          },
          {
            "id": "method.returnElements.18",
            "name": "returnElements",
            "signature": "returnElements(java.lang.Object pageClassObj, java.lang.String webElementName)",
            "return_type": "static java.util.List<org.openqa.selenium.WebElement>",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
            "description": "Method to return List of WebElement using Page Object as reference",
            "source_slide": 8,
            "capability_category": "element_lookup"
          },
          {
            "id": "method.selectDropDownOption.19",
            "name": "selectDropDownOption",
            "signature": "selectDropDownOption(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
            "description": "Method to select a specified item in Dropdown box",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.selectDropDownOption.20",
            "name": "selectDropDownOption",
            "signature": "selectDropDownOption(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
            "description": "Method to select a specified item in Dropdown box",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.selectDropDownOptionVarArgs.21",
            "name": "selectDropDownOptionVarArgs",
            "signature": "selectDropDownOptionVarArgs(org.openqa.selenium.WebElement dropDownElement, java.lang.String... selValue)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String... selValue",
            "description": "Method to select a specified item in Dropdown box on Multiple options basis",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.selectDropnDownOptionAsIndex.22",
            "name": "selectDropnDownOptionAsIndex",
            "signature": "selectDropnDownOptionAsIndex(org.openqa.selenium.WebElement dropDownElement, java.lang.String dropDownIndex)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String dropDownIndex",
            "description": "",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.setLastIteration.23",
            "name": "setLastIteration",
            "signature": "setLastIteration(boolean isLastIteration)",
            "return_type": "static void",
            "parameters_text": "boolean isLastIteration",
            "description": "Framework will set number of iteration for each Test cases",
            "source_slide": 8,
            "capability_category": "miscellaneous"
          },
          {
            "id": "method.setPassWordToTextBox.24",
            "name": "setPassWordToTextBox",
            "signature": "setPassWordToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet)",
            "return_type": "static java.lang.String",
            "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet",
            "description": "Method to set password to specified text box element.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.setTextToAngularTextBox.25",
            "name": "setTextToAngularTextBox",
            "signature": "setTextToAngularTextBox(org.openqa.selenium.WebElement textBox, java.lang.String textToBeEntered)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement textBox, java.lang.String textToBeEntered",
            "description": "Sets the value of an input field, as though you typed it in.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.setTextToTextBox.26",
            "name": "setTextToTextBox",
            "signature": "setTextToTextBox(java.lang.String textBoxFieldName, java.lang.Object pageClassObj, java.lang.CharSequence... valueToSet)",
            "return_type": "static java.lang.String",
            "parameters_text": "java.lang.String textBoxFieldName, java.lang.Object pageClassObj, java.lang.CharSequence... valueToSet",
            "description": "Sets the value of an input field, as though you typed it in.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.setTextToTextBox.27",
            "name": "setTextToTextBox",
            "signature": "setTextToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.CharSequence... valueToSet)",
            "return_type": "static java.lang.String",
            "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.CharSequence... valueToSet",
            "description": "Sets the value of an input field, as though you typed it in.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.setTextToTextBox.28",
            "name": "setTextToTextBox",
            "signature": "setTextToTextBox(org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement textBoxField, java.lang.String valueToSet",
            "description": "On Angular pages some times sendkeys sends only partial text to handle this issue, method sends the string in character only if set value does not match inputed string.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.verifyChecked.29",
            "name": "verifyChecked",
            "signature": "verifyChecked(org.openqa.selenium.WebElement sObj, java.lang.Boolean bExpflag, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.Boolean bExpflag, java.lang.String sMsg",
            "description": "Soft assert that a toggle-button (checkbox/radio) has been checked.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.getParent.30",
            "name": "getParent",
            "signature": "getParent(org.openqa.selenium.WebElement element)",
            "return_type": "static org.openqa.selenium.WebElement",
            "parameters_text": "org.openqa.selenium.WebElement element",
            "description": "Method to return parent element of the specified element",
            "source_slide": 8,
            "capability_category": "element_lookup"
          },
          {
            "id": "method.getRemoteWebDriver.31",
            "name": "getRemoteWebDriver",
            "signature": "getRemoteWebDriver(java.lang.String webBrowser, java.lang.String nodeUrl)",
            "return_type": "static org.openqa.selenium.WebDriver",
            "parameters_text": "java.lang.String webBrowser, java.lang.String nodeUrl",
            "description": "Method to initiate remote webDriver object based on used specified browser type and node url when execution type is set to grid Browser Types: 1.",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.getSelectedDropDownOption.32",
            "name": "getSelectedDropDownOption",
            "signature": "getSelectedDropDownOption(java.lang.Object pageClassObj, java.lang.String dropDownElementName)",
            "return_type": "static java.lang.String",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName",
            "description": "Method to return the Selected Option of the dropdown",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.getWebDriver.33",
            "name": "getWebDriver",
            "signature": "getWebDriver(java.lang.String webBrowser)",
            "return_type": "static org.openqa.selenium.WebDriver",
            "parameters_text": "java.lang.String webBrowser",
            "description": "Framework will initiate webDriver object based on used specified browser type Browser Types: 1.",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.getWebElementText.34",
            "name": "getWebElementText",
            "signature": "getWebElementText(org.openqa.selenium.WebElement element)",
            "return_type": "static java.lang.String",
            "parameters_text": "org.openqa.selenium.WebElement element",
            "description": "Get Web element Text",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.isCheckBoxChecked.35",
            "name": "isCheckBoxChecked",
            "signature": "isCheckBoxChecked(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement",
            "description": "Method to return the True or false Status if checkbox is check or not",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.isCheckBoxChecked.36",
            "name": "isCheckBoxChecked",
            "signature": "isCheckBoxChecked(org.openqa.selenium.WebElement fieldName, boolean... waitForElement)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement fieldName, boolean... waitForElement",
            "description": "Method to return the True or false Status if checkbox is check or not",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.isLastIteration.37",
            "name": "isLastIteration",
            "signature": "isLastIteration()",
            "return_type": "static boolean",
            "parameters_text": "",
            "description": "Framework will check last iteration for each Test cases.",
            "source_slide": 8,
            "capability_category": "miscellaneous"
          },
          {
            "id": "method.ispageloaded.38",
            "name": "ispageloaded",
            "signature": "ispageloaded(java.lang.String document)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.String document",
            "description": "Method to verify is page Loaded successfully",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.isWebEementEnabled.39",
            "name": "isWebEementEnabled",
            "signature": "isWebEementEnabled(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForElement",
            "description": "Method to check if element enabled on the page.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.isWebElementDisplayed.40",
            "name": "isWebElementDisplayed",
            "signature": "isWebElementDisplayed(java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForEle)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String fieldName, boolean... waitForEle",
            "description": "Method to check if element exists on the page and is displayed",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.isWebElementDisplayed.41",
            "name": "isWebElementDisplayed",
            "signature": "isWebElementDisplayed(org.openqa.selenium.WebElement element, boolean... waitForElement)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement element, boolean... waitForElement",
            "description": "Method to check if element exists on the page and is displayed",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.isWebElementsDisplayed.42",
            "name": "isWebElementsDisplayed",
            "signature": "isWebElementsDisplayed(java.util.List<org.openqa.selenium.WebElement> elements, boolean... waitForElement)",
            "return_type": "static boolean",
            "parameters_text": "java.util.List<org.openqa.selenium.WebElement> elements, boolean... waitForElement",
            "description": "Method to check if list of elements exists on the page and is displayed",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyDropDownOptionExists.43",
            "name": "verifyDropDownOptionExists",
            "signature": "verifyDropDownOptionExists(java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String dropDownElementName, java.lang.String selValue, boolean... args",
            "description": "Method to verify if the specified item is found in Dropdown box",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.verifyDropDownOptionExists.44",
            "name": "verifyDropDownOptionExists",
            "signature": "verifyDropDownOptionExists(org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args)",
            "return_type": "static boolean",
            "parameters_text": "org.openqa.selenium.WebElement dropDownElement, java.lang.String selValue, boolean... args",
            "description": "Method to verify if the specified item is found in Dropdown box",
            "source_slide": 8,
            "capability_category": "dropdown_handling"
          },
          {
            "id": "method.verifyEditable.45",
            "name": "verifyEditable",
            "signature": "verifyEditable(org.openqa.selenium.WebElement locator, java.lang.Boolean bExpflag, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement locator, java.lang.Boolean bExpflag, java.lang.String sMsg",
            "description": "Soft assert whether the specified input element is editable (e.g., hasn't been disabled).",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyElementIsDisplayed.46",
            "name": "verifyElementIsDisplayed",
            "signature": "verifyElementIsDisplayed(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
            "description": "Soft assert that the specified element is displayed somewhere on the page.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyElementIsEnabled.47",
            "name": "verifyElementIsEnabled",
            "signature": "verifyElementIsEnabled(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
            "description": "Soft assert that the specified element is enable somewhere on the page he test will continue even if the verify fails.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyElementIsNotDisplayed.48",
            "name": "verifyElementIsNotDisplayed",
            "signature": "verifyElementIsNotDisplayed(org.openqa.selenium.WebElement ele, java.lang.String details, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement ele, java.lang.String details, java.lang.String sMsg",
            "description": "Soft assert that the specified element is not displayed on the page.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyElementIsNotEnabled.49",
            "name": "verifyElementIsNotEnabled",
            "signature": "verifyElementIsNotEnabled(org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sStep, java.lang.String sMsg",
            "description": "Soft assert that the specified element is not enable on the page he test will continue even if the verify fails.",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.verifyErrorMessage.50",
            "name": "verifyErrorMessage",
            "signature": "verifyErrorMessage(org.openqa.selenium.WebElement sObj, java.lang.String sExpText, java.lang.Boolean bPartialText)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sObj, java.lang.String sExpText, java.lang.Boolean bPartialText",
            "description": "Method will verify error message with expected Value and report the status",
            "source_slide": 8,
            "capability_category": "verification_and_assertions"
          },
          {
            "id": "method.VerifyPartialText.51",
            "name": "VerifyPartialText",
            "signature": "VerifyPartialText(java.lang.String inPartialText, java.lang.String inActualText, boolean ignoreCase)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.String inPartialText, java.lang.String inActualText, boolean ignoreCase",
            "description": "Method to verify partial text of two strings.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.VerifyText.52",
            "name": "VerifyText",
            "signature": "VerifyText(java.lang.String inExpectedText, java.lang.String inActualText, boolean... ignoreCase)",
            "return_type": "static boolean",
            "parameters_text": "java.lang.String inExpectedText, java.lang.String inActualText, boolean... ignoreCase",
            "description": "Method to verify two strings.",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.verifyText.53",
            "name": "verifyText",
            "signature": "verifyText(org.openqa.selenium.WebElement Obj, java.lang.String sExpText, java.lang.String sMsg)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement Obj, java.lang.String sExpText, java.lang.String sMsg",
            "description": "Method to verify element text is as expected value and report the status",
            "source_slide": 8,
            "capability_category": "text_attribute_and_value_handling"
          },
          {
            "id": "method.verifyTitlePage.54",
            "name": "verifyTitlePage",
            "signature": "verifyTitlePage(java.lang.String sExpText)",
            "return_type": "static void",
            "parameters_text": "java.lang.String sExpText",
            "description": "Soft assert the title of the current page contains the provided text.",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.waitForElement.55",
            "name": "waitForElement",
            "signature": "waitForElement(java.lang.Object pageClassObj, java.lang.String webElementName)",
            "return_type": "static void",
            "parameters_text": "java.lang.Object pageClassObj, java.lang.String webElementName",
            "description": "Method to wait for the specified element's presence",
            "source_slide": 8,
            "capability_category": "synchronization"
          },
          {
            "id": "method.waitForElement.56",
            "name": "waitForElement",
            "signature": "waitForElement(org.openqa.selenium.WebElement element)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement element",
            "description": "Method to wait for the specified element's presence",
            "source_slide": 8,
            "capability_category": "synchronization"
          },
          {
            "id": "method.waitForElements.57",
            "name": "waitForElements",
            "signature": "waitForElements(java.util.List<org.openqa.selenium.WebElement> elements)",
            "return_type": "static void",
            "parameters_text": "java.util.List<org.openqa.selenium.WebElement> elements",
            "description": "Method to wait for the list element's presence",
            "source_slide": 8,
            "capability_category": "synchronization"
          },
          {
            "id": "method.waitForPageToLoad.58",
            "name": "waitForPageToLoad",
            "signature": "waitForPageToLoad(org.openqa.selenium.WebDriver driver)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebDriver driver",
            "description": "Method to wait until page loads",
            "source_slide": 8,
            "capability_category": "browser_and_page_lifecycle"
          },
          {
            "id": "method.waitInMillSec.59",
            "name": "waitInMillSec",
            "signature": "waitInMillSec(int inmillSec)",
            "return_type": "static void",
            "parameters_text": "int inmillSec",
            "description": "Hard coded wait in millisecond 1 sec = 1000 millsec",
            "source_slide": 8,
            "capability_category": "synchronization"
          },
          {
            "id": "method.waitTillElementIsNotPresent.60",
            "name": "waitTillElementIsNotPresent",
            "signature": "waitTillElementIsNotPresent(org.openqa.selenium.WebElement sElement, int sec)",
            "return_type": "static void",
            "parameters_text": "org.openqa.selenium.WebElement sElement, int sec",
            "description": "Method will wait for given sec till element is not displayed.",
            "source_slide": 8,
            "capability_category": "synchronization"
          }
        ]
      },
      {
        "class_name": "Reporter.java",
        "package": "/src/main/java/bdd_reporter",
        "import_statement": "import bdd_reporter.Reporter;",
        "purpose": "Extent/HTML report generation and test-step event logging.",
        "source_slide": 10,
        "methods": [
          {
            "id": "method.logEvent.1",
            "name": "logEvent",
            "signature": "logEvent(Status logStatus, String Step,String Details, boolean attachScreenshot)",
            "return_type": "void",
            "parameters_text": "Status logStatus, String Step,String Details, boolean attachScreenshot",
            "description": "",
            "source_slide": 10
          }
        ]
      }
    ],
    "method_selection_rules": [
      {
        "intent": "Click a normal button/link/checkbox/radio",
        "preferred_methods": [
          "Web.clickOnElement(WebElement)",
          "Web.clickOnElement(pageClassObj, webElementName)"
        ],
        "fallback_methods": [
          "Web.actionsClickOnElement(WebElement)",
          "Web.jsClick(WebElement)"
        ],
        "expert_guidance": "Start with clickOnElement. Use Actions click for complex hover/action chains. Use jsClick only when normal click fails due to browser or DOM interaction issues."
      },
      {
        "intent": "Enter text",
        "preferred_methods": [
          "Web.setTextToTextBox(WebElement, CharSequence...)",
          "Web.setTextToTextBox(textBoxFieldName, pageClassObj, CharSequence...)"
        ],
        "fallback_methods": [
          "Web.setTextToAngularTextBox(WebElement, String)"
        ],
        "expert_guidance": "For Angular pages or partial sendKeys issues, use the Angular/character-by-character text method and verify the final value."
      },
      {
        "intent": "Select or verify dropdown values",
        "preferred_methods": [
          "Web.selectDropDownOption",
          "Web.getSelectedDropDownOption",
          "Web.verifyDropDownOptionExists",
          "Web.getDropDownOptionAsText",
          "Web.getDropDownOptionCount"
        ],
        "expert_guidance": "Use get/verify methods before selection when test stability depends on dynamic dropdown loading."
      },
      {
        "intent": "Wait for dynamic UI state",
        "preferred_methods": [
          "Web.waitForElement",
          "Web.waitForElements",
          "Web.waitForPageToLoad",
          "Web.waitTillElementIsNotPresent"
        ],
        "avoid": [
          "Web.waitInMillSec unless there is no stable condition to wait for"
        ],
        "expert_guidance": "Prefer condition-based waits over hard-coded waits."
      },
      {
        "intent": "Validate displayed UI state",
        "preferred_methods": [
          "Web.verifyElementIsDisplayed",
          "Web.verifyElementIsEnabled",
          "Web.verifyText",
          "Web.verifyErrorMessage",
          "Web.VerifyText",
          "Web.VerifyPartialText"
        ],
        "expert_guidance": "Use soft verification methods when test should continue and report all validation issues."
      },
      {
        "intent": "Validate backend data",
        "preferred_methods": [
          "DB.executeQuery",
          "DB.getRecordFromRecordSet",
          "DB.getFilteredRowSetCount",
          "DB.compareDB_Date_With_Web_Date"
        ],
        "expert_guidance": "Use parameterized query methods to avoid brittle SQL and to support reusable validation."
      },
      {
        "intent": "Update backend data for setup/cleanup",
        "preferred_methods": [
          "DB.executeUpdate"
        ],
        "expert_guidance": "Use carefully for controlled test data setup or cleanup only; log the action and close connections."
      },
      {
        "intent": "Report a test step",
        "preferred_methods": [
          "Reporter.logEvent"
        ],
        "expert_guidance": "Log status, step name, details, and screenshot choice. For FAIL/WARNING, include stack trace when available."
      }
    ],
    "config_model": {
      "file": "testConfig.properties",
      "required_sections": [
        "Environment Details",
        "App URL",
        "Database Details",
        "Framework Parameters",
        "Browser Drivers location"
      ],
      "common_runtime_properties": [
        "environment",
        "appUrl",
        "dbName",
        "dbHost/dbConnectionString",
        "dbUser",
        "dbPassword or encrypted password",
        "browser",
        "executionType local/grid",
        "nodeUrl for grid",
        "ScreenshotFolder"
      ],
      "agent_guidance": "Before proposing a test implementation, learner agent should check whether the needed config values exist and are environment-specific."
    },
    "maven_model": {
      "purpose": "Build automation and dependency management for Java automation projects.",
      "standard_structure": {
        "src/main/java": "project source code and framework/library usage",
        "src/test/java": "test source code",
        "pom.xml": "central Maven Project Object Model configuration"
      },
      "pom_responsibilities": [
        "declare dependencies such as Selenium, TestNG, and Cucumber",
        "configure plugins such as Surefire and Compiler",
        "define build steps"
      ],
      "commands": [
        {
          "command": "mvn clean",
          "meaning": "Removes previous build files"
        },
        {
          "command": "mvn test",
          "meaning": "Runs test cases"
        },
        {
          "command": "mvn install",
          "meaning": "Compiles, tests, and packages the project"
        }
      ],
      "flow": [
        "pom.xml",
        "Pull dependencies",
        "Use standard project structure",
        "Execute tests",
        "Generate reports"
      ],
      "ci_cd_note": "CI/CD tools can trigger Maven commands automatically."
    },
    "git_branching_model": {
      "application_repository": [
        "Master branch updated once at sprint end",
        "Create one sprint branch from master at sprint beginning",
        "Team pulls from sprint branch for test development",
        "Users merge back only to sprint branch during sprint",
        "Resolve conflicts before pushing to sprint branch",
        "Merge sprint branch to master at sprint end via merge request",
        "Delete user and sprint branches after master merge",
        "Jenkins jobs pull only from master"
      ],
      "page_object_repository": [
        "Master branch updated once at sprint end",
        "Create one sprint branch from master at sprint beginning",
        "Create one branch per application from sprint branch, such as PSC or PartWeb",
        "Users pull application branch for development",
        "Users merge back only to application branch during sprint",
        "Resolve conflicts at application branch to avoid leaking conflicts to sprint branch",
        "Merge sprint branch to master at sprint end via merge request",
        "Delete application, user, and sprint branches after master merge",
        "Jenkins jobs pull only from master"
      ]
    },
    "testng_model": {
      "definition": "TestNG is a Java testing framework with annotations, flexible configuration, parallel execution, data-driven testing, parameters, dependent methods, and broad test category support.",
      "execution_hierarchy": [
        "Suite",
        "Tests",
        "TestNG Class",
        "Test Method"
      ],
      "key_features": [
        "Annotations",
        "Parallel execution using thread pools",
        "Multithread safety testing",
        "Flexible test configuration",
        "@DataProvider data-driven testing",
        "Parameters",
        "Powerful execution model",
        "Tool/plugin support in Eclipse, IDEA, Maven, etc.",
        "Runtime/logging support",
        "Dependent methods",
        "Unit, functional, end-to-end, and integration test support"
      ]
    },
    "learning_units": [
      {
        "id": "lesson.001",
        "title": "What ETF is and why it exists",
        "objectives": [
          "Define test automation framework",
          "Explain ETF abstraction layer",
          "List framework benefits"
        ],
        "concept_refs": [
          "concept.framework",
          "concept.etf_architecture"
        ]
      },
      {
        "id": "lesson.002",
        "title": "Framework application execution flow",
        "objectives": [
          "Explain setup-to-report flow",
          "Map utility classes to execution stages",
          "Describe config-driven execution"
        ],
        "concept_refs": [
          "concept.utility_classes"
        ]
      },
      {
        "id": "lesson.003",
        "title": "Web utility usage",
        "objectives": [
          "Choose correct Web.java method for UI actions",
          "Use waits and verification methods correctly",
          "Handle Angular text entry and dropdowns"
        ],
        "utility_class_refs": [
          "Web.java"
        ]
      },
      {
        "id": "lesson.004",
        "title": "Database utility usage",
        "objectives": [
          "Connect to database",
          "Run query/update operations",
          "Validate web data against database data"
        ],
        "utility_class_refs": [
          "DB.java"
        ]
      },
      {
        "id": "lesson.005",
        "title": "Reporting strategy",
        "objectives": [
          "Log test steps",
          "Attach screenshots",
          "Report warnings/failures with details"
        ],
        "utility_class_refs": [
          "Reporter.java"
        ]
      },
      {
        "id": "lesson.006",
        "title": "Maven, Git, and TestNG operating model",
        "objectives": [
          "Run Maven commands",
          "Follow sprint branching strategy",
          "Understand TestNG hierarchy and capabilities"
        ],
        "concept_refs": [
          "concept.maven",
          "concept.git_strategy",
          "concept.testng"
        ]
      }
    ],
    "qa_expert_best_practices": [
      "Keep test methods readable and delegate repeated actions to ETF utilities.",
      "Prefer Page Object references over raw locators in test logic.",
      "Use explicit/condition waits before interacting with dynamic elements.",
      "Avoid hard-coded waits unless a reliable condition does not exist.",
      "Log business-meaningful steps, not every low-level implementation line.",
      "Capture screenshots for failures and important proof points.",
      "Close database connections and remove WebDriver instances during cleanup.",
      "Keep test data externalized and config-driven.",
      "Use DB validation only for scenarios where UI-only validation is insufficient.",
      "Resolve Git conflicts in lower-level branches before pushing to sprint/master.",
      "Ensure CI/CD jobs run from master after approved merges."
    ],
    "sample_agent_queries_supported": [
      "Which ETF method should I use to click an element?",
      "How do I validate a web value against database value?",
      "What properties should testConfig.properties contain?",
      "Explain Maven automation flow.",
      "What is the correct Git branching strategy for page objects?",
      "How should Reporter.logEvent be used?",
      "What is the TestNG execution hierarchy?"
    ],
    "source_slide_index": [
      {
        "slide_number": 1,
        "summary": "AUTOMATION FRAMEWORK"
      },
      {
        "slide_number": 2,
        "summary": "Introduction Architecture Utility Classes in framework Web Utility Class DB Utility Class Reporter Utility Class Test Config file Working with framework Git Operations | Content"
      },
      {
        "slide_number": 3,
        "summary": "Framework is a structured combination of guidelines, coding standards, tools, and practices designed to efficiently create, maintain, and scale automated tests. | What is a Empower Test Framework (Test Automation Framework) ?"
      },
      {
        "slide_number": 4,
        "summary": "Ensures consistency in test scripts Improves reusability of code Enhances test maintainability Supports scalability for growing projects Integrates with reporting and CI/CD tools | Why Use a Test Automation Framework?"
      },
      {
        "slide_number": 5,
        "summary": "Empower Test Framework ( ETF) Architecture | *ETF Common Libraries and Functions – Reusable functions like Date formatters, Password Decryption, Read properties files, email formats, etc. | ETF | TOOLS | ETF is an abstraction layer which reduces the need for a tester to have tool specific knowledge, provides the much-needed standardization of the common practices across the various automation tools , thereby enabling the testers to focus on developing automated tests quickly and efficiently."
      },
      {
        "slide_number": 6,
        "summary": "Framework State flow diagram"
      },
      {
        "slide_number": 7,
        "summary": "Utility Classes in framework | CommonLib.java : All common usage methods are available DB.java : DataBase related functions are available Log.java : Log related function are available ReadProperties.java : Helps to read test config property file Stock.java : Test data read related functions are available Web.java : UI and Selenium functions are available Reporter.java : Report related functions are available Access all the method with the help of ClassName.MethodName()."
      },
      {
        "slide_number": 8,
        "summary": "Web Utility Class | UI and Selenium Related methods are available Web Utility method are in Web.java Located in the package /src/main/java/bdd_lib Import the class import bdd_lib.web; Methods Details listed"
      },
      {
        "slide_number": 9,
        "summary": "DB Utility Class | Database access related functions are available in the class DB Utility method are in DB.java Located in the package /src/main/java/bdd_lib Import the class import bdd_lib.DB; Methods Details listed"
      },
      {
        "slide_number": 10,
        "summary": "Reporter Utility Class | Reporter Utility class has method helps to generate the extent report Reporter Utility method are in Web.java Located in the package /src/main/java/bdd_reporter Import the class import bdd_reporter.Reporter; Methods Details listed"
      },
      {
        "slide_number": 11,
        "summary": "Test Config File | Enter all the test config property file details in the working projects Details will capture by framework and execution will start accordingly | Environment Details | App URL | Database Details Enter your DB Details | Framework Parameters | Browser Drivers location"
      },
      {
        "slide_number": 12,
        "summary": "Working with Framework | Framework already placed as maven dependency in M2 folder. No other steps are required to import the framework Once Project is imported to your workspace, perform the maven Build. Then you can access framework methods Specify all the testConfig.properties file values in the project"
      },
      {
        "slide_number": 13,
        "summary": "Benefits of Test Automation Frameworks"
      },
      {
        "slide_number": 14,
        "summary": "What is Maven? | Maven is a build automation tool used primarily for Java projects. It helps manage project structure, dependencies, builds, and reports. It standardizes the way projects are built and maintained."
      },
      {
        "slide_number": 15,
        "summary": "Why Maven? | Standard Project Structure Automatic Dependency Management Simple Build Execution Easy Integration with CI/CD Tools Supports Multi-Module Projects"
      },
      {
        "slide_number": 16,
        "summary": "MAVEN - Project Structure | src/main/java directory contains the project source code src/test/java directory contains the test source pom.xml file is the project's Project Object Model, or POM. pom.xml file is the core of a project's configuration in Maven. It is a single configuration file that contains the majority of information required to build a project in just the way you want."
      },
      {
        "slide_number": 17,
        "summary": "MAVEN POM File | Declares project dependencies (Selenium, TestNG, Cucumber, etc.) Configures plugins (Surefire, Compiler, etc.) Defines build steps"
      },
      {
        "slide_number": 18,
        "summary": "Maven Commands | mvn clean : Removes previous build files mvn test : Runs the test cases mvn install : Compiles, tests, and packages the project"
      },
      {
        "slide_number": 19,
        "summary": "Maven + Automation Flow | Flow: pom.xml --> Pulls Dependencies --> Project Structure --> Test Execution --> Reports CI/CD can trigger these commands automatically."
      },
      {
        "slide_number": 20,
        "summary": "Benefits of Maven in Automation Projects | One-click build and execution Centralized dependency version management Standard folder layout across projects Easy integration with Jenkins and GitLab CI/CD"
      },
      {
        "slide_number": 21,
        "summary": "Git Operations"
      },
      {
        "slide_number": 22,
        "summary": "Git Operations"
      },
      {
        "slide_number": 23,
        "summary": "Test Automation Git Branching Strategy | Master branch gets updated once at the end of every sprint One sprint branch should be created from master branch at the beginning of every sprint Only one sprint branch should be created and it will be used by team members across VS Users should always get the pull of Sprint branch for test development Users should only merge back to sprint branch during the sprint All the merge conflicts must be resolved before pushing to Sprint branch Sprint branch sho"
      },
      {
        "slide_number": 24,
        "summary": "Test Automation Git Branching Strategy | Page Object Repository Branching: Master branch gets updated once at the end of every sprint One sprint branch should be created from master branch at the beginning of every sprint Only one sprint branch should be created and it will be used by team members across VS A branch per Application should be created from Sprint branch (Ex: PSC, PartWeb, etc.) Users should always get the pull of Application branch for test development Users should only merge back"
      },
      {
        "slide_number": 25,
        "summary": "TEstng | TestNG is a testing framework, having powerful functionalities and easier to use Annotations. Run your tests in arbitrarily big thread pools with various policies available (all methods in their own thread, one thread per test class, etc...). Test that your code is multithread safe. Flexible test configuration. Support for data-driven testing (with @DataProvider). Support for parameters. Powerful execution model (no more TestSuite). Supported by a variety of tools and plug-ins (Eclipse,"
      },
      {
        "slide_number": 26,
        "summary": "Test NG - Test Structure | Tests in TestNG are structured at 4 levels. Suite->Tests->TestNG Class->Test Method"
      },
      {
        "slide_number": 27,
        "summary": "Test NG Features"
      },
      {
        "slide_number": 28,
        "summary": "Test Cases Template"
      }
    ]
  }
}




        


        


