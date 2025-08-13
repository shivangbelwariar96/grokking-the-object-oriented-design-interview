# Enhanced System Design Tables - 34 Systems

Of course. While the initial designs are a strong foundation, they can be enhanced to be more robust, scalable, and reflective of real-world complexities, which is what interviewers look for.

The following tables refine your designs by introducing more advanced concepts, improving concurrency handling, applying design patterns more precisely, and ensuring better separation of concerns (SRP). These improvements are critical for demonstrating senior-level thinking in a low-level design round.

---

## 1. Parking Lot System 🚗

This enhanced design better separates concerns. The `ParkingLot` manages the physical space, while a new **`BillingService`** handles all financial calculations, adhering more closely to SRP. Concurrency is managed at the floor and spot level.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`ParkingLot`** | `private static final ParkingLot instance;`<br>`private final List<ParkingFloor> floors;`<br>`private final EntranceGate entryGate;`<br>`private final ExitGate exitGate;` | `public static synchronized ParkingLot getInstance();` **(Singleton)**<br>`public Ticket enter(Vehicle v);`<br>`public double exit(Ticket t);` **(Facade** over the entry/exit/billing process) |
| **`ParkingFloor`** | `private final Map<ParkingSpotType, Deque<ParkingSpot>> availableSpots;` | `public synchronized ParkingSpot findAndPark(Vehicle v);`<br>`public synchronized void releaseSpot(ParkingSpot spot);` **(Uses `ConcurrentLinkedDeque`** for thread-safe LIFO/FIFO) |
| **`ParkingSpot`** | `private final String spotId;`<br>`private final ParkingSpotType type;`<br>`private volatile Vehicle parkedVehicle;` | `public boolean isAvailable();`<br>`public void assignVehicle(Vehicle v);`<br>`public void removeVehicle();` **(`volatile`** keyword ensures visibility of `parkedVehicle` across threads) |
| **`Vehicle`** (Abstract) | `protected final String licensePlate;`<br>`protected final VehicleType type;` | (Base class for `Car`, `Motorcycle`. Could be an interface if no shared implementation is needed.) |
| **`Ticket`** | `private final String ticketId;`<br>`private final Vehicle vehicle;`<br>`private final ParkingSpot spot;`<br>`private final Instant entryTime;` | (**Immutable DTO**. Storing `entryTime` here is crucial for billing.) |
| **`BillingService`** | `private final IPricingStrategy pricingStrategy;` | `public double calculatePrice(Ticket t, Instant exitTime);` **(Single Responsibility:** Pricing logic is now separate from the `ParkingLot`.) |
| **`IPricingStrategy`** (Interface) | | `BigDecimal calculate(Ticket ticket, Instant exitTime);` **(Strategy Pattern:** Allows for different pricing models like `HourlyPricing`, `DailyPricing`.) |
| **`HourlyPricingStrategy`** | | `@Override public BigDecimal calculate(...);` (Concrete implementation) |
| **`VehicleType`** (Enum) | `CAR, MOTORCYCLE, TRUCK` | (Ensures type-safety for vehicles.) |
| **`ParkingSpotType`** (Enum) | `COMPACT, LARGE, MOTORBIKE` | (Ensures type-safety for spots.) |

---

## 2. Elevator System 🏙️

This design solidifies the Mediator and State patterns. The `ElevatorController` uses a **`BlockingQueue`** to handle incoming requests asynchronously, preventing the main thread from blocking and ensuring requests are processed fairly.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`ElevatorController`** | `private static final ElevatorController instance;`<br>`private final List<ElevatorCar> elevators;`<br>`private final BlockingQueue<Request> requestQueue;` | `public static synchronized getInstance();` **(Singleton)**<br>`public void submitRequest(Request request);` **(Mediator Pattern:** Decouples buttons from elevators. Uses a `BlockingQueue` to safely handle concurrent requests.) |
| **`ElevatorCar`** (Runnable) | `private int currentFloor;`<br>`private Direction direction;`<br>`private ElevatorState state;`<br>`private final PriorityQueue<Integer> upRequests;`<br>`private final PriorityQueue<Integer> downRequests;` | `public void run();` (The core logic loop)<br>`public void setState(ElevatorState state);` **(Context** for State Pattern)<br>`public void addNewRequest(int floor);` |
| **`ElevatorState`** (Interface) | | `void handle(ElevatorCar car);` **(State Pattern:** A cleaner way to manage behavior than a large switch statement. Each state implements this.) |
| **`MovingUpState`, `IdleState`, `DoorOpenState`** | | (Concrete states manage logic for moving, stopping, opening/closing doors, etc. `IdleState` might check request queues to decide where to go next.) |
| **`Request`** | `private final int originFloor;`<br>`private final Direction direction;` | (**Immutable DTO** representing an external button press.) |
| **`Button`** (Abstract) | `protected final ElevatorController controller;` | `public abstract void press();` (Sends a request to the controller) |
| **`InternalButton`** | `private final int destinationFloor;` | (Button inside the elevator car.) |
| **`ExternalButton`** | `private final int floorNumber;`<br>`private final Direction direction;` | (Up/Down button on a building floor.) |
| **`Direction`** (Enum) | `UP, DOWN, IDLE` | (Represents elevator direction and request type.) |

