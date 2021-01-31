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
    name =  Column(String(50),unique=True)
    director =  Column(String(50))
    year  =  Column(Integer)
    earnings = Column(Integer, nullable=True)
    
    def __repr__(self):
        return f"Movie(name={self.name}, year={self.year}, director={self.director}, earnings = {self.earnings})"

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




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None)]



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
    {"name":"The Darkk Knight", "director": director, "year": 2008},
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




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Following, year=1998, director=Christopher Nolan, earnings = None),
     Movie(name=Memento, year=2000, director=Christopher Nolan, earnings = None),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan, earnings = None),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan, earnings = None),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Inception, year=2010, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = None)]



Lets try using filtering


```python
session.query(Movie).filter_by(name="Interstellar").first()
```




    Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = None)



... maybe a little more complex filtering


```python
from sqlalchemy import and_

session.query(Movie).filter(and_(Movie.year>2007, Movie.name.like('The Dark%'))).all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None)]



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




    [Movie(name=Following, year=1998, director=Christopher Nolan, earnings = None),
     Movie(name=Memento, year=2000, director=Christopher Nolan, earnings = None),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan, earnings = None),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan, earnings = None),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Inception, year=2010, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = None)]




```python
from sqlalchemy import desc

session.query(Movie).order_by(desc(Movie.year)).all()
```




    [Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None),
     Movie(name=Inception, year=2010, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan, earnings = None),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan, earnings = None),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan, earnings = None),
     Movie(name=Memento, year=2000, director=Christopher Nolan, earnings = None),
     Movie(name=Following, year=1998, director=Christopher Nolan, earnings = None)]



Let's try using the data from a query

## Aggregation

First let add a few movies to the database - props to `Srashti13` for curating the dataset https://github.com/Srashti13/Movie-Data-set.



```python
import pandas as pd

movies = pd.read_csv('https://raw.githubusercontent.com/Srashti13/Movie-Data-set/master/Moviedf_full2.csv')
```


```python
movies = movies[["Movie.Name", "Year", "Director", "Total.Gross.Earning.million."]]
movies.columns = ["Name", "year", "director", "earnings"]
```


```python
movies.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>year</th>
      <th>director</th>
      <th>earnings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Black Panther</td>
      <td>2018</td>
      <td>Ryan Coogler</td>
      <td>700059566</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Avengers: Infinity War</td>
      <td>2018</td>
      <td>Anthony Russo, Joe Russo</td>
      <td>678815482</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Incredibles 2</td>
      <td>2018</td>
      <td>Brad Bird</td>
      <td>608563044</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Jurassic World: Fallen Kingdom</td>
      <td>2018</td>
      <td>J.A. Bayona</td>
      <td>416769345</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Deadpool 2</td>
      <td>2018</td>
      <td>David Leitch</td>
      <td>318491426</td>
    </tr>
  </tbody>
</table>
</div>




```python
for movie_tuple in movies.iterrows():
    movie = movie_tuple[1]
    exists = session.query(Movie).filter_by(name=movie.Name).first()
    if not exists:
        movie_object = Movie(
            name = movie.Name, 
            year = movie.year, 
            director = movie.director, 
            earnings=movie.earnings
        )
        
        session.add(movie_object)
    
session.commit()
```


```python
session\
    .query(Movie)\
    .order_by(desc(Movie.earnings))\
    .limit(5)\
    .all()
```




    [Movie(name=Star Wars: The Force Awakens, year=2015, director=J.J. Abrams, earnings = 936662225),
     Movie(name=Avatar, year=2009, director=James Cameron, earnings = 749766139),
     Movie(name=Black Panther, year=2018, director=Ryan Coogler, earnings = 700059566),
     Movie(name=Avengers: Infinity War, year=2018, director=Anthony Russo, Joe Russo, earnings = 678815482),
     Movie(name=Jurassic World, year=2015, director=Colin Trevorrow, earnings = 652270625)]



## Group by and aggregate


```python
session\
    .query(
        func.sum(Movie.name)
    ).all()


session\
    .query(func.sum(Item.nominal))\
    .filter(Item.mod.in_ (selected_mods))\
    .group_by(Item.item_type)\
    .all()

```




    [(0.0)]




```python
from sqlalchemy import func

result = session\
    .query(
        Movie.year,
        func.avg(Movie.earnings).label('avg_earnings')
    )\
    .filter(Movie.earnings.isnot(None))\
    .group_by(Movie.year)
```


