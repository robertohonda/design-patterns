# Design Patterns
Design patterns are reusable solutions to commonly occurring software design problems.

Design patterns are categorized into three main groups:
- Creational
- Structural
- Behavioral

## 1. Creational Design Patterns
Creational patterns focus on object creation mechanisms, aiming to create objects in a flexible and reusable way.

### 1.1 Abstract Factory Pattern

The Abstract Factory Pattern provides an interface for creating **families of related objects**
without specifying their concrete implementations.

```TS
interface Button {
  render(): void;
}

interface Modal {
  render(): void;
}

class LightButton implements Button {
  render() {
    console.log("Render light button");
  }
}

class LightModal implements Modal {
  render() {
    console.log("Render light modal");
  }
}

class DarkButton implements Button {
  render() {
    console.log("Render dark button");
  }
}

class DarkModal implements Modal {
  render() {
    console.log("Render dark modal");
  }
}

interface UIFactory {
  createButton(): Button;
  createModal(): Modal;
}

class LightThemeFactory implements UIFactory {
  createButton(): Button {
    return new LightButton();
  }

  createModal(): Modal {
    return new LightModal();
  }
}

class DarkThemeFactory implements UIFactory {
  createButton(): Button {
    return new DarkButton();
  }

  createModal(): Modal {
    return new DarkModal();
  }
}

function renderUI(factory: UIFactory) {
  const button = factory.createButton();
  const modal = factory.createModal();

  button.render();
  modal.render();
}

const themeFactory: UIFactory =
  Math.random() > 0.5
    ? new LightThemeFactory()
    : new DarkThemeFactory();

renderUI(themeFactory);
```

### Pros

✅ Ensures compatible products  
✅ Easy to swap families    
✅ Strong adherence to DIP  
✅ Scales well for complex systems  

### Cons

❌ More abstraction  
❌ Harder to add new product types  
❌ Overkill for small apps  

### 1.2 Factory Method Pattern
Defines an interface for creating an object but lets subclasses alter the type of objects that will be created.

```TS
// 1️⃣ Common interface
interface Payment {
  pay(amount: number): void;
}

// 2️⃣ Concrete implementations
class PaypalPayment implements Payment {
  pay(amount: number): void {
    console.log(`Paying ${amount} with PayPal`);
  }
}

class CreditCardPayment implements Payment {
  pay(amount: number): void {
    console.log(`Paying ${amount} with Credit Card`);
  }
}

// 3️⃣ Allowed payment types
type PaymentType = "paypal" | "credit";

// 4️⃣ Factory
class PaymentFactory {
  private static registry: Record<PaymentType, new () => Payment> = {
    paypal: PaypalPayment,
    credit: CreditCardPayment,
  };

  static create(type: PaymentType): Payment {
    const PaymentClass = this.registry[type];
    return new PaymentClass();
  }
}

// 5️⃣ Client code
const payment = PaymentFactory.create("paypal");
payment.pay(100);
```

### Pros
✅ Decouples creation from usage  
✅ Centralized creation logic  
✅ Easier to test (mock Payment)  
✅ Supports SOLID principles  

### Cons
❌ Extra abstraction  
❌ Factory can grow too large  
❌ Simple cases don’t need it  

### 1.3 Singleton Pattern
Ensures that a class has only one instance and provides a global access point to it.

```TS
class AppConfig {
  private static instance: AppConfig;

  private constructor() {}

  static getInstance(): AppConfig {
    if (!AppConfig.instance) {
      AppConfig.instance = new AppConfig();
    }
    return AppConfig.instance;
  }
}

const configA = AppConfig.getInstance();
const configB = AppConfig.getInstance();

console.log(configA === configB);      // true
```

### Pros
✅ Guarantees a single shared instance  
✅ Consistent global state  
✅ Useful for configuration, logging, caching  

### Cons
❌ Acts like global state  
❌ Harder to mock and test  
❌ Overuse leads to tight coupling  

### 1.4 Builder Pattern

