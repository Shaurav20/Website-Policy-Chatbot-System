# Website-Policy-Chatbot-System
RAG system for querying website policy documents 

**📚 Website Policy Chatbot System**

A complete RAG (Retrieval-Augmented Generation) system for querying a website's (here I have used my website's name, which is Trovinest, in creating the workflows) policy documents using n8n workflows with Pinecone vector database, Google Gemini embeddings, and DeepSeek LLM.

**🎯 Overview**

This system solves the problem of providing accurate, policy-specific information to users while maintaining strict data privacy and conversation structure. It consists of two interconnected n8n workflows:

- **Data Ingestion Workflow** - Processes PDF policy documents into Pinecone vector database
- **Chatbot Workflow** - Provides intelligent Q&A based on stored policy information

**📂 Project Structure**

website-chatbot-system/

├── workflows/

│ ├── data-ingestion.json # PDF to Pinecone workflow

│ └── chatbot.json # Q&A chatbot workflow

├── docs/

│ ├── architecture.md # Detailed architecture

│ ├── setup-guide.md # Step-by-step setup

│ └── troubleshooting.md # Common issues & solutions

├── scripts/

│ ├── test-ingestion.py # Test ingestion process

│ └── validate-responses.py # Validate chatbot outputs

└── README.md # This file

**🚀 Quick Start**

**Prerequisites**

