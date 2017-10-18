# AC-iOS Protocols and Extensions (feat. a bit of Delegation)
![](https://media.giphy.com/media/3o7TKynapnSWtRUTfi/giphy.gif)

---
### Readings

1. [The Swift Programming Language (Swift 4): Protocols](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html)

---
### Vocabulary

1. **Protocol**: ... A protocol is a list of prerequisites to inherit a title. A protocol is: you need this, this, and this. To be called this. ```You need to drink, party, and procrastinate to be called a stereotypical college student.``` [Medium](https://medium.com/ios-os-x-development/introduction-to-protocols-in-swift-3-73f9a9be6b15)
2. **Delegate**: ... pattern in which one object in a program acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. [Apple](https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html)
3. **Extension**: ... Technique to add new functionality to an existing class, structure, enumeration, or protocol type. You can extend an existing type to adopt and conform to a new protocol, even if you don’t have access to the source code for the existing type.

---
### 0. Objectives

1. Begin to understand the protocols and the delegate design pattern in programming
2. Practice handling delegation somehow

---
### 1. Delegation - Wha-huh?

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

### BUT WAIT, WHAT ABOUT EXTENSIONS

It's 4pm, and the slump is hitting big boss hard. So he assigns `Employee` a new task for them to `grabCoffee`.

```swift
// Drama ensues
extension Employee {
    func grabCoffee() {
        print("This is beneath me. I quit")
    }
}
```

---
### Moving on
