# Python Classes

## Constructor method
The **`__init__()`  is the constructor method of a class**. 
This method is automatically called when a new instance of a class is instantiated.
It is used to initialize an object's state when it is created: assigning values to object properties and perform initialization operations.

``` python
class BookShop:

    # constructor
    def __init__(self, title):
        self.title = title

    def print_book(self):
        print('The tile of the book is', self.title)


b = BookShop('Sandman')
b.print_book()
# The tile of the book is Sandman
```

## Self
**Self represent the instance of the class**, in the class. **You can access the attributes and methods of a class, internally, using the self keyword**. Anyway, keep in mind that self *it is not considered as a keyword in python*, although it is used in different places.

## Class decorators

### @property, @\<property\>.setter and @\<property\>.deleter
@property is a python standard decorator used to add functionalities to class methods and have them behave as getter, setter or deleter for a class property.

We can define three functions that wors on the three aspects of a property:
- **@property**: access to property value, the method should have the same name of the property
- **@\<property\>.setter**: to change the property value
- **@\<property\>.deleter**: to delete the instance attribute

``` python
class House:

	def __init__(self, price):
		self._price = price

	@property
	def price(self):
		return self._price
	
	@price.setter
	def price(self, new_price):
		if new_price > 0 and isinstance(new_price, float):
			self._price = new_price
		else:
			print("Please enter a valid price")

	@price.deleter
	def price(self):
		del self._price
```