# Design Patterns
----

### What is Design Patterns

In software engineering, a **design pattern** is a general repeatable solution to a commonly occurring problem in software design.  

**Design patterns can speed up the development process by providing tested, proven development paradigms.  Design patterns helps to prevent sublet issues that can cause major problems and improve code reability**

Design patterns usually classified into three major categories
 - Creational Patterns
 - Structural Patterns
 - Behavioral Patterns

Each of the pattern type helps to solve certain class of problems. We are not gonig to cover all the patterns in this document, but we will cover the ones we have frequent usage of instead.


Creational Patterns
----

### Singleton

#### Intent
- Ensure a class or object only one instance, and provide a global point of access to it



#### Problem
Application sometimes needs *one and only one* instance of an object during the life time of the application instance.  


#### Solution
It is recommend to name all your singletons in a `singleton.py` file within your project.  Optionally, you can also name the file to whatever you prefer, but expose the singleton object via `__all__` of the module's `__init__.py` 

```python
# in singleton.py

# Only expose the singleton objects you want to be exposed for this module
__all__ = ["s3client"]

class __S3Client():
    def __init__(self, ...):
        # Implementation of your object
        pass

s3client = __S3Client()

```




Structural Patterns
-----

### Adapter




Behavior Patterns
-----

### Chain of responsiblity