- [n8n](https://n8n.io/) instance (self-hosted or cloud)
- [Pinecone](https://www.pinecone.io/) account
- [Google AI Studio](https://makersuite.google.com/) API key
- [DeepSeek](https://platform.deepseek.com/) API key
- Google Sheets API access

**Step 1: Configure Credentials in n8n**

**Required Credentials:**

- **Pinecone API**
  - Name: PineconeApi account
  - API Key: From Pinecone console
- **Google Gemini API**
  - Name: Google Gemini(PaLM) Api account
  - API Key: From Google AI Studio
- **DeepSeek API**
  - Name: DeepSeek account
  - API Key: From DeepSeek platform
- **Google Sheets API**
  - Name: Google Sheets account
  - OAuth2 credentials

**Step 2: Import Workflows**

- **Data Ingestion Workflow**:
  - Download data-ingestion.json
  - In n8n: Workflows → Import from file
  - Configure:
    - Update PDF URL in HTTP Request node
    - Verify Pinecone index name matches your index
    - Test with Execute workflow
- **Chatbot Workflow**:
  - Download chatbot.json
  - Import into n8n
  - Configure:
    - Update Google Sheets document ID
    - Test chat interface

**Step 3: Set Environment Variables**

bash

_\# Pinecone Configuration_

PINECONE_API_KEY=your_pinecone_api_key

PINECONE_ENVIRONMENT=us-east1-gcp

PINECONE_INDEX_NAME=chatbot-database-2

_\# AI Services_

GOOGLE_GEMINI_API_KEY=your_gemini_api_key

DEEPSEEK_API_KEY=your_deepseek_api_key

_\# Google Sheets_

GOOGLE_SHEETS_DOC_ID=1fDLqzQFbpq9OoyGehMUvs4LGWVG6evyKEuIej7S0dFM

**⚙️ Workflow Details**

**1\. Data Ingestion Workflow**

**Purpose**: Convert PDF policy documents into searchable vector embeddings

**Nodes Flow**:

text

Manual Trigger → HTTP Request → Extract from File → Text Splitter → Document Loader → Google Gemini Embeddings → Pinecone Vector Store

**Key Configurations**:

- **Chunk Size**: 2000 characters
- **Chunk Overlap**: 100 characters
- **Embedding Model**: models/gemini-embedding-001
- **Metadata**: policy_type, document_name, version, source

**2\. Chatbot Workflow**

**Purpose**: Provide intelligent Q&A based on policy documents

**Nodes Flow**:

text

Chat Trigger → AI Agent → Memory Buffer → DeepSeek LLM → Pinecone Retrieval → Response Processing → Google Sheets Logging → Chat Response

**Key Features**:

- **Strict Policy Limitation**: Only answers based on the website's policies
- **User Identity Collection**: Mandatory name and email collection
- **Structured Responses**: JSON format enforcement
- **Conversation Logging**: All interactions stored in Google Sheets

**🎯 Agent Configuration**

The AI Agent is configured with strict instructions:

**Role Definition**

- Polite, respectful, and professional website assistant
- Scope limited to website policies only
- No external knowledge or assumptions

**Workflow Steps**

- **User Identity Collection**: Mandatory name and email before answering
- **Validation**: Strict validation of name and email formats
- **Policy Retrieval**: Context retrieval only from Pinecone vector store
- **Context-Based Answers**: Direct answers with source citations
- **Fallback Response**: When information not found in policies

**Response Format (Strict JSON)**

json

{

"response": "User-facing text",

"user_name": "Collected name or null",

"user_email": "Collected email or null"

}

**🔧 Problem Solving & Implementation**

**Challenge 1: PDF Document Loading Issues**

**Problem**: Google Drive PDFs couldn't be directly loaded into n8n workflow due to authentication and access restrictions.

**Solution**:

- Uploaded the Privacy Policy PDF to my website server's (Hostinger) public.html directory
- Used HTTP Request node with direct URL: <https://trovinest.com/Privacy%20Policy.pdf>
- Processed through Extract from File node for text extraction

**Challenge 2: Vector Database Dimension Mismatch**

**Problem**: Initial Pinecone index configuration didn't match Google Gemini embedding dimensions.

**Solution**:

- Created new Pinecone index with manual configuration:
  - Vector Type: Dense
  - Dimension: 3072 (optimized for models/gemini-embedding-001)
  - Metric: Cosine similarity (for semantic search)
  - Cloud provider: AWS
- Verified embedding compatibility between Gemini and Pinecone

**Challenge 3: Intelligent Chunking for Policy Documents**

**Problem**: Policy documents contain structured information that needs proper segmentation for accurate retrieval.

**Solution**: Implemented Recursive Character Text Splitter with:

- **Chunk Size**: 2000 characters
- **Chunk Overlap**: 100 characters
- **Strategy**: Recursive splitting by paragraphs, then sentences

**Chunking Optimization Process**:

- **Initial Testing**: Started with default 1000-character chunks
- **Problem**: Policy clauses were being split mid-sentence
- **Adjustment**: Increased to 2000 characters to capture complete clauses
- **Overlap**: Added 100-character overlap to maintain context across chunks
- **Validation**: Tested retrieval accuracy with sample queries

**Challenge 4: Strategic Metadata Implementation**

**Problem**: Retrieved documents lacked contextual information for precise filtering and source attribution.

**Solution**: Enhanced data ingestion with comprehensive metadata:

- **Added metadata fields** in Default Data Loader node:
  - policy_type: "privacy" (document categorization)
  - document_name: "privacy_policy" (specific document identification)
  - version: "2025-12" (version control for policy updates)
  - source: "privacy_policy_page" (traceability to original source)
- **Metadata benefits during chatbot execution**:
  - **Filtered retrieval**: Enables querying specific policy types only
  - **Version-aware responses**: Provides information from correct policy versions
  - **Source attribution**: Cites exact document sources in answers

**Challenge 5: Chatbot Response Control**

**Problem**: AI agent was providing generic information or going outside policy scope.

**Solution**: Implemented strict agent instructions with:

- **Role Enforcement**: Explicit Trovinest assistant definition
- **Scope Limitation**: Hard-coded policy-only responses
- **Validation Workflow**: Mandatory user identity collection
- **JSON Enforcement**: Structured response format

**Agent Instruction Structure**:

ROLE & TONE → SCOPE LIMITATION → STEP 0: INTRO → STEP 1: VALIDATION →

STEP 2: RETRIEVAL → STEP 3: ANSWER → STEP 4: FALLBACK → STEP 5: OUTPUT

**Challenge 6: Conversation State Management**

**Problem**: Maintaining user identity and conversation context across sessions.

**Solution**:

- **Memory Buffer Window**: Stores recent conversation history
- **Structured Data Collection**: Enforces name/email collection
- **Session Tracking**: Unique session IDs for each chat
- **Persistent Storage**: Google Sheets for all interactions

**Challenge 7: Error Handling in Production**

**Problem**: Workflow failures without proper logging or recovery.

**Solution**:

- **Google Sheets Logging**: All interactions with timestamps
- **Conditional Logic**: IF node for validation checks
- **Structured Error Messages**: Clear feedback to users
- **Execution Monitoring**: n8n built-in execution history

**📊 Data Storage**

**Pinecone Vector Database**

- **Index**: chatbot-database-2
- **Dimensions**: 3072
- **Metric**: Cosine similarity
- **Content**: Trovinest policy documents with metadata
- **Metadata Fields**: policy_type, document_name, version, source

**Google Sheets Logging**

- **Document ID**: 1fDLqzQFbpq9OoyGehMUvs4LGWVG6evyKEuIej7S0dFM
- **Sheet Name**: Chat_data
- **Logged Fields**:
  - Timestamp: Conversation time
  - User Message: Original query
  - Name: Collected user name
  - AI_Response: System response
  - Email ID: Collected email
  - Session ID: Unique session identifier

**🔍 Testing the System**

**Test Ingestion Process**

- Execute Data Ingestion workflow manually
- Monitor Pinecone dashboard for vector count increase
- Verify metadata accuracy in stored vectors
- Check for proper chunk segmentation

**Test Chatbot Functionality**

- **Initial Interaction**:

text

User: "What is your privacy policy?"

Expected: Request for name and email

- **Identity Collection**:

text

User: "John Doe, <john@email.com>"

Expected: Validation and storage

- **Policy Query**:

text

User: "How is my data protected?"

Expected: Policy-specific answer with citations

- **Out-of-Scope Query**:

text

User: "What's the weather today?"

Expected: Fallback response about policy scope

**🐛 Troubleshooting**

**Common Issues & Solutions**

**Issue 1: PDF Extraction Fails**

**Symptoms**: No text output from Extract from File node  
**Solutions**:

- Verify PDF is accessible via URL
- Check if PDF is text-based (not scanned images)
- Test with smaller PDF file first
- Use alternative PDF extraction methods

**Issue 2: Pinecone Connection Errors**

**Symptoms**: "Index not found" or "Authentication failed"  
**Solutions**:

- Verify Pinecone API key in credentials
- Check index name matches exactly
- Ensure index is in "Ready" state
- Validate environment region

**Issue 3: Chatbot Not Collecting Identity**

**Symptoms**: Agent answers without asking for name/email  
**Solutions**:

- Check system prompt in AI Agent node
- Verify memory buffer configuration
- Test agent with simplified instructions first
- Check JSON parsing in Code node

**Issue 4: Slow Response Times**

**Symptoms**: Chat responses take >10 seconds  
**Solutions**:

- Reduce chunk size for faster retrieval
- Limit retrieved documents count
- Consider caching frequent queries

**Issue 5: Google Sheets Permission Errors**

**Symptoms**: "Permission denied" when appending data  
**Solutions**:

- Re-authenticate Google Sheets credentials
- Share spreadsheet with service account
- Check OAuth scopes in credential setup
- Verify sheet name and ID

**Testing Requirements**

**Must Test**:

- Ingestion with new document types
- Chatbot with edge cases
- Error scenarios
- Performance under load

**Validation Criteria**:

- No API keys in exported workflows
- All nodes properly connected
- Error handling functional
- Documentation updated

**Code Standards**

- **n8n Workflows**: Clear node naming, comments where needed
- **Documentation**: Complete, accurate, up-to-date
- **Configuration**: Environment variables, not hard-coded values
- **Security**: No sensitive data in repository

**📄 License**

\[Specify your license - e.g., MIT, Proprietary\]

**For Technical Support**:

- Check troubleshooting guide first
- Review n8n documentation
- Contact system administrator

**Lessons Learned**

- **Chunking is Critical**: Document structure determines optimal chunk size
- **Agent Instructions Matter**: Clear, strict prompts prevent scope creep
- **Testing is Essential**: Multiple test scenarios save production issues
- **Monitoring is Non-Negotiable**: Proactive monitoring prevents major failures

**Last Updated**: \[Current Date\]  
**Version**: 1.0.0  
**Maintainer**: \[Your Name/Team\]