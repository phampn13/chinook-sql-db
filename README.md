### Load the "Chinook_Sqlite.sqlite" database and verify the connection.


```python
import sqlite3

# Open the chinook database connection
# https://www.kaggle.com/datasets/ranasabrii/chinook?resource=download
conn = sqlite3.connect('./Chinook_Sqlite.sqlite')

# Create a command from the conn
cursor = conn.cursor()

# Execute SQL query
cursor.execute("SELECT COUNT(*) FROM album")

# Get result (one)
count_result  = cursor.fetchone()

print(f"Total of albums: {count_result[0]}")

cursor.close();
```

    Total of albums: 347
    

### Function to print a structure of a SQLite table


```python
def table_info(table_name):
    cursor = conn.cursor()

    # SQLite PRAGMA table_info(table) returns a list of columns in the table
    cursor.execute(f"PRAGMA table_info({table_name});")

    # Fetch all columns
    cols = cursor.fetchall()

    for column in cols:
        print(column)

    print()
    cursor.close();

```


```python
table_info('album')
table_info('track')
```

    (0, 'AlbumId', 'INTEGER', 1, None, 1)
    (1, 'Title', 'NVARCHAR(160)', 1, None, 0)
    (2, 'ArtistId', 'INTEGER', 1, None, 0)
    
    (0, 'TrackId', 'INTEGER', 1, None, 1)
    (1, 'Name', 'NVARCHAR(200)', 1, None, 0)
    (2, 'AlbumId', 'INTEGER', 0, None, 0)
    (3, 'MediaTypeId', 'INTEGER', 1, None, 0)
    (4, 'GenreId', 'INTEGER', 0, None, 0)
    (5, 'Composer', 'NVARCHAR(220)', 0, None, 0)
    (6, 'Milliseconds', 'INTEGER', 1, None, 0)
    (7, 'Bytes', 'INTEGER', 0, None, 0)
    (8, 'UnitPrice', 'NUMERIC(10,2)', 1, None, 0)
    
    

### Retrieve all tracks from all albums.


```python
cursor = conn.cursor()

cursor.execute("""
SELECT a.Title AS AlbumTitle,
       g.Name AS GenreName,
       t.TrackId, t.Name AS TrackName, t.UnitPrice
FROM track AS t
INNER JOIN album AS a ON t.AlbumId = a.AlbumId
INNER JOIN Genre AS g ON t.GenreId = g.GenreId
INNER JOIN MediaType AS mt ON t.MediaTypeId = mt.MediaTypeId
ORDER BY a.Title
LIMIT 10
""")

# Fetch all records
records = cursor.fetchall()

print("All tracks from all albums. LIMIT 10")

for rec in records:
    # 0: AlbumTitle, 1: GenreName, ...
    print(f"{rec[0]}, {rec[1]}, {rec[2]}, {rec[3]}, {rec[4]}")

print()
cursor.close();

```

    All tracks from all albums. LIMIT 10
    ...And Justice For All, Metal, 1893, Blackened, 0.99
    ...And Justice For All, Metal, 1894, ...And Justice For All, 0.99
    ...And Justice For All, Metal, 1895, Eye Of The Beholder, 0.99
    ...And Justice For All, Metal, 1896, One, 0.99
    ...And Justice For All, Metal, 1897, The Shortest Straw, 0.99
    ...And Justice For All, Metal, 1898, Harvester Of Sorrow, 0.99
    ...And Justice For All, Metal, 1899, The Frayed Ends Of Sanity, 0.99
    ...And Justice For All, Metal, 1900, To Live Is To Die, 0.99
    ...And Justice For All, Metal, 1901, Dyers Eve, 0.99
    20th Century Masters - The Millennium Collection: The Best of Scorpions, Rock, 3288, Rock You Like a Hurricane, 0.99
    
    

### Find all albums that have at least 20 tracks.