The Builder Pattern is used to construct complex objects step by step.  
It allows creating different representations of an object using the same construction process.

```TS
class User {
  constructor(
    public name: string,
    public age: number,
    public email: string,
    public phone?: string,
    public address?: string
  ) {}
}

class UserBuilder {
  private name!: string;
  private age!: number;
  private email!: string;
  private phone?: string;
  private address?: string;

  setName(name: string): this {
    this.name = name;
    return this;
  }

  setAge(age: number): this {
    this.age = age;
    return this;
  }

  setEmail(email: string): this {
    this.email = email;
    return this;
  }

  setPhone(phone: string): this {
    this.phone = phone;
    return this;
  }

  setAddress(address: string): this {
    this.address = address;
    return this;
  }

  build(): User {
    return new User(this.name, this.age, this.email, this.phone, this.address);
  }
}

const user = new UserBuilder()
  .setName("John")
  .setAge(45)
  .setEmail("john.doe@example.com")
  .setPhone("123-456")
  .setAddress("Washington")
  .build();
```

### Pros
✅ Easy to add new rules  
✅ Highly readable  

### Cons
❌ More classes  
❌ Overkill for simple validation  

### 1.5 Prototype Pattern
The Prototype Pattern creates new objects by **cloning an existing object** instead of creating them from scratch.

```TS
interface Prototype<T> {
  clone(): T;
}

class UserProfile implements Prototype<UserProfile> {
  constructor(
    public role: string,
    public permissions: string[],
    public theme: string
  ) {}

  clone(): UserProfile {
    return new UserProfile(
      this.role,
      [...this.permissions], // deep copy
      this.theme
    );
  }
}

const adminTemplate = new UserProfile(
  "admin",
  ["read", "write", "delete"],
  "dark"
);

const admin1 = adminTemplate.clone();
admin1.theme = "light";

const admin2 = adminTemplate.clone();

console.log(adminTemplate.theme); // dark
console.log(admin1.theme);        // light
console.log(admin2.theme);        // dark
```

### Pros
✅ Fast object creation  
✅ Avoids complex constructors  
✅ Reduces duplication  
✅ Good for templates  

### Cons
❌ Cloning logic can get complex  
❌ Deep copies must be handled carefully  
❌ Not ideal for very dynamic objects  
❌ Circular references might be very tricky

## 2. Structural Design Patterns
Structural patterns focus on how classes and objects are composed to form larger, flexible structures.

### 2.1 Decorator Pattern
The Decorator Pattern allows you to **add new behavior to an object dynamically**
without modifying its original class.

It works by **wrapping** an object and extending its behavior.

```TS
interface Notifier {
  send(message: string): void;
}

class EmailNotifier implements Notifier {
  send(message: string): void {
    console.log(`Email: ${message}`);
  }
}

class SMSDecorator implements Notifier {
  constructor(private notifier: Notifier) {}

  send(message: string): void {
    this.notifier.send(message);
    console.log(`SMS: ${message}`);
  }
}

class PushDecorator implements Notifier {
  constructor(private notifier: Notifier) {}

  send(message: string): void {
    this.notifier.send(message);
    console.log(`Push: ${message}`);
  }
}

const notifier =
  new PushDecorator(
    new SMSDecorator(
      new EmailNotifier()
    )
  );

notifier.send("Order shipped");
```

### Pros
✅ Open/Closed Principle  
✅ Flexible combinations  
✅ Single Responsibility  
✅ Reusable behavior  

### Cons
❌ Many small classes  
❌ Debugging can be harder  
❌ Order of decorators matters  

### 2.2 Adapter Pattern
The Adapter Pattern allows **objects with incompatible interfaces** to work together
by converting one interface into another that the client expects.

```TS
class LegacyLogger {
  logMessage(msg: string) {
    console.log("LEGACY:", msg);
  }
}

interface Logger {
  log(message: string): void;
}

class LoggerAdapter implements Logger {
  constructor(private legacy: LegacyLogger) {}

  log(message: string): void {
    this.legacy.logMessage(message);
  }
}
```

