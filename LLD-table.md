# LLD System Design Tables - 34 Systems


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

The design is improved by separating search functionality into a dedicated **`SearchService`** that uses the Strategy Pattern (`ISearchStrategy`). This makes the system more modular and allows for new search types (e.g., by genre, publication year) to be added easily.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Library`** | `private final Catalog catalog;`<br>`private final MemberService memberService;`<br>`private final LoanService loanService;` | `public List<BookItem> search(...);`<br>`public boolean checkoutBook(...);` **(Facade:** Simplifies common library operations.) |
| **`Catalog`** | `private final Map<String, Book> booksByIsbn;`<br>`private final SearchService searchService;` | `public Book getBook(String isbn);` |
| **`SearchService`** | | `public List<Book> search(ISearchStrategy strategy, String query);` **(Strategy Pattern** lets us easily add new search types.) |
| **`ISearchStrategy`** (Interface) | | `List<Book> search(String query, Catalog catalog);` |
| **`SearchByTitle`, `SearchByAuthor`** | | (Concrete search algorithms.) |
| **`Book`** (Subject) | `private final String ISBN;`<br>`private final String title;`<br>`private final List<Member> reservationWaitlist;` | `public void addToWaitlist(Member m);`<br>`public void notifyNextMember();` **(Observer Pattern** for reservations) |
| **`BookItem`** (extends Book) | `private final String barcode;`<br>`private BookStatus status;` | `public boolean checkout(Member member);` (A specific physical copy.) |
| **`Member`** (Observer) | `private final String memberId;`<br>`private final List<BookItem> booksLoaned;` | `public void onBookAvailable(Book book);` (Observer's update method, called when a reserved book is available.) |
| **`BookStatus`** (Enum) | `AVAILABLE, LOANED, RESERVED, LOST` | (Manages the state of a physical book copy.) |

---

## 15. Vending Machine 🍬

This is a canonical example of the State Pattern. The refinement here is to introduce a **`Coin`** enum with values and a dedicated **`CashBox`** for managing money, improving type safety and separating money handling from the main machine logic.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`VendingMachine`** | `private State currentState;`<br>`private Inventory inventory;`<br>`private CashBox cashBox;`<br>`private Item selectedItem;` | `public void insertCoin(Coin c);`<br>`public void selectItem(int code);`<br>`public void dispenseItem();`<br>`public void setState(State newState);` **(Context** for State Pattern) |
| **`State`** (Interface) | | `void insertCoin(VendingMachine m, Coin c);`<br>`void selectItem(VendingMachine m, int code);`<br>`void dispenseItem(VendingMachine m);` **(State Pattern)** |
| **`NoCoinState`, `HasCoinState`, `SoldState`, `SoldOutState`** | | (Concrete states handle actions. `NoCoinState` accepts coins. `HasCoinState` accepts item selection. `SoldState` dispenses and transitions.) |
| **`Inventory`** | `private final Map<Integer, ItemSlot> slots;` | `public Item getItem(int itemCode);`<br>`public int getQuantity(int itemCode);`<br>`public synchronized void dispenseItem(int itemCode);` |
| **`ItemSlot`** | `private final Item item;`<br>`private int count;` | `public synchronized void decrementCount();` (Must be synchronized.) |
| **`CashBox`** | `private BigDecimal currentBalance;`<br>`private BigDecimal totalCash;` | `public void addCoin(Coin c);`<br>`public BigDecimal getCurrentBalance();`<br>`public void clearBalance();`<br>`public BigDecimal returnChange();` |
| **`Item`** | `private final String name;`<br>`private final BigDecimal price;` | (**Immutable DTO** for item details.) |
| **`Coin`** (Enum) | `NICKEL(0.05), DIME(0.10), QUARTER(0.25), DOLLAR(1.00);`<br>`private final BigDecimal value;` | (**Type-safe representation of coins** with their values.) |

---

## 16. Blackjack and Deck of Cards 🃏

This classic OOP problem is enhanced by making the Dealer's play logic an explicit **Strategy (`IDealerPlayStrategy`)**, allowing for different house rules (e.g., "hit on soft 17") without changing the `Dealer` class itself.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`BlackjackGame`** | `private final Deck deck;`<br>`private final Dealer dealer;`<br>`private final List<Player> players;`<br>`private GameState gameState;` | `public void startGame();`<br>`public void playRound();`<br>`private void determineWinners();` (Orchestrates the game flow.) |
| **`Card`** | `private final Suit suit;`<br>`private final Rank rank;` | `public int getValue();` (**Immutable**: A card's value and suit never change.) |
| **`Deck`** | `private final Deque<Card> cards;` | `public void shuffle();`<br>`public synchronized Card deal();` **(Using `ConcurrentLinkedDeque`** is ideal for a multi-threaded game.) |
| **`Hand`** | `private final List<Card> cards;` | `public void addCard(Card card);`<br>`public int calculateValue();` (Handles Aces being 1 or 11.)<br>`public boolean isBusted();` |
| **`Participant`** (Abstract) | `protected final Hand hand;` | `public void hit(Deck deck);`<br>`public void stand();` |
| **`Player`** (extends Participant) | `private final String name;`<br>`private BigDecimal balance;` | `public void placeBet(BigDecimal amount);` |
| **`Dealer`** (extends Participant) | `private final IDealerPlayStrategy playStrategy;` | `public void play(Deck deck);` (Delegates decision-making to its strategy.) |
| **`IDealerPlayStrategy`** (Interface) | | `boolean shouldHit(Hand hand);` **(Strategy Pattern:** Implements house rules, e.g., `HitOnSoft17Strategy`, `StandOn17Strategy`.) |
| **`Suit`** (Enum) | `HEARTS, DIAMONDS, CLUBS, SPADES` | (Type-safe suits.) |
| **`Rank`** (Enum) | `TWO(2), ..., JACK(10), ..., ACE(11)`<br>`private final int value;` | (Type-safe ranks and their base values.) |

---

## 17. Food Delivery System (DoorDash) 🍔

This system is heavily driven by the Observer pattern. The design is refined to use a **`OrderDispatchSystem` (Mediator)** to decouple restaurants and drivers, preventing them from needing direct knowledge of each other and centralizing the complex assignment logic.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`OrderService`** | `private final OrderDispatchSystem dispatchSystem;`<br>`private final PaymentService paymentService;` | `public Order placeOrder(...);` **(Facade** for the entire ordering process.) |
| **`Order`** (Subject) | `private OrderStatus status;`<br>`private final List<IOrderObserver> observers;` | `public void setStatus(OrderStatus newStatus);` (Notifies all observers)<br>`public void addObserver(IOrderObserver o);`<br>`public void removeObserver(IOrderObserver o);` |
| **`IOrderObserver`** (Interface) | | `void update(Order order);` **(Observer Pattern:** Decouples the order from entities that care about its status.) |
| **`Customer`, `Restaurant`, `Driver`** | | `@Override public void update(Order order);` (Each party reacts differently. Restaurant starts cooking, Driver starts moving, etc.) |
| **`OrderDispatchSystem`** (Mediator) | `private final LocationService locationService;` | `public void assignDriverToOrder(Order order);` **(Finds the best driver for an order** without the restaurant needing to know about drivers.) |
| **`LocationService`** | `private final Map<String, Location> driverLocations;`<br>`private final Map<String, Location> restaurantLocations;` | `Driver findBestDriverFor(Restaurant r);` |
| **`OrderStatus`** (Enum) | `PLACED, CONFIRMED, PREPARING, READY_FOR_PICKUP, PICKED_UP, DELIVERED, CANCELLED` | (A more detailed lifecycle for a food delivery order.) |

---

## 18. Online Auction System ⚖️

This design combines the State and Observer patterns. It's improved by adding a **`Bid`** class to represent the action as an immutable DTO and making the `placeBid` method on `Auction` **`synchronized`**, which is critical to prevent race conditions from multiple bidders.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`AuctionService`** | `private final Map<String, Auction> auctions;` | `public Auction createAuction(...);`<br>`public boolean placeBid(String auctionId, ...);` **(Facade** for auction management.) |
| **`Auction`** (Subject) | `private Bid highestBid;`<br>`private AuctionState state;`<br>`private final transient List<IBidder> bidders;` | `public synchronized boolean placeBid(Bid bid);` **(Thread-safe** to prevent race conditions. Delegates validation to the current state object.)<br>`public void addBidder(IBidder bidder);`<br>`public void setState(AuctionState s);` **(Context** for State) |
| **`IBidder`** (Observer) | | `void onNewHighBid(Auction auction, Bid newHighestBid);` **(Observer Pattern)** |
| **`User`** (implements IBidder) | `private final String userId;` | `@Override public void onNewHighBid(...);` (A concrete bidder who gets notified.) |
| **`AuctionState`** (Interface) | | `boolean placeBid(Auction a, Bid bid);`<br>`void endAuction(Auction a);` **(State Pattern:** Controls what can happen during each auction phase.) |
| **`PendingState`, `ActiveState`, `ClosedState`** | | (Concrete states. Bids are only accepted in `ActiveState`. `PendingState` might transition to `ActiveState` at a scheduled time.) |
| **`Bid`** | `private final BigDecimal amount;`<br>`private final IBidder bidder;`<br>`private final Instant timestamp;` | (**Immutable DTO** representing a single bid action.) |
| **`AuctionStatus`** (Enum) | `PENDING, ACTIVE, CLOSED, CANCELLED` | (Managed by the State pattern.) |

---

## 19. Hotel Management System 🏨

The design is enhanced by separating concerns: a **`RoomInventory`** manages room status, and a **`BookingService`** handles the logic of reservations. This applies SRP more strictly and makes the inventory management explicitly thread-safe.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Hotel`** | `private final RoomInventory inventory;`<br>`private final BookingService bookingService;`<br>`private final SearchService searchService;` | `public List<Room> searchRooms(...);`<br>`public Booking makeBooking(...);` **(Facade:** A simplified entry point for hotel operations.) |
| **`RoomInventory`** | `private final Map<String, Room> rooms;`<br>`private final Map<RoomType, Deque<Room>> availableRooms;` | `public synchronized void checkIn(Booking b);`<br>`public synchronized void checkOut(Booking b);` **(All inventory operations must be thread-safe**.) |
| **`BookingService`** | `private final IPricingStrategy pricingStrategy;` | `public Booking createBooking(...);`<br>`public BigDecimal calculateCost(Booking b);` (Uses Strategy for pricing.) |
| **`Room`** (Abstract) | `protected final String roomNumber;`<br>`protected RoomStatus status;`<br>`protected RoomStyle style;` | `public boolean isAvailable();` |
| **`RoomFactory`** | | `public static Room createRoom(RoomType type, ...);` **(Factory Pattern:** Used during initial hotel setup to create different room subtypes like `StandardRoom`, `Suite`.) |
| **`Booking`** | `private final Guest guest;`<br>`private final Room room;`<br>`private final DateRange dates;` | (**Immutable DTO** for a confirmed booking.) |
| **`IPricingStrategy`** (Interface) | | `BigDecimal calculatePrice(BookingDetails details);` **(Strategy Pattern:** Allows for `SeasonalPricing`, `WeekdayPricing`, `LoyaltyPricing`.) |
| **`RoomStatus`** (Enum) | `AVAILABLE, OCCUPIED, UNDER_MAINTENANCE` | (Manages the state of a room.) |
| **`RoomType`** (Enum) | `STANDARD, DELUXE, SUITE` | (Used by the Factory.) |