```python
cursor = conn.cursor()

cursor.execute("""
SELECT a.AlbumId,
       a.Title AS AlbumTitle,
       COUNT(t.TrackId) AS TotalTracks
FROM Track AS t
INNER JOIN album AS a ON t.AlbumId = a.AlbumId
GROUP BY a.AlbumId, a.Title
HAVING COUNT(t.TrackId) >= 20
ORDER BY TotalTracks
""")

# Fetch all records
records = cursor.fetchall()

print("All albums that have at least 20 tracks")

for rec in records:
    print(f"AlbumId={rec[0]}, AlbumTitle={rec[1]}, TotalTracks={rec[2]}")

print()
cursor.close();
```

    All albums that have at least 20 tracks
    AlbumId=37, AlbumTitle=Greatest Kiss, TotalTracks=20
    AlbumId=54, AlbumTitle=Chronicle, Vol. 1, TotalTracks=20
    AlbumId=55, AlbumTitle=Chronicle, Vol. 2, TotalTracks=20
    AlbumId=115, AlbumTitle=Sex Machine, TotalTracks=20
    AlbumId=221, AlbumTitle=My Generation - The Very Best Of The Who, TotalTracks=20
    AlbumId=39, AlbumTitle=International Superhits, TotalTracks=21
    AlbumId=167, AlbumTitle=Acústico MTV, TotalTracks=21
    AlbumId=51, AlbumTitle=Up An' Atom, TotalTracks=22
    AlbumId=224, AlbumTitle=Acústico, TotalTracks=22
    AlbumId=250, AlbumTitle=The Office, Season 2, TotalTracks=22
    AlbumId=24, AlbumTitle=Afrociberdelia, TotalTracks=23
    AlbumId=228, AlbumTitle=Heroes, Season 1, TotalTracks=23
    AlbumId=255, AlbumTitle=Instant Karma: The Amnesty International Campaign to Save Darfur, TotalTracks=23
    AlbumId=83, AlbumTitle=My Way: The Best Of Frank Sinatra [Disc 1], TotalTracks=24
    AlbumId=231, AlbumTitle=Lost, Season 2, TotalTracks=24
    AlbumId=253, AlbumTitle=Battlestar Galactica (Classic), Season 1, TotalTracks=24
    AlbumId=230, AlbumTitle=Lost, Season 1, TotalTracks=25
    AlbumId=251, AlbumTitle=The Office, Season 3, TotalTracks=25
    AlbumId=229, AlbumTitle=Lost, Season 3, TotalTracks=26
    AlbumId=73, AlbumTitle=Unplugged, TotalTracks=30
    AlbumId=23, AlbumTitle=Minha Historia, TotalTracks=34
    AlbumId=141, AlbumTitle=Greatest Hits, TotalTracks=57
    
    

### List all albums and their average track unit prices. Sort the results by average track unit prices in descending order.


```python
cursor = conn.cursor()

cursor.execute("""
SELECT a.AlbumId,
       a.Title AS AlbumTitle,
       ROUND( AVG(t.UnitPrice), 2) AS AVGTrackPrice
FROM track AS t
INNER JOIN album AS a ON t.AlbumId = a.AlbumId
GROUP BY a.AlbumId, a.Title
ORDER BY AVGTrackPrice DESC
LIMIT 10
""")

# Fetch all records
records = cursor.fetchall()

print("All albums and their average track unit prices")

for rec in records:
    print(f"AlbumId={rec[0]}, AlbumTitle={rec[1]}, AVGTrackPrice=${rec[2]}")

print()
cursor.close();

```

    All albums and their average track unit prices
    AlbumId=226, AlbumTitle=Battlestar Galactica: The Story So Far, AVGTrackPrice=$1.99
    AlbumId=227, AlbumTitle=Battlestar Galactica, Season 3, AVGTrackPrice=$1.99
    AlbumId=228, AlbumTitle=Heroes, Season 1, AVGTrackPrice=$1.99
    AlbumId=229, AlbumTitle=Lost, Season 3, AVGTrackPrice=$1.99
    AlbumId=230, AlbumTitle=Lost, Season 1, AVGTrackPrice=$1.99
    AlbumId=231, AlbumTitle=Lost, Season 2, AVGTrackPrice=$1.99
    AlbumId=249, AlbumTitle=The Office, Season 1, AVGTrackPrice=$1.99
    AlbumId=250, AlbumTitle=The Office, Season 2, AVGTrackPrice=$1.99
    AlbumId=251, AlbumTitle=The Office, Season 3, AVGTrackPrice=$1.99
    AlbumId=253, AlbumTitle=Battlestar Galactica (Classic), Season 1, AVGTrackPrice=$1.99
    
    


