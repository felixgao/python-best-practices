# Design Patterns
----

### What are Design Patterns?

In software engineering, a **design pattern** is a general repeatable solution to a commonly occurring problem in software design.  

**Design patterns can speed up the development process by providing tested, proven development paradigms. Design patterns help to prevent subtle issues that can cause major problems and improve code readability.**

Design patterns are usually classified into three major categories:
 - Creational Patterns
 - Structural Patterns
 - Behavioral Patterns

Each pattern type helps to solve a certain class of problems. We are not going to cover all the patterns in this document, but we will cover the ones we frequently use.

## Creational Patterns
----

### Singleton

#### Intent
- Ensure a class has only one instance, and provide a global point of access to it.

#### Problem
Applications sometimes need *one and only one* instance of an object during the lifetime of the application instance.

#### Solution
It is recommended to name all your singletons in a `singleton.py` file within your project. Optionally, you can also name the file whatever you prefer, but expose the singleton object via `__all__` of the module's `__init__.py`.

Since we don't want to expose the ability to create a new instance of the object to external modules and Python modules are imported once and globally available after import, it is important to make sure the name of the class starts with `__` (dunder) and use `__all__` to limit the symbols that need to be exported.

```python
# in singleton.py

# Only expose the singleton objects you want to be exposed for this module
__all__ = ["s3client"]

class __S3Client:
    def __init__(self, ...):
        # Implementation of your object
        pass

s3client = __S3Client()
```

## Structural Patterns
-----

### Adapter

#### Intent
- Wrap an existing class with a new interface.
- Wrap an old component to a new system.
- Let classes work together that couldn't otherwise because of incompatible interfaces.

#### Problem
Take OCR as an example. Each provider calls the actual method that does OCR slightly differently. For example, EasyOCR uses a method called `readtext`, Google Vision SDK uses `document_text_detection`, and PyTesseract uses `ocr_client` to obtain the OCR result.

#### Solution

```python
# in interface.py

from abc import ABC, abstractmethod

# Our expected interface
class OCRClient(ABC):
    """
    Interface for OCR Client
    """

    @abstractmethod
    def ocr(self, image):
        pass

# in adapter.py

# Adapter Class
class GVisionOCRClientAdapter(OCRClient):
    def __init__(self):
        self._client = GVisionOCRClientAdaptee()

    def ocr(self, image):
        return self._client.adaptee_ocr(image=image)

# Adaptee Class
class GVisionOCRClientAdaptee:
    def __init__(self):
        self.private_client = vision.from_service_account_json('some_location.json')

    def adaptee_ocr(self, image):
        return self.private_client.document_text_detection(image)
```

## Behavioral Patterns
-----

### Registry

#### Intent
- Keep track of all subclasses of a given class.
- Create objects of subclasses that have different behaviors.

#### Problem
The need for some kind of manager class to manage all related subclasses. You want the manager class to iterate over all available subclasses and call a particular method on each/some subclasses. You want to streamline a factory class, which takes an input and creates/returns an object of that type.

#### Solution

```python
from typing import Type

class Document(ABC):
    _REGISTRY = {}

    def __new__(cls, *args, **kwargs):
        name = cls.__name__
        prefix = "Document"
        assert name.startswith(prefix)
        form_type = name[len(prefix):].lower()
        cls._REGISTRY[form_type] = cls

    @classmethod
    def factory(cls, form_type: str) -> Type[Document]:
        return cls._REGISTRY[form_type]()

    @abstractmethod
    def extract(self, *args, **kwargs):
        pass

class Document1040(Document):
    def extract(self, file: ByteIO, fields: Iterable[str] = None) -> Document1040Results:
        # some extract logic
        pass

class Document1099K(Document):
    def extract(self, file: ByteIO, fields: Iterable[str] = None) -> Document1099KResult:
        # different logic to extract this form
        pass

# To use this
doc_1040 = Document.factory("1040")
result_1040 = doc_1040.extract(["ssn", "agi"])

doc_1099k = Document.factory("1099k")
result_1099k = doc_1099k.extract()

# A more powerful use case would be in a loop, you will not see large if/else blocks to 
# process files based on the file type.
for doc_content, doc_type in all_documents:
    extractor = Document.factory(doc_type)
    result = extractor.extract()
```

### Chain of Responsibility

#### Intent
When an object needs to be processed by a potential chain of successive handlers. Each handler may process it, pass it over, or break the chain and stop the task from propagating to the next handler.

#### Problem
The need for decoupling the sender of a request and its receiver. It is unknown until runtime what kind of request the object is, therefore, it needs a different processing handler to handle the request. Sometimes, more than one handler needs to pass over the data.

#### Solution

```python
class NamedEntityHandler(ABC):
    def __init__(self, successor: Optional["NamedEntityHandler"] = None):
        self._successor = successor

    @abstractmethod
    def handle(self, text: str) -> Entity:
        pass

class PersonNameHandler(NamedEntityHandler):
    def handle(self, text: str) -> Entity:
        name = PersonNameEntityParser.parse(text)
        if self._successor is not None:
            name = self._successor.handle(name)
        return name

class NameConfidenceCalibrationHandler(NamedEntityHandler):
    def handle(self, name_entity: Entity) -> Entity:
        return NameConfidenceCalibrator.calibrate(name_entity)

class CompanyNameHandler(NamedEntityHandler):
    def handle(self, text: str) -> Entity:
        return CompanyNameEntityParser.parse(text)

class DateHandler(NamedEntityHandler): 
    def handle(self, text: str) -> Entity:
        return DateParser.parse(text)

class AmountHandler(NamedEntityHandler):
    def handle(self, text: str) -> Entity:
        return AmountParser.parse(text)

class NamedEntityExtractor:
    def __init__(self):
        self._person_name_handler = PersonNameHandler(NameConfidenceCalibrationHandler())
        self._company_name_handler = CompanyNameHandler()
        self._address_handler = AddressHandler()
        self._date_handler = DateHandler()
        self._amount_handler = AmountHandler()
        self._doc_handler = {
            DocTypeEnum.F_1040: [self._person_name_handler, self._address_handler, self._date_handler, self._amount_handler],
            DocTypeEnum.F_W2: [self._person_name_handler, self._address_handler, self._amount_handler],
            DocTypeEnum.F_BILL: [self._company_name_handler, self._date_handler, self._amount_handler]
        }

    def parse_entity(self, text: str, doc_type: DocTypeEnum) -> List[Entity]:
        parse_chain = self._doc_handler.get(doc_type)
        entity_list = [parser.handle(text) for parser in parse_chain]
        return list(filter(lambda x: x is not None, entity_list))
```

### Strategy

#### Intent
Define a family of interchangeable algorithms by a client (client decides which algorithm to use either dynamically or statically).

#### Problem
There are multiple ways of solving the same problem, each of which has its own advantages/disadvantages. To be able to quickly change the behavior of the program.


