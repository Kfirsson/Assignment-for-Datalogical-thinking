BEGIN
   DECLARE df AS DATAFRAME BY READING 'NewsDB-Python/datatsets/dataset.csv'
   DECLARE keywords AS LIST OF STRINGS BY SPLITTING 'Keywords' COLUMN BY ';'
   DECLARE keyword_list AS LIST OF STRINGS BY FLATTENING keywords
   DECLARE counter AS COUNTER FOR keyword_list
   SET ten_most_common TO counter.MOST_COMMON(10)

   DECLARE five_most_common AS LIST OF STRINGS BY EXTRACTING KEYWORDS FROM ten_most_common[0:5]
   SET articles_with_top_keywords TO df WHERE 'Keywords' CONTAINS ANY OF five_most_common
   SET articles_with_top_keywords['Description'] TO FIRST 250 CHARACTERS OF articles_with_top_keywords['Description']

   DECLARE authors AS LIST OF STRINGS BY FILTERING df WHERE 'Keywords' CONTAINS ANY OF keywords IN ten_most_common
   SET author_counter AS COUNTER FOR authors
   SET top_ten_authors TO author_counter.MOST_COMMON(10)

   OPEN FILE 'index.html' FOR WRITING
   WRITE HTML CONTENT INCLUDING:
      - HEAD WITH TITLE, STYLESHEETS, AND FONTS
      - BODY CONTAINING:
         - H1 FOR TOP 10 KEYWORDS AND FREQUENCY
         - P WITH RESEARCH AND DATASET QUESTIONS
         - TABLE OF ten_most_common
         - H1 FOR ARTICLES WITH TOP 5 KEYWORDS
         - TABLE OF articles_with_top_keywords
         - H1 FOR TOP 10 AUTHORS AND THEIR KEYWORD FREQUENCY
         - LOOP THROUGH top_ten_authors TO:
            - LIST THEIR ASSOCIATED KEYWORDS AND FREQUENCIES
            - WRITE ARTICLES FOR EACH AUTHOR AS NECESSARY
   CLOSE FILE
END