```python
table_info('Invoice')
table_info('InvoiceLine')
```

    (0, 'InvoiceId', 'INTEGER', 1, None, 1)
    (1, 'CustomerId', 'INTEGER', 1, None, 0)
    (2, 'InvoiceDate', 'DATETIME', 1, None, 0)
    (3, 'BillingAddress', 'NVARCHAR(70)', 0, None, 0)
    (4, 'BillingCity', 'NVARCHAR(40)', 0, None, 0)
    (5, 'BillingState', 'NVARCHAR(40)', 0, None, 0)
    (6, 'BillingCountry', 'NVARCHAR(40)', 0, None, 0)
    (7, 'BillingPostalCode', 'NVARCHAR(10)', 0, None, 0)
    (8, 'Total', 'NUMERIC(10,2)', 1, None, 0)
    
    (0, 'InvoiceLineId', 'INTEGER', 1, None, 1)
    (1, 'InvoiceId', 'INTEGER', 1, None, 0)
    (2, 'TrackId', 'INTEGER', 1, None, 0)
    (3, 'UnitPrice', 'NUMERIC(10,2)', 1, None, 0)
    (4, 'Quantity', 'INTEGER', 1, None, 0)
    
    

### Query the top 10 albums with the highest total number of sold tracks.


```python
cursor = conn.cursor()

cursor.execute("""
SELECT a.AlbumId,
       a.Title AS AlbumTitle,
       COUNT(*) AS SoldTracks
FROM InvoiceLine AS il
INNER JOIN Track AS t ON il.TrackId = t.TrackId
INNER JOIN album AS a ON t.AlbumId = a.AlbumId
GROUP BY a.AlbumId, a.Title
ORDER BY COUNT(*) DESC
LIMIT 10
""")

# Fetch all records
records = cursor.fetchall()

print("Top 10 albums with the highest total number of sold tracks")

for rec in records:
    print(f"AlbumId={rec[0]}, AlbumTitle={rec[1]}, SoldTracks={rec[2]}")

print()
cursor.close();

```

    Top 10 albums with the highest total number of sold tracks
    AlbumId=23, AlbumTitle=Minha Historia, SoldTracks=27
    AlbumId=141, AlbumTitle=Greatest Hits, SoldTracks=26
    AlbumId=73, AlbumTitle=Unplugged, SoldTracks=25
    AlbumId=224, AlbumTitle=Acústico, SoldTracks=22
    AlbumId=37, AlbumTitle=Greatest Kiss, SoldTracks=20
    AlbumId=21, AlbumTitle=Prenda Minha, SoldTracks=19
    AlbumId=55, AlbumTitle=Chronicle, Vol. 2, SoldTracks=19
    AlbumId=221, AlbumTitle=My Generation - The Very Best Of The Who, SoldTracks=19
    AlbumId=39, AlbumTitle=International Superhits, SoldTracks=18
    AlbumId=54, AlbumTitle=Chronicle, Vol. 1, SoldTracks=18
    
    

### Find all albums that have no sales.


