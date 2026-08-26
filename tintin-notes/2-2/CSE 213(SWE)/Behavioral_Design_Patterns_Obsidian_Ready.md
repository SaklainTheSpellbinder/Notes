# Behavioral Design Patterns — Complete CT Guide

This 261-slide deck covers **six behavioral patterns**:

1.  Strategy
2.  Template Method
3.  Observer
4.  Mediator
5.  Command
6.  State

Many slides are progressive animations of the same diagram, so I have merged repeated slides while preserving every distinct concept, example, warning, code structure, and comparison. The major examples are ducks and navigation for Strategy, coffee/tea for Template Method, Weather-O-Rama for Observer, UI dialogs and air-traffic control for Mediator, a text editor for Command, and the gumball machine for State.

---

# 1. First understand what “behavioral” means

Structural patterns mainly answer:

> How should classes and objects be connected?

Behavioral patterns mainly answer:

> Once the objects exist, how should they communicate, divide responsibility, and perform work?

The slides describe behavioral patterns as dealing with:

-  object interaction;
-  communication;
-  responsibility flow;
-  algorithms;
-  assignment of responsibilities.

The six patterns in this deck can be divided into three groups:

| Main concernPatterns           |                           |
| ------------------------------ | ------------------------- |
| Different ways to perform work | Strategy, Template Method |
| Communication between objects  | Observer, Mediator        |
| Requests and changing behavior | Command, State            |

---

# 2. The fastest pattern-detection table

When reading a question, identify the **main pain**.

| What the question is complaining aboutLikely pattern                            |                 |
| ------------------------------------------------------------------------------- | --------------- |
| “There are several algorithms or ways to do the same task”                      | Strategy        |
| “The overall sequence is fixed, but some steps differ”                          | Template Method |
| “When one object changes, many objects must be updated”                         | Observer        |
| “Many components are directly connected and know too much about each other”     | Mediator        |
| “Buttons, menus, shortcuts, queues, undo, or logging must represent operations” | Command         |
| “The same object behaves differently depending on its current mode/state”       | State           |

The most important distinction is:

> Do not choose a pattern because the class diagram looks familiar. Choose it because its **intent** matches the problem.

Strategy and State, for example, have extremely similar class diagrams. Their intentions are different.

---

# 3. The class-order memory sheet

Memorize these six lines.

## Strategy

```text
Strategy interface

    ↑

Concrete strategies

Context HAS a Strategy

Client selects Strategy
```

## Template Method

```text
Abstract superclass

    contains final template method

    contains abstract variable steps

    contains concrete common steps

    contains optional hook

Concrete subclasses override variable steps
```

## Observer

```text
Subject interface

Observer interface

ConcreteSubject HAS List\<Observer>

ConcreteObservers register with Subject
```

## Mediator

```text
Mediator interface

ConcreteMediator knows components

Components HAVE Mediator

Components notify Mediator

Mediator coordinates components
```

## Command

```text
Command interface

ConcreteCommand HAS Receiver

Invoker HAS Command

Client connects Invoker → Command → Receiver
```

## State

```text
State interface

Concrete states

Context HAS current State

Concrete State usually HAS Context

Context delegates to current State
```

State may change Context's current State

This “who holds whom” structure prevents most coding mistakes.

---

# Pattern 1: Strategy

## 4. Strategy Pattern — basic intuition

Suppose a navigation application can calculate a route using:

-  road;
-  walking;
-  public transport;
-  cycling.

The application performs the same general responsibility—building a route—but the **algorithm changes**.

A bad design would be:

```java
if (type.equals("road")) {
// road algorithm
} else if (type.equals("walk")) {
// walking algorithm
} else if (type.equals("bus")) {
// public transport algorithm
}
```

Whenever a new route algorithm is added, this class must change.

Strategy says:

> Take every replaceable algorithm, put it in a separate class, give all algorithms one common interface, and let the main object hold one of them.

The slide definition is:

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from the clients that use it.

---

## 5. The duck problem from the slides

Initially, the design has a superclass:

```text
Duck

 ├── swim()

 ├── quack()

 └── display()
```

Different ducks inherit from it.

Then `fly()` is added to `Duck`.

Problem:

-  Mallard ducks can fly.
-  Redhead ducks can fly.
-  Rubber ducks cannot fly.
-  Wooden decoy ducks cannot fly or quack.

If `fly()` is inherited by every duck, the rubber duck starts flying.

### Attempt 1: Override methods that do not apply

```java
class RubberDuck extends Duck {
    @Override
void fly() {
// do nothing
    }
}
```

This is poor because:

-  many subclasses contain meaningless methods;
-  new duck classes require more overriding;
-  behavior is duplicated;
-  changes must be repeated in multiple classes.

### Attempt 2: `Flyable` and `Quackable` interfaces

```java
interface Flyable {
void fly();
}
interface Quackable {
void quack();
}
```

This stops non-flying ducks from receiving `fly()`.

However, it creates another problem: every flying duck may implement the same flying code separately.

If the flying algorithm changes—wingbeat rate, angle, maximum altitude—you must modify several classes.

The lesson from pages 19–23 is:

> Identify what varies and separate it from what stays the same.

In the duck application:

-  duck identity and display stay relatively stable;
-  flying behavior varies;
-  quacking behavior varies.

Therefore, flying and quacking become separate strategy families.

---

## 6. Strategy structure

```text
                         FlyBehavior

                              ↑

               ┌──────────────┴─────────────┐

         FlyWithWings                   FlyNoWay

Duck HAS FlyBehavior

Duck.performFly() delegates to flyBehavior.fly()
```

The critical word is **delegates**.

The duck does not implement the flying algorithm itself. It asks another object to perform it.

```java
public void performFly() {
flyBehavior.fly();
}
```

---

## 7. Complete Java example: duck strategies

```java
import java.util.Objects;
// ---------- Strategy interfaces ----------
interface FlyBehavior {
void fly();
}
interface QuackBehavior {
void quack();
}
// ---------- Concrete flying strategies ----------
class FlyWithWings implements FlyBehavior {
    @Override
public void fly() {
System.out.println("Flying with wings");
    }
}
class FlyNoWay implements FlyBehavior {
    @Override
public void fly() {
System.out.println("Cannot fly");
    }
}
class FlyRocketPowered implements FlyBehavior {
    @Override
public void fly() {
System.out.println("Flying with a rocket");
    }
}
// ---------- Concrete quacking strategies ----------
class NormalQuack implements QuackBehavior {
    @Override
public void quack() {
System.out.println("Quack");
    }
}
class Squeak implements QuackBehavior {
    @Override
public void quack() {
System.out.println("Squeak");
    }
}
class MuteQuack implements QuackBehavior {
    @Override
public void quack() {
System.out.println("Silence");
    }
}
// ---------- Context ----------
abstract class Duck {
private FlyBehavior flyBehavior;
private QuackBehavior quackBehavior;
protected Duck(
FlyBehavior flyBehavior,
QuackBehavior quackBehavior) {
setFlyBehavior(flyBehavior);
setQuackBehavior(quackBehavior);
    }
public void performFly() {
flyBehavior.fly();
    }
public void performQuack() {
quackBehavior.quack();
    }
public void setFlyBehavior(FlyBehavior flyBehavior) {
this.flyBehavior =
Objects.requireNonNull(flyBehavior);
    }
```

