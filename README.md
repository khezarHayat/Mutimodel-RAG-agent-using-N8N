# Mutimodel-RAG-agent-using-N8N
A multimodal RAG agent built with n8n, Supabase, Gemini Vision, and custom HTML. It accepts PDFs, images, videos, audio, CSVs, and other files, processes their content, stores embeddings in a vector database, and lets users ask questions and receive context-based AI answers.
Multimodal RAG Agent

A multimodal Retrieval-Augmented Generation (RAG) system built with n8n, Supabase, Gemini Vision, embeddings, and custom HTML interfaces.

The system allows users to upload different types of files, process their content, store the extracted information as vector embeddings, and then ask questions about the uploaded content. Instead of being limited to text or PDFs, the workflow is designed to work with multiple forms of data, including documents, images, videos, audio, CSV files, and other supported file types.

Overview

The project is divided into two main workflows:

Multimodal File Processing Workflow
Question and Answer Workflow

The first workflow handles the uploaded file and converts its content into searchable information. The second workflow receives the user's question, retrieves the most relevant information from the vector database, and uses an AI model to generate the final response.

The overall architecture is:

User
 │
 ▼
Custom HTML Interface
 │
 ├── Upload File
 │
 ▼
n8n Upload Workflow
 │
 ▼
File Processing
 │
 ▼
Content Extraction
 │
 ▼
Gemini / Gemini Vision
 │
 ▼
Text + Visual Information
 │
 ▼
Chunking
 │
 ▼
Embeddings
 │
 ▼
Supabase Vector Database
 │
 └── rag_multimodal_documents
              │
              │
              ▼
       Question Workflow
              │
              ▼
        User Question
              │
              ▼
       Question & Answer Chain
              │
              ▼
          Retriever
              │
              ▼
    Supabase Vector Search
              │
              ▼
      Relevant Documents
              │
              ▼
            Gemini
              │
              ▼
        Final AI Answer
1. File Upload

The user starts from a custom HTML interface.

The interface allows the user to upload different types of content rather than restricting the application to a single file format.

Examples include:

PDF
Images
Videos
Audio
CSV
Documents
Other supported files

The uploaded file is sent to an n8n webhook, which starts the processing workflow.

The goal is to make the system flexible enough that users can provide information in different formats without manually converting everything into text first.

2. Multimodal Processing

After receiving the file, the n8n workflow determines how the content should be processed.

For visual information, Gemini Vision is used to understand information contained in images and other visual content.

For example, if the user uploads an invoice image, the system can extract useful information such as:

Company name
Invoice details
Products
Prices
Dates
Descriptions
Other visible information

This information is then converted into useful textual content that can be indexed and searched.

The same concept can be extended to other supported multimedia inputs depending on the processing pipeline.

3. Creating Embeddings

After extracting the useful content, the workflow divides the information into smaller chunks.

These chunks are converted into vector embeddings.

The embeddings allow the system to perform semantic search rather than simply searching for exact words.

For example, a user might ask:

Which company issued this invoice?

Even if the exact words from the question do not appear in a particular chunk, the vector search can identify content that has a similar meaning.

4. Supabase Vector Database

The processed chunks and their embeddings are stored in Supabase using PostgreSQL and pgvector.

The multimodal workflow uses a dedicated table:

rag_multimodal_documents

The table stores information such as:

id
content
metadata
embedding

The content contains the processed information.

The metadata contains additional information about the source and extracted content.

The embedding contains the vector representation used for similarity search.

5. Separate RAG Databases

One important part of this project was separating the original RAG workflow from the new multimodal RAG workflow.

The project contains two separate vector tables:

rag_documents

for the original RAG workflow, and:

rag_multimodal_documents

for the multimodal RAG workflow.

This prevents documents from different workflows from being mixed during retrieval.

I also created separate PostgreSQL matching functions:

match_documents

for the original RAG system, and:

match_mm_documents

for the multimodal RAG system.

The multimodal matching function searches:

FROM rag_multimodal_documents

This separation was important because a RAG system can return incorrect context if documents from different knowledge bases are accidentally searched together.

6. Asking Questions

After the file has been processed, the user can use the same HTML-based interface to ask questions.

For example:

Which company issued this invoice?

The question is sent to a second n8n webhook:

Webhook-ASK-Question

The question is then passed into the Question and Answer Chain.

7. Retrieval

The Question and Answer Chain uses a Retriever connected to the Supabase Vector Store.

The question is converted into an embedding and compared with the embeddings stored in:

rag_multimodal_documents

The system identifies the most relevant chunks from the uploaded content.

Instead of sending the entire database to the AI model, the system retrieves only the information that is most relevant to the question.

This is the core idea behind Retrieval-Augmented Generation.

8. AI Answer Generation

The retrieved information is then provided to the Gemini model along with the user's question.

Gemini uses the retrieved context to generate the final response.

The result is returned to the user through the HTML interface.

For example:

User:
Which company issued this invoice?


RAG:
Searches the uploaded invoice content.


Gemini:
Generates an answer based on the retrieved information.

This makes the system more useful than a simple chatbot because the response is connected to the user's uploaded data.

Technologies Used
n8n

Used to build and connect the complete automation workflow, including:

Webhooks
File processing
Data transformation
AI processing
Embeddings
Retrieval
Question answering
Supabase

Used as the backend database and vector store.

It stores the processed document chunks, metadata, and embeddings.

Gemini / Gemini Vision

Used for AI processing and understanding multimodal content, particularly visual information.

pgvector

Used to store and search vector embeddings inside PostgreSQL.

HTML

A custom HTML interface is used as the frontend for:

File uploads
User questions
Receiving AI responses
Key Features
Multimodal file input
PDF processing
Image understanding
Video and audio workflow support
CSV and document processing
Gemini Vision integration
Automatic content processing
Text chunking
Vector embeddings
Semantic search
Supabase vector database
RAG-based question answering
Separate vector databases for different RAG workflows
Custom HTML interface
n8n workflow automation
Example Workflow

A typical user interaction looks like this:

1. User opens the HTML interface.


2. User uploads a file.


3. The file is sent to n8n.


4. n8n processes the file.


5. Gemini/Gemini Vision extracts useful information.


6. The content is divided into chunks.


7. Embeddings are generated.


8. Data is stored in Supabase.


9. User asks a question.


10. The question is sent to the second n8n workflow.


11. The Retriever searches the multimodal vector database.


12. Relevant chunks are retrieved.


13. Gemini receives the question and retrieved context.


14. Gemini generates the final answer.


15. The answer is returned to the user.
Why I Built This

The goal of this project was to move beyond a basic text-based RAG system and build something that can work with different types of real-world data.

Real-world information is rarely available only as plain text. It can exist inside documents, images, invoices, videos, audio recordings, spreadsheets, and other formats.

A multimodal RAG architecture makes it possible to process these different sources and make their information searchable through natural-language questions.

This project also helped me understand the importance of the complete RAG pipeline:

Input
→ Processing
→ Extraction
→ Chunking
→ Embeddings
→ Vector Storage
→ Retrieval
→ Context
→ LLM
→ Answer

The project is still being improved, with more multimodal capabilities and better retrieval features planned for future versions.
