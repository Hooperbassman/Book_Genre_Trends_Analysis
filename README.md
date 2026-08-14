# Goodreads Book Genre Trends Analysis

## Project Overview

This project analyzes trends in Goodreads book data using Python, Pandas, SQLite, SQL, and data visualization.

The project combines a large Goodreads dataset with a second dataset containing popular Science Fiction and Fantasy books. The data is cleaned and transformed into a relational database so that books, authors, genres, source categories, ratings, publication years, and cross-dataset relationships can be analyzed efficiently.

The primary goal is to investigate how book and genre representation has changed over time and whether genre representation is associated with reader ratings and engagement.

The project combines:

- Data preparation and cleaning
- Relational database design
- Entity-Relationship Diagram (ERD) development
- SQLite database construction
- Cross-dataset integration
- SQL analysis
- Data visualization
- Git and GitHub version control

The primary Goodreads analysis focuses on books published between **1900 and 2016**.

---

## Research Questions

This project explores three primary questions:

1. **How has the representation of Fiction and Nonfiction books changed over time?**

2. **Which specific genres are driving the changes observed in the Fiction and Nonfiction trend?**

3. **How does genre representation relate to average reader ratings and reader engagement?**

The project also uses the secondary Science Fiction and Fantasy dataset to investigate relationships between the two Goodreads data sources.

---

# Data Sources

The project uses two Goodreads-derived datasets.

## Primary Dataset — Goodreads Books

**Goodreads's Books**

- Author: Ishan RealState
- Source: Kaggle
- Original source of the book information: Goodreads
- Dataset license: MIT

Dataset:

[https://www.kaggle.com/datasets/ishanrealstate/goodreads-cleaned-dataset](https://www.kaggle.com/datasets/ishanrealstate/goodreads-cleaned-dataset)

The primary dataset contains book-level information including:

- Book title
- Author
- Publication year
- Average/star rating
- Number of ratings
- ISBN information
- Genre information

The original data was distributed across multiple Parquet files organized by source genre.

---

## Secondary Dataset — Goodreads Science Fiction and Fantasy Books

**Goodreads Pop Science Fiction and Fantasy Books**

- Author: Michael Cai
- Source: Kaggle
- Original source of the book information: Goodreads

Dataset:

[https://www.kaggle.com/datasets/michaelcai2021/goodreads-pop-science-fiction-and-fantasy-books?resource=download](https://www.kaggle.com/datasets/michaelcai2021/goodreads-pop-science-fiction-and-fantasy-books?resource=download)

The second dataset contains popular Goodreads Science Fiction and Fantasy books.

The source files were:

- `goodreads_fan_books_clean.csv`
- `goodreads_sf_books_clean.csv`

The two files were combined into a single dataset and assigned an original source category:

- `Fantasy`
- `Science Fiction`

The secondary dataset contains information including:

- Book title
- Author
- Publication year
- Average rating
- Number of ratings
- Number of shelves
- Series name
- Series number
- Original source genre

This dataset was integrated with the primary Goodreads dataset using normalized book-title and author matching.

---

---

# Data Preparation

The data preparation process is contained in `data_preparation.ipynb`.

The purpose of this notebook is to transform the original datasets into clean, validated datasets suitable for relational database construction.

## Primary Goodreads Dataset

The preparation process includes:

1. Reading the original Parquet files.
2. Combining the source files.
3. Selecting the fields required for the project.
4. Converting publication years and rating fields to appropriate numeric types.
5. Cleaning author information.
6. Cleaning and normalizing genre information.
7. Removing records outside the 1900–2016 publication-year range where appropriate.
8. Creating a normalized `book_key`.
9. Identifying duplicate book records.
10. Creating separate book, author, and genre datasets.
11. Creating many-to-many relationship tables.
12. Preserving the original source genre information.
13. Assigning primary keys.
14. Converting relationship tables to foreign keys.
15. Validating the resulting relationships.
16. Saving the prepared datasets as Parquet files.

## Secondary Dataset

The Science Fiction and Fantasy dataset is prepared separately.

The process includes:

1. Loading the Fantasy and Science Fiction CSV files.
2. Assigning the original `source_genre`.
3. Combining the datasets.
4. Removing unnecessary source index columns.
5. Removing exact duplicate records.
6. Standardizing column names.
7. Converting numeric fields.
8. Cleaning text fields.
9. Creating normalized title and author matching fields.
10. Creating a primary key for each secondary-dataset record.
11. Matching secondary-dataset books against the primary Goodreads dataset.
12. Validating the cross-dataset relationships.
13. Saving the integrated datasets as Parquet files.

---

# Prepared Datasets

The data preparation stage produces the following primary relational datasets:

- `books.parquet`
- `authors.parquet`
- `book_authors.parquet`
- `genres.parquet`
- `book_genres.parquet`
- `source_genres.parquet`
- `book_source_genres.parquet`

The secondary dataset and cross-dataset relationship are stored as:

- `scifi_fantasy_books.parquet`
- `book_dataset_matches.parquet`

The prepared datasets are stored in:

```text
Data/prepared/
```

Keeping the prepared datasets separate from the raw source data makes the database creation process reproducible without requiring the original cleaning process to be repeated.

---

# Cross-Dataset Integration

The two Goodreads datasets are connected through the `book_dataset_matches` relationship table.

Because the datasets do not share a guaranteed common primary key, normalized versions of the book title and author name were created for matching.

The matching process:

1. Normalizes book titles.
2. Normalizes author names.
3. Creates a title/author matching combination.
4. Matches records between the two datasets.
5. Removes duplicate relationships.
6. Assigns a unique `scifi_fantasy_book_id` to the secondary dataset.
7. Connects matching records through `book_dataset_matches`.

The cross-dataset relationship allows the project to compare information from the two sources without merging their records into a single book table.

---

# Relational Database Design

The cleaned data is stored in a relational SQLite database:

```text
Data/goodreads_capstone.db
```

The database uses primary keys, foreign keys, and junction tables to represent relationships between entities.

The final database contains **nine tables**:

1. `books`
2. `authors`
3. `book_authors`
4. `genres`
5. `book_genres`
6. `source_genres`
7. `book_source_genres`
8. `scifi_fantasy_books`
9. `book_dataset_matches`

---

## `books`

Contains the primary Goodreads book records.

Important fields include:

- `book_id`
- `book_key`
- `name`
- `pub_year`
- `star_rating`
- `num_ratings`
- `isbn_clean`

`book_id` is the primary key.

`book_key` provides a normalized unique identifier for the book records.

---

## `authors`

Contains one record for each unique author.

Important fields include:

- `author_id`
- `author_name`

`author_id` is the primary key.

---

## `book_authors`

A junction table establishing the many-to-many relationship between books and authors.

A book may have multiple authors, and an author may have written multiple books.

The composite primary key is:

```text
(book_id, author_id)
```

---

## `genres`

Contains the unique Goodreads genre classifications associated with books.

Important fields include:

- `genre_id`
- `genre_name`

---

## `book_genres`

A junction table establishing the many-to-many relationship between books and genres.

A book may belong to multiple genres, and a genre may contain many books.

The composite primary key is:

```text
(book_id, genre_id)
```

---

## `source_genres`

Contains the original source categories represented by the Goodreads Parquet files.

This table preserves the original source classification separately from the normalized genre relationships.

---

## `book_source_genres`

A junction table identifying which original source categories contained each book.

This allows the project to preserve the original source relationships even when the same book appeared in multiple source datasets.

---

## `scifi_fantasy_books`

Contains the records from the separate Science Fiction and Fantasy dataset.

Important fields include:

- `scifi_fantasy_book_id`
- `title`
- `author`
- `pub_year`
- `avg_rate`
- `num_rate`
- `shelved`
- `series_name`
- `series_num`
- `source_genre`

---

## `book_dataset_matches`

A cross-dataset relationship table connecting matching Goodreads records between the primary and secondary datasets.

Important fields include:

- `book_key`
- `scifi_fantasy_book_id`

The composite primary key is:

```text
(book_key, scifi_fantasy_book_id)
```

This table allows the two datasets to remain independent while still providing a relational connection between matching records.

---

# Entity-Relationship Diagram

The project uses an Entity-Relationship Diagram (ERD) to document the structure of the relational database.

![alt text](images/ERD.png)

The ERD illustrates:

- Primary keys
- Foreign keys
- One-to-many relationships
- Many-to-many relationships
- Junction tables
- The relationship between the primary Goodreads dataset and the secondary Science Fiction/Fantasy dataset

### Database Structure

The database is organized around the `books` entity.

Books connect to authors through `book_authors` and to genres through `book_genres`.

Books also retain their original source classifications through `book_source_genres`.

The secondary Science Fiction and Fantasy dataset is stored independently in `scifi_fantasy_books` and connected to the primary dataset through `book_dataset_matches`.

This structure prevents duplicate book records from being created simply because the same book appears in multiple genres or source datasets.

---

# Project Structure

The project is organized into separate notebooks corresponding to the major stages of the project.

````text
An_Analysis_of_Goodreads_Book_Trends_Over_Time/
│
├── Data/
│   ├── Goodreads_Books/
│   │   └── genres_top100/
│   │
│   ├── SciFi_Fantasy/
│   │   ├── goodreads_fan_books_clean.csv
│   │   └── goodreads_sf_books_clean.csv
│   │
│   ├── prepared/
│   │   ├── books.parquet
│   │   ├── authors.parquet
│   │   ├── book_authors.parquet
│   │   ├── genres.parquet
│   │   ├── book_genres.parquet
│   │   ├── source_genres.parquet
│   │   ├── book_source_genres.parquet
│   │   ├── scifi_fantasy_books.parquet
│   │   └── book_dataset_matches.parquet
│   │
│   └── goodreads_capstone.db
│
├── images/
│   └── ERD.png
│
├── notebooks/
│   ├── data_preparation.ipynb
│   ├── database_creation.ipynb
│   └── SQL_analysis.ipynb
│
├── .gitignore
├── README.md
└── requirements.txt
````

# Requirements

The project requires Python 3 and the following Python libraries:

- pandas
- numpy
- matplotlib
- pyarrow
- sqlite3

`sqlite3` is included with standard Python installations and does not normally need to be installed separately.

---

# Setting Up the Project

## 1. Clone the Repository

Clone the repository from GitHub:

```bash
git clone https://github.com/Hooperbassman/Book_Genre_Trends_Analysis.git
```

Move into the project directory:

```bash
cd Book_Genre_Trends_Analysis
```

---

# Virtual Environment Setup

A virtual environment keeps the project's Python packages isolated from other Python projects.

Python's built-in `venv` module is used to create the environment.

## Windows

Open Command Prompt, PowerShell, or Git Bash from the project directory.

Create the virtual environment:

```bash
python -m venv .venv
```

### Activate on Windows Command Prompt

```cmd
.venv\Scripts\activate
```

### Activate on Windows PowerShell

```powershell
.venv\Scripts\Activate.ps1
```

If PowerShell prevents the activation script from running, the execution policy may need to be adjusted for the current user:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Activate using Git Bash

```bash
source .venv/Scripts/activate
```

---

## macOS and Linux

Create the virtual environment:

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

---

# Installing Dependencies

After activating the virtual environment, install the required packages:

```bash
pip install pandas numpy matplotlib pyarrow
```

If a `requirements.txt` file is included with the project, dependencies can instead be installed with:

```bash
pip install -r requirements.txt
```

---

# Verifying the Virtual Environment

After activation, the terminal should display the virtual environment name, usually:

```text
(.venv)
```

You can also verify which Python executable is being used.

### Windows

```bash
where python
```

### macOS/Linux

```bash
which python
```

The returned path should point to the project's `.venv` directory.

---

# Running the Project

The notebooks should be run in the following order:

### 1. `data_preparation.ipynb`

This notebook:

- Reads the original Goodreads Parquet datasets.
- Combines the source files.
- Cleans publication years.
- Cleans author information.
- Creates unique book keys.
- Processes Goodreads genres.
- Creates normalized relational DataFrames.
- Loads and prepares the Sci-Fi/Fantasy dataset.
- Creates cross-dataset book matches.
- Assigns primary keys.
- Converts relationship tables to foreign keys.
- Validates the prepared data.
- Saves the prepared datasets as Parquet files.

The prepared datasets include:

- `books.parquet`
- `authors.parquet`
- `book_authors.parquet`
- `genres.parquet`
- `book_genres.parquet`
- `source_genres.parquet`
- `book_source_genres.parquet`
- `scifi_fantasy_books.parquet`
- `book_dataset_matches.parquet`

### 2. `database_creation.ipynb`

This notebook:

- Loads the prepared Parquet datasets.
- Creates the SQLite database.
- Creates the relational tables.
- Defines primary keys and foreign keys.
- Enables foreign-key enforcement.
- Inserts the prepared entity and relationship tables.
- Creates the Sci-Fi/Fantasy dataset table.
- Creates the cross-dataset relationship table.
- Validates database row counts.
- Tests relational queries.
- Performs foreign-key integrity checks.
- Inspects the final database schema.

### 3. `SQL_analysis.ipynb`

This notebook performs SQL-based analysis using the completed SQLite database.

The analysis demonstrates intermediate and advanced SQL techniques including:

- Multiple-table joins
- `GROUP BY`
- Aggregate functions
- `HAVING`
- Subqueries
- Common Table Expressions (`WITH`)
- Filtering
- Ordering
- Cross-table analysis

The SQL analysis includes seven primary queries examining:

1. Genre representation
2. Large genre populations
3. Average ratings among highly represented genres
4. Average rating by genre
5. Publication trends over time
6. Genre trends over time
7. Highly represented and highly rated genres

The SQL analysis satisfies the project requirement to include at least three intermediate/advanced SQL queries.

### 4. `EDA.ipynb`

This notebook uses SQL queries and Python visualization tools to answer the project's primary research questions.

The exploratory analysis builds on the results of the data preparation and SQL analysis stages to create the project's final visualizations.

---

# Deactivating the Virtual Environment

When finished working on the project, deactivate the virtual environment with:

```bash
deactivate
```

This command works on Windows, macOS, and Linux.

The virtual environment can be activated again later using the appropriate command described above.

---

# Exploratory Data Analysis

The exploratory analysis tells a three-part story.

## Chart 1 — Fiction vs. Nonfiction Over Time

The first visualization establishes the overall trend by comparing the representation of Fiction and Nonfiction books across publication years from 1900 through 2016.

To create this comparison, original Goodreads source categories were grouped into broader Fiction and Nonfiction classifications.

The purpose of this chart is to answer:

> **How has the representation of Fiction and Nonfiction changed over time?**

### Classification

The project created two analytical groups from the original source categories.

Fiction includes categories such as:

- Fantasy
- Science Fiction
- Romance
- Mystery
- Thriller
- Horror
- Historical Fiction
- Young Adult
- Adventure
- Action
- Paranormal
- Urban Fantasy
- Westerns
- War
- Military Fiction

Nonfiction includes categories such as:

- History
- Biography
- Biography & Memoir
- Science
- Business
- Economics
- Education
- Philosophy
- Psychology
- Politics
- Sociology
- Self Help
- Health
- Religion
- Theology
- Travel
- Nature
- Art
- Music
- Cookbooks
- Cooking

These classifications were created specifically for this analysis and do not alter the original source genre classifications stored in the database.

![alt text](images/chart1_fiction_vs_nonfiction.png)

### Finding

The chart shows that the Fiction and Nonfiction trends show a notable shift in the composition of the dataset over time.

Nonfiction leads Fiction beginning around 1965 and remains ahead for several decades. Around 2010, however, Fiction experiences a dramatic increase and quickly surpasses Nonfiction by a substantial margin.

This sharp reversal suggests that the overall change in the dataset is not simply a gradual shift between Fiction and Nonfiction, but may be driven by significant growth in specific Fiction genres during the later period.

This establishes the broad trend that the remaining analysis attempts to explain.

---

# Chart 2 — Which Genres Are Driving the Change?

The second visualization moves from broad Fiction/Nonfiction categories to individual genres.

The purpose is to answer:

> **Which specific genres are driving the changes observed in the Fiction and Nonfiction trend?**

The analysis divides publication years into two periods:

- **1900–1979**
- **1980–2016**

The 10 most represented genres were identified and their changes in representation between the two periods were calculated.

A positive value indicates that a genre became more represented in the later period.

A negative value indicates that a genre became less represented in the later period.

![alt text](images/chart2_genre_representation_change.png)

### Finding

The results indicate that the genre-level analysis helps explain the shift observed in Chart 1.

Romance, Contemporary, and Fantasy showed the largest increases in representation between 1900–1979 and 1980–2016, while History and Historical Fiction experienced notable decreases.

The growth of Romance and Fantasy—both primarily Fiction genres—corresponds with the sharp rise in Fiction observed in Chart 1. In contrast, the decline in History, a primarily Nonfiction category, is consistent with the weakening relative representation of Nonfiction during the later period.

This step moves the analysis from simply observing that the overall composition of books changed to identifying the specific genres responsible for those changes.

---

# Chart 3 — Genre Representation vs. Reader Ratings

The third visualization examines whether the most represented genres also tend to have higher or lower average Goodreads ratings.

The analysis answers:

> **Does the number of books represented within a genre appear to be related to reader ratings and engagement?**

For each genre, the analysis calculates:

- Number of unique books
- Average Goodreads rating
- Total number of Goodreads ratings

Genres with fewer than 100 books are excluded to reduce the influence of very small samples.

The following categories are also excluded:

- Audiobook
- Ebooks
- Comics
- Picture Books
- Short Stories
- Graphic Novels

These categories were excluded because they describe formats, media types, or literary forms rather than being directly comparable to traditional genres.

They remain in the underlying database and are excluded only from this particular analysis.

![alt text](images/chart3_genre_representation_vs_rating.png)

### Finding

The results indicate that the relationship between genre representation and average reader ratings appears to be relatively weak.

The highly represented genres generally cluster around an average Goodreads rating of approximately 3.8 stars, with the top 10 most represented genres showing very similar ratings.

Overall, there is little noticeable difference in average rating between highly represented genres and the remaining genres in the dataset.

This suggests that greater genre representation does not necessarily correspond to higher or lower reader ratings.

Point size represents the total number of Goodreads ratings, providing an additional indication of reader engagement.

---

# Overall Data Story

The three visualizations work together to tell a progression-based story.

### 1. Broad Trend

The first chart establishes that the representation of Fiction and Nonfiction has changed across the period from 1900 to 2016.

### 2. Drivers of Change

The second chart investigates that change at the genre level, identifying which specific genres became more or less represented between the earlier and later periods.

### 3. Reader Reception

The third chart then examines whether the genres that are most heavily represented also differ in average reader ratings and reader engagement.

Together, the analysis moves from:

**What changed?**

↓

**Which genres caused the change?**

↓

**How are those genres received by readers?**

This provides a more complete picture of how Goodreads book categories have evolved over time rather than relying on a single measure of genre popularity.

---

# Key Findings and Conclusions

Based on the three visualizations, the project evaluates several broader observations about changes in Goodreads book representation and reader ratings between 1900 and 2016.

### Finding 1 — Fiction and Nonfiction Representation

The Fiction and Nonfiction trends show a significant shift in the composition of the dataset over time.

The Fiction/Nonfiction comparison demonstrates that:

**Nonfiction leads Fiction beginning around 1965 and remains ahead for several decades. Around 2010, however, Fiction experiences a dramatic increase and quickly surpasses Nonfiction by a substantial margin.**

This sharp reversal suggests that the overall change in the dataset is not simply a gradual shift between Fiction and Nonfiction, but may be driven by significant growth in specific Fiction genres during the later period.

### Finding 2 — Genre-Level Changes

The genre-level analysis helps explain the shift observed in the Fiction/Nonfiction comparison.

The genre-level comparison shows that:

**Romance, Contemporary, and Fantasy showed the largest increases in representation between 1900–1979 and 1980–2016, while History and Historical Fiction experienced notable decreases.**

The growth of Romance and Fantasy, both primarily Fiction genres, corresponds with the sharp rise in Fiction observed in Chart 1. In contrast, the decline in History, a primarily Nonfiction category, is consistent with the weakening relative representation of Nonfiction during the later period.

These results suggest that the shift toward Fiction was influenced by changes within specific genres rather than being evenly distributed across all categories.

### Finding 3 — Ratings and Genre Representation

The analysis of genre representation and reader ratings suggests that the popularity or representation of a genre does not have a strong relationship with its average Goodreads rating.

The relationship between genre representation and reader ratings suggests:

**Highly represented genres generally cluster around an average Goodreads rating of approximately 3.8 stars, with the top 10 most represented genres showing very similar ratings.**

There is little noticeable difference in average rating between highly represented genres and the remaining genres in the dataset.

This suggests that greater genre representation does not necessarily correspond to higher or lower reader ratings.

Overall, the analysis indicates that **genre representation changed substantially over time, but those changes do not appear to be strongly associated with differences in average reader ratings.**

### Overall Conclusion

The analysis demonstrates that changes in the Goodreads book landscape cannot be explained solely by looking at broad Fiction versus Nonfiction categories.

Examining individual genres reveals which categories are responsible for changes in representation, while the rating analysis provides additional context about how readers respond to those genres.

The relational database structure made it possible to analyze these relationships without treating duplicate appearances of the same book as separate books.

---

# Limitations

Several limitations should be considered when interpreting the results.

### Goodreads Source Categories

The original Goodreads genre classifications are user-generated and may not represent mutually exclusive or standardized literary categories.

A single book can belong to multiple genres.

### Fiction and Nonfiction Classification

The Fiction and Nonfiction groupings used in Chart 1 were created for this project based on the original source categories.

Some categories may contain books that could reasonably be classified differently.

### Publication Years

The analysis is limited to books with publication years between 1900 and 2016.

Books outside this range are not included in the final analysis.

### Dataset Representation

The dataset represents books collected from Goodreads source categories and should not be interpreted as a complete census of all books published during the period.

Therefore, changes in the dataset's composition should not automatically be interpreted as changes in the publishing industry as a whole.

### Ratings

Average Goodreads ratings represent user ratings recorded by Goodreads and may be affected by differences in the number of ratings received by individual books.

For the genre-rating analysis, a minimum threshold of 100 books per genre was used to reduce the effect of very small samples.

---

# Technologies and Tools

This project was developed using:

- **Python**
- **Pandas**
- **NumPy**
- **Matplotlib**
- **PyArrow**
- **SQLite3**
- **Jupyter Notebook**
- **Git**
- **GitHub**

### Python

Python was used for data preparation, transformation, database construction, SQL analysis, and visualization.

### Pandas

Pandas was used for:

- Data cleaning
- Data transformation
- DataFrame manipulation
- Reading and writing Parquet files
- Preparing relational datasets

### NumPy

NumPy was used for numerical operations and handling array-based data during the data preparation process.

### PyArrow

PyArrow was used to read and write the large Parquet datasets efficiently.

### SQLite

SQLite was used to create the relational database and perform SQL-based analysis.

### Matplotlib

Matplotlib was used to create the project's visualizations.

### Git and GitHub

Git was used for version control and GitHub was used to store and document the project source code.

---

# AI and External Assistance

AI assistance was used during the development of this project.

**ChatGPT by OpenAI** was used as a development and editing assistant for:

- Relational database design inspiration
- Entity-Relationship Diagram planning
- Troubleshooting Python and SQLite errors
- Notebook organization
- Markdown documentation
- README development
- Visualization planning
- Editing and improving explanatory text

The final analytical decisions, data validation, interpretation of results, and project implementation were reviewed and tested by the project author.

AI-generated code was executed and validated within the project environment before being incorporated into the analysis.

---

# Academic Project Requirements

This project was designed to satisfy the following capstone requirements:

- Design a relational database and explain the reasoning behind the design.
- Create and include an Entity-Relationship Diagram (ERD).
- Build the database using SQLite3 and Python.
- Include at least three intermediate/advanced SQL queries using techniques such as joins, grouping, aggregation, `HAVING`, and subqueries.
- Include at least three different chart types.
- Create well-designed, labeled visualizations that communicate a clear analytical story.
- Document data sources and provide appropriate credit.
- Document tools, frameworks, libraries, and external assistance used.

---

# Author

**Andrew Hooper**

Goodreads Book Genre Trends Analysis

2026