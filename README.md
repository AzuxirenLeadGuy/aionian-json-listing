# Aionian JSON Listing

This repository contains the [bible resources from aionian](https://www.aionianbible.org/resources/) in mainstream and well documented formats that can be used with other programming environments, languages and frameworks.

Initially, this repository only housed files in JSON format, and was named accordingly. At this time, the scope of this repository has expanded, but the original name remains unchanged.

The files are parsed from the custom `*.noia` format, which contains the following details

- Metadata: Each noia file contain Key-Value tags, representing the metadata for each bible file
- Bible Content: The bible content, containing the index/position of the bible verses and the content of the verse itself.
- Bible Book Names: In between the content lines, the start of each book has its own row as a comment, with the bookname described for it.

Every `.noia` file is converted into the following formats and made available in this repository (under [CC-4.0 License](https://creativecommons.org/licenses/by/4.0)).
---

- **JSON**: The main JSON document contains at least two dictionaries at its root: `meta` containing the metadata, and `index` containing the key-value pair of book id and its corresponding book name. The content is represented by additional dictionaries that contain a inner dictionary of chapters, which in turn contain a dictionary of a verse number mapped to the verse content. The structure can be understood with the following example:

```json
{
    "tags": { // The metadata available in this document
        "meta-key-1": "meta-value-1",
        "meta-key-2": "meta-value-2",
        ...
    },
    "index": { // The books available in this document
        "43":"John",
        "62":"1 John",
        "63":"2 John",
        "64":"3 John",
        
    },
    "43" : { // The content of this book (43 is John in this example)
        "1" : { // The chapter 1 within this book
            "1":"In the beginning was the Word, and the Word was with God, and the Word was God.",
            "2":"He was in the beginning with God.",
            "3":"Everything was made through Him, and without Him nothing was made that was made.",
            "4":"In Him was life, and that life was the light of mankind.",
            ...
        },
        "2" : { // The chapter 2 of this book
            ...
        },
    },
    "62" : { // The content of this book (62 is 1 John in this example)
        "1" { // Chapter 1 of this book
            ...,
        }
        ...,
    },
    ...
}
```
- **TOML**: The sturcture of TOML is identical to JSON. Each TOML file has the following 2 tables at its root `tags` and `index` along with additional tables for each chapter of each book. This makes the TOML files express the same data with slightly lesser characters compared to JSON. The structure of the TOML documents is as follows:

```toml
[tags] # The table containg the metadata for the bible document
'tag-key-1'='tag-value-1'
'tag-key-2'='tag-value-2'
'tag-key-3'='tag-value-3'
[index] # The table containing the books available in this bible document
43='John'
62='1 John'
63='2 John'
64='3 John'
[43.1] # Chapter 1 of book id 43 (John in this example)
1="In the beginning was the Word, and the Word was with God, and the Word was God."
2="He was in the beginning with God."
3="Everything was made through Him, and without Him nothing was made that was made."
4="In Him was life, and that life was the light of mankind."
...
...
[43.2] # Chapter 2 of book id 43
...
...

[62.1] # Chapter 1 of book id 62 (1 John in this example)
...
...
```

- **SQLite**: For use with a relational database, SQLite is a good option. Each SQLite database contains the following tablels
    - **BOOK**
        - Contains the names of books available in this document
        - Schema: `CREATE TABLE BOOK(book_id INT8 PRIMARY KEY, name TEXT)`
        - Rows: `book_id`, `name`
    - **DATA**
        - Contains the actual bible content, provided one verse at a time
        - Schema: `CREATE TABLE DATA(book_id INT8, chapter_id INT8, verse_id INT8, content TEXT)`
        - Rows: `book_id`, `chapter_id`, `verse_id`, `content`

- **TSV**: These are text document prepared as Tab Separated Values. The entire document is a single table with two columns. The first column in the index column, which contains a hexadecimal string of exactly 10 digits (5 Bytes of data). The second column contains the string value for the corresponding index.
    - Types of Rows: There are two types of rows in this table: Book-Header row and Verse-Content row.
        - **Book-Header Row**: This row contains the following information
            - The ID of the book. It is provided as part of the hex string in the first column within the first two digits, i.e index[0:2]
            - The number of chapters contained within this book, provided as part of the hex string in the first column, included within the next two digits, i.e index[2:4]
            - The number of verses contained within this book, provided as part of the hex string in the first column, included within the next four digits, i.e index[4:8]
            - The length of the string in the second column, provided as part of the hex string in the first column, within the next two digits, i.e index[8:10]
            - The name of the book, provided in the second column.
        - **Verse-Content Row**: Following a Book-Header Row, a fixed amount of Verse-Content Rows follow in the document (The fixed amount being the number of verses in the book). This row contains one verse at a time, and includes the following information.
            - The ID of the book of bible containing tbis verse. It is provided as part of the hex string in the first column within the first two digits, i.e index[0:2]
            - The ID of the chapter for this verse. It is provided as part of the hex string in the first column within the next two digits, i.e index[2:4]
            - The ID of the verse. It is provided as part of the hex string in the first column within the next two digits, i.e index[4:6]
            - The size of the string in the second column. It is provided as part of the hex string in the first column within the next four digits, i.e index[6:10]
            - The string content for this verse, provided in the second column.  
    The following example is provided to understand this structure
```tsv
2b15036f04	John
2b0101004f	In the beginning was the Word, and the Word was with God, and the Word was God.
2b01020021	He was in the beginning with God.
2b01030050	Everything was made through Him, and without Him nothing was made that was made.
2b01040038	In Him was life, and that life was the light of mankind.
...
2b02010057	On the third day there was a wedding in Cana of Galilee, and Jesus’ mother was there.
2b0202003d	Now both Jesus and His disciples were invited to the wedding.
...
3e05006906	1 John
3e010100b6	We declare to you that which was from the beginning, which we have heard, which we have seen with our eyes, which we watched, and our hands have touched, concerning the Word of life.
3e0102008c	The life was revealed, and we have seen, and testify, and declare to you this eternal life which was with the Father and was revealed to us.
3e010300a7	That which we have seen and heard we declare to you, that you may also have fellowship with us. Truly our fellowship is with the Father and with His Son, Jesus Christ.
3e0104003a	We write these things to you so that your joy may be full.


```

Here, the first line of the file represents a **Book-Header** row, thus it has the following information:
- Book ID = 0x2b = 43
- Chapter count = 0x15 = 21
- Verse count = 0x036f = 879
- String length = 0x04 = 4
- Book Name: "John"

Since this book contains a total of 879 verses, it implies that 879 rows of **Content-Verse** rows will follow. Parsing the first line, we can get the following information
- Book ID = 0x2b = 43
- Chapter ID = 0x01 = 1
- Verse ID = 0x01 = 1
- String Length = 0x004f = 79
- Verse Content = "In the beginning was the Word, and the Word was with God, and the Word was God."

Thus, after all rows of the **Content-Verse**, a new **Book-Header** row for a different book is followed, 