---

## 8. What is dynamic about Strategy?

Initially:

```java
model.setFlyBehavior(new FlyNoWay());
```

Later:

```java
model.setFlyBehavior(new FlyRocketPowered());
```

The `ModelDuck` object has not changed class. Its contained strategy object has changed.

That gives runtime behavior replacement.

---

## 9. Strategy detection signals

Choose Strategy when the question says things such as:

-  support multiple payment methods;
-  choose among sorting algorithms;
-  calculate tax differently for different regions;
-  calculate shipping by different policies;
-  compress using ZIP, RAR, or GZIP;
-  authenticate through password, OTP, or biometric;
-  change the route-building algorithm at runtime;
-  eliminate a large conditional selecting an algorithm.

### Strong words

```text
algorithm

policy

method

approach

different ways

select

switch

interchangeable

runtime
```

---

## 10. Strategy mistakes

### Mistake 1: Context implements every algorithm

```java
class Navigator {
void roadRoute() {}
void walkingRoute() {}
void busRoute() {}
}
```

This is not Strategy. The algorithms remain inside the context.

### Mistake 2: Context uses `instanceof`

```java
if (strategy instanceof RoadStrategy) {
    ...
}
```

The context should depend only on the strategy interface.

### Mistake 3: Every strategy has a different method

```java
road.buildRoad();
walk.buildWalkingRoute();
bus.calculateBusRoute();
```

They need one common contract:

```java
interface RouteStrategy {
Route buildRoute(Point a, Point b);
}
```

### Mistake 4: Confusing a value with a strategy

```java
navigator.setRouteType("road");
```

A string is not an algorithm. The strategy object should contain the behavior.

---

## 11. Strategy memory formula

> **Context HAS an interchangeable algorithm.**

Or:

> Strategy changes **how** a task is performed.

---

# Pattern 2: Template Method

## 12. Template Method — basic intuition

Suppose tea and coffee are prepared as follows.

### Coffee

1.  Boil water.
2.  Brew coffee.
3.  Pour into a cup.
4.  Add sugar and milk.

### Tea

1.  Boil water.
2.  Steep tea.
3.  Pour into a cup.
4.  Add lemon.

The sequence is nearly identical.

Common steps:

-  boil water;
-  pour into cup.

Variable steps:

-  brewing or steeping;
-  condiments.

The important observation is not merely that some code is duplicated. It is that both algorithms have the same **skeleton**.

Template Method places that skeleton in a superclass.

The slide definition is:

> Template Method defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps without changing the algorithm’s structure.

---

## 13. Structure

```java
abstract CaffeineBeverage
final prepareRecipe()
    boilWater()
```

    brew()                 abstract

    pourInCup()

    addCondiments()        abstract

Then:

```text
Coffee extends CaffeineBeverage

Tea extends CaffeineBeverage
```

The template method must normally be `final`.

Why?

Because a subclass should be allowed to change certain steps, but not reorder or replace the whole algorithm.

---

## 14. Complete Java example

```java
abstract class CaffeineBeverage {
// Template method: fixed algorithm skeleton.
public final void prepareRecipe() {
boilWater();
brew();
pourInCup();
if (customerWantsCondiments()) {
addCondiments();
        }
    }
// Primitive operations: subclasses must implement.
protected abstract void brew();
protected abstract void addCondiments();
// Common concrete steps.
private void boilWater() {
System.out.println("Boiling water");
    }
private void pourInCup() {
System.out.println("Pouring into cup");
    }
// Hook: optional extension point.
protected boolean customerWantsCondiments() {
return true;
    }
}
class Coffee extends CaffeineBeverage {
    @Override
protected void brew() {
System.out.println(
"Brewing coffee through filter"
        );
    }
    @Override
protected void addCondiments() {
System.out.println(
"Adding sugar and milk"
        );
    }
}
class Tea extends CaffeineBeverage {
    @Override
protected void brew() {
System.out.println("Steeping tea");
    }
    @Override
protected void addCondiments() {
System.out.println("Adding lemon");
    }
}
class BlackCoffee extends Coffee {
    @Override
protected boolean customerWantsCondiments() {
return false;
    }
}
class TemplateMethodDemo {
public static void main(String[] args) {
CaffeineBeverage tea = new Tea();
tea.prepareRecipe();
CaffeineBeverage coffee =
new BlackCoffee();
coffee.prepareRecipe();
    }
}
```

---

## 15. Four types of methods inside Template Method

This distinction is frequently tested.

### 1. Template method

Defines the sequence.

```java
public final void prepareRecipe() {
    ...
}
```

### 2. Abstract primitive operations

Subclasses must implement them.

```java
protected abstract void brew();
```

### 3. Concrete common operations

Superclass implements them once.

```java
private void boilWater() {
    ...
}
```

### 4. Hooks

Have a default implementation. Subclasses may override them.

```java
protected boolean customerWantsCondiments() {
return true;
}
```

A hook often does nothing or returns a default value.

---

## 16. The data-mining example from the slides

Suppose the system imports:

-  DOC files;
-  CSV files;
-  PDF files.

The overall mining algorithm is:

1.  Open file.
2.  Extract data.
3.  Parse data.
4.  Analyze data.
5.  Send report.
6.  Close file.

Only some file-specific steps vary.

```java
abstract class DataMiner {
public final void mine(String path) {
openFile(path);
String rawData = extractData();
String data = parseData(rawData);
analyzeData(data);
sendReport();
closeFile();
    }
protected abstract void openFile(String path);
protected abstract String extractData();
protected abstract String parseData(String rawData);
protected void analyzeData(String data) {
System.out.println("Analyzing data");
    }
protected void sendReport() {
System.out.println("Sending report");
    }
protected abstract void closeFile();
}
```

`PDFDataMiner`, `CSVDataMiner`, and `DocDataMiner` override the file-specific operations.

---

## 17. Template Method detection signals

Choose Template Method when:

-  multiple classes perform nearly the same sequence;
-  only a few steps vary;
-  the sequence must not be reordered;
-  there is substantial duplicated workflow code;
-  subclasses should customize only selected steps.

Typical phrases:

```text
fixed sequence

same workflow

same skeleton

common steps

some steps differ

subclasses customize

cannot change order
```

---

## 18. Template Method mistakes