### Pros
✅ Allows reuse of existing code  
✅ Decouples client from third-party APIs  
✅ Follows Open/Closed Principle  

### Cons
❌ Extra layer of abstraction  
❌ Can hide complexity  

### 2.3 Facade Pattern
Facade provides a simple, unified interface to a complex subsystem.

```TS
class PaymentService {
  pay(amount: number) {
    console.log("Payment processed:", amount);
  }
}

class InventoryService {
  reserve(productId: string) {
    console.log("Product reserved:", productId);
  }
}

class ShippingService {
  ship(productId: string) {
    console.log("Product shipped:", productId);
  }
}

class EmailService {
  sendConfirmation() {
    console.log("Confirmation email sent");
  }
}

class OrderFacade {
  private payment = new PaymentService();
  private inventory = new InventoryService();
  private shipping = new ShippingService();
  private email = new EmailService();

  placeOrder(productId: string, amount: number) {
    this.inventory.reserve(productId);
    this.payment.pay(amount);
    this.shipping.ship(productId);
    this.email.sendConfirmation();
  }
}

const order = new OrderFacade();
order.placeOrder("P123", 100);
```

### Pros
✅ Simplifies API usage  
✅ Centralizes workflow logic  

### Cons
❌ Can become a God Object if it grows too much  
❌ May hide important flexibility  
❌ Clients may bypass it, causing inconsistency  

### 2.4 Proxy Pattern
Proxy provides a stand-in object that controls access to another object.
> “You’re not talking to the real thing directly — I’m managing access for you.”

```TS
interface UserService {
  getUser(id: string): Promise<string>;
}

class RealUserService implements UserService {
  async getUser(id: string): Promise<string> {
    console.log("Fetching user from API...");
    return `User(${id})`;
  }
}

class UserServiceProxy implements UserService {
  private realService = new RealUserService();
  private cache = new Map<string, string>();

  async getUser(id: string): Promise<string> {
    if (this.cache.has(id)) {
      console.log("Returning cached user");
      return this.cache.get(id)!;
    }

    const user = await this.realService.getUser(id);
    this.cache.set(id, user);
    return user;
  }
}

const userService: UserService = new UserServiceProxy();

await userService.getUser("1"); // API call
await userService.getUser("1"); // Cached
```

### Pros
✅ Encourages loose coupling  

### Cons
❌ The response from the service might get delayed.
❌ Introduces additional abstraction  

### 2.5 Composite Pattern
Composite is a structural design pattern that lets you compose objects into tree structures and then work with these structures as if they were individual objects.

```TS
interface FileSystemItem {
  getSize(): number;
}

class File implements FileSystemItem {
  constructor(
    private name: string,
    private size: number
  ) {}

  getSize(): number {
    return this.size;
  }
}

class Folder implements FileSystemItem {
  private items: FileSystemItem[] = [];

  constructor(private name: string) {}

  add(item: FileSystemItem): void {
    this.items.push(item);
  }

  getSize(): number {
    return this.items.reduce(
      (total, item) => total + item.getSize(),
      0
    );
  }
}

const file1 = new File("a.txt", 100);
const file2 = new File("b.txt", 200);

const folder = new Folder("docs");
folder.add(file1);
folder.add(file2);

const root = new Folder("root");
root.add(folder);
root.add(new File("c.txt", 300));

console.log(root.getSize()); // 600
```

### Pros
✅ Treats individual and composite objects uniformly  
✅ Simplifies client code  
✅ Natural fit for tree structures  
✅ Easy to add new leaf types  

### Cons
❌ Can make design overly generic  
❌ Harder to restrict what can be added  
❌ Debugging recursive structures is harder  
❌ May hide performance costs  

### 2.6 Bridge Pattern
Composite is a structural design pattern that lets you compose objects into tree structures and then work with these structures as if they were individual objects.