```python
result.all()
```




    [(2001, 72022668.40816326),
     (2002, 81750440.3367347),
     (2003, 78470774.58585858),
     (2004, 83239208.49),
     (2005, 74872675.85714285),
     (2006, 79145198.73737374),
     (2007, 85367481.36363636),
     (2008, 80702203.48484848),
     (2009, 98637101.44),
     (2010, 91022223.62244898),
     (2011, 90576888.64),
     (2012, 97337851.91836734),
     (2013, 100596875.22),
     (2014, 94919040.54545455),
     (2015, 103213559.59),
     (2016, 103817633.94),
     (2017, 102710781.84),
     (2018, 93537240.24)]



### To print this more nicely


```python
[r._asdict() for r in result]
```




    [{'year': 2001, 'avg_earnings': 72022668.40816326},
     {'year': 2002, 'avg_earnings': 81750440.3367347},
     {'year': 2003, 'avg_earnings': 78470774.58585858},
     {'year': 2004, 'avg_earnings': 83239208.49},
     {'year': 2005, 'avg_earnings': 74872675.85714285},
     {'year': 2006, 'avg_earnings': 79145198.73737374},
     {'year': 2007, 'avg_earnings': 85367481.36363636},
     {'year': 2008, 'avg_earnings': 80702203.48484848},
     {'year': 2009, 'avg_earnings': 98637101.44},
     {'year': 2010, 'avg_earnings': 91022223.62244898},
     {'year': 2011, 'avg_earnings': 90576888.64},
     {'year': 2012, 'avg_earnings': 97337851.91836734},
     {'year': 2013, 'avg_earnings': 100596875.22},
     {'year': 2014, 'avg_earnings': 94919040.54545455},
     {'year': 2015, 'avg_earnings': 103213559.59},
     {'year': 2016, 'avg_earnings': 103817633.94},
     {'year': 2017, 'avg_earnings': 102710781.84},
     {'year': 2018, 'avg_earnings': 93537240.24}]



### Group by and count

Generally a good idea to check averages against the count or standard deviation - general syntax is
```python
session.query(Table.column, func.count(Table.column)).group_by(Table.column).all()
```


```python
session.query(Movie.year, func.count(Movie.year)).group_by(Movie.year).all()
```




    [(1998, 1),
     (2000, 1),
     (2001, 98),
     (2002, 98),
     (2003, 100),
     (2004, 100),
     (2005, 99),
     (2006, 100),
     (2007, 99),
     (2008, 101),
     (2009, 100),
     (2010, 99),
     (2011, 100),
     (2012, 99),
     (2013, 100),
     (2014, 100),
     (2015, 100),
     (2016, 100),
     (2017, 100),
     (2018, 100)]



### Getting number of items


```python
session\
    .query(Movie)\
    .count()
```




    1795



## Rolling back

Let's say we accidently add a movie with the same name, despite having the `Column` set to `unique`