---

## 3. LRU Cache 💾

The design is enhanced to explicitly use a **`ReentrantLock`** for fine-grained control, which offers better performance under high contention than synchronizing entire methods.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`LRUCache<K, V>`** | `private final int capacity;`<br>`private final Map<K, Node<K, V>> cacheMap;`<br>`private final DoublyLinkedList<K, V> dll;`<br>`private final ReentrantLock lock = new ReentrantLock();` | `public V get(K key);`<br>`public void put(K key, V value);` **(Methods use `lock.lock()` and `lock.unlock()`** for thread-safety. This is more performant than synchronizing the whole method.) |
| **`DoublyLinkedList<K, V>`** (private) | `private final Node<K, V> head, tail;` | `private void removeNode(Node<K, V> node);`<br>`private void addToFront(Node<K, V> node);`<br>`private Node<K, V> removeLast();` (Encapsulates the list logic.) |
| **`Node<K, V>`** (private) | `K key, V value;`<br>`Node<K, V> prev, next;` | (Private static nested class representing a node in the doubly-linked list.) |
| **Note on `LinkedHashMap`** | | For a practical solution, show awareness of Java's **`LinkedHashMap`**. It can be configured as an LRU cache by constructing it with `accessOrder=true` and overriding the `removeEldestEntry` method. This is often a "good enough" and much simpler solution. |

---

## 4. Movie Ticket Booking System 🎬