```TS
interface Theme {
  getBackgroundColor(): string;
  getTextColor(): string;
}

class LightTheme implements Theme {
  getBackgroundColor(): string {
    return "#ffffff";
  }

  getTextColor(): string {
    return "#000000";
  }
}

class DarkTheme implements Theme {
  getBackgroundColor(): string {
    return "#1e1e1e";
  }

  getTextColor(): string {
    return "#ffffff";
  }
}

abstract class UIComponent {
  constructor(protected theme: Theme) {}

  abstract render(): void;
}

class Button extends UIComponent {
  render(): void {
    console.log(
      `Button with bg=${this.theme.getBackgroundColor()} text=${this.theme.getTextColor()}`
    );
  }
}

const light = new LightTheme();
const dark = new DarkTheme();

const lightButton = new Button(light);
const darkButton = new Button(dark);


lightButton.render();
darkButton.render();
```

### Pros
✅ Avoids class explosion  
✅ Decouples abstraction from implementation  
✅ Improves extensibility  
✅ Strong alignment with DIP  

### Cons
❌ Increases initial complexity  
❌ More classes to manage  
❌ Overkill for simple hierarchies  

### 2.7 Flyweight Pattern
Flyweight minimizes memory usage by sharing common (intrinsic) state between many objects, while keeping unique (extrinsic) state outside.
> “Don’t duplicate what can be shared.”

```TS
class CharacterStyle {
  constructor(
    public font: string,
    public size: number,
    public color: string
  ) {}
}

class CharacterStyleFactory {
  private static styles = new Map<string, CharacterStyle>();

  static getStyle(font: string, size: number, color: string): CharacterStyle {
    const key = `${font}-${size}-${color}`;

    if (!this.styles.has(key)) {
      this.styles.set(key, new CharacterStyle(font, size, color));
    }

    return this.styles.get(key)!;
  }
}

class Character {
  constructor(
    public char: string,
    public x: number,
    public y: number,
    public style: CharacterStyle
  ) {}

  draw(): void {
    console.log(
      `Draw '${this.char}' at (${this.x}, ${this.y}) with ${this.style.font}`
    );
  }
}

const style = CharacterStyleFactory.getStyle("Arial", 12, "black");

const c1 = new Character("H", 0, 0, style);
const c2 = new Character("i", 10, 0, style);

c1.draw();
c2.draw();
```

### Pros
✅ Significantly reduces memory usage  
✅ Encourages separation of shared vs unique state  

### Cons
❌ Changing a flyweight affects all consumers  
❌ Increases code complexity  
❌ You might be trading RAM over CPU cycles when some of the context data needs to be recalculated each time somebody calls a flyweight method.  

## 3. Behavioral Design Patterns
Behavioral patterns focus on communication between objects and the assignment of responsibilities.

### 3.1 Strategy Pattern
Strategy defines a family of algorithms, encapsulates each one, and makes them interchangeable.

```TS
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPaymentStrategy implements PaymentStrategy {
  pay(amount: number): void {
    console.log(`Paid ${amount} using Credit Card`);
  }
}

class PaypalPaymentStrategy implements PaymentStrategy {
  pay(amount: number): void {
    console.log(`Paid ${amount} using PayPal`);
  }
}

class Checkout {
  constructor(private readonly paymentStrategy: PaymentStrategy) {}

  process(amount: number): void {
    this.paymentStrategy.pay(amount);
  }
}

const creditCheckout = new Checkout(new CreditCardPaymentStrategy());
creditCheckout.process(100);

const paypalCheckout = new Checkout(new PaypalPaymentStrategy());
paypalCheckout.process(100);
```

### Pros
✅ Eliminates large conditional statements  
✅ Makes algorithms interchangeable  
✅ Improves testability  
✅ Strong adherence to OCP and DIP 

### Cons
❌ Increases number of classes  
❌ Client must understand which strategy to choose  
❌ Overkill for simple logic  


### 3.2 Observer Pattern
Observer defines a one-to-many dependency:
when one object changes state, all dependents are notified automatically.

