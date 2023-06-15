# Reproduction Process (Design Process)

## Reproduction Process (Design Process)

Open the VS Code editor and create a file named app.py.

Then open the terminal, preferably using the Git Bash terminal, and enter the following commands to set up the compilation environment:

~~~
pip install langchain pypdf2 python-dotenv streamlit
~~~

Streamlit is an open-source Python library used for rapidly building data applications. It helps developers easily create interactive web applications without requiring frontend development experience. Streamlit is designed to allow data scientists and engineers to focus on data analysis and model building rather than spending a lot of time on interface design and web development.

Langchain is a powerful framework designed to assist developers in building end-to-end applications using language models. It provides a set of tools, components, and interfaces to simplify the process of creating applications supported by large language models (LLMs) and chat models. Langchain allows easy management of interactions with language models, connecting multiple components together, and integrating additional resources such as APIs and databases.

PyPDF2 is a pure Python library for processing PDF files. It can be used to extract information from PDF documents, split, merge, and crop pages, as well as add content and annotations to PDF documents. PyPDF2 does not directly handle text or graphics rendering; it primarily focuses on manipulating the structure and content of PDF files.

Create a .env file to store the parameters:

~~~
OPENAI_API_KEY = "Please input your API_KEY"
~~~

Please note that this variable must be named 'OPENAI_API_KEY' because we are referencing an external source from Langchain, and it only recognizes this variable name.

