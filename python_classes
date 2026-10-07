class Employee:

    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def display(self):
        print("Name:", self.name)
        print("Salary:", self.salary)


emp1 = Employee("Hari", 100000)

emp1.display()



# Inheritence (passing parent functionality to children)
====================

class Employee:
    def work(self):
        print("I am working")

class Devops(Employee):
    def devops_eng(self):
        print("im a devops engineer")

class Dev(Employee):
    def dev_eng(self):
        print("im a dev engineer")


devops = Devops()
devops.work()
devops.devops_eng() 
    "I am working"
    "im a devops engineer"

dev=Dev()
dev.work()
dev.dev_eng()
    "I am working"
    "im a dev engineer"


# polymorphism -> overriding ( overding the method , to override functionality) , overloading ( overloading method with addtional functionality)
===================== 
class Employee():
    def work(self):
        print("I am working")

class Devops(Employee):
    def work(self):
        print("i work as devops engineer")

devops=Devops()
devops.work()
    "i work as devops engineer"




# Python Encapsulation
# Encapsulation is about protecting data inside a class.

# It means keeping data (properties) and methods together in a class, while controlling how the data can be accessed from outside the class.

# This prevents accidental changes to your data and hides the internal details of how your class works.
===================================================
class Employee():
    def __init__(self,name,age):
        self.name=name
        self.__age=age
    
    def display(self):
        print(self.name)
        print(self.__age)



# Abstraction is the process of hiding implementation details and exposing only the essential functionality to the user. It is used to hide the implementation details from the user and expose only necessary parts, making the code simpler and easier to interact with.
==================================================
class Employee():
    @abstractmethod
    def work():
        pass #no implemntation here 
        #The implementation of an @abstractmethod lives inside the child classes (subclasses)

class child1(Employee):
    def work():
        print("printing children") #implemntation here 