```TS
interface Subscriber {
  update(videoTitle: string): void;
}

class YouTubeChannel {
  private subscribers: Subscriber[] = [];

  subscribe(sub: Subscriber): void {
    this.subscribers.push(sub);
  }

  unsubscribe(sub: Subscriber): void {
    this.subscribers = this.subscribers.filter(s => s !== sub);
  }

  upload(videoTitle: string): void {
    console.log(`New video: ${videoTitle}`);
    this.notify(videoTitle);
  }

  private notify(videoTitle: string): void {
    this.subscribers.forEach(sub => sub.update(videoTitle));
  }
}

class User implements Subscriber {
  constructor(private name: string) {}

  update(videoTitle: string): void {
    console.log(`${this.name} received notification: ${videoTitle}`);
  }
}

const channel = new YouTubeChannel();

const alice = new User("Alice");
const bob = new User("Bob");

channel.subscribe(alice);
channel.subscribe(bob);

channel.upload("Observer Pattern Explained");

// Bob unsubscribes
channel.unsubscribe(bob);

channel.upload("Strategy Pattern Explained");
```

### Pros
✅ Decouples publisher from subscribers  
✅ Supports event-driven architecture  
✅ Easy to add/remove listeners  
✅ Scales well with many dependents  

### Cons
❌ Can cause memory leaks if not unsubscribed    
❌ Notification chains may be hard to debug   
❌ Order of execution may be unpredictable   
❌ Cascading updates can hurt performance  

### 3.3 Command Pattern
Command encapsulates a request as an object so it can be executed, queued, logged, or undone.
> “Turn actions into objects.”

```TS
interface Command {
  execute(): void;
  undo(): void;
}

class Light {
  turnOn(): void {
    console.log("Light ON");
  }

  turnOff(): void {
    console.log("Light OFF");
  }
}

class TurnOnLightCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }

  undo(): void {
    this.light.turnOff();
  }
}

class TurnOffLightCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }

  undo(): void {
    this.light.turnOn();
  }
}

class RemoteControl {
  private history: Command[] = [];

  press(command: Command): void {
    command.execute();
    this.history.push(command);
  }

  undo(): void {
    const last = this.history.pop();
    last?.undo();
  }
}

const light = new Light();
const remote = new RemoteControl();

const on = new TurnOnLightCommand(light);
const off = new TurnOffLightCommand(light);

remote.press(on);
remote.press(off);

remote.undo(); // Light ON again
```

### Pros
✅ Decouples sender from receiver  
✅ Supports undo/redo operations  
✅ Enables queuing, logging, scheduling  
✅ Easy to add new commands (OCP)  

### Cons
❌ Increases number of classes  
❌ Can add boilerplate  
❌ Overkill for simple method calls  

### 3.4 Chain of Responsibility
Chain of Responsibility passes a request along a chain of handlers.
Each handler decides whether to process it or pass it forward.

```TS
abstract class Handler {
  private next?: Handler

  setNext(handler: Handler): Handler {
    this.next = handler
    return handler
  }

  handle(request: Request): void {
    if (this.next) {
      this.next.handle(request)
    }
  }
}

class LoggingHandler extends Handler {
  handle(request: Request) {
    console.log("Logging request:", request.url)
    super.handle(request)
  }
}

class AuthHandler extends Handler {
  handle(request: Request) {
    if (!request.headers["auth"]) {
      console.log("Unauthorized ❌")
      return
    }

    console.log("Authenticated ✅")
    super.handle(request)
  }
}

class ControllerHandler extends Handler {
  handle(request: Request) {
    console.log("Handling business logic 🚀")
  }
}

const logger = new LoggingHandler()
const auth = new AuthHandler()
const controller = new ControllerHandler()

logger.setNext(auth).setNext(controller)

logger.handle({
  url: "/dashboard",
  headers: { auth: "token" }
} as any)
```