### Mistake 1: The template method is abstract

```java
abstract void prepareRecipe();
```

That destroys the common skeleton.

The variable steps should be abstract, not the overall algorithm.

### Mistake 2: The template method is not final

A subclass could replace the entire workflow.

```java
@Override
public void prepareRecipe() {
// Completely different sequence
}
```

### Mistake 3: Every step is abstract

Then no useful code is reused.

### Mistake 4: Using Template Method only because two classes share one method

There must be a meaningful algorithmic sequence, not merely one duplicated helper method.

---

## 19. Strategy versus Template Method

This comparison appears explicitly in the slides.

| StrategyTemplate Method                  |                                             |
| ---------------------------------------- | ------------------------------------------- |
| Based on composition                     | Based on inheritance                        |
| Context has a strategy object            | Subclass extends an abstract class          |
| Replaces an entire algorithm or behavior | Replaces selected steps                     |
| Behavior can change at runtime           | Behavior is normally fixed by subclass type |
| Works at object level                    | Works at class level                        |
| Strategies are independent objects       | Steps belong to subclass implementation     |

### Easy example

You need different complete route algorithms:

> Strategy.

You need one fixed report-generation process where only loading and formatting differ:

> Template Method.

### Memory line

> Strategy swaps the **worker**.
>  Template Method lets subclasses fill the **blanks in a recipe**.

---

## 20. Important Bloch-style warning for Template Method

Do not call a template method from a superclass constructor.

Bad:

```java
abstract class Base {
Base() {
```

prepare(); // calls overridable methods

    }

abstract void prepare();

}

The subclass fields may not have been initialized when `prepare()` runs.

This follows the general Effective Java principle:

> Constructors should not invoke overridable methods.

Template Method is inheritance-heavy, so use it only when the superclass-subclass relationship is stable and genuinely represents an “is-a” relationship.

---

# Pattern 3: Observer

## 21. Observer — basic intuition

Consider a weather station.

`WeatherData` receives:

-  temperature;
-  humidity;
-  pressure.

Several displays require the data:

-  current conditions display;
-  statistics display;
-  forecast display.

A bad implementation is:

```java
void measurementsChanged() {
currentDisplay.update(...);
statisticsDisplay.update(...);
forecastDisplay.update(...);
}
```

Problems:

- `WeatherData` knows every concrete display;
-  adding a display requires modifying `WeatherData`;
-  removing a display requires modifying `WeatherData`;
-  displays cannot subscribe and unsubscribe dynamically.

Observer says:

> The subject stores a collection of observers. When its state changes, it calls the common update method on every registered observer.

The slide definition is:

> Define a one-to-many dependency between objects so that when one object changes state, all dependents are notified and updated automatically.

Terminology:

```text
Publisher = Subject

Subscriber = Observer
```

---

## 22. Magazine-subscription intuition

A magazine publisher does not manually search for readers each month.

Instead:

1.  A customer subscribes.
2.  The publisher stores the subscriber.
3.  A new issue is published.
4.  Every subscriber is notified or receives the issue.
5.  A subscriber may unsubscribe.

That is Observer.

---

## 23. Structure

```text
Subject

 ├── registerObserver(observer)

 ├── removeObserver(observer)

 └── notifyObservers()

Observer

 └── update(...)

ConcreteSubject HAS List\<Observer>

ConcreteObservers implement update()
```

---

## 24. Complete Java example

