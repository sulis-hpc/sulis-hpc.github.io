---
layout: page
title: Database IO
parent: Advanced Topics
nav_order: 2
---
## Database IO

To avoid generating too many files on the disk and risking hitting your quota for files you can use a structured data format to store the data from many jobs in a single file. We'll here briefly describe how you can approach combining IO from multiple jobs

## Returning data in Dask

If you follow the instructions on using Dask then you will see the that the clients return values to the Dask scheduler as they complete. If the data returned by the client is small enough then you can simply store the returned data in memory and write it to disk periodically. When you are combining data from multiple simulations you have to map between the input parameters to the output results. An easy way of storing this mapping on disk is to use a standard structured file format, such as JSON. This code shows a general approach to this problem using Python's built in JSON writer to write output to disk.

```python
from dask.distributed import Client, progress, as_completed
import time
import random
import json
client = Client(threads_per_worker=40, n_workers=1)

def op(val):
    time.sleep(random.random() * 100)
    return val + 1

values = range(10)
futures = {}
#Create the futures for the results and store them
#in a map to map the future to the input value
#Dask future objects are hashable so can be used as a key
#for a dict
for el in values:
    futures[client.submit(op,el)] = el

#Create another map mapping input values to results
#Uses as_complete to get results in the order that they
#become available
results = {}
for el in as_completed(futures.keys()):
    results[futures[el]] = el.result()
    file = open("data.json","w")
    json.dump(results, file)
    file.close()
    print(results)

#Finally write out the sorted results
sorted_results = {key:results[key] for key in sorted(results)}
file = open("data.json","w")
json.dump(sorted_results, file)
file.close()
```

### Database IO

The aim of databases is to structure data so that it can be stored and retrieved easily and quickly. Many database systems work by having a central program that runs continually and is connected to by clients that read and write data. This maps quite well to the idea of writing data from scientific codes and avoids a lot of the problems with IO load on clusters. While not universally true, think of a database as a set of *tables* of data. When you create a database table you specify what types of data it contains by specifying *columns* and then you add *records* that actually contain the data specified by the columns. Generally if you visualise a simple database table it looks like a page in a spreadsheet with column headers at the top and one or more rows of data below them (although databases can get much more complicated than spreadsheets)

The major problem with using databases for storing your data is that most (but not all) databases expect commands to be given to them in a specific language called Structured Query Language or SQL (pronounced either as separate letters or as the word sequel)

SQL is a very simple language and can be learned fairly easily. For example creating a table and adding two records to it looks like

```SQL
CREATE TABLE mytable (id int, name varchar(20));
INSERT INTO mytable VALUES(1, "Bob");
INSERT INTO mytable VALUES(2, "Alice");
COMMIT;
```
but you have to convert the data that you want to store in the database into SQL statements of this form which can be time consuming. Fortunately there is a simpler approach - Object Relational Mapping (ORM). The idea of ORM is that you can combine the idea of object oriented programming (there exist programatic objects that group together related data) and that of the database (There is related data that you can combine into a table for storage). The upshot of this is that you can create classes that describe the columns of a table and then when you create an instance of that class and filling in the member variables you are creating a record that the ORM library be told to store in the database.

Here we're going to introduce one database for Python called SQLAlchemy which includes the ORM functionality. SQLAlchemy also works with multiple database systems so you can easily switch between database systems and keep the core of your code the same. SQLAlchemy is very powerful and this is going to be a very brief overview.

In order to use SQLAlchemy we have to choose which database engine we're going to use. The simplest one is SQLite database engine. This uses a simple file on disk (or you can create a temporary database in memory) to hold a database and there is no central server. First you have to install the SQLAlchemy package for Python so once you have loaded your preferred Python module, install it with `pip3 install --user sqlalchemy`. The next thing to do with an ORM system is define a class that you are going to map to your data. As a simple example consider a problem that takes two floating point value inputs and returns a single floating point output

```Python
from sqlalchemy import create_engine
from sqlalchemy import Column, Date, Integer, Float, String
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine('sqlite:///data.db', echo=True)
Base = declarative_base()

class Data(Base):
    __tablename__ = "data"

    id = Column(Integer, primary_key=True)
    input_I = Column(Float)
    input_J = Column(Float)
    result_K = Column(Float)

    #----------------------------------------------------------------------
    def __init__(self, I, J, K):
        """"""
        self.input_I = I
        self.input_J = J
        self.result_K = K

# create tables
Base.metadata.create_all(engine)
```
This example does four things

1 ) It creates a database engine that handles all of the requests to the database. The URL-like string in the call to create_engine defines both the name and the type of the database, here an sqlite database called data.db. The optional "echo=True" parameter makes the code print what it is doing as it runs and is probably not wanted for a real program.