Concurrency is improved significantly. Instead of locking the entire `Show` object, we introduce a **`SeatLockProvider`** to achieve finer-grained locking on a per-seat basis, which is far more scalable and realistic.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`BookingSystem`** | `private final MovieService movieService;`<br>`private final TheaterService theaterService;`<br>`private final BookingService bookingService;` | `public List<Show> getShows(...);`<br>`public Booking bookTickets(...);` **(Facade** that delegates to specialized services.) |
| **`Show`** | `private final Movie movie;`<br>`private final CinemaHall hall;`<br>`private final Map<String, SeatStatus> seatStatuses;` | `public Map<String, SeatStatus> getSeatMap();` (**Concurrency handled by BookingService**, not by synchronizing methods here.) |
| **`BookingService`** | `private final SeatLockProvider lockProvider;`<br>`private final PaymentService paymentService;` | `public Booking createBooking(Show show, List<Seat> seats, User user);` **(Manages the booking workflow:** lock seats -> process payment -> confirm booking.) |
| **`SeatLockProvider`** | `private final Map<String, Lock> seatLocks;` | `boolean lockSeats(Show show, List<Seat> seats, String userId);`<br>`void unlockSeats(Show show, List<Seat> seats, String userId);` **(Manages temporary locks on seats using `ReentrantLock`** for each seat to prevent race conditions.) |
| **`Booking`** | `private final String bookingId;`<br>`private final Show show;`<br>`private Payment payment;`<br>`private BookingStatus status;` | `public boolean makePayment(PaymentStrategy strategy);` |
| **`PaymentService`** | | `boolean processPayment(PaymentStrategy strategy, BigDecimal amount);` **(Single Responsibility:** Payment logic is externalized.) |
| **`Seat`** | `private final String seatId;`<br>`private final SeatType type;`<br>`private final int row;`<br>`private final int col;` | (**Immutable DTO** representing a seat's physical properties.) |
| **`SeatType`** (Enum) | `REGULAR, PREMIUM, RECLINER` | (Type-safe seat categories.) |
| **`SeatStatus`** (Enum) | `AVAILABLE, BOOKED, LOCKED` | (Represents the current state of a seat for a specific `Show`.) |
| **`BookingStatus`** (Enum) | `PENDING, CONFIRMED, CANCELLED` | (Lifecycle of a booking.) |

---

## 5. E-commerce Website 🛒

The design is enhanced with a dedicated `ProductCatalog` for searching/Browse and an `InventoryService` that uses **`AtomicInteger`** for thread-safe stock management, a more efficient approach than method-level synchronization for simple counters.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`User`** | `private final ShoppingCart cart;`<br>`private final List<Order> orderHistory;` | `public void addItemToCart(...);`<br>`public Order checkout(IPaymentStrategy paymentMethod);` |
| **`ProductCatalog`** | `private final SearchService searchService;`<br>`private final InventoryService inventoryService;` | `public List<Product> searchProducts(String query);`<br>`public Product getProductDetails(String productId);` **(Facade** for product discovery.) |
| **`InventoryService`** | `private final Map<String, AtomicInteger> productStock;` | `public static synchronized getInstance();` **(Singleton)**<br>`public boolean reserveStock(String productId, int quantity);`<br>`public void releaseStock(String productId, int quantity);` **(Uses `AtomicInteger`** for thread-safe stock counts.) |
| **`ShoppingCart`** | `private final Map<Product, Integer> items;` | `public synchronized void addItem(Product p, int qty);`<br>`public BigDecimal calculateTotal();` (A user's temporary, session-based cart.) |
| **`Order`** (Subject) | `private final List<OrderItem> items;`<br>`private OrderStatus status;`<br>`private final List<IOrderObserver> observers;` | `public void setStatus(OrderStatus newStatus);` (Notifies observers like email service)<br>`public void registerObserver(IOrderObserver o);` **(Observer Pattern)** |
| **`OrderItem`** | `private final Product product;`<br>`private final int quantity;`<br>`private final BigDecimal priceAtPurchase;` | (**Immutable DTO**. Critically, it saves the **price at the time of purchase**.) |
| **`IOrderObserver`** (Interface) | | `void onStatusUpdate(Order order);` (Allows `EmailNotifier`, `SmsNotifier`, `WarehouseNotifier` to react to order status changes.) |
| **`OrderStatus`** (Enum) | `PENDING, CONFIRMED, SHIPPED, DELIVERED` | (Represents the lifecycle of an order.) |
| **`IPaymentStrategy`** (Interface) | | `boolean processPayment(BigDecimal amount);` **(Strategy Pattern)** |

---

## 6. Ride-Sharing Service (Uber/Lyft) 🚕

The design is improved by making the driver location updates more explicit through a `LocationService` using a **`ConcurrentHashMap`** for thread-safety. A dedicated `TripService` now manages the lifecycle of a trip, applying SRP more effectively.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`RideDispatchSystem`** | `private final LocationService locationService;`<br>`private final TripService tripService;` | `public Trip requestRide(Rider rider, Location destination, VehicleType type);`<br>`public void cancelTrip(Trip trip);` **(Facade/Mediator:** A central point to coordinate the entire process.) |
| **`LocationService`** | `private final Map<String, Location> driverLocations;` | `public Driver findNearestDriver(Location loc, VehicleType type);`<br>`public void updateDriverLocation(String driverId, Location newLoc);` **(Uses a `ConcurrentHashMap`** for `driverLocations` for thread-safety.) |
| **`TripService`** | `private final FareService fareService;` | `public Trip createTrip(Rider r, Driver d, Location start, Location end);`<br>`public void startTrip(Trip t);`<br>`public void endTrip(Trip t);` **(Single Responsibility:** Manages trip state transitions.) |
| **`Driver`** (extends User) | `private final Vehicle vehicle;`<br>`private volatile DriverStatus status;`<br>`private Location currentLocation;` | `public void acceptTrip(Trip trip);`<br>`public void updateLocation(Location newLoc);` (Calls `LocationService.updateDriverLocation`) |
| **`Trip`** | `private Rider rider;`<br>`private Driver driver;`<br>`private TripStatus status;`<br>`private BigDecimal fare;` | (Represents the full lifecycle of a ride from request to completion.) |
| **`FareService`** | | `BigDecimal calculateFare(Trip trip, IFareStrategy strategy);` **(Strategy Pattern** via `IFareStrategy` allows dynamic fare calculation like `StandardFareStrategy` vs. `SurgePricingStrategy`.) |
| **`IFareStrategy`** (Interface) | | `BigDecimal calculate(TripDetails details);` |
| **`DriverStatus`** (Enum) | `AVAILABLE, BUSY, OFFLINE` | (Manages driver availability.) |
| **`TripStatus`** (Enum) | `REQUESTED, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED` | (Manages trip lifecycle.) |
| **`Location`** | `private final double lat, lon;` | (**Immutable value object**.) |

---

## 7. Car Rental System 🚙

This refined design introduces a **`VehicleInventory`** per location, making the management of vehicles more structured and realistic for a multi-location agency. All inventory methods must be thread-safe.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`CarRentalSystem`** | `private final List<RentalLocation> locations;`<br>`private final ReservationService reservationService;` | `public List<Vehicle> searchVehicles(...);`<br>`public Reservation makeReservation(...);` **(Facade:** Simplifies interactions for the end-user.) |
| **`RentalLocation`** | `private final String address;`<br>`private final VehicleInventory inventory;` | `public List<Vehicle> getAvailableVehicles(...);`<br>`public Bill returnVehicle(Reservation r);` |
| **`VehicleInventory`** | `private final Map<VehicleType, List<Vehicle>> availableVehicles;`<br>`private final Map<String, Vehicle> allVehicles;` | `public synchronized void addVehicle(Vehicle v);`<br>`public synchronized Vehicle reserveVehicle(VehicleType type);`<br>`public synchronized void markAsAvailable(Vehicle v);` **(All methods must be thread-safe**.) |
| **`Vehicle`** (Abstract) | `protected final String licensePlate;`<br>`protected volatile VehicleStatus status;` | (Base class for different vehicle models like `Sedan`, `SUV`.) |
| **`ReservationService`** | `private final BillingService billingService;` | `public Reservation createReservation(...);`<br>`public void cancelReservation(...);`<br>`public Bill processReturn(Reservation r, ...);` |
| **`Reservation`** | `private final String reservationId;`<br>`private final User user;`<br>`private final Vehicle vehicle;`<br>`private final Instant pickupTime, dropoffTime;` | (**Immutable DTO** representing a confirmed booking.) |
| **`Bill`** | `private final Reservation reservation;`<br>`private final BigDecimal totalAmount;` | `public boolean processPayment(IPaymentStrategy p);` |
| **`VehicleType`** (Enum) | `ECONOMY, SUV, LUXURY, TRUCK` | (Type-safe vehicle categories.) |
| **`VehicleStatus`** (Enum) | `AVAILABLE, RESERVED, IN_SERVICE, RENTED` | (Manages the state of a specific vehicle.) |
| **`IPaymentStrategy`** (Interface) | | `boolean pay(BigDecimal amount);` **(Strategy Pattern)** |

---

## 8. ATM System 💰

The design is improved by explicitly including the **Command Pattern** for transactions. This decouples the ATM from the execution of specific financial operations, making it extensible for new transaction types (e.g., `FundsTransfer`).

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`ATM`** | `private ATMState currentState;`<br>`private final CardReader reader;`<br>`private final CashDispenser dispenser;`<br>`private final BankService bankServiceProxy;` | `public void insertCard(Card card);`<br>`public void ejectCard();`<br>`public void setState(ATMState newState);` **(Context** for State Pattern) |
| **`ATMState`** (Interface) | | `void insertCard(ATM atm, Card card);`<br>`void enterPin(ATM atm, String pin);`<br>`void selectOperation(ATM atm, ...);` **(State Pattern)** |
| **`IdleState`, `HasCardState`, `AuthenticatedState`** | | (Each concrete state implements the allowed behaviors. E.g., `IdleState` only allows `insertCard`, while `AuthenticatedState` allows transactions.) |
| **`BankService`** (Interface) | | `boolean authenticate(String cardId, String pin);`<br>`TransactionStatus executeTransaction(ITransaction txn);` **(Facade/Proxy:** Hides complexity of the bank's backend.) |
| **`ITransaction`** (Interface) | | `TransactionStatus execute();` **(Command Pattern:** Encapsulates a request like withdrawal or deposit as an object.) |
| **`Withdrawal`, `Deposit`, `BalanceInquiry`** | `private final Account account;`<br>`private final BigDecimal amount;` | `@Override public TransactionStatus execute();` **(Concrete command objects**. They contain all info needed to perform the action.) |
| **`Card`** | `private final String cardId;` | (**Immutable DTO** representing the user's ATM card.) |
| **`CashDispenser`** | `private final Map<Denomination, Integer> bills;` | `public synchronized boolean dispense(BigDecimal amount);` |

---

## 9. Notification System 📧

This design is enhanced by explicitly using an **`ExecutorService`** for asynchronous, non-blocking notification delivery, a critical feature for a high-throughput, real-world system.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`NotificationService`** | `private final ChannelFactory channelFactory;`<br>`private final ExecutorService threadPool;` | `public void send(Notification notification);` **(Submits a task to the `threadPool` to send notifications asynchronously**, preventing the caller from blocking.) |
| **`Notification`** | `private final Recipient recipient;`<br>`private final MessageContent content;`<br>`private final List<ChannelType> channels;` | (**Immutable** object created via the Builder Pattern. Guarantees a valid, complete notification object before sending.) |
| **`NotificationBuilder`** | (Mutable attributes for recipient, content, etc.) | `public NotificationBuilder to(...);`<br>`public NotificationBuilder withContent(...);`<br>`public Notification build();` **(Builder Pattern)** |
| **`IChannel`** (Interface) | | `boolean send(Notification notification);` **(Strategy Pattern:** Each channel implements its own sending logic.) |
| **`EmailChannel`, `SmsChannel`, `PushChannel`** | | `@Override public boolean send(Notification n);` (Concrete channel implementations, interacting with external APIs like SendGrid, Twilio, etc.) |
| **`ChannelFactory`** | `private final Map<ChannelType, IChannel> channelCache;` | `public IChannel getChannel(ChannelType type);` **(Factory Pattern / Singleton:** Provides channel instances, potentially reusing them.) |
| **`Recipient`** | `private String email;`<br>`private String phoneNumber;`<br>`private String deviceToken;` | (DTO containing all possible contact points for a user.) |
| **`ChannelType`** (Enum) | `EMAIL, SMS, PUSH` | (Type-safe representation of notification channels.) |

---

## 10. Logging Framework 📝

The design is enhanced by clarifying the roles. `LogManager` is a factory for loggers. The `Logger` itself is the entry point, and the **Chain of Responsibility** is implemented through its hierarchical parent-child relationship.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`LogManager`** | `private static final LogManager instance;`<br>`private final Logger rootLogger;` | `public static synchronized getInstance();` **(Singleton)**<br>`public Logger getLogger(String name);` **(Factory Method** for creating or retrieving loggers.) |
| **`Logger`** | `private final String name;`<br>`private LogLevel level;`<br>`private final List<IAppender> appenders;`<br>`private final Logger parent;` | `public void log(LogLevel lvl, String msg);` (If lvl is sufficient, logs via appenders, then passes to parent: **Chain of Responsibility**.) |
| **`IAppender`** (Interface) | | `void append(LoggingEvent event);` **(Strategy Pattern:** Defines the output destination.) |
| **`ConsoleAppender`, `FileAppender`, `NetworkAppender`** | | `@Override public void append(LoggingEvent event);` **(These must be thread-safe** as multiple threads can log concurrently.) |
| **`ILayout`** (Interface) | | `String format(LoggingEvent event);` **(Strategy Pattern:** Defines the message format.) |
| **`PatternLayout`, `JsonLayout`** | | `@Override public String format(LoggingEvent event);` (Concrete formatters.) |
| **`LoggingEvent`** | `private final LogLevel level;`<br>`private final String message;`<br>`private final Instant timestamp;`<br>`private final String threadName;` | (**Immutable DTO** carrying all log information.) |
| **`LogLevel`** (Enum) | `TRACE, DEBUG, INFO, WARN, ERROR, FATAL` | (Defines standard logging verbosity levels.) |

---

## 11. Chat Application (WhatsApp/Slack) 💬

The design is improved by adding a **`MessageStore`** for persistence and a **`ConnectionManager`** to handle the real-world complexity of user sessions and connections (e.g., WebSockets), separating concerns from the `UserManager`.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`ChatServer`** (Mediator) | `private final UserManager userManager;`<br>`private final ChatManager chatManager;`<br>`private final MessageStore messageStore;` | `public void sendMessage(Message msg);` **(Mediator Pattern:** Routes messages between users/chats.)<br>`public void deliverMessage(Message msg);` (Pushes message to recipient's connection.) |
| **`UserManager`** | `private final Map<String, User> allUsers;`<br>`private final ConnectionManager connectionManager;` | `public void userOnline(User u, Connection c);`<br>`public void userOffline(User u);` |
| **`Chat`** (Abstract) | `protected final String chatId;`<br>`protected final List<User> participants;` | `public void addMessage(Message msg);` (Adds message to chat history.) |
| **`PrivateChat`, `GroupChat`** | | (Inherits from `Chat` for 1-to-1 and group conversations.) |
| **`User`** (Observer) | `private final String userId;`<br>`private Connection connection;` | `public void sendMessage(...);`<br>`public void receiveMessage(Message msg);` (Callback method triggered by the `ChatServer`.) |
| **`MessageStore`** | `// Connection to a database like Cassandra` | `void saveMessage(Message msg);`<br>`List<Message> getChatHistory(String chatId);` **(Persistence Layer)** |
| **`Message`** | `private final String messageId;`<br>`private final User sender;`<br>`private final String content;`<br>`private final Instant timestamp;` | (**Immutable DTO** for message content.) |
| **`UserStatus`** (Enum) | `ONLINE, OFFLINE, AWAY` | (Manages user presence.) |
| **`MessageStatus`** (Enum) | `SENT, DELIVERED, READ` | (Manages the delivery lifecycle of a message.) |

---

## 12. Publish-Subscribe System (Kafka Lite) 📡

This design is refined by making the **`Topic`** explicitly manage its own consumer groups and their offsets using `AtomicInteger`, which is a core concept in real pub-sub systems like Kafka for tracking consumer progress.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Broker`** | `private static final Broker instance;`<br>`private final Map<String, Topic> topics;` | `public static synchronized getInstance();` **(Singleton/Facade)**<br>`public void publish(String topicName, Message msg);`<br>`public void subscribe(String topicName, String groupId, ISubscriber sub);` |
| **`Topic`** | `private final String name;`<br>`private final BlockingQueue<Message> messageQueue;`<br>`private final Map<String, List<ISubscriber>> subscribersByGroup;`<br>`private final Map<String, AtomicInteger> offsetsByGroup;` | `public void addSubscriber(String groupId, ISubscriber sub);`<br>`public void publishMessage(Message msg);` (Notifies subscribers)<br>`public void dispatchMessages();` (A worker thread could run this to push messages.) |
| **`ISubscriber`** (Interface) | | `void onMessage(Message message);` **(Observer Pattern:** The "consumer" contract.)<br>`String getSubscriberId();`<br>`String getGroupId();` |
| **`SubscriberImpl`** | `private final String id;`<br>`private final String groupId;` | `@Override public void onMessage(Message message);`<br>`// ... getters for id and groupId` (A concrete consumer.) |
| **`Message`** | `private final String key;`<br>`private final String payload;`<br>`private final Instant timestamp;` | (**Immutable DTO** representing the data being published.) |
| **`Publisher`** | `private final Broker broker;` | `public void publish(String topicName, Message msg);` (A client class that sends messages to the broker.) |

---

## 13. Stack Overflow ❓

The design is enhanced by introducing a **`VotingService`** to handle the complex reputation logic (SRP) and adding an abstract **`Post`** base class for `Question` and `Answer` to reduce code duplication.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Post`** (Abstract) | `protected final Member author;`<br>`protected int voteCount;`<br>`protected final List<Comment> comments;` | `public void addComment(...);`<br>`public void edit(...)` |
| **`Question`** (extends Post) | `private final String title;`<br>`private final List<Answer> answers;`<br>`private QuestionState state;` | `public void addAnswer(Answer answer);`<br>`public void setState(QuestionState s);` **(Context** for State Pattern) |
| **`Answer`** (extends Post) | `private boolean isAccepted;` | `public void markAsAccepted();` |
| **`Member`** | `private final String userId;`<br>`private int reputation;`<br>`private final List<Badge> badges;` | `public void createQuestion(...);`<br>`public void addAnswer(...);` |
| **`VotingService`** | | `public synchronized void vote(Member voter, Post post, VoteType type);` **(Single Responsibility:** Manages voting logic and reputation updates.) |
| **`QuestionState`** (Interface) | | `void addAnswer(Question q, Answer a);`<br>`void close(Question q);` **(State Pattern:** Encapsulates behavior associated with a question's state.) |
| **`OpenState`, `ClosedState`, `OnHoldState`** | | (Concrete states defining permissible actions. E.g., you can't add an answer to a closed question.) |
| **`VoteType`** (Enum) | `UPVOTE, DOWNVOTE` | (Type-safe voting actions.) |
| **`QuestionStatus`** (Enum) | `OPEN, CLOSED, ON_HOLD, DELETED` | (Managed by the State pattern.) |

---

## 14. Library Management System 📚

The design is improved by separating search functionality into a dedicated **`SearchService`**