```python
from sqlalchemy import exc
try:
    dark_knight = Movie(name = "The Dark Knight", director = "Christopher Nolan", year = 2008)
    session.add(dark_knight)
    session.commit()
except exc.IntegrityError as e:
    print(e)
```

    (sqlite3.IntegrityError) UNIQUE constraint failed: movie.name
    [SQL: INSERT INTO movie (name, director, year, earnings) VALUES (?, ?, ?, ?)]
    [parameters: ('The Dark Knight', 'Christopher Nolan', 2008, None)]
    (Background on this error at: http://sqlalche.me/e/13/gkpj)


We need to roll back this transaction to restore order


```python
session.rollback()
```


```python
session.query(Movie).filter_by(name="The Dark Knight").all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None)]



## Updating rows

Let's try updating the earnings for all Nolan movies for which we have data.

We first start in `pandas` land


```python
nolan_movies = movies.query("director=='Christopher Nolan'")
nolan_movies
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>year</th>
      <th>director</th>
      <th>earnings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>113</th>
      <td>Dunkirk</td>
      <td>2017</td>
      <td>Christopher Nolan</td>
      <td>188045546</td>
    </tr>
    <tr>
      <th>415</th>
      <td>Interstellar</td>
      <td>2014</td>
      <td>Christopher Nolan</td>
      <td>188020017</td>
    </tr>
    <tr>
      <th>601</th>
      <td>The Dark Knight Rises</td>
      <td>2012</td>
      <td>Christopher Nolan</td>
      <td>448139099</td>
    </tr>
    <tr>
      <th>805</th>
      <td>Inception</td>
      <td>2010</td>
      <td>Christopher Nolan</td>
      <td>292576195</td>
    </tr>
    <tr>
      <th>1000</th>
      <td>The Dark Knight</td>
      <td>2008</td>
      <td>Christopher Nolan</td>
      <td>533345358</td>
    </tr>
    <tr>
      <th>1260</th>
      <td>The Prestige</td>
      <td>2006</td>
      <td>Christopher Nolan</td>
      <td>53089891</td>
    </tr>
    <tr>
      <th>1307</th>
      <td>Batman Begins</td>
      <td>2005</td>
      <td>Christopher Nolan</td>
      <td>205343774</td>
    </tr>
    <tr>
      <th>1640</th>
      <td>Insomnia</td>
      <td>2002</td>
      <td>Christopher Nolan</td>
      <td>67355513</td>
    </tr>
    <tr>
      <th>1787</th>
      <td>Memento</td>
      <td>2001</td>
      <td>Christopher Nolan</td>
      <td>25544867</td>
    </tr>
  </tbody>
</table>
</div>



Let's compare with what we have in `sqlalchemy`


```python
session.query(Movie).filter_by(director="Christopher Nolan").all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Following, year=1998, director=Christopher Nolan, earnings = None),
     Movie(name=Memento, year=2000, director=Christopher Nolan, earnings = None),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan, earnings = None),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan, earnings = None),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Inception, year=2010, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = None),
     Movie(name=Dunkirk, year=2017, director=Christopher Nolan, earnings = 188045546)]




```python
for movie_tuple in nolan_movies.iterrows():
    movie = movie_tuple[1]
    existing_nolan_movie = session.query(Movie).filter_by(name=movie.Name)
    if existing_nolan_movie:
        existing_nolan_movie.update({Movie.earnings: movie.earnings}) 
        
session.commit()
```

We now see that we have correct earnings for all our Nolan movies


```python
session.query(Movie).filter_by(director="Christopher Nolan").all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = 533345358),
     Movie(name=Following, year=1998, director=Christopher Nolan, earnings = None),
     Movie(name=Memento, year=2000, director=Christopher Nolan, earnings = 25544867),
     Movie(name=Insomnia, year=2003, director=Christopher Nolan, earnings = 67355513),
     Movie(name=Batman Begins, year=2005, director=Christopher Nolan, earnings = 205343774),
     Movie(name=The Prestige, year=2006, director=Christopher Nolan, earnings = 53089891),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=Inception, year=2010, director=Christopher Nolan, earnings = 292576195),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = 448139099),
     Movie(name=Interstellar, year=2014, director=Christopher Nolan, earnings = 188020017),
     Movie(name=Dunkirk, year=2017, director=Christopher Nolan, earnings = 188045546)]



## Deleting rows

Let's delete the erronous record with `The Darkk Knight`


```python
session.query(Movie).filter_by(name="The Darkk Knight").delete()
```




    1




```python
session.query(Movie).filter(and_(Movie.year>2007, Movie.name.like('The Dark%'))).all()
```




    [Movie(name=The Dark Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Darkk Knight, year=2008, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Knight Rises, year=2012, director=Christopher Nolan, earnings = None),
     Movie(name=The Dark Tower, year=2017, director=Nikolaj Arcel, earnings = 50701325)]



## More complex data models

<img src="erd.png">