2 ) It creates an example of an ORM base object using the declarative_base() factory method. This builds an object that can be used to declare object definitions

3 ) It creates the Data object that is a class derived from the base class created by the call to declarative_base. This class is what's going to define the data that we store in the database and instances of this class are going to be the actual data that we store. Note the member "__tablename__" which defines which table in the database these objects are stored in. Note that several variables are created by specifying them as Column(*Type*), where *Type* is the type of the column. Here there are four columns. The first column *id* is an integer that is flagged as the primary key. The primary key of a database is the column that is used to sort the data and is the column that it can be searched on fastest. By default this key increments by 1 for each record

4 ) The final call to Base.metadata.create_all(engine) signals that all of the types that have been created and that the database engine should create the tables with the columns that are specified in the construction of the object.

On it's own this code doesn't do anything useful but if we save it as "define_data.py" we can move on to creating data and reading it back.

Creating data is fairly simple once you have defined the class, but there is a bit of boilerplate.

```Python
from sqlalchemy.orm import sessionmaker
from define_data import *

# create a Session
Session = sessionmaker(bind=engine)
session = Session()

# Create data objects and add them to the session
d = Data(1,2,3)
session.add(d)
session.add(Data(1.2,2.5,9.8))

# Actually store the data into the database
session.commit()

# Create objects again
session.add(Data(1,2,3))
session.add(Data(1.2,2.5,9.8))

# store them again
session.commit()
```

First thing to note is that this python script imports that "define_data" script that we wrote above. Generally any code that makes use of the database (using SQLAlchemy anyway) needs access to the definition for the class and it also uses the engine that we created in that script. You can create a new engine and that will work as well, but it is not necessary.

The next thing that the script does is use another factory method *sessionmaker* to create a sessions. Sessions are transactions to the database both make writing data to the database easier and make it easier to write multiple pieces of data to the database before the database actually writes it to disk which helps with performance. Note the slightly unusual idiom where you first of all create Session (uppercase S) with sessionmaker which creates a class of sessions tied to your engine and then the next line creates session (lowercase s) that is an instance of that class. It isn't necessary to fully understand what's going on here and this code can be used as a black box.

Once you have created your actual connection session you can add data to it, and you do this by calling the session's add method and passing it an instance of the Data class that we defined in define_data. This is done in the same was as creating any Python object by calling the initialiser method. Back in define_data we saw that the initialiser method takes three parameters and uses them to set the values of the columns so the first example creates a data object with (I = 1, J = 2, K = 3) and stores it to the variable d. Then it calls "session.add(d)" to add that data to the database (strictly to the current session, but all going well it will be added to the database). The next line shows that you can simply put the call to the initialiser straight into the call to "session.add".

Once you have added all of the data that you want to commit to the database to the session you call "session.commit()". This actually pushes the data to the database and generally causes the data to be written to disk (it is also possible to cancel a session to discard any data that you said to write in the session, but that isn't common in this use case).

Sessions aren't done with when you call the commit method so you can keep adding more data and then commit it again once you are finished. You don't need to do anything when you are finished with the session and it will all clean up properly when the session object goes out of scope.

Now that you have written data to the database the next question is recovering the data from the database.  This is very similar to writing the data

```Python
from sqlalchemy.orm import sessionmaker
from define_data import *

# create a Session
Session = sessionmaker(bind=engine)
session = Session()

# Create objects  
for record in session.query(Data):
    print (str(record.id), " : ", str(record.input_I), ", ", str(record.input_J), " : ", str(record.result_K))
```

You once again import the define_data file to get access to the definition of our data object and then we once again make a session object. Once you have the session object you can simply iterate over the data stored for a given data object by using "session.query(Data)". Note that the "Data" there without and parameters is just referring to the definition of the data class, not any given instance. The "record" variable in the loop then becomes an instance of the Data class with the instance's members being the values that were store in the database. SQLAlchemy deals with making sure that there is a decent balance of performance and memory usage.

### Using Databases for IO

You can combine database IO with the previous suggestion of using Dask by bringing data back to the scheduler and writing it into the database there. The sessions then make it convenient to combine output until you want to write them to disk. The main advantage of using databases for IO is that they have their own mechanisms for dealing with multiple clients accessing the database at the same time. So long as there aren't *too many* clients trying to write to the database at the same time you can simply have each one of your clients using their own copy of the data writing code and writing output data whenever they want to and the database engine will sort out about making sure that the data that is written is consistent and sensible. Even with a lot of clients writing at the same time performance degrades but consistency of the final data does not.