---

## 20. Chess Game ♟️

This design correctly uses the Strategy Pattern for piece movement. It's refined by adding a **`Game`** class to manage state (like whose turn it is) and a **`Move`** class (Command/DTO) to encapsulate a player's complete action.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Game`** | `private final Board board;`<br>`private final Player[] players;`<br>`private Player currentPlayer;`<br>`private GameStatus status;` | `public boolean makeMove(Player p, Move m);` (Validates the move is for the current player and the move itself is valid.) |
| **`Board`** | `private final Square[][] squares;` | `public void setupBoard();`<br>`public Square getSquare(int x, int y);`<br>`public void executeMove(Move m);` |
| **`Piece`** (Abstract) | `protected final Color color;`<br>`private final IMoveStrategy moveStrategy;` | `public List<Move> getValidMoves(Board b, Square start);` (Delegates to its `moveStrategy`.) |
| **`King`, `Knight`, `Pawn`, etc.** | | (Inherits from `Piece`, initialized with its specific movement strategy.) |
| **`IMoveStrategy`** (Interface) | | `List<Move> getValidMoves(Board b, Square start);` **(Strategy Pattern:** Encapsulates the unique move algorithm for each piece type.) |
| **`KnightMoveStrategy`, `PawnMoveStrategy`** | | (Concrete strategies. `PawnMoveStrategy` would handle en passant and promotion logic.) |
| **`Move`** | `private final Player player;`<br>`private final Square start, end;`<br>`private final Piece pieceMoved;`<br>`private final Piece pieceKilled;` | **(Command/DTO:** Represents a single, validated move in the game.) |
| **`Color`** (Enum) | `WHITE, BLACK` | (Type-safe player colors.) |
| **`GameStatus`** (Enum) | `ACTIVE, CHECK, CHECKMATE, STALEMATE, RESIGNATION` | (Represents the current state of the game.) |

---

## 21. Tic-Tac-Toe Game ⭕❌

A simple but effective design. It is improved by abstracting the **`Player`** class, which allows for an easy implementation of both a `HumanPlayer` and an `AIPlayer` (e.g., using minimax) without changing the main game loop.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Game`** | `private final Board board;`<br>`private final Player[] players;`<br>`private int currentPlayerIndex;`<br>`private GameStatus status;` | `public void start();` (Manages the main game loop: get move -> validate -> apply -> check winner -> switch player.) |
| **`Board`** | `private final int size;`<br>`private final Mark[][] grid;` | `public boolean placeMark(int row, int col, Mark m);`<br>`public boolean isFull();`<br>`public GameStatus checkStatus();` (Encapsulates all board logic.) |
| **`Player`** (Abstract) | `protected final String name;`<br>`protected final Mark playingMark;` | `public abstract Move makeMove(Board board);` (Contract for any type of player.) |
| **`HumanPlayer`** | | `@Override public Move makeMove(Board board);` (Reads move from console/UI.) |
| **`AIPlayer`** | | `@Override public Move makeMove(Board board);` (Uses an algorithm like minimax to determine the best move.) |
| **`Move`** | `private final int row, col;` | (**Immutable DTO** to represent a player's desired move.) |
| **`Mark`** (Enum) | `X, O, EMPTY` | (Type-safe representation for what can be on a board square.) |
| **`GameStatus`** (Enum) | `IN_PROGRESS, DRAW, X_WINS, O_WINS` | (Represents the outcome of the game.) |

---

## 22. Snake and Ladder Game 🐍🪜

This straightforward design is improved by modeling **`Jumper`** as an abstract class, making `Snake` and `Ladder` more explicit types. The main `Game` class uses a **Queue** to naturally handle turn rotation for players.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Game`** | `private final Board board;`<br>`private final Queue<Player> players;`<br>`private final Dice dice;` | `public void startGame();` (Manages the turn-based game loop using a **Queue** for players, which naturally handles turn rotation.) |
| **`Board`** | `private final int size;`<br>`private final Map<Integer, Jumper> jumpers;` | `public int getNextPosition(int currentPos, int diceVal);` (Calculates move, then checks if the destination square has a `Jumper`.) |
| **`Jumper`** (Abstract) | `protected final int startPosition;`<br>`protected final int endPosition;` | `public int getEndPosition();` (Base class for snakes and ladders.) |
| **`Snake`** (extends Jumper) | | (A `Jumper` where `endPosition < startPosition`. A constructor can enforce this rule.) |
| **`Ladder`** (extends Jumper) | | (A `Jumper` where `endPosition > startPosition`. A constructor can enforce this rule.) |
| **`Player`** | `private final String name;`<br>`private int currentPosition;` | `public void setPosition(int position);`<br>`public String getName();` |
| **`Dice`** | `private final int numDice;`<br>`private final Random random;` | `public int roll();` (Encapsulates the logic for rolling one or more dice.) |

---

## 23. File Storage System (Google Drive) 📁

The Composite Pattern is perfect here. The design is enhanced by adding **`AccessControlList` (ACL)** to represent permissions, a critical feature of any real multi-user file system, and using a **`ConcurrentHashMap`** for children in a directory to ensure thread-safety.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`FileSystem`** | `private final Directory root;`<br>`private final UserManager userManager;` | `public void createFile(String path, ...);`<br>`public void createDirectory(String path, ...);` **(Facade** over the file system operations.) |
| **`IStorageItem`** (Interface) | | `String getName();`<br>`int getSize();`<br>`void delete();`<br>`Instant getCreationDate();` **(Composite Pattern: Component** - Defines the common interface.) |
| **`File`** (Leaf) | `private final String name;`<br>`private final byte[] content;`<br>`private final AccessControlList acl;` | `@Override public int getSize();` **(Leaf** in Composite Pattern.) |
| **`Directory`** (Composite) | `private final String name;`<br>`private final Map<String, IStorageItem> children;`<br>`private final AccessControlList acl;` | `@Override public int getSize();` (Recursively calculates size of all children.)<br>`public void addChild(IStorageItem item);` **(Uses `ConcurrentHashMap`** for children for thread safety.) |
| **`AccessControlList`** | `private final User owner;`<br>`private final Map<User, PermissionType> permissions;` | `public boolean hasPermission(User user, PermissionType perm);` (Manages read/write/execute permissions for users/groups.) |
| **`PermissionType`** (Enum) | `READ, WRITE, EXECUTE` | (Type-safe permissions.) |

---

## 24. Airline Reservation System ✈️

This design correctly separates `Flight` (the route) from `FlightInstance` (the specific occurrence). It's improved by adding an **`Aircraft`** class to model the physical plane and its seat layout, which is a more realistic representation of how airlines manage fleets.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`AirlineService`** | `private final FlightService flightService;`<br>`private final BookingService bookingService;` | `public List<FlightInstance> searchFlights(...);`<br>`public Booking makeBooking(...);` **(Facade** for all airline operations.) |
| **`Flight`** | `private final String flightNumber;`<br>`private final Airport departure, arrival;` | (**Immutable DTO** representing a route, e.g., "UA245 SFO-JFK".) |
| **`FlightInstance`** | `private final Flight flight;`<br>`private final LocalDate date;`<br>`private final Aircraft aircraft;`<br>`private final Map<String, SeatStatus> seatMap;` | `public List<Seat> getAvailableSeats();`<br>`public synchronized boolean reserveSeat(String seatNumber);` (Represents a specific, bookable flight.) |
| **`Aircraft`** | `private final String model;`<br>`private final String tailNumber;`<br>`private final List<Seat> seats;` | (**Immutable**. Represents the physical plane with its fixed seat layout.) |
| **`Seat`** | `private final String seatNumber;`<br>`private final SeatClass seatClass;` | (**Immutable DTO** representing a seat's physical properties.) |
| **`Booking`** | `private final FlightInstance flightInstance;`<br>`private final List<Passenger> passengers;`<br>`private final List<Seat> seatsBooked;` | `public boolean confirm(IPaymentStrategy p);` |
| **`IPaymentStrategy`** (Interface) | | `boolean pay(BigDecimal amount);` **(Strategy Pattern)** |
| **`SeatClass`** (Enum) | `ECONOMY, ECONOMY_PLUS, BUSINESS, FIRST` | (Type-safe seating classes.) |
| **`SeatStatus`** (Enum) | `AVAILABLE, RESERVED, BOOKED` | (Status of a seat on a specific `FlightInstance`.) |

---

## 25. Shopping Cart System 🛍️

This is a focused design. The crucial point, which is now emphasized, is that **`OrderItem`** must be immutable and store the **`priceAtPurchase`** to protect against price changes after an item is added to the cart but before checkout is complete.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`ShoppingCart`** | `private final String userId;`<br>`private final Map<String, CartItem> items;` | `public synchronized void addItem(Product p, int qty);`<br>`public synchronized void updateItemQuantity(...);`<br>`public synchronized void clearCart();` (All methods are synchronized. Often managed per user session.) |
| **`CartItem`** | `private final Product product;`<br>`private int quantity;` | `public void incrementQuantity(int amount);`<br>`public BigDecimal getSubtotal();` |
| **`Product`** | `private final String productId;`<br>`private final String name;`<br>`private final BigDecimal currentPrice;` | (**Immutable DTO** for product details.) |
| **`CheckoutService`** | `private final InventoryService inventory;`<br>`private final PaymentService paymentService;`<br>`private final OrderRepository orderRepo;` | `public Order checkout(ShoppingCart cart, IPaymentStrategy strategy);` **(Facade:** Coordinates inventory, payment, and order creation.) |
| **`Order`** | `private final String orderId;`<br>`private final List<OrderItem> orderItems;`<br>`private final BigDecimal total;` | (**Immutable** object created from a `ShoppingCart` at checkout.) |
| **`OrderItem`** | `private final String productId;`<br>`private final int quantity;`<br>`private final BigDecimal priceAtPurchase;` | (**Immutable DTO**. Crucially stores the **price at the time of purchase**, which might differ from the product's current price.) |

---

## 26. Online Stock Brokerage System 📈

Concurrency is paramount. This enhanced design makes the `OrderBook` logic more explicit, using **`ReentrantLock`** for the critical matching operation and **`PriorityQueue`** to correctly model the bid (max-heap) and ask (min-heap) matching process.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`BrokerageService`** | `private final AccountService accountService;`<br>`private final Exchange exchange;` | `public boolean placeOrder(String accountId, Order order);` **(Facade** for user interactions.) |
| **`Account`** | `private BigDecimal cashBalance;`<br>`private final Portfolio portfolio;` | `public synchronized boolean placeBuyOrder(Order order);`<br>`public synchronized boolean placeSellOrder(Order order);` (Checks for funds/shares before sending to exchange.) |
| **`Order`** | `private final OrderType type;`<br>`private final String stockTicker;`<br>`private final int quantity;`<br>`private final IOrderExecutionStrategy execStrategy;` | `public boolean execute(Exchange exchange);` (Delegates to its strategy.) |
| **`IOrderExecutionStrategy`** (Interface) | | `void execute(Order o, OrderBook book);` **(Strategy Pattern:** Defines how to fill an order.) |
| **`MarketOrderStrategy`, `LimitOrderStrategy`** | | (Concrete strategies. `LimitOrderStrategy` has a `limitPrice` attribute.) |
| **`Exchange`** (Singleton) | `private static final Exchange instance;`<br>`private final Map<String, OrderBook> orderBooks;` | `public static synchronized getInstance();`<br>`public void submitOrder(Order order);` (Places order in the correct `OrderBook`.) |
| **`OrderBook`** | `private final PriorityQueue<Order> buyOrders; // Max-heap`<br>`private final PriorityQueue<Order> sellOrders; // Min-heap`<br>`private final ReentrantLock lock = new ReentrantLock();` | `public void addOrder(Order order);` (Adds order and triggers `matchOrders()`.)<br>`private void matchOrders();` **(The core, thread-safe matching engine**, protected by the lock.) |
| **`OrderType`** (Enum) | `BUY, SELL` | (Type-safe order types.) |

---

## 27. Digital Wallet Service (PayPal) 💳

This design is improved by making the **`Wallet`** the central Subject in the Observer pattern and defining a clear **`Transaction`** class that captures the full context of a financial operation as an immutable DTO.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|:---|:---|:---|
| **`Wallet`** (Subject) | `private BigDecimal balance;`<br>`private final List<IFinancialInstrument> instruments;`<br>`private final transient List<IWalletListener> listeners;` | `public synchronized Transaction sendMoney(...);`<br>`public synchronized Transaction addFunds(...);`<br>`private void notifyListeners(Transaction t);` (All balance-changing methods must be **synchronized**.) |
| **`IFinancialInstrument`** (Interface) | | `TransactionResult charge(BigDecimal amount);` **(Strategy-like interface** for funding sources like `BankAccount`, `CreditCard`.) |
| **`BankAccount`, `CreditCard`** | | (Implementations for different ways to add/withdraw money.) |
| **`Transaction`** | `private final String txnId;`<br>`private final Wallet fromWallet, toWallet;`<br>`private final BigDecimal amount;`<br>`private final TransactionType type;`<br>`private TransactionStatus status;` | (**Immutable DTO** representing a complete financial transaction.) |
| **`IWalletListener`** (Observer) | | `void onTransactionComplete(Transaction transaction);`<br>`void onTransactionFailed(Transaction transaction);` **(Observer Pattern)** |
| **`TransactionLogger`, `UiNotifier`** | | (Concrete observers that react to wallet events. `TransactionLogger` would write to a database for audit.) |
| **`TransactionStatus`** (Enum) | `PENDING, COMPLETED, FAILED, REVERSED` | (Manages the state of a transaction.) |
| **`TransactionType`** (Enum) | `SEND, RECEIVE, ADD_FUNDS, WITHDRAWAL` | (Categorizes transaction types.) |

---


## 28. Restaurant Management System 🍽️

The design is enhanced by creating a Waiter actor who acts as a Facade for the customers at a Table. This simplifies the interaction with the ordering and payment systems and more accurately models the real world.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **Restaurant** | `private final Menu menu;`<br>`private final List<Table> tables;`<br>`private final KitchenDisplaySystem kds;` | (Main class to bootstrap the system.) |
| **Waiter** | `private final List<Table> assignedTables;`<br>`private final OrderService orderService;` | `public void createOrder(Table t);`<br>`public void addToOrder(Order o, MenuItem item, int qty);`<br>`public void sendOrderToKitchen(Order o);`<br>`public Bill checkout(Order o);` (Facade for a table's dining experience.) |
| **Order** | `private final Table table;`<br>`private final Map<MenuItem, Integer> items;`<br>`private volatile OrderStatus status;` | `public void addItem(...);`<br>`public void setStatus(OrderStatus newStatus);` (Changing status might notify KDS.) |
| **KitchenDisplaySystem** | `private static final KDS instance;`<br>`private final Queue<Order> pendingOrders;` | `public static synchronized getInstance();` (Singleton/Observer)<br>`public void onNewOrderReceived(Order order);`<br>`public void onOrderReady(Order order);` |
| **Table** | `private final int tableId;`<br>`private TableStatus status;`<br>`private Order currentOrder;` | (Represents the physical table.) |
| **Menu** | `private final List<MenuItem> items;` | `public MenuItem findItem(String itemId);` |
| **TableStatus** (Enum) | AVAILABLE, OCCUPIED, RESERVED, DIRTY | (Manages the state of a dining table.) |
| **OrderStatus** (Enum) | PENDING, CONFIRMED, PREPARING, READY, SERVED, PAID | (Manages the lifecycle of a customer's order.) |

## 29. Course Registration System 🎓

Concurrency is critical here. The design is enhanced by making the key registration methods on both Student and CourseOffering synchronized to prevent race conditions like over-enrollment or a student registering for conflicting courses.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **RegistrationSystem** | `private final CourseCatalog catalog;` | `public boolean processRegistration(Student s, CourseOffering o);` (Facade that coordinates between student and offering, checking prerequisites and capacity.) |
| **Student** | `private final Map<String, CourseOffering> registeredCourses;`<br>`private final Transcript transcript;` | `public synchronized boolean registerForCourse(CourseOffering o);`<br>`public synchronized boolean dropCourse(CourseOffering o);` (Synchronized to prevent a student from multiple conflicting operations.) |
| **Course** | `private final String courseCode;`<br>`private final String title;`<br>`private final List<Course> prerequisites;` | (Immutable DTO representing the abstract catalog entry, e.g., "CS101".) |
| **CourseOffering** | `private final Course course;`<br>`private final int capacity;`<br>`private final List<Student> enrolledStudents;` | `public synchronized boolean addStudent(Student s);` (Checks capacity.)<br>`public synchronized void removeStudent(Student s);` (Represents a specific section of a course.) |
| **CourseCatalog** | `private static final CourseCatalog instance;`<br>`private final Map<String, Course> allCourses;`<br>`private final List<CourseOffering> currentOfferings;` | `public static synchronized getInstance();` (Singleton)<br>`public List<CourseOffering> findOfferings(String courseCode);` |
| **Transcript** | `private final Map<Course, Grade> completedCourses;` | `public boolean hasPrerequisites(List<Course> prereqs);` |

## 30. Online Bookstore 📖

This design is refined by introducing a dedicated SearchService using the Strategy Pattern (ISearchStrategy). This decouples the search logic from the main Bookstore facade and allows for easy addition of new search methods (e.g., by genre, publisher).

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **Bookstore** | `private final Inventory inventory;`<br>`private final SearchService searchService;`<br>`private final CheckoutService checkoutService;` | `public List<Book> search(ISearchStrategy strategy, String query);`<br>`public Order checkout(ShoppingCart cart, ...);` (Facade that simplifies the primary user workflows.) |
| **Inventory** | `private final Map<String, AtomicInteger> stockByIsbn;`<br>`private final Map<String, Book> booksByIsbn;` | `public synchronized void addBook(Book b, int qty);`<br>`public int getStockLevel(String isbn);` (Uses AtomicInteger for thread-safe stock management.) |
| **SearchService** | `private final Inventory inventory;` | `public List<Book> search(ISearchStrategy strategy, String query);` |
| **ISearchStrategy** (Interface) | | `List<Book> search(String query, Inventory inventory);` (Strategy Pattern: For searching by Title, Author, ISBN, Genre, etc.) |
| **Book** | `private final String ISBN;`<br>`private final String title;`<br>`private final List<Author> authors;`<br>`private final BigDecimal price;` | (Immutable DTO for book details.) |
| **ShoppingCart** | `private final Map<String, Integer> itemsByIsbn;` | `public void addItem(Book book, int quantity);` |
| **Order** | `private final List<OrderItem> items;`<br>`private final BigDecimal total;` | (Immutable record of a completed purchase.) |
| **OrderItem** | `private final Book book;`<br>`private final BigDecimal priceAtPurchase;` | (Immutable. It's crucial this stores the price at time of purchase.) |

## 31. Social Media Platform (Generic) 🌐

The design is enhanced with a NewsFeedService that uses the Strategy Pattern for feed generation. This is a core feature and allows for pluggable ranking algorithms (e.g., chronological vs. relevance-based).

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **User** | `private final Set<User> followers;`<br>`private final Set<User> following;` | `public void follow(User user);`<br>`public Post createPost(PostType type, ...);` (Uses the PostFactory.) |
| **Post** (Abstract) | `protected final User author;`<br>`protected int likeCount;`<br>`protected final List<Comment> comments;` | `public abstract void display();`<br>`public synchronized void addLike();` |
| **TextPost, ImagePost, VideoPost** | | `@Override public void display();` (Concrete post types with different content.) |
| **PostFactory** | | `public static Post createPost(User u, PostType type, ...);` (Factory Pattern: Decouples the client from the creation logic of different post types.) |
| **NewsFeedService** | `private final IFeedGenerationStrategy feedStrategy;` | `public List<Post> getFeed(User user);` (Delegates generation to the chosen strategy.) |
| **IFeedGenerationStrategy** (Interface) | | `List<Post> generateFeed(User user, Set<User> followedUsers);` (Strategy Pattern: Allows for different feed ranking algorithms, e.g., ChronologicalFeed, RelevanceFeed.) |
| **Comment** | `private final User author;`<br>`private final String text;` | (Immutable DTO for a comment.) |
| **PostType** (Enum) | TEXT, IMAGE, VIDEO | (Used by the Factory to create the correct post type.) |

## 32. Facebook (Advanced Social Media) 👍

This builds on the generic platform by adding more complex entities. The FriendRequest is modeled as a stateful object, which is a better representation of the real-world process than a simple method call.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **Member** (extends User) | `private final Map<String, Member> friends;` | `public void sendFriendRequest(Member recipient);`<br>`public void processFriendRequest(FriendRequest req, boolean accepted);` |
| **FriendRequest** | `private final Member fromUser, toUser;`<br>`private RequestStatus status;`<br>`private final Instant timestamp;` | `public void accept();`<br>`public void decline();` (A stateful object representing a pending friend request.) |
| **Post** | `private final Map<ReactionType, List<Member>> reactions;` | `public synchronized void addReaction(Member m, ReactionType type);` (More advanced than a simple like count.) |
| **Group** | `private final String name;`<br>`private final Map<String, Member> members;`<br>`private final List<Post> posts;` | `public void addMember(Member member);`<br>`public void createPost(...);` (A container for posts and members.) |
| **Page** | `private final String name;`<br>`private final Set<Member> followers;` | (Represents an entity like a business that members can follow.) |
| **Event** | `private final String title;`<br>`private final Map<Member, EventRSVP> attendees;` | `public void rsvp(Member member, EventRSVP status);` (Manages events and attendance.) |
| **ReactionType** (Enum) | LIKE, LOVE, HAHA, WOW, SAD, ANGRY | (Richer set of reactions.) |
| **RequestStatus** (Enum) | PENDING, ACCEPTED, DECLINED, IGNORED | (Manages friend request lifecycle.) |
| **EventRSVP** (Enum) | ATTENDING, MAYBE, NOT_ATTENDING | (Manages event attendance status.) |

## 33. Cricinfo (Live Sports Commentary) 🏏

This is a classic real-time system driven by the Observer pattern. The design is refined by making Ball an immutable DTO that contains all information about a single delivery, ensuring that once an event has happened, its record cannot be altered.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **Match** (Subject) | `private final List<Innings> innings;`<br>`private MatchStatus status;`<br>`private final transient List<ILiveScoreViewer> viewers;` | `public void addBall(Innings i, Ball b);` (Notifies all viewers of the new ball.)<br>`public void addViewer(ILiveScoreViewer v);` (Subject's registration method.) |
| **ILiveScoreViewer** (Observer) | | `void updateScore(Match match, Ball lastBall);` (Observer Pattern: Defines how viewers receive updates.) |
| **MobileAppViewer, WebsiteViewer** | | `@Override public void updateScore(...);` (Concrete observers can render the update differently.) |
| **Innings** | `private final Team battingTeam, bowlingTeam;`<br>`private int totalRuns;`<br>`private int wicketsFallen;`<br>`private final List<Over> overs;` | (Represents one team's turn to bat.) |
| **Over** | `private final Player bowler;`<br>`private final List<Ball> balls;` | (Composed of up to 6 valid Ball events.) |
| **Ball** | `private final int runsScored;`<br>`private final Wicket wicket; // Can be null`<br>`private final BallType type;`<br>`private final String commentary;` | (Immutable DTO. Represents the smallest unit of action. Once a ball is bowled, its outcome is final.) |
| **Wicket** | `private final Player batsmanOut;`<br>`private final WicketType type;` | (Immutable DTO representing the dismissal of a batsman.) |
| **WicketType** (Enum) | BOWLED, CAUGHT, LBW, RUN_OUT, STUMPED | (Type-safe representation of dismissal types.) |
| **BallType** (Enum) | NORMAL, WIDE, NO_BALL | (Type-safe ball types.) |

## 34. API Rate Limiter ⏱️

This design uses the Strategy Pattern for different rate-limiting algorithms. The TokenBucketStrategy is enhanced to be explicitly thread-safe using synchronized on the core logic, as multiple requests for the same user can arrive concurrently. A Factory creates limiters based on user subscription tiers.

| Class/Enum/Interface | Key Attributes (Variables) | Key Methods (Functions) & Concepts |
|---------------------|---------------------------|-----------------------------------|
| **RateLimiterService** | `private final IRateLimitingStrategy defaultStrategy;`<br>`private final Map<String, IRateLimitingStrategy> userStrategies;` | `public boolean isRequestAllowed(String userId);` (Facade that checks if a request from a user should be accepted or rejected.) |
| **IRateLimitingStrategy** (Interface) | | `boolean isAllowed();` (Strategy Pattern: Allows for different algorithms like TokenBucket, LeakyBucket, FixedWindowCounter.) |
| **TokenBucketStrategy** | `private final int bucketCapacity;`<br>`private final int refillRate; // tokens per second`<br>`private final AtomicInteger currentTokens;`<br>`private volatile long lastRefillTimestamp;` | `public synchronized boolean isAllowed();` (Refills tokens based on time elapsed, then attempts to consume a token. This must be synchronized.) |
| **TokenBucketFactory** | | `public IRateLimitingStrategy createForUser(User user);` (Factory Pattern: Creates a specific rate limiter instance for a user based on their subscription tier.) |
| **User** | `private final String userId;`<br>`private final SubscriptionTier tier;` | (User object containing information that might affect their rate limit.) |
| **SubscriptionTier** (Enum) | FREE(10), BASIC(100), PREMIUM(1000);<br>`private final int requestsPerMinute;` | (Defines different service levels, used by the factory to configure the rate limiter.) |
