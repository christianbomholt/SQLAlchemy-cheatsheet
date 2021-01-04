## Creating a simple database model


```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Movie(Base):
    __tablename__ = 'movie'
    id = Column(Integer, primary_key=True)
    name =  Column(String(50))
    director =  Column(String(50))
    year  =  Column(Integer)
    
    def __repr__(self):
        return f"Movie(name={self.name}, year={self.year}, director={self.director})"

engine = create_engine('sqlite:///:memory:')

Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)

session = Session()
```

### Adding objects


```python
dark_knight = Movie(name = "The Dark Knight", director = "Christopher Nolan", year = 2008)
session.add(dark_knight)
session.commit()
```

Let's query all the objects to see that `The Dark Knight` got added correctly


```python
session.query(Movie).all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan)]



Note  that the object is printed nicely due to the `__repr__` method

Let's add a few more movies


```python
director = "Christopher Nolan"
movies = [
    {"name":"Following", "director": director, "year": 1998},
    {"name":"Memento", "director": director, "year": 2000},
    {"name":"Insomnia", "director": director, "year": 2003},
    {"name":"Batman Begins", "director": director, "year": 2005},
    {"name":"The Prestige", "director": director, "year": 2006},
#    {"name":"The Dark Knight", "director": director, "year": 2008},
    {"name":"Inception", "director": director, "year": 2010},
    {"name":"The Dark Knight Rises", "director": director, "year": 2012},
    {"name":"Interstellar", "director": director, "year": 2014},
]
```


```python
for movie in movies:
    movie_object = Movie(**movie)
    session.add(movie_object)
    session.commit()
```

We now see that we have a substantial amount of movies


```python
session.query(Movie).all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan),
     Movie(name=Following, year=1998, director=Christopher Nolan),
     Movie(name=Memento, year=2000, director=Christopher Nolan),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan),
     Movie(name=Inception, year=2010, director=Christopher Nolan),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan)]



Lets try using filtering


```python
session.query(Movie).filter_by(name="Interstellar").first()
```




    Movie(name=Interstellar, year=2014, director=Christopher Nolan)



... maybe a little more complex filtering


```python
from sqlalchemy import and_

session.query(Movie).filter(and_(Movie.year>2007, Movie.name.like('The Dark%'))).all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan)]



#### What´s the difference between `filter_by`and `filter`

`filter_by` is used for simple queries on the column names using regular `kwargs`, like

```python
db.users.filter_by(name='Joe')
```

The same can be accomplished with filter, not using `kwargs`, but instead using the `'=='` equality operator, which has been overloaded on the `db.users.name` object:

```python
db.users.filter(db.users.name=='Joe')`

```

You can also write more powerful queries using `filter`, such as expressions like:

```python
db.users.filter(or_(db.users.name=='Ryan', db.users.country=='England'))

```

We can order the data in our table using `order_by`


```python
session.query(Movie).order_by(Movie.year).all()
```




    [Movie(name=Following, year=1998, director=Christopher Nolan),
     Movie(name=Memento, year=2000, director=Christopher Nolan),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan),
     Movie(name=The Dark Knight, year=2008, director=Christopher Nolan),
     Movie(name=Inception, year=2010, director=Christopher Nolan),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan)]




```python
from sqlalchemy import desc
session.query(Movie).order_by(desc(Movie.year)).all()
```




    [Movie(name=Interstellar, year=2014, director=Christopher Nolan),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan),
     Movie(name=Inception, year=2010, director=Christopher Nolan),
     Movie(name=The Dark Knight, year=2008, director=Christopher Nolan),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan),
     Movie(name=Memento, year=2000, director=Christopher Nolan),
     Movie(name=Following, year=1998, director=Christopher Nolan)]



Let's try using the data from a query

## More complex data models


```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import scoped_session,sessionmaker
from zope.sqlalchemy import ZopeTransactionExtension
from sqlalchemy import (
    Column,
    Integer,
    String,
    Boolean,
    ForeignKey,
    DateTime,
    Sequence,
    Float
)
import datetime

DBSession = scoped_session(sessionmaker(extension=ZopeTransactionExtension()))
Base = declarative_base()

class Book(Base):  #<------------------------- 
    __tablename__  = "books"    #matches the name of the actual database table
    id             = Column(Integer,Sequence('book_seq'),primary_key=True) # plays nice with all major database engines
    name           = Column(String(50))                                    # string column need lengths
    author_id      = Column(Integer,ForeignKey('authors.id'))              # assumes there is a table in the database called 'authors' that has an 'id' column
    price          = Column(Float)
    date_added     = Column(DateTime, default=datetime.datetime.now)       # defaults can be specified as functions
    promote        = Column(Boolean,default=False) 
```

## Engine connection string


```python
#the general form of a connection string:
`dialect+driver://username:password@host:port/database` 

#SQLITE:
'sqlite:///:memory:' #store everything in memory, data is lost when program exits
'sqlite:////absolute/path/to/project.db')  #Unix/Mac
'sqlite:///C:\\path\\to\\project.db' #Windows
r'sqlite:///C:\path\to\project.db' #Windows alternative

#PostgreSQL

'postgresql://user:pass@localhost/mydatabase'
'postgresql+psycopg2://user:pass@localhost/mydatabase'
'postgresql+pg8000://user:pass@localhost/mydatabase'

#Oracle
'oracle://user:pass@127.0.0.1:1521/sidname'
'oracle+cx_oracle://user:pass@tnsname'

#Microsoft SQL Server
'mssql+pyodbc://user:pass@mydsn'
'mssql+pymssql://user:pass@hostname:port/dbname'
```