```python
cursor = conn.cursor()

cursor.execute("""
SELECT a.AlbumId,
       a.Title AS AlbumTitle
FROM album AS a
WHERE NOT EXISTS (
   SELECT il.* FROM InvoiceLine AS il
   INNER JOIN Track AS t ON il.TrackID = t.TrackID
   WHERE t.AlbumId = a.AlbumID -- Join with the outer query via AlbumID
)
""")

# Fetch all records
records = cursor.fetchall()

print("All albums that have no sales")

for rec in records:
    print(f"AlbumId={rec[0]}, AlbumTitle={rec[1]}")

print()
cursor.close();
```

    All albums that have no sales
    AlbumId=226, AlbumTitle=Battlestar Galactica: The Story So Far
    AlbumId=260, AlbumTitle=Cake: B-Sides and Rarities
    AlbumId=262, AlbumTitle=Quiet Songs
    AlbumId=264, AlbumTitle=Realize
    AlbumId=267, AlbumTitle=Worlds
    AlbumId=268, AlbumTitle=The Best of Beethoven
    AlbumId=272, AlbumTitle=Adorate Deum: Gregorian Chant from the Proper of the Mass
    AlbumId=273, AlbumTitle=Allegri: Miserere
    AlbumId=275, AlbumTitle=Vivaldi: The Four Seasons
    AlbumId=276, AlbumTitle=Bach: Violin Concertos
    AlbumId=277, AlbumTitle=Bach: Goldberg Variations
    AlbumId=281, AlbumTitle=Sir Neville Marriner: A Celebration
    AlbumId=282, AlbumTitle=Mozart: Wind Concertos
    AlbumId=284, AlbumTitle=Beethoven: Symhonies Nos. 5 & 6
    AlbumId=285, AlbumTitle=A Soprano Inspired
    AlbumId=286, AlbumTitle=Great Opera Choruses
    AlbumId=289, AlbumTitle=Tchaikovsky: The Nutcracker
    AlbumId=290, AlbumTitle=The Last Night of the Proms
    AlbumId=291, AlbumTitle=Puccini: Madama Butterfly - Highlights
    AlbumId=293, AlbumTitle=Pavarotti's Opera Made Easy
    AlbumId=294, AlbumTitle=Great Performances - Barber's Adagio and Other Romantic Favorites for Strings
    AlbumId=295, AlbumTitle=Carmina Burana
    AlbumId=296, AlbumTitle=A Copland Celebration, Vol. I
    AlbumId=297, AlbumTitle=Bach: Toccata & Fugue in D Minor
    AlbumId=298, AlbumTitle=Prokofiev: Symphony No.1
    AlbumId=302, AlbumTitle=Mascagni: Cavalleria Rusticana
    AlbumId=305, AlbumTitle=Great Recordings of the Century - Mahler: Das Lied von der Erde
    AlbumId=309, AlbumTitle=Palestrina: Missa Papae Marcelli & Allegri: Miserere
    AlbumId=311, AlbumTitle=Strauss: Waltzes
    AlbumId=313, AlbumTitle=Bizet: Carmen Highlights
    AlbumId=315, AlbumTitle=Handel: Music for the Royal Fireworks (Original Version 1749)
    AlbumId=317, AlbumTitle=Mozart Gala: Famous Arias
    AlbumId=318, AlbumTitle=SCRIABIN: Vers la flamme
    AlbumId=319, AlbumTitle=Armada: Music from the Courts of England and Spain
    AlbumId=328, AlbumTitle=Charpentier: Divertissements, Airs & Concerts
    AlbumId=332, AlbumTitle=The Ultimate Relexation Album
    AlbumId=336, AlbumTitle=Prokofiev: Symphony No.5 & Stravinksy: Le Sacre Du Printemps
    AlbumId=339, AlbumTitle=Great Recordings of the Century: Paganini's 24 Caprices
    AlbumId=341, AlbumTitle=Great Recordings of the Century - Shubert: Schwanengesang, 4 Lieder
    AlbumId=342, AlbumTitle=Locatelli: Concertos for Violin, Strings and Continuo, Vol. 3
    AlbumId=345, AlbumTitle=Monteverdi: L'Orfeo
    AlbumId=346, AlbumTitle=Mozart: Chamber Music
    AlbumId=347, AlbumTitle=Koyaanisqatsi (Soundtrack from the Motion Picture)
    
    


```python
# Close the conn
# conn.close()
```


```python

```