```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import random

Base = declarative_base()

class Item(Base):
    __tablename__ = 'items'
    id = Column(Integer, primary_key=True)
    name =  Column(String(50),unique=True)
    nominal  =  Column(Integer)
    rarity =  Column(String(50))
    
    def __repr__(self):
        return f"Item(name={self.name}, nominal={self.nominal}, rarity={self.rarity})"


class LinkAttachments(Base):
    __tablename__ = "link_attachments"
    id = Column(Integer, primary_key=True)
    itemname = Column(String)
    attachname = Column(String)
    
    def __repr__(self):
        return f"Link(itemname={self.itemname}, attachname={self.attachname})"

class LinkMags(Base):
    __tablename__ = "link_mags"
    id = Column(Integer, primary_key=True)
    itemname = Column(String)
    magname = Column(String)    
    
class LinkBullets(Base):
    __tablename__ = "link_bullets"
    id = Column(Integer, primary_key=True)
    itemname = Column(String)
    bulletname = Column(String)

class LinkBulletMag(Base):
    __tablename__ = "link_bullet_to_mags"
    id = Column(Integer, primary_key=True)
    magname = Column(String)
    bulletname = Column(String)
    
class Attachments(Base):
    __tablename__ = "attachments"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    
    def __repr__(self):
        return f"Attach(name={self.name})"

class Bullets(Base):
    __tablename__ = "bullets"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    bulletcount = Column(Integer, default=7)
    
    def __repr__(self):
        return f"Bullet(name={self.name}, bullet_count={self.bulletcount})"

class Magazines(Base):
    __tablename__ = "magazines"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    magbulletcount = Column(Integer, default=30)
    
    def __repr__(self):
        return f"Mag(name={self.name}, bullets_in_mag={self.magbulletcount})"

    
engine = create_engine('sqlite:///:memory:')

Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)

session = Session()
```


```python
import json

with open('DumpAttatch.json', 'r') as myfile:
    data=myfile.read()
    
attachments = json.loads(data)["HlyngeWeapons"]
```

### Lets load in the items to have some baseline data


```python
for item in attachments:
    item_name = item.get("name")
    exists = session.query(Item).filter_by(name=item_name).first()
    if not exists:
        item_obj = Item(name = item_name, nominal = random.randint(1, 300), rarity = "Common")
        session.add(item_obj)
    
session.commit()
```


```python
session.query(Item).limit(5).all()
```




    [Item(name=access, nominal=237, rarity=Common),
     Item(name=Weapon_Base, nominal=49, rarity=Common),
     Item(name=DefaultWeapon, nominal=192, rarity=Common),
     Item(name=PistolCore, nominal=293, rarity=Common),
     Item(name=RifleCore, nominal=29, rarity=Common)]




```python
attachments[100]
```




    {'name': 'LAW', 'attachments': [], 'bullets': ['Ammo_LAW_HE'], 'magazines': []}



### Next we load in the appropriate attachments


```python
for item in attachments:
    item_name = item.get("name")
    for attach in item.get("attachments"):
        
        exists = session.query(Attachments).filter_by(name=attach).first()
        if not exists:
            item_obj = Attachments(name = attach)
            session.add(item_obj)
        
        exists = session.query(LinkAttachments).filter_by(attachname=attach, itemname=item_name).first()
        if not exists:
            item_obj = LinkAttachments(attachname=attach, itemname=item_name)
            session.add(item_obj)

session.commit()     
```


```python
session.query(Attachments).limit(5).all()
```




    [Attach(name=FNP45_MRDSOptic),
     Attach(name=Crossbow_RedpointOptic),
     Attach(name=PistolOptic),
     Attach(name=AK_WoodBttstck),
     Attach(name=AK_WoodBttstck_Black)]




```python
session.query(LinkAttachments).limit(5).all()
```




    [Link(itemname=Crossbow, attachname=FNP45_MRDSOptic),
     Link(itemname=Crossbow, attachname=Crossbow_RedpointOptic),
     Link(itemname=Crossbow, attachname=PistolOptic),
     Link(itemname=AK101, attachname=AK_WoodBttstck),
     Link(itemname=AK101, attachname=AK_WoodBttstck_Black)]



### An example of how to get all attachments for a specific item


```python
session.query(
         Item, Attachments,
    ).filter(
         Item.name == "Crossbow",
    ).filter(
        Item.name == LinkAttachments.itemname
    ).filter(
        LinkAttachments.attachname == Attachments.name
    ).all()
```




    [(Item(name=Crossbow, nominal=47, rarity=Common),
      Attach(name=FNP45_MRDSOptic)),
     (Item(name=Crossbow, nominal=47, rarity=Common),
      Attach(name=Crossbow_RedpointOptic)),
     (Item(name=Crossbow, nominal=47, rarity=Common), Attach(name=PistolOptic))]



#### Or simpler if we dont need the item


```python
session.query(
         Attachments,
    ).filter(
         Item.name == "Crossbow",
    ).filter(
        Item.name == LinkAttachments.itemname
    ).filter(
        LinkAttachments.attachname == Attachments.name
    ).all()
```




    [Attach(name=FNP45_MRDSOptic),
     Attach(name=Crossbow_RedpointOptic),
     Attach(name=PistolOptic)]