### Pros
✅ Reduces large conditional logic  
✅ Decouples sender from concrete handler  
✅ Easy to reorder or extend chain  
✅ Follows Open/Closed Principle  
✅ Good for middleware pipelines  

### Cons
❌ Request may go unhandled  
❌ Debugging can be harder  
❌ Order dependency can cause subtle bugs  

### 3.5 State Pattern
State allows an object to change its behavior when its internal state changes.
Instead of giant if/else or switch statements,
you move behavior into separate state classes.

```TS
interface PlayerContext {
  setState(state: PlayerState): void;
}

interface PlayerState {
  play(): void;
  pause(): void;
  stop(): void;
}

class MediaPlayer implements PlayerContext {
  private state: PlayerState;

  constructor() {
    this.state = new StoppedState(this);
  }

  setState(state: PlayerState) {
    this.state = state;
  }

  play() { this.state.play(); }
  pause() { this.state.pause(); }
  stop() { this.state.stop(); }
}

class StoppedState implements PlayerState {
  constructor(private player: PlayerContext) {}

  play(): void {
    console.log("Starting playback...");
    this.player.setState(new PlayingState(this.player));
  }

  pause(): void {
    console.log("Can't pause. Player is stopped.");
  }

  stop(): void {
    console.log("Already stopped.");
  }
}

class PlayingState implements PlayerState {
  constructor(private player: PlayerContext) {}

  play(): void {
    console.log("Already playing.");
  }

  pause(): void {
    console.log("Pausing...");
    this.player.setState(new PausedState(this.player));
  }

  stop(): void {
    console.log("Stopping...");
    this.player.setState(new StoppedState(this.player));
  }
}

class PausedState implements PlayerState {
  constructor(private player: PlayerContext) {}

  play(): void {
    console.log("Resuming...");
    this.player.setState(new PlayingState(this.player));
  }

  pause(): void {
    console.log("Already paused.");
  }

  stop(): void {
    console.log("Stopping...");
    this.player.setState(new StoppedState(this.player));
  }
}

const player = new MediaPlayer();

player.play();  // Starting playback...
player.pause(); // Pausing...
player.play();  // Resuming...
player.stop();  // Stopping...
```

### Pros
✅ Single Responsibility Principle. Organize the code related to particular states into separate classes  
✅ Open/Closed Principle. Introduce new states without changing existing state classes or the context  
✅ Eliminates large conditional statements

### Cons
❌ Increases number of classes  
❌ Can feel overengineered for simple cases  
❌ State transitions can become hard to track  

### 3.6 Mediator Pattern
Mediator defines an object that centralizes communication between other objects.

```TS
interface Mediator {
  send(message: string, sender: User): void;
}

class ChatRoom implements Mediator {
  private users: User[] = [];

  addUser(user: User) {
    this.users.push(user);
  }

  send(message: string, sender: User): void {
    for (const user of this.users) {
      if (user !== sender) {
        user.receive(message);
      }
    }
  }
}

class User {
  constructor(
    public name: string,
    private mediator: Mediator
  ) {}

  send(message: string) {
    console.log(`${this.name} sends: ${message}`);
    this.mediator.send(message, this);
  }

  receive(message: string) {
    console.log(`${this.name} receives: ${message}`);
  }
}

const chatRoom = new ChatRoom();

const alice = new User("Alice", chatRoom);
const bob = new User("Bob", chatRoom);
const charlie = new User("Charlie", chatRoom);

chatRoom.addUser(alice);
chatRoom.addUser(bob);
chatRoom.addUser(charlie);

alice.send("Hello everyone!");
```

### Pros
✅ Reduces direct dependencies between objects  
✅ Centralizes communication logic  

### Cons
❌ Single point of failure  
❌ Mediator can become a “God object”  
❌ Performance overhead  

### 3.7 Iterator Pattern
Iterator provides a way to access elements of a collection sequentially without exposing its internal structure.

