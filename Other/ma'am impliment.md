Let's break down the steps needed to implement and run the provided code effectively. I'll guide you through the process, ensuring you understand each step and can run the code successfully.

### **Step 1: Set Up Your Environment**

1. **Install Required Python Packages**
   - Ensure you have Python installed. You can check your Python version by running `python --version` in your terminal.
   - Install the required libraries using pip:
     ```bash
     pip install requests beautifulsoup4 pandas nltk textstat
     ```

2. **Prepare Your Directory Structure**
   - Create a directory where you will place your `Input.xlsx`, `Output Data Structure.xlsx`, and the script files.
   - The structure should look something like this:
     ```
     /project-directory/
     ├── Input.xlsx
     ├── Output Data Structure.xlsx
     ├── script.py              # The script you provided
     ├── Articles/              # This will be created by the script
     ```

### **Step 2: Implement the Data Extraction Script**

Copy the provided code for **Objective 1** into a file, say `extract_articles.py`, and place it in your project directory.

**Code:**
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import os

# Load the Excel file
input_file = 'Input.xlsx'  # Update this path if necessary
input_data = pd.read_excel(input_file)

# Create a directory to save the articles
output_dir = 'Articles'
if not os.path.exists(output_dir):
    os.makedirs(output_dir)

# Function to extract article title and text
def extract_article_text(url):
    response = requests.get(url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Modify these selectors according to the website's structure
        title = soup.find('h1')  # Assuming the title is in an <h1> tag
        article_text = soup.find_all('p')  # Assuming paragraphs are in <p> tags
        
        if title:
            title_text = title.get_text().strip()
        else:
            title_text = "No Title Found"
        
        article_content = "\n".join([p.get_text() for p in article_text])
        
        return title_text, article_content
    else:
        return None, None

# Iterate over each URL and extract the article text
for index, row in input_data.iterrows():
    url_id = row['URL_ID']
    url = row['URL']
    
    title, content = extract_article_text(url)
    
    if title and content:
        # Save the extracted text to a file
        output_file = os.path.join(output_dir, f"{url_id}.txt")
        with open(output_file, 'w', encoding='utf-8') as f:
            f.write(f"{title}\n\n{content}")
        print(f"Article {url_id} saved successfully.")
    else:
        print(f"Failed to extract article {url_id}.")

print("All articles processed.")
```

### **Step 3: Run the Data Extraction Script**

1. **Run the Script**:
   - Ensure `Input.xlsx` is in the same directory as `extract_articles.py`.
   - Run the script in your terminal:
     ```bash
     python extract_articles.py
     ```

2. **Check the Output**:
   - After running the script, an `Articles` directory should be created with `.txt` files named after each `URL_ID`. These files will contain the article's title and content.

### **Step 4: Implement the Text Analysis Script**

Now, copy the provided code for **Objective 2** into a new file, say `text_analysis.py`.

**Code:**
```python
import os
import pandas as pd
import re
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize, sent_tokenize
from nltk import download
import textstat

# Download required NLTK data files
download('punkt')
download('stopwords')

# Load the Excel file with URLs and output structure
input_file = 'Input.xlsx'
output_structure_file = 'Output Data Structure.xlsx'

input_data = pd.read_excel(input_file)
output_structure = pd.read_excel(output_structure_file)

# Define positive and negative word lists (sample words, you may need to expand this list)
positive_words = ['good', 'great', 'excellent', 'positive', 'fortunate', 'correct', 'superior']
negative_words = ['bad', 'poor', 'wrong', 'negative', 'inferior', 'unfortunate', 'sad']

# Function to count syllables in a word
def count_syllables(word):
    return textstat.syllable_count(word)

# Function to calculate the required textual analysis metrics
def analyze_text(text):
    word_tokens = word_tokenize(text)
    words = [word for word in word_tokens if word.isalpha()]
    sentences = sent_tokenize(text)
    
    word_count = len(words)
    sentence_count = len(sentences)
    
    # POSITIVE and NEGATIVE SCORE
    positive_score = sum(1 for word in words if word.lower() in positive_words)
    negative_score = sum(1 for word in words if word.lower() in negative_words)
    
    # POLARITY and SUBJECTIVITY SCORE
    polarity_score = (positive_score - negative_score) / (positive_score + negative_score + 0.000001)
    subjectivity_score = (positive_score + negative_score) / (word_count + 0.000001)
    
    # Average Sentence Length
    avg_sentence_length = word_count / sentence_count if sentence_count > 0 else 0
    
    # Complex Word Count
    complex_words = [word for word in words if count_syllables(word) > 2]
    complex_word_count = len(complex_words)
    
    # Percentage of Complex Words
    percentage_complex_words = complex_word_count / word_count if word_count > 0 else 0
    
    # Fog Index
    fog_index = 0.4 * (avg_sentence_length + percentage_complex_words)
    
    # Syllable Per Word
    syllable_per_word = sum(count_syllables(word) for word in words) / word_count if word_count > 0 else 0
    
    # Personal Pronouns
    personal_pronouns = re.findall(r'\b(I|we|my|ours|us)\b', text, re.I)
    personal_pronouns_count = len(personal_pronouns)
    
    # Average Word Length
    avg_word_length = sum(len(word) for word in words) / word_count if word_count > 0 else 0
    
    return {
        'POSITIVE SCORE': positive_score,
        'NEGATIVE SCORE': negative_score,
        'POLARITY SCORE': polarity_score,
        'SUBJECTIVITY SCORE': subjectivity_score,
        'AVG SENTENCE LENGTH': avg_sentence_length,
        'PERCENTAGE OF COMPLEX WORDS': percentage_complex_words,
        'FOG INDEX': fog_index,
        'AVG NUMBER OF WORDS PER SENTENCE': avg_sentence_length,  # Same as AVG SENTENCE LENGTH
        'COMPLEX WORD COUNT': complex_word_count,
        'WORD COUNT': word_count,
        'SYLLABLE PER WORD': syllable_per_word,
        'PERSONAL PRONOUNS': personal_pronouns_count,
        'AVG WORD LENGTH': avg_word_length
    }

# Prepare the output data structure
output_data = []

# Iterate over each URL and perform the analysis
for index, row in input_data.iterrows():
    url_id = row['URL_ID']
    url = row['URL']
    
    # Read the corresponding text file saved earlier
    try:
        with open(f'Articles/{url_id}.txt', 'r', encoding='utf-8') as file:
            text = file.read()
        
        # Perform the analysis
        analysis_results = analyze_text(text)
        
        # Prepare the output row
        output_row = {
            'URL_ID': url_id,
            'URL': url
        }
        output_row.update(analysis_results)
        
        output_data.append(output_row)
        
        print(f"Analysis completed for {url_id}.")
    
    except Exception as e:
        print(f"Failed to process {url_id}: {e}")

# Convert the results to a DataFrame and save to an Excel file
output_df = pd.DataFrame(output_data)
output_df = output_df[output_structure.columns]  # Ensure columns are in the correct order
output_df.to_excel('Textual_Analysis_Output.xlsx', index=False)

print("Textual analysis and data saved successfully.")
```

### **Step 5: Run the Text Analysis Script**

1. **Run the Script**:
   - Ensure `Input.xlsx`, `Output Data Structure.xlsx`, and the `Articles` folder are all in the same directory as `text_analysis.py`.
   - Run the script in your terminal:
     ```bash
     python text_analysis.py
     ```

2. **Check the Output**:
   - After running the script, a `Textual_Analysis_Output.xlsx` file should be generated in the same directory, containing the results of the text analysis.

### **Step 6: Verify the Output**

- Open `Textual_Analysis_Output.xlsx` to verify that the columns are correctly populated according to the structure in `Output Data Structure.xlsx`.

### **Conclusion**

You should now have a working setup to extract article content from URLs, perform text analysis, and save the results to an Excel file. If you encounter any issues or errors, feel free to ask for further assistance!