### Adding magazines


```python
for item in attachments:
    item_name = item.get("name")
    for mag in item.get("magazines"):
        
        exists = session.query(Magazines).filter_by(name=mag).first()
        if not exists:
            item_obj = Magazines(name = mag)
            session.add(item_obj)
        
        exists = session.query(LinkMags).filter_by(magname=mag, itemname=item_name).first()
        if not exists:
            item_obj = LinkMags(magname=mag, itemname=item_name)
            session.add(item_obj)

session.commit()
```


```python
session.query(
         Magazines,
    ).filter(
         Item.name == "AK101",
    ).filter(
        Item.name == LinkMags.itemname
    ).filter(
        LinkMags.magname == Magazines.name
    ).all()
```




    [Mag(name=Mag_AK101_30Rnd, bullets_in_mag=30),
     Mag(name=Mag_AK101_30Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_AK101_30Rnd_Green, bullets_in_mag=30)]



### Adding bullets


```python
for item in attachments:
    item_name = item.get("name")
    for bullet in item.get("bullets"):
        
        exists = session.query(Bullets).filter_by(name=bullet).first()
        if not exists:
            item_obj = Bullets(name = bullet)
            session.add(item_obj)
        
        exists = session.query(LinkBullets).filter_by(bulletname=bullet, itemname=item_name).first()
        if not exists:
            item_obj = LinkBullets(bulletname=bullet, itemname=item_name)
            session.add(item_obj)
            
        for mag in item.get("magazines"):
            exists = session.query(LinkBulletMag).filter_by(bulletname=bullet, magname=mag).first()
            if not exists:
                item_obj = LinkBulletMag(bulletname=bullet, magname=mag)
                session.add(item_obj)
            
session.commit()
```


```python
session.query(
         Bullets,
    ).filter(
         Item.name == "AK101",
    ).filter(
        Item.name == LinkBullets.itemname
    ).filter(
        LinkBullets.bulletname == Bullets.name
    ).all()
```




    [Bullet(name=Ammo_556x45, bullet_count=7),
     Bullet(name=Ammo_556x45Tracer, bullet_count=7)]



### Querying which mags fits the bullets 


```python
session.query(
         Magazines
    ).filter(
         Bullets.name == "Ammo_556x45",
    ).filter(
        Bullets.name == LinkBulletMag.bulletname
    ).filter(
        LinkBulletMag.magname == Magazines.name
    ).all()
```




    [Mag(name=Mag_AK101_30Rnd, bullets_in_mag=30),
     Mag(name=Mag_AK101_30Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_AK101_30Rnd_Green, bullets_in_mag=30),
     Mag(name=Mag_STANAG_30Rnd, bullets_in_mag=30),
     Mag(name=Mag_STANAGCoupled_30Rnd, bullets_in_mag=30),
     Mag(name=Mag_CMAG_10Rnd, bullets_in_mag=30),
     Mag(name=Mag_CMAG_20Rnd, bullets_in_mag=30),
     Mag(name=Mag_CMAG_30Rnd, bullets_in_mag=30),
     Mag(name=Mag_CMAG_40Rnd, bullets_in_mag=30),
     Mag(name=Mag_CMAG_10Rnd_Green, bullets_in_mag=30),
     Mag(name=Mag_CMAG_20Rnd_Green, bullets_in_mag=30),
     Mag(name=Mag_CMAG_30Rnd_Green, bullets_in_mag=30),
     Mag(name=Mag_CMAG_40Rnd_Green, bullets_in_mag=30),
     Mag(name=Mag_CMAG_10Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_CMAG_20Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_CMAG_30Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_CMAG_40Rnd_Black, bullets_in_mag=30),
     Mag(name=Mag_M249_Box200Rnd, bullets_in_mag=30)]



And the converse...


```python
session.query(
         Bullets
    ).filter(
         Magazines.name == "Mag_M249_Box200Rnd",
    ).filter(
        Magazines.name == LinkBulletMag.magname
    ).filter(
        LinkBulletMag.bulletname == Bullets.name
    ).all()
```




    [Bullet(name=Ammo_556x45, bullet_count=7),
     Bullet(name=Ammo_556x45Tracer, bullet_count=7)]



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