```java
import java.util.ArrayList;
import java.util.List;
interface Subject {
void registerObserver(Observer observer);
void removeObserver(Observer observer);
void notifyObservers();
}
interface Observer {
void update(
float temperature,
float humidity,
float pressure
    );
}
class WeatherData implements Subject {
private final List\<Observer> observers =
new ArrayList<>();
private float temperature;
private float humidity;
private float pressure;
    @Override
public void registerObserver(Observer observer) {
if (!observers.contains(observer)) {
observers.add(observer);
        }
    }
    @Override
public void removeObserver(Observer observer) {
observers.remove(observer);
    }
    @Override
public void notifyObservers() {
// Copy protects the iteration if an observer
// unsubscribes while being notified.
List\<Observer> snapshot =
new ArrayList<>(observers);
for (Observer observer : snapshot) {
observer.update(
temperature,
humidity,
pressure

            );

        }

    }

public void setMeasurements(

float temperature,

float humidity,

float pressure) {

this.temperature = temperature;

this.humidity = humidity;

this.pressure = pressure;

measurementsChanged();

    }

private void measurementsChanged() {

notifyObservers();

    }

}

class CurrentConditionsDisplay

implements Observer {

private float temperature;

private float humidity;

    @Override

public void update(

float temperature,

float humidity,

---

## 25. Push versus pull Observer

The weather code in the slides uses the **push model**:

```java
observer.update(
temperature,
humidity,
```

pressure

);

The subject pushes all values.

### Push

```java
void update(float temp, float humidity, float pressure);
```

Advantages:

-  simple;
-  observer receives everything immediately.

Disadvantages:

-  every observer receives data it may not need;
-  changing the data list changes the observer interface.

### Pull

The subject sends itself:

```java
interface Observer {
void update(WeatherData source);
}
```

The observer pulls only what it needs:

```java
public void update(WeatherData source) {
float temperature =
source.getTemperature();
}
```

The generalized diagram near the Observer definition uses this publisher-reference style.

---

## 26. Who knows whom?

The Subject knows observers only through the interface:

```java
List<Observer>
```

It must not know:

```text
CurrentConditionsDisplay

ForecastDisplay

StatisticsDisplay
```

The observers may know the Subject to:

-  register;
-  unregister;
-  pull data.

The dependency is still relatively loose because both sides communicate through interfaces.

---

## 27. Observer detection signals

Choose Observer when:

-  one event affects many listeners;
-  subscribers can appear or disappear dynamically;
-  the subject should not know concrete receivers;
-  a GUI event has many listeners;
-  a stock-price change updates multiple screens;
-  a new message notifies several services;
-  a model change updates views.

Typical words:

```text
notify

subscribe

unsubscribe

listener

event

publisher

subscriber

one-to-many

when X changes

all interested objects
```

---

## 28. Observer mistakes

### Mistake 1: Subject directly calls concrete classes

```java
currentDisplay.update();
forecastDisplay.update();
```

### Mistake 2: One observer field instead of a collection

Observer is normally one-to-many.

### Mistake 3: Observers cannot unsubscribe

Dynamic registration is a central property.

### Mistake 4: Subject contains observer-specific logic

```java
if (observer instanceof ForecastDisplay) {
    ...
}
```

### Mistake 5: Forgetting lifecycle management

If an observer remains registered after it is no longer needed, it may:

-  continue receiving events;
-  prevent garbage collection;
-  create memory leaks.

---

## 29. Modern Java refinement

The slides show an observer registering itself in its constructor.

```java
public CurrentConditionsDisplay(Subject subject) {
subject.registerObserver(this);
}
```

This is common in teaching examples, but explicit registration is often clearer:

```java
CurrentConditionsDisplay display =
new CurrentConditionsDisplay();
subject.registerObserver(display);
```

It avoids exposing `this` before construction has completely finished.

A convenient production API can return a subscription object:

```java
interface Subscription {
void unsubscribe();
}
```

Then:

```java
Subscription subscription =
weatherData.subscribe(display);
subscription.unsubscribe();
```

---

# Pattern 4: Mediator

## 30. Mediator — basic intuition

The slides first use air-traffic control.

Aircraft do not coordinate landing by directly talking to every other aircraft.

Without a control tower, every pilot would need to know:

-  every other aircraft;
-  their positions;
-  their intentions;
-  their landing sequence.

Instead:

```text
Aircraft → Control Tower → Aircraft
```

The tower is the mediator.

---

## 31. UI-dialog problem from the slides

Consider a customer-profile dialog containing:

-  text fields;
-  checkboxes;
-  buttons;
-  selection controls.

Examples of interaction:

-  selecting “Business” enables the company-name field;
-  pressing Apply validates every field;
-  changing one field may disable another control.

Bad design:

```java
class BusinessCheckBox {
private ApplyButton applyButton;
private TextField companyField;
private ProfileDialog dialog;
private OtherField otherField;
}
```

Now each control knows several colleagues.

Consequences:

-  components become tightly coupled;
-  a checkbox cannot be reused in another form;
-  changes spread across many classes;
-  the dependency graph becomes chaotic.

Mediator says:

> Components stop talking to one another directly. They report events to a mediator, and the mediator coordinates the response.

The slide definition is:

> Mediator restricts direct communications between objects and forces them to collaborate only through a mediator, reducing chaotic dependencies.

---

## 32. Structure

```text
Mediator

 └── notify(sender, event)

ConcreteMediator

 ├── knows ComponentA

 ├── knows ComponentB

 └── coordinates them

Component

 └── has Mediator

ConcreteComponent

 └── sends events to Mediator
```

The communication shape is:

```text
Component A

     ↓

  Mediator

     ↓

Component B
```

Not:

```text
Component A → Component B
```

---

## 33. Valid Java implementation

The slide pseudocode shows a `Component` class and later writes something resembling:

```java
class Button implements Component
```

In Java, that would be invalid if `Component` is a class. A class **extends** another class and **implements** an interface.

The correct Java form is:

```java
class Button extends Component
```

Here is a complete version.

```java
interface Mediator {
void notify(Component sender, String event);
}
abstract class Component {
protected final Mediator mediator;
protected Component(Mediator mediator) {
this.mediator = mediator;
    }
}
class CheckBox extends Component {
private boolean checked;
public CheckBox(Mediator mediator) {
super(mediator);
    }
public void setChecked(boolean checked) {
this.checked = checked;
mediator.notify(
this,
"checked-changed"
        );
    }
public boolean isChecked() {
return checked;
    }
}
class TextField extends Component {
private String text = "";
private boolean enabled = true;
public TextField(Mediator mediator) {
super(mediator);
    }
public void setText(String text) {
this.text = text;
mediator.notify(this, "text-changed");
    }
public String getText() {
return text;
    }
public void setEnabled(boolean enabled) {
this.enabled = enabled;
System.out.println(
"Text field enabled: " + enabled
        );
    }
public boolean isEnabled() {
return enabled;
    }
}
class Button extends Component {
public Button(Mediator mediator) {
super(mediator);
    }
public void click() {
mediator.notify(this, "click");
    }
}
class ProfileDialog implements Mediator {
private final CheckBox businessCheckBox;
private final TextField companyNameField;
private final Button applyButton;
public ProfileDialog() {
businessCheckBox =
new CheckBox(this);
```

---

## 34. What did Mediator improve?

Before:

```text
Checkbox knows Button

Checkbox knows TextField

Button knows Checkbox

Button knows every field

TextField knows other controls
```

After:

```text
Every component knows only Mediator
```

Mediator knows the dialog's components

The dependency has been centralized.

This improves component reuse.

A generic `CheckBox` can now be placed in another dialog because it does not contain business-specific knowledge about company-name fields or Apply buttons.

---

## 35. Mediator detection signals

Choose Mediator when:

-  many peer components communicate;
-  every component knows several other components;
-  individual components are difficult to reuse;
-  changing one component requires changing many colleagues;
-  coordination rules belong to a larger context;
-  a dialog, controller, room, control tower, or chat room naturally coordinates participants.

Typical words:

```text
coordinate

central controller

many-to-many

chaotic dependencies

components interact

control tower

dialog controller

colleagues
```

---

## 36. Mediator drawbacks

Mediator removes complexity from components, but that complexity moves somewhere.

The concrete mediator may become a large “god object”:

```java
if (...) {
} else if (...) {
} else if (...) {
} else if (...) {
}
```

Solutions:

-  one mediator per dialog or bounded subsystem;
-  extract validation services;
-  use typed events;
-  split a large mediator into smaller coordinators.

A mediator should coordinate components, not absorb every business responsibility in the system.

---

## 37. Observer versus Mediator

This comparison appears directly in the slides.

| ObserverMediator                                |                                                       |
| ----------------------------------------------- | ----------------------------------------------------- |
| One-to-many notification                        | Many components coordinated centrally                 |
| Publisher broadcasts a change                   | Components report events to coordinator               |
| Subscribers usually do not depend on each other | Components may logically affect each other            |
| Dynamic subscribe/unsubscribe is central        | Central interaction logic is central                  |
| Communication is primarily one-way              | Communication may be redirected in several directions |

### Example

A weather station announces a temperature change to all displays:

> Observer.

A dialog receives a checkbox event and decides which fields and buttons to update:

> Mediator.

### Memory line

> Observer **broadcasts**.
>  Mediator **coordinates**.

---

# Pattern 5: Command

## 38. Command — basic intuition

The slides use a text editor containing:

-  toolbar buttons;
-  menu items;
-  keyboard shortcuts.

The same operation—Save—can be triggered through:

-  Save button;
-  File → Save menu item;
-  Ctrl+S shortcut.

Bad options include:

### Subclass every UI control

```text
SaveButton

CopyButton

PasteButton

OpenButton
```

This creates many subclasses.

### Duplicate the saving code

```java
class SaveButton {
void click() {
// saving code
    }
}
class SaveMenuItem {
void select() {
// same saving code
    }
}
class SaveShortcut {
void press() {
// same saving code
    }
}
```

### Make shortcuts depend on buttons

```java
shortcut.press() {
saveButton.click();
}
```

Now unrelated GUI components are coupled.

Command says:

> Represent the request itself as an object.

Instead of a button knowing how to save, the button holds a `SaveCommand`.

---

## 39. Command structure

```text
Invoker → Command → Receiver
```

For a text editor:

```text
Button → SaveCommand → Editor
```

Roles:

### Command

Common operation interface.

```java
interface Command {
void execute();
}
```

### ConcreteCommand

Stores:

-  receiver;
-  operation parameters;
-  information required for execution;
-  possibly backup data for undo.

### Receiver

Performs the real business operation.

```java
class Editor {
void save() { ... }
}
```

### Invoker

Triggers the command.

```java
class Button {
Command command;
void click() {
command.execute();
    }
}
```

### Client

Constructs and connects everything.

```java
Editor editor = new Editor();
Command save = new SaveCommand(editor);
Button button = new Button(save);
```

The slide definition is:

> Command turns a request into a standalone object containing all information about the request. This allows requests to be parameterized, delayed, queued, and made undoable.

---

## 40. Complete Java example

```java
import java.util.ArrayDeque;
import java.util.Deque;
interface Command {
void execute();
default void undo() {
// Commands without undo need do nothing.
    }
default boolean isUndoable() {
return false;
    }
}
// ---------- Receiver ----------
class Editor {
private String text = "";
public String getText() {
return text;
    }
public void setText(String text) {
this.text = text;
System.out.println(
"Editor text: " + text
        );
    }
public void append(String value) {
setText(text + value);
    }
public void save() {
System.out.println(
"Saving: " + text
        );
    }
}
// ---------- Concrete commands ----------
class SaveCommand implements Command {
private final Editor editor;
public SaveCommand(Editor editor) {
this.editor = editor;
    }
    @Override
public void execute() {
editor.save();
    }
}
class AppendTextCommand implements Command {
private final Editor editor;
private final String value;
private String backup;
public AppendTextCommand(
Editor editor,
String value) {
this.editor = editor;
this.value = value;
    }
    @Override
public void execute() {
backup = editor.getText();
editor.append(value);
    }
    @Override
public void undo() {
editor.setText(backup);
    }
    @Override
public boolean isUndoable() {
return true;
    }
}
// ---------- Invokers ----------
```

---

## 41. Why command parameters should be preconfigured

The slides emphasize that commands “know what to do.”

Instead of:

```java
command.execute(editor, "Hello", 5, true);
```

Prefer:

```java
Command command =
new AppendTextCommand(
editor,
"Hello"
        );
command.execute();
```

The request object already contains:

-  receiver;
-  parameters;
-  operation.

This is why it can be:

-  queued;
-  stored;
-  logged;
-  passed as an argument;
-  executed later.

---

## 42. Queuing requests

A queue can store commands without knowing their concrete types.

```java
Queue<Command> queue =
new ArrayDeque<>();
queue.add(new SaveCommand(editor));
queue.add(new AppendTextCommand(
editor,
"Queued text"
));
while (!queue.isEmpty()) {
queue.remove().execute();
}
```

A worker can execute these later.

That is substantially harder if requests exist only as direct method calls.

---

## 43. Macro commands

One command can contain multiple commands.

```java
class MacroCommand implements Command {
private final Command[] commands;
public MacroCommand(Command... commands) {
this.commands = commands;
    }
    @Override
public void execute() {
for (Command command : commands) {
command.execute();
        }
    }
    @Override
public void undo() {
for (int i = commands.length - 1;
i >= 0;
i--) {
commands[i].undo();
        }
    }
}
```

The reverse order during undo is important.

If operations execute:

```text
A → B → C
```

Undo normally occurs:

```text
C → B → A
```

---

## 44. Command detection signals

Choose Command when:

-  operations are initiated through buttons, menus, or shortcuts;
-  the same action has several triggers;
-  operations must be queued;
-  operations must be logged;
-  operations must be retried;
-  operations need undo/redo;
-  a request must be passed as a parameter;
-  sender and receiver must be decoupled.

Typical words:

```text
request

action

execute

button

menu

shortcut

undo

redo

queue

history

transaction

job

task

receiver
```

---

## 45. Command mistakes

### Mistake 1: Business logic remains in the invoker

```java
class Button {
void click() {
editor.save();
    }
}
```

The button is still coupled to the receiver.

### Mistake 2: Concrete command has no receiver or behavior

```java
class SaveCommand {
String name = "save";
}
```

A command is not merely a label.

### Mistake 3: Invoker checks command types

```java
if (command instanceof SaveCommand) {
    ...
}
```

It should simply call `execute()`.

### Mistake 4: Undo does not preserve previous state

To reverse a state-changing operation, the command usually needs:

-  a backup;
-  an inverse operation;
-  or a stored memento.

---

## 46. Command versus Mediator

This comparison appears in the slides.

| CommandMediator                                      |                                                  |
| ---------------------------------------------------- | ------------------------------------------------ |
| Creates a request object                             | Creates a coordinator object                     |
| Sender points to Command                             | Components point to Mediator                     |
| Command points to Receiver                           | Mediator points to components                    |
| Usually one directional: sender → command → receiver | Redirects communication among several components |
| Main concern: represent an action                    | Main concern: organize communication             |

### Example

A Save button contains a `SaveCommand`:

> Command.

A dialog decides how clicking Apply affects ten controls:

> Mediator.

They may coexist:

```text
Button

  → notifies DialogMediator

      → executes SaveCommand

          → EditorService
```

---

## 47. Command versus Strategy

They can also look similar because both wrap behavior.

| StrategyCommand                        |                                              |
| -------------------------------------- | -------------------------------------------- |
| Represents how to perform an algorithm | Represents a request to perform an operation |
| Usually selected and used repeatedly   | May represent one execution                  |
| Often returns a result                 | Often performs an action                     |
| Usually has no history                 | Frequently stored in history or queue        |
| Does not necessarily know a receiver   | Commonly contains a receiver                 |

Memory:

> Strategy is a **method of solving**.
>  Command is an **order to act**.

---

# Pattern 6: State

## 48. State — basic intuition

A gumball machine behaves differently depending on whether it:

-  is sold out;
-  has no quarter;
-  has a quarter;
-  is dispensing a gumball;
-  is in winner mode.

The same input has different outcomes.

For example, `insertQuarter()`:

-  in `NoQuarterState`: accept the quarter;
-  in `HasQuarterState`: reject the extra quarter;
-  in `SoldState`: ask the user to wait;
-  in `SoldOutState`: reject because the machine is empty.

Initially, this may be implemented using integer constants:

```java
static final int SOLD_OUT = 0;
static final int NO\_QUARTER = 1;
static final int HAS\_QUARTER = 2;
static final int SOLD = 3;
int state = NO\_QUARTER;
```

Then every operation contains conditionals:

```java
void insertQuarter() {
if (state == HAS\_QUARTER) {
        ...
    } else if (state == NO\_QUARTER) {
        ...
    } else if (state == SOLD\_OUT) {
        ...
    } else if (state == SOLD) {
        ...
    }
}
```

This is repeated in:

- `insertQuarter()`;
- `ejectQuarter()`;
- `turnCrank()`;
- `dispense()`.

When Winner State is added, every method may need modification.

The slides call this the “nightmare of change.”

State says:

> Move each state’s behavior into a separate state object. The context delegates its operations to the current state.

The slide definition is:

> Allow an object to alter its behavior when its internal state changes. The object appears to change its class.

---

## 49. State structure

```text
State

 ├── operationA()

 ├── operationB()

 └── operationC()

ConcreteStateA

ConcreteStateB

ConcreteStateC

Context HAS current State

Context.operation() → state.operation()
```

Usually, in the slide’s design:

```text
ConcreteState HAS Context
```

That allows the state to perform transitions:

```java
machine.setState(
machine.getHasQuarterState()
);
```

---

## 50. Complete gumball-machine core

```java
interface State {
void insertQuarter();
void ejectQuarter();
void turnCrank();
void dispense();
}
class GumballMachine {
private final State soldOutState;
private final State noQuarterState;
private final State hasQuarterState;
private final State soldState;
private State state;
private int count;
public GumballMachine(int count) {
this.count = count;
soldOutState =
new SoldOutState(this);
noQuarterState =
new NoQuarterState(this);
hasQuarterState =
new HasQuarterState(this);
soldState =
new SoldState(this);
state = count > 0
? noQuarterState
                : soldOutState;
    }
public void insertQuarter() {
state.insertQuarter();
    }
public void ejectQuarter() {
state.ejectQuarter();
    }
public void turnCrank() {
state.turnCrank();
state.dispense();
    }
void setState(State state) {
this.state = state;
    }
void releaseBall() {
if (count > 0) {
System.out.println(
"A gumball comes out"
            );
count--;
        }
    }
int getCount() {
return count;
    }
State getSoldOutState() {
return soldOutState;
    }
State getNoQuarterState() {
return noQuarterState;
    }
State getHasQuarterState() {
return hasQuarterState;
    }
State getSoldState() {
```

### No-quarter state

```java
class NoQuarterState implements State {
private final GumballMachine machine;
public NoQuarterState(
GumballMachine machine) {
this.machine = machine;
    }
    @Override
public void insertQuarter() {
System.out.println(
"Quarter accepted"
        );
machine.setState(
machine.getHasQuarterState()
        );
    }
    @Override
public void ejectQuarter() {
System.out.println(
"No quarter to eject"
        );
    }
    @Override
public void turnCrank() {
System.out.println(
"Insert a quarter first"
        );
    }
    @Override
public void dispense() {
System.out.println(
"Payment required"
        );
    }
}
```

### Has-quarter state

```java
class HasQuarterState implements State {
private final GumballMachine machine;
public HasQuarterState(
GumballMachine machine) {
this.machine = machine;
    }
    @Override
public void insertQuarter() {
System.out.println(
"Quarter already inserted"
        );
    }
    @Override
public void ejectQuarter() {
System.out.println(
"Quarter returned"
        );
machine.setState(
machine.getNoQuarterState()
        );
    }
    @Override
public void turnCrank() {
System.out.println(
"Crank turned"
        );
machine.setState(
machine.getSoldState()
        );
    }
    @Override
public void dispense() {
System.out.println(
"Turn the crank first"
        );
    }
}
```

### Sold state

```java
class SoldState implements State {
private final GumballMachine machine;
public SoldState(
GumballMachine machine) {
this.machine = machine;
    }
    @Override
public void insertQuarter() {
System.out.println(
"Please wait"
        );
    }
    @Override
public void ejectQuarter() {
System.out.println(
"Too late to eject"
        );
    }
    @Override
public void turnCrank() {
System.out.println(
"Already turned"
        );
    }
    @Override
public void dispense() {
machine.releaseBall();
if (machine.getCount() > 0) {
machine.setState(
machine.getNoQuarterState()
            );
        } else {
System.out.println(
"Machine is sold out"
            );
machine.setState(
machine.getSoldOutState()
            );
        }
    }
}
```

### Sold-out state

```java
class SoldOutState implements State {
private final GumballMachine machine;
public SoldOutState(
GumballMachine machine) {
this.machine = machine;
    }
    @Override
public void insertQuarter() {
System.out.println(
"Machine is sold out"
        );
    }
    @Override
public void ejectQuarter() {
System.out.println(
"No quarter inserted"
        );
    }
    @Override
public void turnCrank() {
System.out.println(
"No gumballs available"
        );
    }
    @Override
public void dispense() {
System.out.println(
"Nothing dispensed"
        );
    }
}
```

### Client

```java
class StateDemo {
public static void main(String[] args) {
GumballMachine machine =
new GumballMachine(2);
machine.insertQuarter();
machine.turnCrank();
machine.insertQuarter();
machine.ejectQuarter();
    }
}
```

---

## 51. Adding the Winner State

The slides introduce a policy:

> Ten percent of the time, a customer receives two gumballs.

Without State, a new state requires rewriting several conditional methods.

With State:

1.  Add `WinnerState`.
2.  Add a winner-state object to `GumballMachine`.
3.  Change the transition decision in `HasQuarterState`.
4.  Existing unrelated state behavior remains localized.

Simplified transition:

```java
@Override
public void turnCrank() {
int winner = random.nextInt(10);
if (winner == 0
&& machine.getCount() > 1) {
machine.setState(
machine.getWinnerState()
        );
    } else {
machine.setState(
machine.getSoldState()
        );
    }
}
```

Winner behavior:

```java
class WinnerState implements State {
private final GumballMachine machine;
public WinnerState(
GumballMachine machine) {
this.machine = machine;
    }
    @Override
public void dispense() {
System.out.println(
"Winner! Two gumballs"
        );
machine.releaseBall();
if (machine.getCount() == 0) {
machine.setState(
machine.getSoldOutState()
            );
return;
        }
machine.releaseBall();
if (machine.getCount() > 0) {
machine.setState(
machine.getNoQuarterState()
            );
        } else {
machine.setState(
machine.getSoldOutState()
            );
        }
    }
    @Override
public void insertQuarter() {
System.out.println("Please wait");
    }
    @Override
public void ejectQuarter() {
System.out.println(
"Cannot eject now"
        );
    }
    @Override
public void turnCrank() {
System.out.println(
"Already turned"
        );
    }
}
```

---

## 52. Invalid operations belong to states

Each state determines what is valid.

For example:

```java
class SoldOutState {
void insertQuarter() {
System.out.println(
"Cannot insert: sold out"
        );
    }
}
```

This is better than one large context method checking every possible state.

All behavior for `SoldOutState` is found in one class.

---

## 53. Document-publication example

The slides also show a document whose `publish()` operation varies by state.

Possible states:

```text
Draft

Moderation

Published
```

Behavior:

-  Draft: send document for moderation.
-  Moderation: approve or reject.
-  Published: publishing again may do nothing.

This is State because the document’s behavior depends on its current lifecycle stage.

---

## 54. State detection signals

Choose State when:

-  behavior depends on current state;
-  the object moves through a lifecycle;
-  the same operation means different things in different states;
-  large `if` or `switch` statements check a state variable;
-  adding a state requires modifying many methods;
-  transitions are important.

Typical words:

```text
state

status

mode

lifecycle

transition

draft

approved

locked

unlocked

connected

disconnected

pending

completed
```

---

## 55. State mistakes

### Mistake 1: Keep the large switch inside Context

```java
switch (state) {
case NO\_QUARTER:
case HAS\_QUARTER:
    ...
}
```

Then State objects add no value.

### Mistake 2: Every state contains the full context logic

Only behavior specific to that state should be moved.

### Mistake 3: Client manually controls every transition

In the slide’s form of State, concrete states normally know valid transitions.

The client says:

```java
machine.insertQuarter();
```

Not:

```java
machine.setState(HAS_QUARTER);
```

### Mistake 4: Use State for two tiny cases that will never grow

A small `boolean enabled` does not automatically justify a pattern with ten classes.

---

## 56. State versus Strategy

This comparison appears explicitly at the end of the slides.

Both:

-  use composition;
-  place behavior in helper objects;
-  make Context delegate work;
-  may have nearly identical UML structures.

But their intentions differ.

| StrategyState                                   |                                                   |
| ----------------------------------------------- | ------------------------------------------------- |
| Different algorithms for one task               | Different behavior for different lifecycle states |
| Strategies are normally independent             | States may know one another                       |
| Client or configuration chooses strategy        | Current state or context drives transitions       |
| Switching is optional and externally controlled | Switching is part of normal object behavior       |
| Strategy usually does not know Context          | State often stores Context                        |
| Focus: how work is done                         | Focus: what is allowed now                        |

### Example

User selects Credit Card instead of PayPal:

> Strategy.

Order automatically changes from Pending to Paid to Shipped:

> State.

### Memory line

> Strategy is **chosen behavior**.
>  State is **condition-dependent behavior**.

---

# 57. Master comparison of all six patterns

| PatternMain questionMain relationshipWho selects/controls behavior? |                                              |                                               |                           |
| ------------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------- | ------------------------- |
| Strategy                                                            | Which algorithm should be used?              | Context has Strategy                          | Client/configuration      |
| Template Method                                                     | Which steps of this fixed algorithm vary?    | Subclass inherits superclass                  | Subclass type             |
| Observer                                                            | Who should be notified when this changes?    | Subject has observer list                     | Subscribers register      |
| Mediator                                                            | Who should coordinate these components?      | Components and mediator reference one another | Mediator                  |
| Command                                                             | How can this request become an object?       | Invoker has Command; Command has Receiver     | Client configures command |
| State                                                               | What behavior is valid in the current state? | Context has State; State often has Context    | State transitions/context |

---

# 58. The most commonly confused pairs

## Strategy versus Template Method

```text
Whole algorithm varies        → Strategy

Only selected steps vary      → Template Method

Runtime swapping needed       → Strategy

Inheritance-based recipe      → Template Method
```

## Observer versus Mediator

```text
One object announces to many  → Observer

Many peers need coordination  → Mediator
```

## Command versus Strategy

```text
How should task be performed? → Strategy

Please perform this request   → Command
```

## Command versus Mediator

```text
Represent an action           → Command

Coordinate components         → Mediator
```

## State versus Strategy

```text
Externally selected policy    → Strategy

Internally changing lifecycle → State
```

## Observer versus Command

```text
Event broadcast to listeners  → Observer

Event represented as action   → Command
```

They can coexist. A subject may notify observers by publishing Command objects.

---

# 59. How to solve a pattern-identification question

Use this order.

## Step 1: Find what is changing

Is it:

-  an algorithm?
-  selected steps?
-  number of listeners?
-  component interaction?
-  an operation/request?
-  lifecycle state?

## Step 2: Find the problematic coupling

Ask:

> Which class currently knows too much?

Examples:

-  Duck knows every flying variation.
-  WeatherData knows every display.
-  Checkbox knows every field.
-  Button knows Editor business logic.
-  GumballMachine knows every state case.

## Step 3: Identify what must become an object

| VariationNew object          |          |
| ---------------------------- | -------- |
| Algorithm                    | Strategy |
| Workflow step implementation | Subclass |
| Listener                     | Observer |
| Coordinator                  | Mediator |
| Request                      | Command  |
| Current mode                 | State    |

## Step 4: Draw the central arrow

```text
Context → Strategy

Subject → Observer[]

Component → Mediator

Invoker → Command → Receiver

Context → State

Subclass → Abstract superclass
```

## Step 5: Check the intent

Do not stop after matching the UML. State and Strategy use similar UML, but the problem intention must decide.

---

# 60. How to write each pattern by hand without forgetting classes

## Strategy template

```java
interface Strategy {
void execute();
}
class StrategyA implements Strategy {
public void execute() {}
}
class Context {
private Strategy strategy;
Context(Strategy strategy) {
this.strategy = strategy;
    }
void setStrategy(Strategy strategy) {
this.strategy = strategy;
    }
void doWork() {
strategy.execute();
    }
}
```

Mnemonic:

```text
Interface → implementations → Context has interface
```

---

## Template Method template

```java
abstract class AbstractClass {
public final void templateMethod() {
commonStep();
variableStep();
hook();
    }
private void commonStep() {}
protected abstract void variableStep();
protected void hook() {}
}
class ConcreteClass extends AbstractClass {
protected void variableStep() {}
}
```

Mnemonic:

```text
Final recipe → abstract steps → concrete subclasses
```

---

## Observer template

```java
interface Observer {
void update();
}
interface Subject {
void add(Observer observer);
void remove(Observer observer);
void notifyObservers();
}
class ConcreteSubject implements Subject {
List\<Observer> observers;
public void notifyObservers() {
for (Observer observer : observers) {
observer.update();
        }
    }
}
```

Mnemonic:

```text
Subject has list
```

---

## Mediator template

```java
interface Mediator {
void notify(Component sender, String event);
}
abstract class Component {
protected Mediator mediator;
}
class ConcreteMediator implements Mediator {
private ComponentA a;
private ComponentB b;
public void notify(
Component sender,
String event) {
// coordinate components
    }
}
```

Mnemonic:

```text
Components know mediator;

mediator knows components
```

---

## Command template

```java
interface Command {
void execute();
}
class ConcreteCommand implements Command {
private Receiver receiver;
public void execute() {
receiver.action();
    }
}
class Invoker {
private Command command;
void trigger() {
command.execute();
    }
}
```

Mnemonic:

```text
Invoker → Command → Receiver
```

---

## State template

```java
interface State {
void handle();
}
class ConcreteStateA implements State {
private Context context;
public void handle() {
context.setState(
new ConcreteStateB(context)
        );
    }
}
class Context {
private State state;
void request() {
state.handle();
    }
void setState(State state) {
this.state = state;
    }
}
```

Mnemonic:

```text
Context has current State;

State may change Context
```

---

# 61. Joshua Bloch-inspired and modern Java improvements

These are additions beyond the slide material.

## Prefer composition over inheritance

This strongly supports:

-  Strategy;
-  Command;
-  State;
-  Mediator.

Template Method uses inheritance, so do not choose it merely to remove a little duplication. Choose it when subclasses genuinely share a stable algorithmic skeleton.

---

## Use constructor injection

Bad:

```java
class Context {
private Strategy strategy;
Context() {
strategy = null;
    }
}
```

Better:

```java
Context(Strategy strategy) {
this.strategy =
Objects.requireNonNull(strategy);
}
```

The object is valid immediately after construction.

---

## Use lambdas for small stateless strategies

Because Strategy is often a single-method interface:

```java
@FunctionalInterface
interface DiscountStrategy {
double apply(double price);
}
```

It can be implemented using lambdas:

```java
DiscountStrategy tenPercent =
price -> price \* 0.90;
DiscountStrategy fixed100 =
price -> price - 100;
```

The classic class-based form is usually better for handwritten pattern answers because it makes the structure visible. Lambdas are useful in production when behavior is small and stateless.

The same applies to simple Commands:

```java
Runnable saveCommand = editor::save;
```

However, use a class when the command needs:

-  parameters;
-  history;
-  undo information;
-  identity;
-  complex logic.

---

## Use standard functional interfaces where appropriate

Instead of inventing:

```java
interface ValueTransformer {
String transform(String value);
}
```

Java may already provide:

```java
Function<String, String>
```

Other useful interfaces include:

- `Predicate<T>`;
- `Consumer<T>`;
- `Supplier<T>`;
- `Comparator<T>`;
- `Function<T, R>`.

Do not use a generic functional interface when a domain-specific name makes the design substantially clearer.

---

## Avoid magic strings and integers

Bad Mediator event:

```text
notify(this, "click");
```

Better:

```java
enum Event {
CLICK,
CHECKED\_CHANGED,
```

TEXT\_CHANGED

}

Bad state representation:

```java
int state = 2;
```

At minimum use an enum:

```java
enum MachineState {
SOLD\_OUT,
NO\_QUARTER,
HAS\_QUARTER,
```

SOLD

}

For behavior-rich states, full State objects remain better.

---

## Keep stateless objects immutable

A stateless strategy may be shared:

```java
final class FlyNoWay
implements FlyBehavior {
static final FlyNoWay INSTANCE =
new FlyNoWay();
private FlyNoWay() {}
public void fly() {
System.out.println("Cannot fly");
    }
}
```

Then:

```java
duck.setFlyBehavior(
FlyNoWay.INSTANCE
);
```

Do not share state objects if they contain mutable state specific to one context.

---

## Do not expose mutable collections

Bad:

```java
public List<Observer> getObservers() {
return observers;
}
```

External code can corrupt the subject’s list.

Prefer:

```java
public List<Observer> getObservers() {
return List.copyOf(observers);
}
```

Or do not expose it at all.

---

## Separate undoable and non-undoable commands

The compact example used a default no-op `undo()` for readability.

A stricter design can use:

```java
interface Command {
void execute();
}
interface UndoableCommand extends Command {
void undo();
}
```

This avoids forcing every command to pretend it supports undo.

---

# 62. Builder versus Abstract Factory

These are **creational**, not behavioral, and are not among the six patterns in this deck. Since you specifically mentioned the confusion, use this rule.

## Builder

Use Builder when constructing **one complex object step by step**.

```text
Builder → one final Product
```

Example:

```java
Computer computer =
new ComputerBuilder()
```

.cpu("Ryzen")

.ram(32)

.storage(1000)

.build();

The questions are:

-  Which parts are included?
-  In what stages is the object assembled?
-  Are many optional parameters present?
-  Can the same construction process make different representations?

### Memory

> Builder assembles **one thing**.

---

## Abstract Factory

Use Abstract Factory when creating **a family of related, compatible objects**.

```text
Factory → ProductA + ProductB + ProductC
```

Example:

```java
GUIFactory factory =
new DarkThemeFactory();
Button button =
factory.createButton();
CheckBox checkBox =
factory.createCheckBox();
```

The question is:

-  Which product family should be used?
-  Must all created products match one another?

### Memory

> Abstract Factory chooses **one family**.

---

## Direct difference

| BuilderAbstract Factory                         |                                              |
| ----------------------------------------------- | -------------------------------------------- |
| Builds one complex product                      | Creates several related product types        |
| Step-by-step construction                       | Individual factory methods                   |
| Product returned after assembly                 | Products returned separately                 |
| Focus on configuration and construction process | Focus on compatible product families         |
| `builder.cpu().ram().build()`                   | `factory.createButton()`, `createCheckBox()` |

A gaming-computer builder assembling CPU, RAM, GPU, and storage into one computer:

> Builder.

A Windows factory producing Windows Button, Windows Menu, and Windows CheckBox:

> Abstract Factory.

---

# 63. Final exam-level decision chart

```text
Does behavior depend on current lifecycle state?

    Yes → State
```

Otherwise, is a request/action being stored,

queued, logged, undone, or assigned to buttons?

    Yes → Command

Otherwise, does one object notify many listeners?

    Yes → Observer

Otherwise, are many peer objects too tightly coupled?

    Yes → Mediator

Otherwise, are multiple complete algorithms interchangeable?

    Yes → Strategy

Otherwise, is there one fixed algorithm skeleton

with only selected steps varying?

    Yes → Template Method

---

# 64. One-line revision sheet

```text
Strategy:
```

Encapsulate interchangeable algorithms.

Template Method:

Fix the algorithm skeleton; subclasses fill variable steps.

Observer:

One subject notifies many registered observers.

Mediator:

A central object coordinates peer components.

Command:

Turn a request into an executable object.

State:

Delegate behavior to the object representing current state.

And the most important ownership diagram:

```text
Strategy: Context → Strategy

Template: Concrete subclass → Abstract template

Observer: Subject → Observer[]

Mediator: Component → Mediator → Components

Command: Invoker → Command → Receiver
```

State: Context ↔ State

That diagram, combined with the pattern intent, is the safest way to remember which interface or class comes next when writing code by hand.
