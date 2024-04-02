```python
from dataclasses import dataclass
from enum import Enum

class Gender(Enum):
    MALE = 1
    FEMALE = 2


@dataclass
class Person:
    name: str
    age: int
    gender: Gender
    dream: str

    def toString(self):
        return f"My name is {self.name} i'm {self.age} yo, {self.gender}, and i'm dreaming about {self.dream}"


people: [Person] = [
    Person(name="Mark", age=20, gender=Gender.MALE, dream="airplanes"),
    Person(name="Anjela", age=34, gender=Gender.FEMALE, dream="ocean"),
    Person(name="John", age=40, gender=Gender.MALE, dream="cigarette"),
    Person(name="Lily", age=8, gender=Gender.FEMALE, dream="treehouse"),
    Person(name="Jake", age=4, gender=Gender.MALE, dream="banana"),
    Person(name="Finn", age=4, gender=Gender.MALE, dream="pineapple"),
    Person(name="Pem", age=28, gender=Gender.FEMALE, dream="Jim"),
    Person(name="Jim", age=29, gender=Gender.MALE, dream="Pem"),
]


def check_people_with_collection_manipulation():
    people_under_thirty = list(filter(lambda person: person.age < 30, people))
    for person in people_under_thirty:
        print(person.toString())

    number_of_people_with_j_name = len(list(filter(lambda person: person.name[0] == 'J', people)))
    print(number_of_people_with_j_name)

    fruits: list[str] = ['apple', 'banana', 'pineapple']
    fruit_dreams = list(filter(lambda person: person.dream in fruits, people))
    for person in fruit_dreams:
        print(person.toString())

    oldest_person = max(map(lambda person: person.age, people))
    print(oldest_person)

    number_of_males_and_females = dict(map(lambda person: (person.gender, len(list(filter(lambda filter_person: filter_person.gender == person.gender, people)))), people))
    print(number_of_males_and_females)

    females: list[Person] = list(filter(lambda person: person.gender == Gender.FEMALE, people))
    for female in females:
        print(female.toString())

if __name__ == "__main__":
    check_people_with_collection_manipulation()
```