```TS
class Song {
  constructor(public title: string) {}
}

interface CustomIterator<T> {
  hasNext(): boolean;
  next(): T;
}

class Playlist {
  private songs: Song[] = [];

  add(song: Song) {
    this.songs.push(song);
  }

  createIterator() {
    return new PlaylistIterator(this.songs);
  }

  createRandomIterator() {
    return new RandomPlaylistIterator(this.songs);
  }

  createSortedIterator() {
    return new SortedPlaylistIterator(this.songs);
  }
}

class PlaylistIterator implements CustomIterator<Song> {
  private index = 0;

  constructor(private songs: Song[]) {}

  hasNext(): boolean {
    return this.index < this.songs.length;
  }

  next(): Song {
    return this.songs[this.index++];
  }
}

class RandomPlaylistIterator implements CustomIterator<Song> {
  private index = 0;
  private shuffled: Song[];

  constructor(songs: Song[]) {
    this.shuffled = [...songs].sort(() => Math.random() - 0.5);
  }

  hasNext(): boolean {
    return this.index < this.shuffled.length;
  }

  next(): Song {
    return this.shuffled[this.index++];
  }
}

class SortedPlaylistIterator implements CustomIterator<Song> {
  private index = 0;
  private sorted: Song[];

  constructor(songs: Song[]) {
    this.sorted = [...songs].sort((a, b) =>
      a.title.localeCompare(b.title)
    );
  }

  hasNext(): boolean {
    return this.index < this.sorted.length;
  }

  next(): Song {
    return this.sorted[this.index++];
  }
}

const playlist = new Playlist();

playlist.add(new Song("Song A"));
playlist.add(new Song("Song B"));
playlist.add(new Song("Song C"));

console.log("Playlist");

const iterator = playlist.createIterator();

while (iterator.hasNext()) {
  const song = iterator.next();
  console.log(song.title);
}

console.log("Random playlist");

const random = playlist.createRandomIterator();

while (random.hasNext()) {
  console.log(random.next().title);
}

console.log("Sorted playlist");

const sorted = playlist.createSortedIterator();

while (sorted.hasNext()) {
  console.log(sorted.next().title);
}
```

### Pros
✅ Follows Single Responsibility Principle  
✅ Simplifies iteration logic  

### Cons
❌ Adds extra classes / abstraction  
❌ Slight overhead for simple collections  

### 3.8 Template Method Pattern
The Template Method pattern defines the skeleton of an algorithm in a base class, while letting subclasses override specific steps without changing the algorithm structure.

```TS
abstract class DataImporter {
  // Template method (algorithm skeleton)
  import(): void {
    const raw = this.load();
    const parsed = this.parse(raw);
    this.validate(parsed);
    this.save(parsed);
  }

  protected abstract load(): string;

  protected abstract parse(data: string): any[];

  protected validate(data: any[]): void {
    console.log("Validating data...");
  }

  protected save(data: any[]): void {
    console.log("Saving", data.length, "records to DB");
  }
}

class CsvImporter extends DataImporter {
  protected load(): string {
    console.log("Loading CSV file...");
    return "John,30\nAlice,25";
  }

  protected parse(data: string): any[] {
    console.log("Parsing CSV...");
    return data.split("\n").map(row => {
      const [name, age] = row.split(",");
      return { name, age: Number(age) };
    });
  }
}

class JsonApiImporter extends DataImporter {
  protected load(): string {
    console.log("Fetching JSON API...");
    return JSON.stringify([
      { name: "John", age: 30 },
      { name: "Alice", age: 25 }
    ]);
  }

  protected parse(data: string): any[] {
    console.log("Parsing JSON...");
    return JSON.parse(data);
  }
}

const csvImporter = new CsvImporter();
csvImporter.import();

console.log("-----");

const jsonImporter = new JsonApiImporter();
jsonImporter.import();

### Pros
✅ Avoids code duplication  
✅ Easy to extend  
✅ Follows Open/Closed Principle  

### Cons
❌ Uses inheritance  
❌ Can create tight coupling  
❌ Base class can become too complex  
❌ Maintenance can be difficult  
```