Create a .gitignore file and copy all the files from gitignore/Python.gitignore at main · github/gitignore. There is nothing special to explain about this step.[gitignore/Python.gitignore at main · github/gitignore](https://github.com/github/gitignore/blob/main/Python.gitignore)

Write a simple Streamlit application in app.py using Python, which creates a PDF file upload feature on a web page:

~~~
from dotenv import load_dotenv
import streamlit as st

def main():
    load_dotenv()
    st.set_page_config(page_title="Ask your PDF")
    st.title("Ask your PDF")
    
if __name__ == "__main__":
    main()
~~~

## Reproduction Steps (Detailed Explanation)

### Workflow Diagram

![Ask a Book Questions Workflow](http://bennycheung.github.io/images/ask-a-book-questions-with-langchain-openai/Ask_Book_Questions_Workflow.jpg)

1. Extract Text
2. Split Text
3. Embedding Process and Knowledge Base Creation
4. Rank Similarity Based on Input Embeddings

### Extract Text

Refine the previous code to extract the text content from the uploaded PDF file.

~~~
from dotenv import load_dotenv
import streamlit as st
from PyPDF2 import PdfReader

def main():
    load_dotenv()
    st.set_page_config(page_title="Ask your PDF")
    st.title("Ask your PDF")
    pdf = st.file_uploader("Upload your PDF", type="pdf")
    if pdf is not None:
        pdf_reader = PdfReader(pdf)
        text = ""
        for page in pdf_reader.pages:
            text += page.extract_text()
~~~

Here is the detailed explanation of the code:

1. Import libraries:
- from dotenv import load_dotenv: Import the load_dotenv function from the dotenv library to load environment variables from the .env file.
- import streamlit as st: Import the streamlit library and set its alias as st for convenience.
- from PyPDF2 import PdfReader: Import the pdfreader function from the PyPDF2 library for reading PDF files.

2. Define the main function:
- load_dotenv(): Call the load_dotenv function to load environment variables.
- st.set_page_config(page_title="Ask your PDF"): Use the st.set_page_config function to set the web page's title as "Ask your PDF".
- st.title("Ask your PDF"): Use the st.title function to display the title "Ask your PDF" on the web page.
- pdf = st.file_uploader("Upload your PDF", type="pdf"): Use the st.file_uploader function to create a file uploader on the web page, allowing users to upload PDF files. The prompt text for the file uploader is "Upload your PDF", and it only accepts PDF files.
- if pdf is not None:: Use an if statement to check if a PDF file has been uploaded by the user (i.e., pdf is not None). If a file has been uploaded, execute the following code block:
- pdf_reader = PdfReader(pdf): Create a PdfReader instance to read the uploaded PDF file.
- text = "": Initialize an empty string variable text to store the extracted text from the PDF file.
- for page in pdf_reader.pages:: Iterate through each page of the PDF file and execute the following code block:
- text += page.extract_text(): Use the extract_text() method to extract the text from the current page and append it to the text variable.

   ### Split Text

  Next, split the extracted text into smaller chunks of text. Continuing from the previous code:

   ~~~
   from dotenv import load_dotenv
   import streamlit as st
   from PyPDF2 import PdfReader
   from langchain.text_splitter import CharacterTextSplitter
   
   def main():
       load_dotenv()
       st.set_page_config(page_title="Ask your PDF")
       st.title("Ask your PDF")
   
       pdf = st.file_uploader("Upload your PDF", type="pdf")
   
       if pdf is not None:
           pdf_reader = PdfReader(pdf)
           text = ""
           for page in pdf_reader.pages:
               text += page.extract_text()
           # split into chunks
           text_splitter = CharacterTextSplitter(
               separator="\n",
               chunk_size=1000,
               chunk_overlap=200,
               length_function=len,
           )
           chunks = text_splitter.split_text(text)
   ~~~

   Next, split the extracted text into smaller chunks of text. Continuing from the previous code:

   `from langchain.text_splitter import CharacterTextSplitter`: Import the CharacterTextSplitter class from the langchain.text_splitter module.
  
   - `text_splitter = CharacterTextSplitter(...)`: Create an instance of CharacterTextSplitter to split the extracted text into smaller chunks. The following parameters are explained:
     - `chunk_size=1000`: Set the text separator as a newline character.
     - `chunk_overlap=200`: Set the maximum length of each text chunk to 1000 characters.
     - `length_function=len`: Set the overlap length between adjacent text chunks to 200.
   - `chunks = text_splitter.split_text(text)`: This method splits the extracted text text into multiple smaller text chunks and stores the result in the chunks variable.

### Embedding Process and Building Knowledge Base

This code segment uses OpenAIEmbeddings to create embedding vectors for these text chunks and stores them in a FAISS vector store (knowledge base), following the previous code:

~~~
from dotenv import load_dotenv
import streamlit as st
from PyPDF2 import PdfReader
from langchain.text_splitter import CharacterTextSplitter
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS

def main():
    load_dotenv()
    st.set_page_config(page_title="Ask your PDF")
    st.title("Ask your PDF")

    pdf = st.file_uploader("Upload your PDF", type="pdf")

    if pdf is not None:
        pdf_reader = PdfReader(pdf)
        text = ""
        for page in pdf_reader.pages:
            text += page.extract_text()

        # split into chunks
        text_splitter = CharacterTextSplitter(
            separator="\n",
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len,
        )

        chunks = text_splitter.split_text(text)

        # create embeddings
        embeddings = OpenAIEmbeddings()
        knowledge_base = FAISS.from_texts(chunks, embeddings)
~~~

- `from langchain.embeddings.openai import OpenAIEmbeddings`: Import the OpenAIEmbeddings class from the langchain.embeddings.openai module.
- `from langchain.vectorstores import FAISS`: Import the FAISS class from the langchain.vectorstores module.
- `embeddings = OpenAIEmbeddings()`: Create an instance of OpenAIEmbeddings to generate embedding vectors for the text.
- `knowledge_base = FAISS.from_texts(chunks, embeddings)`: Call the FAISS.from_texts method to generate embedding vectors for each text chunk in chunks using the embeddings instance and store these vectors in a FAISS vector store (knowledge base).

### Ranking Similarity Based on Input Embedding

This code segment adds the functionality to receive user input queries and calculate the similarity between the queries and the text chunks. It then uses the load_qa_chain function from the langchain library to load a pre-trained question-answering model. Building upon the previous code, here is the updated code:

~~~
from dotenv import load_dotenv
import streamlit as st
from PyPDF2 import PdfReader
from langchain.text_splitter import CharacterTextSplitter
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chains.question_answering import load_qa_chain
from langchain.llms import OpenAI

def main():
    load_dotenv()
    st.set_page_config(page_title="Ask your PDF")
    st.title("Ask your PDF")

    pdf = st.file_uploader("Upload your PDF", type="pdf")

    if pdf is not None:
        pdf_reader = PdfReader(pdf)
        text = ""
        for page in pdf_reader.pages:
            text += page.extract_text()

        # split into chunks
        text_splitter = CharacterTextSplitter(
            separator="\n",
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len,
        )

        chunks = text_splitter.split_text(text)

        # create embeddings
        embeddings = OpenAIEmbeddings()
        knowledge_base = FAISS.from_texts(chunks, embeddings)

        user_question = st.text_input("Ask your question")
        if user_question:
            docs = knowledge_base.similarity_search(user_question)
            llm = OpenAI()
            chain = load_qa_chain(llm, chain_type="stuff")
            response = chain.run(input_documents=docs, question=user_question)
            st.write(response)
~~~

- `user_question = st.text_input("Ask your question")`:Uses the st.text_input function to create a text input box on the web page, allowing the user to enter a question. The prompt text for the input box is set as "Ask your question".
- `if user_question:`: Uses an if statement to check if the user has entered a question. If the user has entered a question (i.e., user_question is not empty), the following code block is executed:
  - `docs = knowledge_base.similarity_search(user_question)`:Calls the knowledge_base.similarity_search method to calculate the similarity between the user's question and the text chunks, and stores the results in the docs variable.
  - `response = chain.run(input_documents=docs, question=user_question)`:Uses the modified chain.run method to process the user's input question. The input documents (i.e., text chunks similar to the query) and the question are passed as parameters to the run method.
  - `st.write(response)`: Displays the output of the processed question-answer model on the web page.
