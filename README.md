# AC-iOS Protocols (feat. a bit of Delegation)

---
### Readings

1. [The Swift Programming Language (Swift 4): Protocols](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html)

---
### Vocabulary

1. **Protocol**: ... A protocol is a list of prerequisites to inherit a title. A protocol is: you need this, this, and this. To be called this. ```You need to drink, party, and procrastinate to be called a stereotypical college student.``` [Medium](https://medium.com/ios-os-x-development/introduction-to-protocols-in-swift-3-73f9a9be6b15)
2. **Delegate**: ... pattern in which one object in a program acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. [Apple](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html)

---
### 1. Objectives

1. Begin to understand the protocols and the delegate design pattern in programming
2. Handle delegation in preparation for tomorrow

---
### 2. But why Protocols?

A protocol defines a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality. The protocol can then be adopted by a class, structure, or enumeration to provide an actual implementation of those requirements. Any type that satisfies the requirements of a protocol is said to conform to that protocol.

Because types can conform to more than one protocol, they can be decorated with default behaviors from multiple protocols. Unlike multiple inheritance of classes which some programming languages support, protocol extensions do not introduce any additional state.

Protocols can be adopted by classes, structs and enums. Base classes and inheritance are restricted to class types.

When you use a protocol, you define **only** the properties and methods you want the implementing types to have, and not their values. If using properties, their read-write or read-only permissions have to be specified.

```swift
protocol someProtocol {
    var someString: String { get set }
    var someInt: Int { get }

    func someMethod()
    func someMutatingMethod()
}
```

In the below, check out some nuances with using protocols.

```swift
// 1: As always, note the protocols below with the function and property stubs
protocol Animal {
    var name: String { get }
    func pet()
}

protocol Cat {
    var badCat: Bool { get }
    func purr()
}

protocol Dog {
    var goodDoggie: Bool { get }
}

// 2: Welp, here are the animals that we want to create. Notice how this class conforms to both of the protocols we created.
class Kitteh: Animal, Cat {
    var name: String = "Meow Mix" // from Animal
    var badCat: Bool = true // from Cat

    func pet() { // Animal again
        self.purr()
    }

    func purr() { // Cat again
        badCat = !badCat
        print("Purrrrr~~ ")
    }
}

// 3. Check it out, a struct!
struct Doggo: Animal, Dog {
    var name: String = "upDog" // from Animal
    var goodDoggie: Bool = true // from Dog

    func pet() { // from Animal
        print("Ew. Slobber.")
    }
}

var kitty = Kitteh()
var doggy = Doggo()

// 4: You can store them in arrays of their types! Dogs and cats living together, mass hysteria
let petShop: [Animal] = [kitty, doggy] // class, struct

petShop.map { $0.pet() }

protocol ExampleProtocol {
    // 5: This is a get-only property! But-- read on for drama
    var simpleDescription: String { get }
    // 6: Keep note of the mutating func.
    mutating func adjust()
}

struct SimpleStruct: ExampleProtocol {
    var simpleDescription: String = "This is a simple String!"

    mutating func adjust() {
        simpleDescription += "  Now 100% adjusted."
        print(simpleDescription)
    }
}

class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "This is a simple String!"

    func adjust() { // 7. Does not have to be mutating since this is a Class
        simpleDescription += " Now 100% adjusted."
        print(simpleDescription)
    }
}

var simpleClass = SimpleClass()
var simpleStruct = SimpleStruct()
simpleClass.adjust()
simpleStruct.adjust()

// 8. And here we change the description...
simpleClass.simpleDescription = "We changed this"
simpleStruct.simpleDescription = "We changed this too"

// If the protocol only requires a property to be gettable, the requirement can be satisfied by any kind of property, and it’s valid for the property to be also settable if this is useful for your own code. In the above, the Class and Struct strings are by default read-write / gettable and settable .

// 9. Protocols can enforce therse permissions if you cast it as a a protocol
var protoClass = SimpleClass() as ExampleProtocol
var protoStruct: ExampleProtocol = SimpleStruct()

//protoClass.simpleDescription = "Hello" // NOPE
//protoStruct.simpleDescription = "Hello" // NOT HERE NEITHER

// 10. This protocol has a read-write String
protocol EditableProperties {
    var editThis: String { get set }
}

class EditClass: EditableProperties {
    var editThis: String = "Hello"
}

class EditStruct: EditableProperties {
    var editThis: String = "Hello"
}

// 11. YUP. STRINGS ARE CHANGED
var editClass: EditableProperties = EditClass()
editClass.editThis = "Hiya"

var editStruct: EditableProperties = EditStruct()
editStruct.editThis = "Okay"
```
![](https://media.giphy.com/media/3o7TKynapnSWtRUTfi/giphy.gif)
---
### 3. Delegation - Wha-huh?

Imagine a job posting for a personal assistant by some employer:

> **Help Wanted:**
>
> **Seeking**: Personal Assistant
>
> **Needed Skills**: Organizing Calendar, Taking Calls, Running Errands

The employer looking for an assistant is likely busy with other things, so much so that they don't have time to *organize their calendar*, or *take all their calls*, or *run errands*. But, they're willing to delegate out some of their responsibilities to their assistant. The employer doesn't really have preference for how their assistant does these tasks - they're only concerned that the tasks get done. And once something gets done, they only want to be informed by their assistant.

Think of a `protocol` as a job posting looking for certain skills:

```swift

// "job posting" PersonalAssistant
protocol PersonalAssistant {
  func organizeCalendar()
  func takeCalls() -> Bool
  func runErrands()
}

```

The employer doesn't necessarily care who they're hiring, just that they can do the functions required. So, a human that could do those tasks would be as valuable to them as a robot, or cat, or dolphin.

A class/struct/enum that is qualified to be a `PersonalAssistant` does their functions on behalf of their "employer." Their "employer" has delegated out some of their duties, and really is only concerned that they happened. When an object is "qualified" to be a `PersonalAssistant`, it is said that they "**conform**" to the `PersonalAssistant protocol`.

#### The `Employer`

To continue the analogy, let's create a class called `Employer`. This `Employer` will have a property called `delegate` of type `PersonalAssistant?`. Why optional? Well, the `Employer` doesn't necessarily have a `PersonalAssistant` right off the bat -- they may need to "hire" one. For that, we'll add a function called `hirePersonalAssistant`. Lastly, the `Employer` needs to be able to declare that they are busy at a meeting and can't take any calls.

```swift
  class Employer {
    // 1. this is optional because we may have not yet "hired" an assistant
    var delegate: PersonalAssistant?

    // 2. We can hire a new assistant
    func hirePersonalAssistant(_ assistant: PersonalAssistant) {
      self.delegate = assistant
    }

    // 3. Employer is going to a meeting, so their calls need to be handled somehow
    func busyAtAMeeting() {
        if let delegate = self.delegate {
           if delegate.takeCalls() {
                print("Delegate is taking the call")
            }
        } else {
            print("Calls going to voicemail")
        }
    }
  }
```

#### The `Employee: PersonalAssistant`

On the other side of things, we have an `Employee` class. The `Employee` is interested in applying to be a `PersonalAssistant`, which means that they can guarantee that they can fulfill the required tasks of `organizeCalendar()`, `takeCalls()`, and `runErrands`.

```swift
  // This Employee conforms to the PersonalAssistant protocol
  class Employee: PersonalAssistant {

    // 1. This employee has an additional ability outside of the requirements of the job description (being able to greeting people)
    func greet() {
        print("Hi there, I'm your Personal Assistant")
    }

    // 2. But because Employee conforms to the PersonalAssistant protocol/job description,
    //    its required skills are to organizeCalendar, takeCalls, and runErrands
    func organizeCalendar() {
      print("Organizing your calendar")
    }

    func takeCalls() -> Bool {
      print("Answering calls")
      return true
    }
    func runErrands() {
        print("Off running some errands")
    }
  }
```

#### First Day on the Job

To see delegation in action, we could imagine `Employee`'s first day on the job looking like this:

```swift

// 8am, Employer gets into work. An Employee is coming in at 9am for an interview
let boss = Employer()

// 8:30am, boss has a meeting to go to
boss.busyAtAMeeting() // prints "Calls going to voicemail"

// 9am. Employee arrives for the interview to be a PersonalAssistant
let assistant = Employee()
assistant.greet() // prints "Hi there, I'm your Personal Assistant" ... boss thinks this is a little too soon, they haven't gotten the job yet... 🤦‍♂️

// 10am. boss is so impressed by the new assistant! hires them right on the spot❗❗❗ 💰
boss.hirePersonalAssistant(assistant)

// 11am. boss heads into another meeting. but now has an assistant!
boss.busyAtAMeeting()   // assistant prints "Answering calls"
                        // boss prints "Delegate is taking the call"

// 12pm. boss and assistant take lunch and bond 🤝
```
