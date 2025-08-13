Of course. Here are the low-level object-oriented designs for the requested systems, presented in a crisp, interview-ready format. Each design focuses on class structure, relationships, and optimal data structures without function implementations.

-----

## 🅿️ 1. Parking Lot System

This design supports multiple vehicle types, floors, and payment methods. It uses the **Singleton** pattern for the `ParkingLot` to ensure a single entry/exit point and the **Strategy** pattern for payments.

### Core Classes & Relationships

**Relationships:**

  * `ParkingLot` has a **Composition** relationship with `ParkingFloor` and `Ticket`. A floor cannot exist without a lot.
  * `ParkingFloor` has a **Composition** relationship with `ParkingSpot`. A spot is part of a specific floor.
  * `Vehicle` and `ParkingSpot` have an **Association**. A spot can be temporarily associated with a vehicle.
  * `Car`, `Motorcycle`, and `Truck` show **Inheritance** from the abstract class `Vehicle`.
  * `Ticket` has an **Aggregation** relationship with `Vehicle` and `ParkingSpot`.

-----

### Class Definitions

```java
// Using Singleton Pattern to ensure a single instance of the parking lot.
public final class ParkingLot {
    // Attributes
    private String name;
    private String address;
    private List<ParkingFloor> floors; // Composition: Floors are part of the lot
    private static final ParkingLot instance = new ParkingLot(); // Eager initialization of Singleton

    // Constructor
    private ParkingLot() {} // Private constructor for Singleton

    // Methods
    public static ParkingLot getInstance(); // Provides global access to the single instance
    public Ticket assignTicket(Vehicle vehicle); // Assigns a spot and generates a ticket
    public double scanAndPay(Ticket ticket); // Processes payment upon exit
    public void addFloor(ParkingFloor floor); // Adds a new floor to the lot
    public void removeFloor(ParkingFloor floor); // Removes a floor from the lot
}

// Represents a floor in the parking lot.
public class ParkingFloor {
    // Attributes
    private int floorNumber;
    private Map<String, ParkingSpot> spots; // Composition: Key is spotId for O(1) lookup
    private Map<ParkingSpotType, Integer> availableSpots; // Tracks count of available spots by type

    // Constructor
    public ParkingFloor(int floorNumber);

    // Methods
    public ParkingSpot findSpot(Vehicle vehicle); // Finds an available spot for a given vehicle type
    public void parkVehicle(ParkingSpot spot, Vehicle vehicle); // Occupies a spot
    public void freeSpot(ParkingSpot spot); // Releases a spot
    public void displayBoard(); // Displays current availability on the floor
}

// Abstract base class for all vehicles. Demonstrates Abstraction.
public abstract class Vehicle {
    // Attributes
    private String licensePlate;
    private final VehicleType type; // Using enum for type safety

    // Constructor
    public Vehicle(String licensePlate, VehicleType type);

    // Getters
    public String getLicensePlate();
    public VehicleType getType();
}

// Concrete vehicle classes. Demonstrates Inheritance.
public class Car extends Vehicle {
    public Car(String licensePlate) { super(licensePlate, VehicleType.CAR); }
}
public class Motorcycle extends Vehicle {
    public Motorcycle(String licensePlate) { super(licensePlate, VehicleType.MOTORCYCLE); }
}

// Represents a single parking spot.
public class ParkingSpot {
    // Attributes
    private String spotId;
    private boolean isOccupied;
    private final ParkingSpotType type;
    private Vehicle parkedVehicle; // Association with Vehicle

    // Constructor
    public ParkingSpot(String spotId, ParkingSpotType type);

    // Methods
    public void assignVehicle(Vehicle vehicle); // Marks spot as occupied
    public void removeVehicle(); // Marks spot as free
}

// Represents the ticket issued to a vehicle.
public class Ticket {
    // Attributes
    private String ticketId;

    private long entryTime;
    private long exitTime;
    private double amount;
    private TicketStatus status;
    private Vehicle vehicle; // Aggregation
    private ParkingSpot spot; // Aggregation

    // Constructor
    public Ticket(Vehicle vehicle, ParkingSpot spot);

    // Methods
    public void processPayment(PaymentStrategy paymentStrategy); // Polymorphism via Strategy Pattern
}

// Using Strategy Pattern for payment flexibility. Demonstrates Polymorphism.
public interface PaymentStrategy {
    void pay(double amount); // Abstract method
}
public class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount); // Overridden method
}
public class CashPayment implements PaymentStrategy {
    public void pay(double amount); // Overridden method
}

// Enums for type safety and clarity.
public enum VehicleType { CAR, TRUCK, MOTORCYCLE }
public enum ParkingSpotType { COMPACT, LARGE, MOTORCYCLE }
public enum TicketStatus { ACTIVE, PAID, LOST }
```

-----

## ⬆️ 2. Elevator System

This design manages multiple elevators in a building using a central controller. It uses a **Singleton** `ElevatorController` and a **Command** pattern (implicitly through requests) to direct elevators. Each elevator runs on its own thread to operate independently.

### Core Classes & Relationships

**Relationships:**

  * `ElevatorSystem` has a **Composition** relationship with `ElevatorCar` and `BuildingFloor`. The elevators and floors are integral parts of the system.
  * `ElevatorController` acts as a mediator and is a **Singleton**.
  * `ElevatorCar` has an **Association** with `Request`. It processes requests.
  * `InternalButton` and `ExternalButton` show **Inheritance** from the abstract class `Button`.

-----

### Class Definitions

```java
// Singleton controller to manage all elevator cars.
public class ElevatorController {
    // Attributes
    private List<ElevatorCar> elevators; // List of all elevators in the system
    private Queue<Request> requestQueue; // A queue to hold pending requests
    private static final ElevatorController instance = new ElevatorController(); // Singleton instance

    // Constructor
    private ElevatorController() {} // Private constructor

    // Methods
    public static ElevatorController getInstance(); // Global access point
    public void addRequest(Request request); // Adds a new user request to the queue
    public void assignRequestToElevator(); // Logic to dispatch the best elevator for a request
}

// Represents the physical elevator car. Would typically run on its own thread.
public class ElevatorCar implements Runnable {
    // Attributes
    private int id;
    private int currentFloor;
    private Direction direction;
    private ElevatorState state;
    // Optimal data structure: PriorityQueue to handle requests in order based on direction.
    private PriorityQueue<Request> upRequests;
    private PriorityQueue<Request> downRequests;
    private Door door;

    // Constructor
    public ElevatorCar(int id);

    // Methods
    public void run(); // The core logic loop for the elevator thread
    private void move(); // Moves the elevator up or down
    private void openDoor();
    private void closeDoor();
    public void addNewRequest(Request request); // Adds a request to its internal queue
}

// Represents a request made by a user.
public class Request {
    // Attributes
    private int originFloor;
    private int destinationFloor;
    private RequestType type; // From inside or outside the elevator

    // Constructor
    public Request(int originFloor, int destinationFloor, RequestType type);
}

// Represents a floor in the building.
public class BuildingFloor {
    // Attributes
    private int floorNumber;
    private ExternalButton upButton;
    private ExternalButton downButton;

    // Constructor
    public BuildingFloor(int floorNumber);

    // Methods
    public void pressUpButton();
    public void pressDownButton();
}

// Represents the elevator door.
public class Door {
    private DoorState state;
    public void open();
    public void close();
}

// Abstract class for buttons. Demonstrates Abstraction.
public abstract class Button {
    public abstract void press();
}

// Buttons inside the elevator.
public class InternalButton extends Button {
    private int destinationFloorNumber;
    public void press(); // Sends a request to the controller
}

// Buttons on each floor.
public class ExternalButton extends Button {
    private Direction direction;
    public void press(); // Sends a request to the controller
}


// Enums for state management.
public enum Direction { UP, DOWN, IDLE }
public enum ElevatorState { MOVING, IDLE, OUT_OF_SERVICE }
public enum DoorState { OPEN, CLOSED }
public enum RequestType { INTERNAL, EXTERNAL }
```

-----

## 🧠 3. LRU Cache

An LRU (Least Recently Used) Cache evicts the least recently used item when it reaches its capacity. The most optimal implementation uses a **`HashMap`** for O(1) lookups and a **`DoublyLinkedList`** for O(1) insertion/deletion to track recency. This design is generic to support any key-value types.

### Core Classes & Relationships

**Relationships:**

  * `LRUCache` has a **Composition** relationship with `Node`. The nodes are the internal data structure managed entirely by the cache.
  * `LRUCache` uses a `HashMap` and a `DoublyLinkedList` (implemented via `Node` pointers) to achieve its functionality.

-----

### Class Definitions

```java
// The main LRU Cache class, using generics for type flexibility.
public class LRUCache<K, V> {
    // Attributes
    private final int capacity; // The maximum size of the cache
    private Map<K, Node<K, V>> cacheMap; // HashMap for O(1) get operations
    private Node<K, V> head; // Head of the doubly linked list (most recently used)
    private Node<K, V> tail; // Tail of the doubly linked list (least recently used)

    // Constructor
    public LRUCache(int capacity);

    // Core Methods
    public V get(K key); // Retrieves an item and marks it as most recently used
    public void put(K key, V value); // Adds/updates an item, handles eviction if capacity is reached

    // Helper Methods (private for encapsulation)
    private void removeNode(Node<K, V> node); // Removes a node from the linked list
    private void addToFront(Node<K, V> node); // Adds a node to the head of the linked list
}

// Inner static nested class representing a node in the doubly linked list.
// It's static because it doesn't need to access instance members of LRUCache.
private static class Node<K, V> {
    // Attributes
    K key;
    V value;
    Node<K, V> prev; // Pointer to the previous node
    Node<K, V> next; // Pointer to the next node

    // Constructor
    public Node(K key, V value);
}
```

-----

## 🎬 4. Movie Ticket Booking System

This system handles movie listings, showtimes, seat selection, and booking. It is designed to be scalable for multiple cities and cinemas. It utilizes the **Strategy** pattern for payments.

### Core Classes & Relationships

**Relationships:**

  * `BookingSystem` is a **Singleton** facade for the client.
  * A `City` has a **Composition** of `Cinema`s.
  * A `Cinema` has a **Composition** of `CinemaHall`s (screens).
  * A `CinemaHall` has a **Composition** of `Seat`s.
  * `Show` has an **Aggregation** relationship with `Movie` and `CinemaHall`. It connects a movie to a specific screen at a specific time.
  * `Booking` has an **Association** with `User`, `Show`, and a list of `Seat`s.

-----

### Class Definitions

```java
// Main controller class, could be a Singleton.
public class BookingSystem {
    // Attributes
    private List<City> cities;
    private List<Movie> movies;

    // Methods
    public List<Movie> getMoviesByCity(String cityName);
    public List<Cinema> getCinemasByCity(String cityName);
    public List<Show> getShows(Movie movie, Cinema cinema, Date date);
    public boolean bookTickets(User user, Show show, List<Seat> seats);
}

// Represents a movie.
public class Movie {
    // Attributes
    private String movieId;
    private String title;
    private int durationInMinutes;
    private String language;
    private Genre genre;
}

// Represents a cinema complex.
public class Cinema {
    // Attributes
    private String cinemaId;
    private String name;
    private Address address;
    private List<CinemaHall> halls; // Composition

    // Methods
    public List<Show> getShows();
}

// Represents a single screen/hall within a cinema.
public class CinemaHall {
    // Attributes
    private String hallId;
    private String name;
    private List<Seat> seats; // Composition

    // Methods
    public List<Show> getShows();
}

// Represents a specific show of a movie.
public class Show {
    // Attributes
    private String showId;
    private Movie movie; // Aggregation
    private CinemaHall hall; // Aggregation
    private Date startTime;
    private int durationInMinutes;
    // Using a map for quick checking of a seat's availability for this specific show.
    private Map<String, SeatStatus> seatStatus; // Key is seatId

    // Methods
    public boolean isSeatAvailable(Seat seat);
    public void lockSeats(List<Seat> seats); // Temporarily block seats during payment
    public void bookSeats(List<Seat> seats); // Confirm booking for seats
}

// Represents a single seat.
public class Seat {
    // Attributes
    private String seatId; // e.g., "A12"
    private int row;
    private int col;
    private SeatType type;
    private double price;
}

// Represents a confirmed booking.
public class Booking {
    // Attributes
    private String bookingId;
    private User user; // Association
    private Show show; // Association
    private List<Seat> bookedSeats; // Association
    private double totalAmount;
    private Payment payment;
}

// Represents a user of the system.
public class User {
    // Attributes
    private String userId;
    private String name;
    private String password; // Hashed password
}

// Enums for type safety.
public enum SeatType { REGULAR, PREMIUM, RECLINER }
public enum SeatStatus { AVAILABLE, BOOKED, LOCKED }
public enum Genre { ACTION, COMEDY, DRAMA }
```

-----

## 🛒 5. E-commerce Website (e.g., Amazon)

This is a high-level design for a core e-commerce flow: searching products, managing a shopping cart, and placing an order. It employs several patterns like **Strategy** (for payment), **Observer** (for notifications), and **Factory** (for creating different user types or products).

### Core Classes & Relationships

**Relationships:**

  * `User` has an **Inheritance** hierarchy (`Customer`, `Admin`).
  * `Customer` has a **Composition** relationship with `ShoppingCart` and `Address`. A cart belongs to one customer.
  * `Order` has a **Composition** relationship with `OrderItem`. Order items cannot exist without an order.
  * `Product` and `ProductCategory` have a many-to-many **Association**, often managed through a join table in a database.
  * The system uses the **Observer** pattern where `Order` is the subject and `NotificationService` (e.g., EmailService, SMSService) are the observers.
  * It uses the **Strategy** pattern for `PaymentMethod`.

-----

### Class Definitions

```java
// Abstract base class for users.
public abstract class User {
    private String userId;
    private String username;
    private String passwordHash;
    private AccountStatus status;
}

// A customer is a type of user. Demonstrates Inheritance.
public class Customer extends User {
    // Attributes
    private ShoppingCart cart; // Composition
    private List<Address> addresses; // Composition
    private List<Order> orderHistory;

    // Methods
    public void addItemToCart(Product product, int quantity);
    public void removeItemFromCart(Product product);
    public Order checkout();
}

// An admin is a type of user with different permissions.
public class Admin extends User {
    // Methods
    public void addProduct(Product product);
    public void blockUser(Customer customer);
}

// Represents a product for sale.
public class Product {
    // Attributes
    private String productId;
    private String name;
    private String description;
    private double price;
    private ProductCategory category; // Association
    private int stockQuantity;
    private List<Review> reviews;
}

// Represents a customer's shopping cart.
public class ShoppingCart {
    // Attributes
    private List<CartItem> items;

    // Methods
    public void addItem(CartItem item);
    public void removeItem(CartItem item);
    public double calculateTotal();
}

// An item within the shopping cart.
public class CartItem {
    private Product product;
    private int quantity;
}

// Represents a completed order. This is the "Subject" in the Observer pattern.
public class Order {
    // Attributes
    private String orderId;
    private List<OrderItem> orderItems; // Composition
    private Customer customer;
    private Address shippingAddress;
    private double orderTotal;
    private OrderStatus status;
    private List<NotificationObserver> observers = new ArrayList<>(); // Observer pattern

    // Methods
    public void setStatus(OrderStatus newStatus); // Notifies observers on change
    public void registerObserver(NotificationObserver observer);
    public void unregisterObserver(NotificationObserver observer);
    public void notifyObservers();
}

// Item within a confirmed order.
public class OrderItem {
    private Product product;
    private int quantity;
    private double priceAtPurchase;
}

// Observer pattern interface for notifications.
public interface NotificationObserver {
    void update(Order order);
}

// Concrete Observers.
public class EmailService implements NotificationObserver {
    public void update(Order order); // Sends an email notification
}
public class SMSService implements NotificationObserver {
    public void update(Order order); // Sends an SMS notification
}

// Strategy pattern for different payment methods.
public interface PaymentMethod {
    boolean processPayment(double amount);
}
public class CreditCard implements PaymentMethod {
    public boolean processPayment(double amount);
}
public class PayPal implements PaymentMethod {
    public boolean processPayment(double amount);
}

// Enums for type safety.
public enum AccountStatus { ACTIVE, BLOCKED, PENDING_VERIFICATION }
public enum OrderStatus { PENDING, SHIPPED, DELIVERED, CANCELED }
```



Of course. Here are the low-level object-oriented designs for the next set of systems, following the same interview-ready format.

-----

## 🚗 6. Ride-Sharing Service (e.g., Uber)

This design focuses on the core functionality of matching riders with drivers, managing trips, and calculating fares. It uses the **Strategy** pattern for fare calculation and the **Singleton** pattern for the dispatch system. For location-based searches, it assumes a spatial indexing mechanism for efficiency.

### Core Classes & Relationships

**Relationships:**

  * `User` is an **abstract** base class for `Rider` and `Driver`, demonstrating **Inheritance**.
  * `Driver` has a **Composition** relationship with `Vehicle`. A specific vehicle is tied to one driver at a time.
  * `Trip` forms an **Association** between a `Rider` and a `Driver`.
  * `RideDispatchSystem` is a **Singleton** that acts as the central brain, mediating between riders and drivers.
  * `FareCalculationStrategy` is a **Strategy** interface, allowing for different pricing models (e.g., surge, standard).

-----

### Class Definitions

```java
// Central Singleton class to manage the entire ride-hailing process.
public class RideDispatchSystem {
    // Attributes
    private static final RideDispatchSystem instance = new RideDispatchSystem();
    // Using a simple list for LLD; in reality, a Quadtree or spatial index is optimal
    // for finding nearby drivers efficiently (O(log n)).
    private List<Driver> availableDrivers;
    private List<RideRequest> activeRequests;
    private TripManager tripManager;

    // Constructor
    private RideDispatchSystem() {} // Private constructor for Singleton

    // Methods
    public static RideDispatchSystem getInstance(); // Global access point
    public RideRequest createRequest(Rider rider, Location pickup, Location dropoff);
    public Trip matchDriver(RideRequest request); // Core logic to find the best driver
}

// Abstract base class for system users.
public abstract class User {
    private String userId;
    private String name;
    private double rating;
}

// A Rider is a type of User.
public class Rider extends User {
    private PaymentStrategy preferredPaymentMethod;
    public void requestRide(Location pickup, Location dropoff);
    public void cancelRide(Trip trip);
    public void rateDriver(Trip trip, double rating);
}

// A Driver is a type of User.
public class Driver extends User {
    // Attributes
    private Vehicle vehicle; // Composition: Driver has one vehicle
    private DriverStatus status; // e.g., AVAILABLE, ON_TRIP

    // Methods
    public void acceptTrip(Trip trip);
    public void startTrip(Trip trip);
    public void endTrip(Trip trip);
    public void updateLocation(Location newLocation); // Pushes location to dispatch system
}

// Represents a single trip.
public class Trip {
    // Attributes
    private String tripId;
    private Rider rider; // Association
    private Driver driver; // Association
    private Location startLocation;
    private Location endLocation;
    private TripStatus status;
    private FareCalculationStrategy fareStrategy; // Strategy Pattern for fare
    private double fare;

    // Constructor
    public Trip(Rider rider, Driver driver, FareCalculationStrategy fareStrategy);

    // Methods
    public void calculateFare();
}

// Represents a vehicle.
public class Vehicle {
    private String licensePlate;
    private String model;
    private VehicleType type; // e.g., SEDAN, SUV
}

// Represents a geographical location.
public class Location {
    private double latitude;
    private double longitude;
}

// Strategy pattern for calculating fares. Allows for dynamic pricing models.
public interface FareCalculationStrategy {
    double calculateFare(Trip trip);
}
public class StandardFareStrategy implements FareCalculationStrategy {
    public double calculateFare(Trip trip); // Implements base fare logic
}
public class SurgePricingFareStrategy implements FareCalculationStrategy {
    public double calculateFare(Trip trip); // Implements surge pricing logic
}

// Enums for type and state management.
public enum TripStatus { REQUESTED, DRIVER_ASSIGNED, IN_PROGRESS, COMPLETED, CANCELED }
public enum DriverStatus { AVAILABLE, ON_TRIP, OFFLINE }
public enum VehicleType { SEDAN, SUV, HATCHBACK }
```

-----

## 🔑 7. Car Rental System

This design covers vehicle inventory management across multiple locations, reservations, and billing. It uses the **Factory** pattern to create vehicles and the **Strategy** pattern for payments.

### Core Classes & Relationships

**Relationships:**

  * `CarRentalSystem` has a **Composition** of `RentalLocation`s.
  * `RentalLocation` has a **Composition** of `VehicleInventory`.
  * `VehicleInventory` has an **Aggregation** of `Vehicle`s. Vehicles can exist independently but are part of an inventory.
  * `Vehicle` has an **Inheritance** hierarchy (`Car`, `SUV`, `Truck`).
  * `Reservation` links a `Customer` to a `Vehicle`.

-----

### Class Definitions

```java
// The main system facade.
public class CarRentalSystem {
    // Attributes
    private List<RentalLocation> locations;

    // Methods
    public List<RentalLocation> getLocations(String city);
    public Reservation makeReservation(Customer customer, VehicleType type, Date from, Date to);
    public void cancelReservation(Reservation reservation);
}

// Represents a physical rental location.
public class RentalLocation {
    // Attributes
    private String locationId;
    private Address address;
    private VehicleInventory inventory; // Composition

    // Methods
    public List<Vehicle> getAvailableVehicles(VehicleType type, Date from, Date to);
    public Reservation pickupVehicle(Reservation reservation);
    public Bill returnVehicle(Reservation reservation);
}

// Manages the collection of vehicles at a location.
public class VehicleInventory {
    // Using a Map for O(1) access to vehicle lists by type.
    private Map<VehicleType, List<Vehicle>> vehiclesByType;

    // Methods
    public void addVehicle(Vehicle vehicle);
    public void removeVehicle(Vehicle vehicle);
    public List<Vehicle> getVehicles();
}

// Abstract class for all vehicles, demonstrating abstraction.
public abstract class Vehicle {
    // Attributes
    private String licensePlate;
    private String model;
    private int year;
    private double dailyRentalCost;
    private VehicleStatus status;
    private List<VehicleLog> logs; // History of maintenance and rentals
}

// Concrete vehicle classes demonstrating inheritance.
public class Car extends Vehicle {}
public class SUV extends Vehicle {}

// Represents a reservation made by a customer.
public class Reservation {
    // Attributes
    private String reservationId;
    private Customer customer;
    private Vehicle vehicle;
    private Date fromDate;
    private Date toDate;
    private Location pickupLocation;
    private Location dropoffLocation;
    private ReservationStatus status;
}

// Represents the final bill for a rental.
public class Bill {
    // Attributes
    private Reservation reservation;
    private double totalAmount;
    private boolean isPaid;

    // Methods
    // Polymorphism via Strategy Pattern
    public boolean processPayment(PaymentStrategy paymentStrategy);
}

// An entry in the vehicle's history.
public class VehicleLog {
    private VehicleLogType type;
    private String description;
    private Date date;
}

// Enums for managing state and types.
public enum VehicleStatus { AVAILABLE, RENTED, UNDER_MAINTENANCE }
public enum ReservationStatus { CONFIRMED, CANCELED, COMPLETED }
public enum VehicleLogType { RENTAL, MAINTENANCE, ACCIDENT }
public enum VehicleType { CAR, SUV, TRUCK, VAN }
```

-----

## 🏧 8. ATM System

This design models an ATM's interaction with a user and the bank. It prominently features the **State** pattern to manage the ATM's lifecycle and the **Facade** pattern, where the `ATM` class provides a simple interface to a complex subsystem. Transactions are modeled using the **Command** pattern.

### Core Classes & Relationships

**Relationships:**

  * The `ATM` class is a **Facade** for the entire system.
  * `ATM` has a **Composition** relationship with its hardware components (`CardReader`, `Keypad`, etc.).
  * The system uses a **State** pattern, with `ATMState` as the interface and classes like `IdleState`, `HasCardState`, etc., as concrete states. The `ATM` class holds a reference to its current state.
  * `Transaction` is an interface for the **Command** pattern, encapsulating different operations.

-----

### Class Definitions

```java
// Facade for the system and context for the State pattern.
public class ATM {
    // Attributes
    private ATMState currentState;
    private CardReader cardReader;
    private Keypad keypad;
    private Screen screen;
    private CashDispenser cashDispenser;
    private BankService bankService; // Interface to bank's network

    // Constructor
    public ATM();

    // Methods delegated to the current state object.
    public void insertCard(Card card);
    public void enterPIN(String pin);
    public void selectOperation(TransactionType type, double amount);
    public void ejectCard();
    // Setter for changing state.
    public void setCurrentState(ATMState newState);
}

// State pattern interface.
public interface ATMState {
    void insertCard(ATM atm, Card card);
    void enterPIN(ATM atm, String pin);
    void selectOperation(ATM atm, TransactionType type, double amount);
    void ejectCard(ATM atm);
}

// Concrete state implementation.
public class IdleState implements ATMState {
    // Override all methods from ATMState interface.
    // e.g., insertCard will change the ATM's state to HasCardState.
}
public class HasCardState implements ATMState { /* ... */ }
public class AuthenticatedState implements ATMState { /* ... */ }

// Hardware components (Composition).
public class CardReader { public Card readCard(); }
public class Keypad { public String getInput(); }
public class Screen { public void displayMessage(String message); }
public class CashDispenser { public void dispenseCash(double amount); }

// Interface representing the connection to the bank.
public interface BankService {
    boolean authenticate(String cardNumber, String pin);
    double getBalance(String cardNumber);
    boolean executeTransaction(String cardNumber, double amount);
}

// Command pattern for transactions.
public interface Transaction {
    void execute(BankService service, String cardNumber, double amount);
}
public class WithdrawTransaction implements Transaction {
    public void execute(BankService service, String cardNumber, double amount);
}
public class DepositTransaction implements Transaction {
    public void execute(BankService service, String cardNumber, double amount);
}
public class BalanceInquiry implements Transaction {
    public void execute(BankService service, String cardNumber, double amount); // amount not used
}

// Represents the user's card.
public class Card {
    private String cardNumber;
    private String cardHolderName;
    private Date expiryDate;
}

// Enum for transaction types.
public enum TransactionType { WITHDRAW, DEPOSIT, BALANCE_INQUIRY }
```

-----

## 🔔 9. Notification System

A highly scalable and flexible notification system. It uses the **Strategy** pattern to handle different delivery channels, the **Builder** pattern for constructing complex notifications, and is designed to integrate with a message queue.

### Core Classes & Relationships

**Relationships:**

  * `NotificationService` uses the **Strategy** pattern via the `NotificationChannel` interface to send notifications.
  * `Notification` objects are complex and are best created with a **Builder** pattern.
  * The `Notification` has a **Composition** relationship with `Recipient` and `MessageContent`.
  * The system can be enhanced by the **Decorator** pattern to add features (e.g., logging, priority) to a channel.

-----

### Class Definitions

```java
// Main service for sending notifications.
public class NotificationService {
    // Attributes
    // In a real system, this would push to a message queue (e.g., Kafka, RabbitMQ)
    private MessageQueueClient queueClient;

    // Methods
    public void send(Notification notification);
}

// The core Notification object. Use a Builder for its construction.
public final class Notification { // final class as it's an immutable value object
    // Attributes
    private final String notificationId;
    private final Recipient recipient;
    private final MessageContent content;
    private final List<ChannelType> channels; // Target channels (EMAIL, SMS, etc.)
    private final Priority priority;

    // Private constructor to be used by the Builder.
    private Notification(NotificationBuilder builder);

    // Only getters, no setters, to ensure immutability.

    // Static nested Builder class.
    public static class NotificationBuilder {
        // Attributes matching Notification, but mutable.
        // ...
        public NotificationBuilder to(Recipient recipient);
        public NotificationBuilder withContent(MessageContent content);
        public NotificationBuilder viaChannel(ChannelType channel);
        public NotificationBuilder withPriority(Priority priority);
        public Notification build(); // Creates the final Notification object
    }
}

// Strategy interface for different delivery channels.
public interface NotificationChannel {
    boolean send(Notification notification);
    ChannelType getChannelType();
}

// Concrete strategy implementations for each channel.
public class EmailChannel implements NotificationChannel {
    public boolean send(Notification notification); // Logic to send via SMTP
}
public class SMSChannel implements NotificationChannel {
    public boolean send(Notification notification); // Logic to use a Twilio/SMS gateway
}
public class PushNotificationChannel implements NotificationChannel {
    public boolean send(Notification notification); // Logic to use APNS/FCM
}

// Factory to get the correct channel handler.
public class ChannelFactory {
    public NotificationChannel getChannel(ChannelType type);
}

// Represents the person or system receiving the notification.
public class Recipient {
    private String userId;
    private String emailAddress;
    private String phoneNumber;
    private String deviceToken;
}

// Represents the message content itself.
public class MessageContent {
    private String subject;
    private String body;
    // Map for template variables, e.g., {"username", "John"}
    private Map<String, String> templateParams;
}

// Enums for type safety.
public enum ChannelType { EMAIL, SMS, PUSH, SLACK }
public enum Priority { HIGH, MEDIUM, LOW }
```

-----

## 📜 10. Logging Framework

A design inspired by popular frameworks like Log4j or Logback. It uses a **Chain of Responsibility** for logger hierarchy, and the **Strategy** pattern for Appenders (output destinations) and Layouts (formatting).

### Core Classes & Relationships

**Relationships:**

  * `Logger`s form a hierarchy. A `Logger` has an optional reference to a parent `Logger`, enabling the **Chain of Responsibility** pattern for event propagation.
  * `Logger` has a **Composition** of `Appender`s.
  * `Appender` is a **Strategy** interface for output destinations.
  * `Appender` has a **Composition** relationship with a `Layout`, which is a **Strategy** for formatting.
  * `LogManager` is a **Singleton** used as a factory and registry for `Logger` instances.

-----

### Class Definitions

```java
// Manages the creation and retrieval of logger instances. Singleton/Registry pattern.
public class LogManager {
    // Attributes
    private static final LogManager instance = new LogManager();
    // Using a map to cache and retrieve loggers by name.
    private Map<String, Logger> loggerRegistry;

    // Methods
    public static LogManager getInstance();
    public Logger getLogger(String name); // Factory method for loggers
}

// The main class clients interact with.
public class Logger {
    // Attributes
    private final String name; // Hierarchical name, e.g., "com.mycompany.service"
    private LogLevel level; // The minimum level this logger will handle
    private List<Appender> appenders; // Composition of Appenders
    private Logger parent; // For Chain of Responsibility

    // Constructor (usually protected or package-private, controlled by LogManager).
    protected Logger(String name);

    // Core logging methods with Method Overloading.
    public void debug(String message);
    public void info(String message);
    public void warn(String message);
    public void error(String message, Throwable throwable);

    // Private method that does the actual logging logic.
    private void log(LogLevel level, String message, Throwable t);
}

// Strategy interface for log output destinations.
public interface Appender {
    void append(LoggingEvent event);
    void setLayout(Layout layout); // An appender has a layout
    Layout getLayout();
}

// Concrete Appender implementations.
public class ConsoleAppender implements Appender { /* ... */ }
public class FileAppender implements Appender {
    private String filePath;
    /* ... */
}
public class DatabaseAppender implements Appender { /* ... */ }

// Strategy interface for formatting log messages.
public interface Layout {
    String format(LoggingEvent event);
}

// Concrete Layout implementations.
public class SimpleLayout implements Layout {
    // Formats as "LEVEL - message"
    public String format(LoggingEvent event);
}
public class JsonLayout implements Layout {
    // Formats the event as a JSON string.
    public String format(LoggingEvent event);
}

// A data object containing all information about a single logging act.
public class LoggingEvent {
    // Attributes
    private final long timestamp;
    private final LogLevel level;
    private final String message;
    private final String loggerName;
    private final String threadName;
    private final Throwable throwable; // Optional exception info

    // Constructor
    public LoggingEvent(/* ... */);

    // Getters for all attributes.
}

// Enum for logging levels.
public enum LogLevel {
    DEBUG, INFO, WARN, ERROR, FATAL
}
```



Of course. Here are the low-level object-oriented designs for the next five systems, presented in the same concise, interview-ready format.

-----

## 💬 11. Chat Application (e.g., WhatsApp, Slack)

This design uses the **Observer** pattern for real-time message delivery and **Inheritance** to model different chat types. The `ChatServer` acts as a central mediator and a subject for observers (users).

### Core Classes & Relationships

**Relationships:**

  * `Chat` is an **abstract** base class for `PrivateChat` and `GroupChat` (**Inheritance**).
  * `User` and `Chat` have a many-to-many **Association**.
  * `ChatServer` is a **Singleton** and the central hub. It uses the **Observer** pattern to notify `User`s of new messages.
  * `Chat` has a **Composition** of `Message`s.
  * `Message` can have its own **Inheritance** hierarchy (e.g., `TextMessage`, `ImageMessage`).

-----

### Class Definitions

```java
// Central server acting as a mediator and subject for the Observer pattern.
public class ChatServer {
    // Attributes
    private Map<String, User> connectedUsers; // Key: userId
    private Map<String, Chat> activeChats; // Key: chatId
    private static final ChatServer instance = new ChatServer();

    // Constructor
    private ChatServer() {} // Singleton

    // Methods
    public static ChatServer getInstance();
    public void sendMessage(User sender, Message message, String chatId);
    public void registerUser(User user); // Connects a user (Observer)
    public void unregisterUser(User user);
    public Chat createChat(List<User> participants);
}

// Represents a user who acts as an Observer.
public class User {
    // Attributes
    private String userId;
    private String username;
    private UserStatus status;
    private List<Contact> contacts;

    // Methods
    public void sendNewMessage(String chatId, MessageContent content);
    public void receiveMessage(Message message); // Method called by the Subject (ChatServer)
    public void joinChat(Chat chat);
    public void leaveChat(Chat chat);
}

// Abstract base class for all conversations.
public abstract class Chat {
    // Attributes
    protected String chatId;
    protected List<User> participants;
    protected List<Message> messages;

    // Methods
    public abstract void addParticipant(User user);
    public abstract void removeParticipant(User user);
    public void postMessage(Message message);
}

// Concrete chat types.
public class PrivateChat extends Chat { /* Specific implementation for 1-on-1 */ }
public class GroupChat extends Chat {
    private User admin;
    /* Specific implementation for groups */
}

// Represents a single message.
public class Message {
    // Attributes
    private String messageId;
    private User sender;
    private long timestamp;
    private MessageContent content; // Can be text, image, etc.
    private MessageStatus status;
}

// Can be implemented as an interface/abstract class for different message types.
public class MessageContent {
    private String data;
    private ContentType type;
}

// Enums for state and type management.
public enum UserStatus { ONLINE, OFFLINE, AWAY }
public enum MessageStatus { SENT, DELIVERED, READ }
public enum ContentType { TEXT, IMAGE, VIDEO, FILE }
```

-----

## 📡 12. Publish-Subscribe System

This is a classic implementation of the **Observer** pattern. The `Broker` manages `Topic`s, which are the subjects. `ISubscriber` implementations are the observers that get notified when a message is published to a topic they follow.

### Core Classes & Relationships

**Relationships:**

  * This system is a direct implementation of the **Observer** design pattern.
  * `Broker` has a **Composition** of `Topic`s.
  * `Topic` (the Subject) has a list of `ISubscriber`s (the Observers).
  * `IPublisher` and `ISubscriber` are **interfaces**, promoting loose coupling.

-----

### Class Definitions

```java
// The central broker or facade for the system.
public class Broker {
    // Attributes
    private Map<String, Topic> topics; // Key: topicName
    private static final Broker instance = new Broker();

    // Constructor
    private Broker() {} // Singleton

    // Methods
    public static Broker getInstance();
    public void createTopic(String topicName);
    public void subscribe(String topicName, ISubscriber subscriber);
    public void unsubscribe(String topicName, ISubscriber subscriber);
    public void publishMessage(String topicName, Message message);
}

// Represents a topic, acting as the "Subject" in the Observer pattern.
public class Topic {
    // Attributes
    private final String name;
    private final List<ISubscriber> subscribers; // The list of Observers
    // Each topic has its own queue of messages.
    private final Queue<Message> messageQueue;

    // Methods
    public void addSubscriber(ISubscriber subscriber);
    public void removeSubscriber(ISubscriber subscriber);
    public void addMessage(Message message); // Adds a new message to the queue
    public void notifySubscribers(); // Pushes messages to all subscribers
}

// Interface for any entity that can subscribe to topics.
public interface ISubscriber {
    String getId();
    void onMessage(Message message); // The update method called by the Subject
}

// Interface for any entity that can publish messages.
public interface IPublisher {
    void publish(String topicName, Message message, Broker broker);
}

// Concrete subscriber implementation.
public class ConcreteSubscriber implements ISubscriber {
    private final String id;
    public void onMessage(Message message);
}

// The data object that is transmitted.
public class Message {
    // Attributes
    private final String messageId;
    private final String payload; // Can be a generic type or JSON string
    private final long timestamp;
}
```

-----

## ❓ 13. Stack Overflow

This design models the core entities of a Q\&A platform like Stack Overflow. It uses **Inheritance** for user roles and content types and the **State** pattern for managing a question's lifecycle.

### Core Classes & Relationships

**Relationships:**

  * `Content` is an **abstract** base class for `Question`, `Answer`, and `Comment` (**Inheritance**).
  * `User` is an **abstract** base class for `Member` and `Moderator` (**Inheritance**).
  * `Question` has a **Composition** of `Answer`s. Both `Question` and `Answer` have a **Composition** of `Comment`s.
  * `Question` has a many-to-many **Association** with `Tag`.
  * A `Question`'s lifecycle is managed by the **State** pattern (`QuestionState` interface).

-----

### Class Definitions

```java
// Abstract base class for all posted content.
public abstract class Content {
    protected String contentId;
    protected String text;
    protected Member author;
    protected Date creationTime;
    protected int voteCount;

    public void incrementVoteCount();
}

// Represents a question, which is a type of content.
public class Question extends Content {
    // Attributes
    private String title;
    private List<Answer> answers;
    private List<Tag> tags;
    private QuestionState state; // State pattern for lifecycle
    private int bounty;

    // Methods
    public void addAnswer(Answer answer);
    public void addTag(Tag tag);
    public void close();
    public void delete();
}

// Represents an answer, which is a type of content.
public class Answer extends Content {
    private boolean isAccepted;
    public void markAsAccepted();
}

// Represents a comment, a simpler form of content.
public class Comment extends Content { /* No extra fields needed for basic LLD */ }

// Abstract base class for users.
public abstract class User {
    private String userId;
    private String username;
    private String email;
}

// A member is a user who can post and vote.
public class Member extends User {
    private int reputation;
    private List<Badge> badges;
    public void addReputation(int points);
    public void createQuestion(String title, String text);
    public void createAnswer(Question question, String text);
}

// State pattern for managing the question's lifecycle.
public interface QuestionState {
    void close(Question question);
    void delete(Question question);
    void reopen(Question question);
}
public class OpenState implements QuestionState { /* ... */ }
public class ClosedState implements QuestionState { /* ... */ }

// Represents a tag for categorizing questions.
public class Tag {
    private String tagId;
    private String name;
}
```

-----

## 📚 14. Library Management System

This design uses the **Facade** pattern (`Library` class) to provide a simple interface for complex operations like issuing and returning books. The **Observer** pattern is used to notify members about reserved book availability.

### Core Classes & Relationships

**Relationships:**

  * The `Library` class is a **Facade**.
  * `User` is an **abstract** base class for `Member` and `Librarian`.
  * A `Book` is a logical entity, while a `BookItem` is a physical copy. This is a key distinction. A `Book` has a one-to-many **Composition** with `BookItem`.
  * The system uses the **Observer** pattern where `Book` is the subject and `Member`s are observers for reservations.
  * `SearchService` is a **Strategy** interface for different search algorithms.

-----

### Class Definitions

```java
// Facade for the entire system.
public class Library {
    // Attributes
    private Catalog catalog;
    private List<Member> members;

    // Methods
    public List<BookItem> searchBook(SearchStrategy strategy, String query);
    public boolean issueBook(Member member, String barcode);
    public boolean returnBook(Member member, String barcode);
    public Fine calculateFine(Member member, String barcode);
}

// Represents a physical copy of a book.
public class BookItem {
    // Attributes
    private final String barcode;
    private final Book bookDetails; // Link to the logical book
    private BookStatus status;
    private Date issueDate;
    private Date dueDate;

    // Methods
    public boolean checkout(Member member);
    public boolean checkin();
}

// Represents the logical book entity (title, author, etc.).
// This is the "Subject" in the Observer pattern for reservations.
public class Book {
    // Attributes
    private final String ISBN;
    private final String title;
    private final String author;
    private final List<Member> reservationObservers; // List of observers

    // Methods
    public void addObserver(Member member);
    public void removeObserver(Member member);
    public void notifyObservers(); // Called when a BookItem becomes available
}

// A user of the library. Acts as an "Observer" for reservations.
public class Member extends User {
    // Attributes
    private List<BookItem> booksLoaned;

    // Methods
    public void update(Book book); // Method called by the Subject (Book)
}

// Represents a librarian with special privileges.
public class Librarian extends User {
    // Methods
    public void addBookItem(BookItem bookItem);
    public void removeBookItem(BookItem bookItem);
    public void blockMember(Member member);
}

// Strategy pattern for searching.
public interface SearchStrategy {
    List<BookItem> search(String query, Catalog catalog);
}
public class SearchByTitle implements SearchStrategy { /* ... */ }
public class SearchByAuthor implements SearchStrategy { /* ... */ }

// Enums for state management.
public enum BookStatus { AVAILABLE, LOANED, RESERVED, LOST }
```

-----

## 🏭 15. Vending Machine

This design is a prime example of the **State** pattern. The machine's behavior (`insertCoin`, `selectItem`) changes based on its current state, providing a robust and easy-to-manage workflow. The `VendingMachine` class acts as the **Facade** to the user.

### Core Classes & Relationships

**Relationships:**

  * `VendingMachine` is the context for the **State** pattern and a **Facade** for the user.
  * `State` is an interface with concrete implementations like `NoCoinInsertedState`, `CoinInsertedState`, etc.
  * `VendingMachine` has a **Composition** relationship with its `Inventory` and `CashInventory`.
  * `Inventory` is composed of `ItemSlot`s.

-----

### Class Definitions

```java
// The main Vending Machine class, acting as the Facade and State pattern Context.
public class VendingMachine {
    // Attributes
    private State currentState;
    private Inventory inventory;

    private CashInventory cashInventory;
    private double currentBalance;
    private Item selectedItem;

    // Constructor
    public VendingMachine();

    // Methods that delegate actions to the current state object.
    public void insertCoin(Coin coin);
    public void selectItem(int itemCode);
    public void dispenseItem();
    public List<Coin> cancelAndEjectCoins();

    // Getters and Setters used by State objects to manage the context.
    public void changeState(State newState);
    public void addToBalance(double amount);
    // ... other getters/setters
}

// State pattern interface.
public interface State {
    void insertCoin(VendingMachine machine, Coin coin);
    void selectItem(VendingMachine machine, int itemCode);
    void dispenseItem(VendingMachine machine);
    List<Coin> cancelAndEjectCoins(VendingMachine machine);
}

// Concrete state implementations.
public class NoCoinInsertedState implements State {
    // insertCoin will change state to CoinInsertedState.
    // other methods will likely throw an exception or do nothing.
}
public class CoinInsertedState implements State {
    // selectItem will validate and change state to DispensingState.
    // cancelAndEjectCoins will return balance and change to NoCoinInsertedState.
}
public class DispensingState implements State { /* ... */ }
public class SoldOutState implements State { /* ... */ }

// Manages the items available for sale.
public class Inventory {
    // Map of item codes to the physical slot holding the items.
    private Map<Integer, ItemSlot> slots;

    // Methods
    public Item getItem(int itemCode);
    public void decreaseItemCount(int itemCode);
    public int getItemCount(int itemCode);
}

// Represents a slot in the machine holding one type of item.
public class ItemSlot {
    private Item item;
    private int count;
}

// Represents a product.
public class Item {
    private String name;
    private double price;
}

// Enum for accepted coins.
public enum Coin {
    PENNY(0.01), NICKEL(0.05), DIME(0.10), QUARTER(0.25);
    private double value;
    Coin(double value) { this.value = value; }
    public double getValue() { return value; }
}
```




Of course. Here are the low-level object-oriented designs for the final set of systems, presented in the same crisp, interview-ready format.

-----

## 🃏 16. Blackjack and Deck of Cards

This design models a generic deck of cards and a specific implementation for the game of Blackjack. It emphasizes clear separation of concerns between the card representation, the deck, player hands, and the game logic itself.

### Core Classes & Relationships

**Relationships:**

  * `Dealer` and `Player` both inherit from an **abstract** class `AbstractParticipant`, which holds a `Hand` (**Inheritance**).
  * `BlackjackGame` has a **Composition** relationship with `Deck`, `Dealer`, and a list of `Player`s.
  * `Hand` is composed of `Card` objects (**Composition**).
  * `Deck` is composed of `Card` objects (**Composition**).
  * `Suit` and `Rank` are **Enums** to ensure type safety for `Card` properties.

-----

### Class Definitions

```java
// Represents a single playing card. This class is immutable.
public final class Card {
    // Attributes
    private final Suit suit;
    private final Rank rank;

    // Constructor
    public Card(Suit suit, Rank rank);

    // Methods
    public int getValue(); // Returns the Blackjack value (e.g., King = 10, Ace = 11/1)
    public Suit getSuit();
    public Rank getRank();
}

// Represents a deck of cards. Can be extended for multi-deck shoes.
public class Deck {
    // Attributes
    // A Deque is optimal as it provides efficient LIFO (deal from top) operations.
    private Deque<Card> cards;

    // Constructor
    public Deck(int numberOfSets); // Create a deck with 1 or more sets of 52 cards

    // Methods
    public void shuffle();
    public Card deal();
    public int cardsLeft();
}

// Represents the cards held by a participant.
public class Hand {
    // Attributes
    private List<Card> cards;

    // Methods
    public void addCard(Card card);
    public int calculateValue(); // Calculates the total value of the hand, handling Aces correctly
    public void clear();
}

// Abstract base class for game participants.
public abstract class AbstractParticipant {
    // Attributes
    protected Hand hand;
    protected String name;

    // Methods
    public void hit(Deck deck);
    public abstract boolean wantsToHit(); // Abstract method to be defined by subclasses
}

// Represents a human player.
public class Player extends AbstractParticipant {
    // Attributes
    private double balance;

    // Methods
    public void placeBet(double amount);
    @Override
    public boolean wantsToHit(); // Logic to get player input (hit or stand)
}

// Represents the dealer with specific game rules.
public class Dealer extends AbstractParticipant {
    @Override
    public boolean wantsToHit(); // Implements dealer's rule (e.g., hit until 17)
    public void revealFirstCard();
}

// Orchestrates the entire game flow.
public class BlackjackGame {
    // Attributes
    private Deck deck;
    private Dealer dealer;
    private List<Player> players;

    // Methods
    public void startGame();
    private void placeBets();
    private void dealInitialCards();
    private void executePlayerTurns();
    private void executeDealerTurn();
    private void determineWinners();
}

// Enums for type safety and clarity.
public enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES }
public enum Rank { TWO, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING, ACE }
```

-----

## 🍔 17. Food Delivery System (e.g., DoorDash)

This design uses the **Observer** pattern to keep all parties (customer, restaurant, driver) updated on the order status. It also uses the **Strategy** pattern for calculating delivery fees and handling payments.

### Core Classes & Relationships

**Relationships:**

  * `Order` is the "Subject" in the **Observer** pattern. `Customer`, `Restaurant`, and `Driver` are "Observers".
  * `User` is an **abstract** base class for `Customer`, `Driver`, and `RestaurantManager`.
  * `Restaurant` has a **Composition** of `Menu`, which in turn is composed of `MenuItem`s.
  * `Order` has an **Association** with a `Customer`, `Restaurant`, and `Driver`.
  * `DeliveryFeeStrategy` is a **Strategy** interface for pricing.

-----

### Class Definitions

```java
// Central facade and service orchestrator.
public class FoodDeliveryService {
    // Attributes
    private List<Restaurant> restaurants;
    private OrderDispatchSystem dispatchSystem; // Singleton for matching drivers

    // Methods
    public List<Restaurant> findRestaurants(String areaCode);
    public Order placeOrder(Customer customer, Restaurant restaurant, List<MenuItem> items);
}

// The "Subject" of the Observer pattern.
public class Order {
    // Attributes
    private String orderId;
    private Customer customer;
    private Restaurant restaurant;
    private Driver driver;
    private List<MenuItem> items;
    private OrderStatus status;
    private List<IOrderObserver> observers; // List of observers

    // Methods
    public void addObserver(IOrderObserver observer);
    public void removeObserver(IOrderObserver observer);
    public void notifyObservers();
    public void setStatus(OrderStatus newStatus); // Triggers notifyObservers()
}

// Observer interface.
public interface IOrderObserver {
    void update(Order order); // Method called by the Subject
}

// Abstract base class for users.
public abstract class User {
    private String userId;
    private String name;
}

// Concrete user types that also act as Observers.
public class Customer extends User implements IOrderObserver {
    public void update(Order order);
}
public class Restaurant extends IOrderObserver { // Restaurants are also observers
    private String restaurantId;
    private String name;
    private Menu menu;
    public void update(Order order); // e.g., print new order in the kitchen
}
public class Driver extends User implements IOrderObserver {
    private Location currentLocation;
    public void update(Order order); // e.g., get notification for new pickup
}

// Represents a restaurant's menu.
public class Menu {
    private List<MenuItem> items;
}

// Represents a single food item.
public class MenuItem {
    private String itemId;
    private String name;
    private String description;
    private double price;
}

// Enums for state management.
public enum OrderStatus { PLACED, ACCEPTED, PREPARING, READY_FOR_PICKUP, OUT_FOR_DELIVERY, DELIVERED, CANCELED }
```

-----

## 🏛️ 18. Online Auction System

This design uses the **Observer** pattern to notify bidders of new bids and the **State** pattern to manage the auction lifecycle. It ensures thread safety for placing bids, a critical requirement for an auction system.

### Core Classes & Relationships

**Relationships:**

  * `Auction` is the "Subject" for the **Observer** pattern, notifying subscribed `Bidder`s.
  * The `Auction` class is also the "Context" for the **State** pattern (`AuctionState`).
  * `User` is an **abstract** class for `Seller` and `Bidder`.
  * `Auction` has an **Association** with an `Item` and a `Seller`.
  * `Auction` has a **Composition** of `Bid`s.

-----

### Class Definitions

```java
// Main service to manage all auctions.
public class AuctionService {
    // Attributes
    private Map<String, Auction> auctions; // Key: auctionId

    // Methods
    public Auction createAuction(Seller seller, Item item, double startPrice, Date endTime);
    public void placeBid(Auction auction, Bidder bidder, double amount);
    public List<Auction> searchAuctions(String keyword);
}

// The "Subject" for Observers and "Context" for the State pattern.
public class Auction {
    // Attributes
    private String auctionId;
    private Item item;
    private Seller seller;
    private Date endTime;
    private double currentHighestBid;
    private Bidder highestBidder;
    private AuctionState state; // State Pattern
    private List<Bidder> bidders; // The list of Observers

    // Methods
    public synchronized void placeBid(Bidder bidder, double amount); // synchronized for thread safety
    public void startAuction();
    public void endAuction();
    public void addObserver(Bidder bidder);
    public void notifyBidders();
    public void changeState(AuctionState newState);
}

// State Pattern Interface.
public interface AuctionState {
    void placeBid(Auction auction, Bidder bidder, double amount);
    void endAuction(Auction auction);
}

// Concrete states.
public class PendingState implements AuctionState { /* ... */ }
public class ActiveState implements AuctionState { /* ... */ }
public class ClosedState implements AuctionState { /* ... */ }

// Represents an item up for auction.
public class Item {
    private String itemId;
    private String name;
    private String description;
}

// Represents a single bid.
public class Bid {
    private String bidId;
    private double amount;
    private Date timestamp;
    private Bidder bidder;
}

// Abstract base user class.
public abstract class User {
    private String userId;
    private String username;
}

// A user who can bid. Also an "Observer".
public class Bidder extends User {
    public void update(Auction auction); // Method called by the Auction Subject
}

// A user who can sell items.
public class Seller extends User { /* ... */ }
```

-----

## 🏨 19. Hotel Management System

This design models the core operations of a hotel, including booking, room management, and guest services. It uses the **Factory** pattern to create different room types and the **Strategy** pattern for dynamic pricing.

### Core Classes & Relationships

**Relationships:**

  * `Room` is an **abstract** base class. A `RoomFactory` uses the **Factory** pattern to create concrete instances like `StandardRoom`, `DeluxeRoom`, etc.
  * `User` is an **abstract** base class for `Guest` and various `Staff` roles.
  * `Hotel` has a **Composition** of `Room`s.
  * `Booking` has an **Association** with a `Guest` and one or more `Room`s.
  * `PricingStrategy` is a **Strategy** interface to allow for flexible pricing models.

-----

### Class Definitions

```java
// Facade for the entire system.
public class Hotel {
    // Attributes
    private String name;
    private Address address;
    private List<Room> rooms;

    // Methods
    public List<Room> searchRooms(RoomType type, Date startDate, Date endDate);
    public Booking makeBooking(Guest guest, List<Room> rooms, Date startDate, Date endDate);
    public void checkIn(Booking booking);
    public Invoice checkOut(Booking booking);
}

// Abstract base class for all rooms.
public abstract class Room {
    // Attributes
    protected String roomNumber;
    protected RoomStyle style;
    protected RoomStatus status;
    protected double basePrice;

    // Methods
    public abstract boolean isAvailable();
    public void checkIn();
    public void checkOut();
}

// Concrete room classes.
public class StandardRoom extends Room { /* ... */ }
public class DeluxeRoom extends Room { /* ... */ }

// Factory for creating different room types.
public class RoomFactory {
    public static Room createRoom(RoomType type, String roomNumber);
}

// Represents a guest booking.
public class Booking {
    // Attributes
    private String bookingId;
    private Guest guest;
    private List<Room> bookedRooms;
    private Date startDate;
    private Date endDate;
    private BookingStatus status;
    private PricingStrategy pricingStrategy; // Strategy Pattern

    // Methods
    public double calculateTotalCost();
}

// Abstract base user class.
public abstract class User {
    private String userId;
    private String name;
}

// A guest staying at the hotel.
public class Guest extends User {
    private List<Booking> bookingHistory;
}

// A hotel staff member.
public class Staff extends User {
    /* ... */
}

// Strategy Pattern for pricing.
public interface PricingStrategy {
    double calculatePrice(Booking booking);
}
public class StandardPricing implements PricingStrategy { /* ... */ }
public class SeasonalPricing implements PricingStrategy { /* ... */ }

// Enums for type and state management.
public enum RoomType { STANDARD, DELUXE, SUITE }
public enum RoomStatus { AVAILABLE, OCCUPIED, UNDER_MAINTENANCE }
public enum BookingStatus { REQUESTED, CONFIRMED, CHECKED_IN, CHECKED_OUT, CANCELED }
```

-----

## ♟️ 20. Chess Game

This design elegantly models a game of chess by using the **Strategy** pattern for move validation, which is the most complex part of chess logic. **Inheritance** is used for the piece hierarchy.

### Core Classes & Relationships

**Relationships:**

  * `Piece` is an **abstract** class, with concrete classes like `Pawn`, `Rook`, etc., using **Inheritance**.
  * Each `Piece` has a `IMoveStrategy`, demonstrating the **Strategy** pattern to encapsulate move validation logic.
  * `Game` has a **Composition** of a `Board` and two `Player`s.
  * `Board` is composed of a 2D array of `Square`s (**Composition**).
  * `Square` has an optional **Association** with a `Piece`.

-----

### Class Definitions

```java
// Main class that orchestrates the game.
public class Game {
    // Attributes
    private Board board;
    private Player whitePlayer;
    private Player blackPlayer;
    private Player currentPlayer;
    private GameStatus status;
    private List<Move> moveHistory;

    // Methods
    public boolean makeMove(Player player, Move move);
    private void switchTurn();
    private boolean isCheckmate();
}

// Represents the 8x8 game board.
public class Board {
    // Attributes
    private Square[][] squares; // 8x8 grid of squares

    // Methods
    public void setupBoard();
    public Square getSquare(int x, int y);
}

// Represents a single square on the board.
public class Square {
    // Attributes
    private int x, y;
    private Piece piece; // Association with a piece

    // Methods
    public Piece getPiece();
    public void setPiece(Piece piece);
}

// Abstract base class for all chess pieces.
public abstract class Piece {
    // Attributes
    protected Color color;
    protected IMoveStrategy moveStrategy; // Strategy Pattern

    // Methods
    public abstract boolean canMove(Board board, Move move);
    public Color getColor();
}

// Strategy interface for move validation.
public interface IMoveStrategy {
    boolean validateMove(Board board, Move move);
}

// Concrete piece and strategy implementations.
public class Knight extends Piece {
    public Knight(Color color) {
        super(color);
        this.moveStrategy = new KnightMoveStrategy();
    }
    public boolean canMove(Board board, Move move) {
        return this.moveStrategy.validateMove(board, move);
    }
}
public class KnightMoveStrategy implements IMoveStrategy {
    public boolean validateMove(Board board, Move move) { /* Knight's L-shape logic */ }
}
// ... similar classes for Pawn, Rook, Bishop, Queen, King

// Represents a player.
public class Player {
    private String name;
    private Color color;
}

// Represents a single move from a start to an end square.
public class Move {
    private Square start;
    private Square end;
    private Piece pieceMoved;
    private Piece pieceKilled;
}

// Enums for type safety.
public enum Color { WHITE, BLACK }
public enum GameStatus { ACTIVE, CHECK, CHECKMATE, STALEMATE, DRAW }
```




Of course. Here is a comprehensive, interview-ready, and optimal Object-Oriented Low-Level Design (LLD) for each of the requested systems. The designs emphasize core OOPS principles, optimal data structures, and real-world applicability.

-----

## 1\. Tic-Tac-Toe Game

This design focuses on modularity and scalability, allowing for easy extension (e.g., to a 4x4 board or different types of players like AI).

### Core Classes & Design

```java
// Main class to orchestrate the entire game flow.
public class Game {
    private Board board; // Composition: Game 'has a' Board.
    private Player[] players; // Aggregation: Game 'has' Players.
    private Player currentPlayer; // The player whose turn it is.
    private GameStatus status; // Enum to track the game's state.
    private final List<Move> moves; // Records the history of all moves.

    // Constructor to initialize the game with players and a board size.
    public Game(Player player1, Player player2, int boardSize) {}

    // Starts the game loop.
    public void startGame() {}

    // A private helper method to switch turns between players.
    private void changeTurn() {}

    // Checks for a win or draw condition after each move.
    private boolean checkWinner() {}
}

// Represents the game board. Manages the grid and piece placements.
public class Board {
    private final int size; // Using 'final' as board size is immutable post-creation.
    private final Piece[][] grid; // 2D array is optimal for a fixed-size grid.

    // Constructor to initialize the board.
    public Board(int size) {}

    // Places a piece on the board if the cell is empty. Returns true on success.
    public boolean placePiece(int row, int col, Piece piece) {}

    // Checks if the board is full, indicating a potential draw.
    public boolean isFull() {}

    // Prints the current state of the board to the console.
    public void display() {}
}

// Represents a player. Can be extended to HumanPlayer and BotPlayer.
// This is an 'abstract' class demonstrating Abstraction and Inheritance.
public abstract class Player {
    private final String name;
    private final Piece playingPiece; // The piece (X or O) assigned to the player.

    // Constructor.
    public Player(String name, Piece playingPiece) {}

    // Abstract method forcing subclasses to implement their move logic. Polymorphism.
    public abstract Move makeMove();

    // Getter for the player's name. Encapsulation.
    public String getName() { return this.name; }

    // Overriding 'equals()' and 'hashCode()' for proper comparison of Player objects.
    @Override
    public boolean equals(Object o) {}
    @Override
    public int hashCode() {}
}

// Concrete implementation for a human player. Inheritance.
public class HumanPlayer extends Player {
    // Constructor calling the parent class constructor using 'super'.
    public HumanPlayer(String name, Piece playingPiece) {
        super(name, playingPiece);
    }

    // Implementation of the move logic for a human.
    @Override
    public Move makeMove() {}
}

// Represents a single move made by a player.
public class Move {
    private final int row;
    private final int col;
    private final Player player;

    // Constructor.
    public Move(int row, int col, Player player) {}
}

// Using an Enum for a fixed set of piece types. Type-safe and clean.
public enum Piece {
    X, O, EMPTY;
}

// Enum for managing the game's state.
public enum GameStatus {
    IN_PROGRESS, FINISHED, DRAW;
}
```

### OOPS Concepts Applied

  * **Encapsulation**: All member variables are `private` with controlled access.
  * **Abstraction**: `Player` is an `abstract` class, hiding move implementation details.
  * **Inheritance**: `HumanPlayer` inherits from `Player`.
  * **Polymorphism**: The `makeMove()` method can be called on any `Player` object, and the specific version for `HumanPlayer` (or a future `BotPlayer`) will be executed.
  * **Composition**: `Game` is *composed of* a `Board`. The `Board`'s lifecycle is managed by the `Game`.
  * **Aggregation**: `Game` *has an aggregation* of `Player`s. Players can exist independently of a single game.
  * **Enums**: Used for type-safe representation of `Piece` and `GameStatus`.
  * **`final` keyword**: Used for immutable fields like `size` and `playingPiece`.

-----

## 2\. Snake and Ladder Game

This design uses inheritance to model the common behavior of snakes and ladders, simplifying the board setup.

### Core Classes & Design

```java
// Main class to manage the game flow.
public class Game {
    private Board board; // Composition.
    private Queue<Player> players; // A Queue is optimal for turn-based play (FIFO).
    private Dice dice; // Composition.
    private GameStatus status;

    // Constructor Overloading: Can start a game with a default dice or a custom one.
    public Game(List<Player> players) {}
    public Game(List<Player> players, Dice dice) {}

    // Starts the game loop.
    public void startGame() {}

    // A single turn for the current player.
    private void playTurn() {}
}

// Represents the game board, composed of cells.
public class Board {
    private final int size;
    private final Cell[] cells; // 1D array is sufficient, mapping 1-100.
    private final Map<Integer, Jumper> jumpers; // Map for efficient lookup of start positions of snakes/ladders.

    // Constructor initializes cells and places jumpers (snakes/ladders).
    public Board(int size, List<Jumper> jumpers) {}

    // Calculates the final position after a move, considering snakes and ladders.
    public int getNextPosition(int currentPosition, int diceValue) {}
}

// Represents a single cell on the board.
public class Cell {
    private final int position; // The number of the cell (1-100).
}

// Abstract class for Snakes and Ladders to demonstrate Inheritance.
// This shows 'is-a' relationship and promotes code reuse.
public abstract class Jumper {
    private final int startPosition;
    private final int endPosition;

    public Jumper(int startPosition, int endPosition) {}

    // Getters for start and end positions.
    public int getStartPosition() { return this.startPosition; }
    public int getEndPosition() { return this.endPosition; }
}

// Snake is a type of Jumper.
public class Snake extends Jumper {
    public Snake(int startPosition, int endPosition) {
        super(startPosition, endPosition); // 'super' keyword usage.
    }
}

// Ladder is a type of Jumper.
public class Ladder extends Jumper {
    public Ladder(int startPosition, int endPosition) {
        super(startPosition, endPosition);
    }
}

// Represents a player in the game.
public class Player {
    private final String name;
    private int currentPosition;

    public Player(String name) {}

    // Updates the player's position on the board.
    public void setPosition(int position) {}
}

// Encapsulates the logic for the dice.
public class Dice {
    private final int numDice;

    public Dice(int numDice) {}

    // Returns a random number between 1*numDice and 6*numDice.
    public int roll() {}
}

// Enum for game status.
public enum GameStatus {
    NOT_STARTED, IN_PROGRESS, FINISHED;
}
```

### OOPS Concepts Applied

  * **Inheritance**: `Snake` and `Ladder` inherit from the `abstract` class `Jumper`.
  * **Composition**: `Game` is composed of a `Board` and `Dice`. `Board` is composed of `Cell`s.
  * **Data Structures**: `Queue` for managing player turns, `Map` for efficient jumper lookups.
  * **Constructor Overloading**: `Game` can be initialized in multiple ways.
  * **`super` keyword**: Used in `Snake` and `Ladder` constructors to call the parent `Jumper` constructor.

-----

## 3\. File Storage System (e.g., Google Drive)

This design uses the **Composite Design Pattern**, a powerful structural pattern to treat individual objects (`File`) and compositions of objects (`Directory`) uniformly.

### Core Classes & Design

```java
// The entry point to the file system. Implemented as a Singleton.
public class FileSystem {
    private static FileSystem instance; // Static member for Singleton pattern.
    private final Directory root;

    // Private constructor to prevent direct instantiation.
    private FileSystem() {}

    // Static method to get the single instance.
    public static synchronized FileSystem getInstance() {}

    // Methods to interact with the file system, e.g., creating files/dirs at a path.
    public void createFile(String path, String content) {}
    public void createDirectory(String path) {}
    public void delete(String path) {}
    public String readFile(String path) {}
}

// Interface defining the common contract for both File and Directory.
// This is the core of the Composite Pattern.
public interface StorageItem {
    String getName();
    int getSize();
    void delete();
}

// Represents a single file. It's a 'Leaf' node in the composite structure.
public class File implements StorageItem {
    private String name;
    private String content;
    private int size;
    private final Directory parent; // Association to its containing directory.

    public File(String name, String content, Directory parent) {}

    // Overridden methods from the StorageItem interface.
    @Override public String getName() {}
    @Override public int getSize() {}
    @Override public void delete() {}
}

// Represents a directory. It's a 'Composite' node that can contain other StorageItems.
public class Directory implements StorageItem {
    private String name;
    private final Directory parent;
    // A Map is optimal for quick lookup of children by name.
    private final Map<String, StorageItem> children;

    public Directory(String name, Directory parent) {}

    public void addChild(StorageItem item) {}
    public void removeChild(String name) {}
    public StorageItem getChild(String name) {}

    // The size of a directory is the sum of the sizes of its children.
    // This is a polymorphic calculation.
    @Override
    public int getSize() {}

    @Override public String getName() {}
    @Override public void delete() {}
}

// Represents a user of the system.
public class User {
    private final String userId;
    private String name;
}

// Manages permissions. Could be extended into a more complex ACL system.
// This is an example of Association between User and StorageItem.
public class Permission {
    private final User user;
    private final StorageItem item;
    private PermissionType type;

    public Permission(User user, StorageItem item, PermissionType type) {}
}

// Enum for permission types.
public enum PermissionType {
    READ, WRITE, EXECUTE;
}
```

### OOPS Concepts Applied

  * **Composite Pattern**: `StorageItem` interface is implemented by both `File` (leaf) and `Directory` (composite), allowing clients to treat them uniformly.
  * **Singleton Pattern**: `FileSystem` class ensures only one instance exists.
  * **Polymorphism**: The `getSize()` method behaves differently for a `File` versus a `Directory`.
  * **Association**: `Permission` class links a `User` and a `StorageItem`. `File` and `Directory` have an association with their `parent`.
  * **High Cohesion**: Each class has a single, well-defined responsibility (e.g., `File` holds content, `Directory` manages children).
  * **Low Coupling**: `FileSystem` interacts with children through the `StorageItem` interface, not concrete classes, reducing dependencies.

-----

## 4\. Airline Reservation System

This is a complex domain model that demonstrates how classes represent real-world entities and their intricate relationships.

### Core Classes & Design

```java
// Represents a specific flight on a particular date.
public class FlightInstance {
    private final String flightNumber; // e.g., "UA249"
    private final Flight flightRoute; // Association: Connects to the general flight route.
    private final LocalDateTime departureTime;
    private final LocalDateTime arrivalTime;
    private Aircraft aircraft; // The specific plane for this flight instance.
    private final Map<String, Seat> seats; // Map of seat number to Seat object for fast lookup.

    public FlightInstance(Flight flightRoute, LocalDateTime departureTime, Aircraft aircraft) {}

    // Checks seat availability.
    public List<Seat> getAvailableSeats() {}
}

// Represents a general flight route (e.g., SFO to JFK).
public class Flight {
    private final String flightNumber;
    private final Airport departureAirport;
    private final Airport arrivalAirport;
    private final int durationInMinutes;

    public Flight(String flightNumber, Airport departure, Airport arrival, int duration) {}
}

// Represents a passenger with their booking information.
public class Passenger {
    private String passengerId;
    private String name;
    private String email;
}

// Represents a single booking, linking passengers to a flight instance.
public class Booking {
    private final String bookingId;
    private final FlightInstance flightInstance; // Association.
    private final List<Passenger> passengers; // Can have multiple passengers in one booking.
    private BookingStatus status;
    private final Payment payment; // Composition: A Booking has a Payment.

    public Booking(FlightInstance flightInstance, List<Passenger> passengers) {}

    // Confirms the booking after successful payment.
    public boolean confirmBooking() {}
    public boolean cancelBooking() {}
}

// Represents a physical aircraft.
public class Aircraft {
    private final String tailNumber;
    private final String model;
    private final List<Seat> seats; // Composition: Aircraft is composed of Seats.

    public Aircraft(String tailNumber, String model, int numEconomy, int numBusiness) {}
}

// Represents a single seat on an aircraft.
public class Seat {
    private final String seatNumber; // e.g., "24A"
    private final SeatClass seatClass;
    private boolean isBooked;

    public Seat(String seatNumber, SeatClass seatClass) {}

    // Methods to book and release the seat.
    public boolean bookSeat() {}
    public void releaseSeat() {}
}

// Interface for different payment strategies. Demonstrates Strategy Pattern.
public interface PaymentStrategy {
    boolean pay(double amount);
}

// Concrete payment strategies.
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String cvv;
    @Override public boolean pay(double amount) {}
}
public class PayPalPayment implements PaymentStrategy {
    private String email;
    @Override public boolean pay(double amount) {}
}

// The payment associated with a booking.
public class Payment {
    private final double amount;
    private PaymentStatus status;
    private final LocalDateTime paymentDate;
    // Uses the strategy pattern for flexibility.
    private final PaymentStrategy paymentStrategy;

    public Payment(double amount, PaymentStrategy strategy) {}

    public boolean processPayment() {}
}

// Other essential classes and enums.
public class Airport { private String airportCode; private String name; }
public enum SeatClass { ECONOMY, BUSINESS, FIRST_CLASS; }
public enum BookingStatus { PENDING, CONFIRMED, CANCELLED, CHECKED_IN; }
public enum PaymentStatus { PENDING, COMPLETED, FAILED, REFUNDED; }

```

### OOPS Concepts Applied

  * **Complex Associations**: The design models many-to-many and one-to-many relationships (e.g., `Flight` has many `FlightInstance`s).
  * **Composition**: `Booking` has a `Payment`, `Aircraft` is composed of `Seats`.
  * **Strategy Pattern**: `PaymentStrategy` interface allows for different payment methods to be used interchangeably without changing the `Payment` class.
  * **Rich Domain Model**: Classes like `Flight`, `FlightInstance`, `Airport` accurately reflect real-world concepts.
  * **Encapsulation with Getters/Setters**: State is protected (e.g., `isBooked` on a `Seat`).
  * **Cohesion & Coupling**: High cohesion in classes (`Aircraft` only deals with aircraft-specifics) and low coupling (a `Booking` doesn't need to know credit card details, only that a `PaymentStrategy` exists).

-----

## 5\. Shopping Cart System

This design models a standard e-commerce flow, separating the concerns of products, the shopping cart, and the final order.

### Core Classes & Design

```java
// Represents a user, can be a Guest or a Registered Member.
// Abstract class to define common user behavior.
public abstract class User {
    private String sessionId; // For guests or logged-in users.
    private ShoppingCart cart; // Each user has a shopping cart.

    public User() {
        this.cart = new ShoppingCart(); // Composition: User manages the lifecycle of their cart.
    }

    public void addItemToCart(Product product, int quantity) {}
    public void removeItemFromCart(Product product) {}
    public Order checkout() {}
}

// A registered user with an account.
public class Member extends User {
    private String userId;
    private String password;
    private List<Address> addresses;
    private List<Order> orderHistory;

    // Overriding the checkout method to associate order with the member's history.
    // Demonstrates Method Overriding and Covariant Return Types if Order subtype was used.
    @Override
    public Order checkout() {}
}

// A temporary user.
public class Guest extends User {
    // No extra fields needed, inherits all cart functionality.
}

// Represents the shopping cart.
public class ShoppingCart {
    // Map is optimal for O(1) access, update, and removal of items.
    private final Map<String, CartItem> items; // Key: ProductID
    private double total;

    public ShoppingCart() {}

    public void addItem(Product product, int quantity) {}
    public void updateItemQuantity(Product product, int quantity) {}
    public void removeItem(Product product) {}
    public void clearCart() {}
}

// Represents a single item within the shopping cart.
public class CartItem {
    private final Product product; // Association to the product.
    private int quantity;
    private double price; // Price at the time of adding to cart.

    public CartItem(Product product, int quantity) {}
    public void updateQuantity(int quantity) {}
}

// Represents a product in the catalog.
public class Product {
    private final String productId;
    private String name;
    private String description;
    private double price;
    private ProductCategory category;
    private int stockQuantity;
}

// Represents a confirmed order after checkout.
public class Order {
    private final String orderId;
    private final String userId; // Can be a guest session ID or member ID.
    private final List<OrderItem> orderItems;
    private final LocalDateTime orderDate;
    private OrderStatus status;
    private final Address shippingAddress;
    private final double orderTotal;

    // Constructor creates an Order from a ShoppingCart.
    public Order(User user, ShoppingCart cart, Address shippingAddress) {}
}

// An item within a confirmed order. Immutable.
public class OrderItem {
    private final String productId;
    private final int quantity;
    private final double priceAtPurchase; // Price is locked in at the time of order.
}

// Other necessary classes & enums.
public class Address { /* street, city, zip, country */ }
public class ProductCategory { /* id, name, description */ }
public enum OrderStatus { PENDING_PAYMENT, PLACED, SHIPPED, DELIVERED, CANCELLED, RETURNED; }
```

### OOPS Concepts Applied

  * **Inheritance**: `Member` and `Guest` extend the `abstract` `User` class.
  * **Method Overriding**: `Member` could override `checkout()` to add order history tracking.
  * **Composition**: A `User` is composed of a `ShoppingCart`.
  * **Aggregation/Association**: A `CartItem` has an association with a `Product`. An `Order` contains a list of `OrderItem`s.
  * **Data Structures**: Using `Map` in `ShoppingCart` for efficient item management is a key design choice.
  * **Immutability**: `OrderItem` is designed to be immutable once created, ensuring the integrity of past orders. `Product` ID is `final`.
  * **Separation of Concerns**: The design clearly separates the temporary `ShoppingCart` from the permanent `Order`. `CartItem` holds a reference to `Product` but also stores the price, insulating the cart from price changes in the main catalog.






Here are the comprehensive, interview-ready LLDs for the next set of systems, following the same format and high standards.

-----

## 6\. Online Stock Brokerage System

This design handles the complexity of different order types, real-time data, and user portfolio management. It uses the **Strategy Pattern** for order execution logic.

### Core Classes & Design

```java
// Represents a user account, which holds portfolios and cash balances.
// Abstract class to support different account types like Cash or Margin accounts.
public abstract class Account {
    private final String accountId;
    private final User owner;
    private double cashBalance;
    private final List<Portfolio> portfolios; // An account can have multiple portfolios.

    public Account(User owner) {}

    // Abstract method to be implemented by subclasses, defines how an order is placed.
    public abstract boolean placeOrder(Order order);
    
    // Method to deposit or withdraw cash.
    public void updateCashBalance(double amount) {}
}

// Concrete implementation of a standard cash account.
public class CashAccount extends Account {
    public CashAccount(User owner) { super(owner); }

    // Logic for placing an order from a cash account (checks available cash).
    @Override public boolean placeOrder(Order order) {}
}

// Manages a collection of stock positions for a user.
public class Portfolio {
    private final String portfolioId;
    private final Map<String, Position> positions; // Map<stockTicker, Position> for O(1) lookup.
    
    public Portfolio() {}
    
    // Updates a position, e.g., after a buy or sell transaction.
    public void updatePosition(String stockTicker, int quantity, double price) {}
}

// Represents a holding of a specific stock.
public class Position {
    private final Stock stock;
    private int quantity;
    private double averageCostPrice;
}

// Represents a single stock.
public class Stock {
    private final String tickerSymbol; // e.g., "GOOGL"
    private String companyName;
    private double currentPrice; // This would be updated by a real-time service.
}

// Main Order class. It uses a strategy for execution.
public class Order {
    private final String orderId;
    private final User user;
    private final Stock stock;
    private final OrderType type; // BUY or SELL
    private final int quantity;
    private OrderStatus status;
    private final LocalDateTime timestamp;
    // Composition: Each order has an execution strategy.
    private final OrderExecutionStrategy executionStrategy;

    public Order(User user, Stock stock, OrderType type, int quantity, OrderExecutionStrategy strategy) {}
    
    // Delegates the execution to its strategy object.
    public boolean execute(Exchange exchange) {}
}

// Strategy Pattern: Interface for different order execution logics.
public interface OrderExecutionStrategy {
    boolean execute(Order order, Exchange exchange);
}

// Concrete Strategy for a Market Order (executes at current market price).
public class MarketOrderStrategy implements OrderExecutionStrategy {
    @Override public boolean execute(Order order, Exchange exchange) {}
}

// Concrete Strategy for a Limit Order (executes at a specific price or better).
public class LimitOrderStrategy implements OrderExecutionStrategy {
    private final double limitPrice;
    public LimitOrderStrategy(double limitPrice) {}
    @Override public boolean execute(Order order, Exchange exchange) {}
}

// Singleton class representing the Stock Exchange where orders are matched.
public class Exchange {
    private static Exchange instance;
    private final Map<String, Queue<Order>> orderBook; // Map<stockTicker, Queue<Order>>

    private Exchange() {}
    public static synchronized Exchange getInstance() {}

    // The core logic for matching buy and sell orders.
    public void matchOrders() {}
    public void placeOrder(Order order) {}
}

// Other essential classes and enums.
public class User { private String userId; private String name; }
public class Transaction { /* id, type, amount, date */ }
public enum OrderType { BUY, SELL; }
public enum OrderStatus { PENDING, EXECUTED, CANCELLED, PARTIALLY_FILLED; }
```

### OOPS Concepts Applied

  * **Strategy Pattern**: `OrderExecutionStrategy` allows for defining different order types (`Market`, `Limit`, `Stop Loss`) without changing the `Order` class.
  * **Singleton Pattern**: `Exchange` ensures a single, global marketplace for orders.
  * **Inheritance & Abstraction**: `Account` is an `abstract` class, allowing for different types like `CashAccount` or `MarginAccount` with distinct `placeOrder` logic.
  * **Composition**: `Order` is composed of an `OrderExecutionStrategy`.
  * **Data Structures**: `Map` for portfolios provides fast access to positions. `Queue` within the `Exchange`'s `orderBook` is ideal for FIFO processing of orders.

-----

## 7\. Digital Wallet Service (e.g., PayPal, Google Pay)

This design focuses on security, transaction management, and notifications using the **Observer Pattern**.

### Core Classes & Design

```java
// Represents the user's digital wallet. This is the central class.
public class Wallet {
    private final String walletId;
    private final User owner;
    private double balance;
    private List<FinancialInstrument> instruments; // Holds cards, bank accounts.
    private final List<Transaction> transactionHistory;
    // Observer Pattern: List of listeners to notify on state change.
    private final transient List<WalletListener> listeners; // 'transient' to exclude from serialization.

    public Wallet(User owner) {}

    // Adds money to the wallet from a linked instrument.
    public void addMoney(FinancialInstrument source, double amount) {}

    // Sends money to another wallet.
    public void sendMoney(Wallet recipient, double amount) {}
    
    // Pays a merchant.
    public void makePayment(double amount) {}

    // Observer Pattern methods.
    public void addListener(WalletListener listener) {}
    public void removeListener(WalletListener listener) {}
    private void notifyListeners(Transaction transaction) {}
}

// Interface for anything that can be used for a transaction (credit card, bank account).
public interface FinancialInstrument {
    boolean processTransaction(double amount);
    String getIdentifier();
}

// Concrete implementation for a bank account.
public class BankAccount implements FinancialInstrument {
    private String accountNumber;
    private String bankName;
    @Override public boolean processTransaction(double amount) {}
    @Override public String getIdentifier() {}
}

// Concrete implementation for a card.
public class Card implements FinancialInstrument {
    private String cardNumber;
    private LocalDate expiryDate;
    private CardType type;
    @Override public boolean processTransaction(double amount) {}
    @Override public String getIdentifier() {}
}

// Base class for transactions, demonstrating inheritance.
public abstract class Transaction {
    private final String transactionId;
    private final double amount;
    private final LocalDateTime timestamp;
    private TransactionStatus status;

    public Transaction(double amount) {}
    public abstract TransactionType getType();
}

// Concrete transaction types.
public class DebitTransaction extends Transaction { /* ... */ @Override public TransactionType getType() { return TransactionType.DEBIT; } }
public class CreditTransaction extends Transaction { /* ... */ @Override public TransactionType getType() { return TransactionType.CREDIT; } }

// Observer Pattern: Interface for entities that want to be notified of wallet activity.
public interface WalletListener {
    void onTransaction(Transaction transaction);
}

// Concrete Observer that sends a notification.
public class NotificationService implements WalletListener {
    private final User user;
    @Override public void onTransaction(Transaction transaction) {
        // Logic to send an email or push notification.
        System.out.println("Notifying " + user.getName() + " of transaction " + transaction.getTransactionId());
    }
}

// Other essential classes and enums.
public class User { private String userId; private String name; private String email; }
public enum TransactionStatus { PENDING, COMPLETED, FAILED, CANCELLED; }
public enum TransactionType { DEBIT, CREDIT; }
public enum CardType { CREDIT, DEBIT; }

```

### OOPS Concepts Applied

  * **Observer Pattern**: `WalletListener` and `NotificationService` provide a clean way to send notifications about wallet activity without coupling the `Wallet` class to the notification logic.
  * **Interface**: `FinancialInstrument` provides a contract for different payment sources, promoting polymorphism.
  * **Inheritance**: `DebitTransaction` and `CreditTransaction` inherit common properties from the `abstract Transaction` class.
  * **Encapsulation**: The wallet's balance can only be modified through controlled methods like `addMoney` or `sendMoney`.
  * **Association**: The `Wallet` is associated with a `User` and a list of `FinancialInstrument`s.

-----

## 8\. Restaurant Management System

This system uses the **Observer Pattern** to decouple the front-of-house (orders) from the back-of-house (kitchen).

### Core Classes & Design

```java
// Central class representing the restaurant.
public class Restaurant {
    private String name;
    private Address address;
    private List<Table> tables;
    private Menu menu;
    // Singleton pattern for the Kitchen Display System (KDS).
    private final KitchenDisplaySystem kds;

    public Restaurant() { this.kds = KitchenDisplaySystem.getInstance(); }
}

// Represents a dining table in the restaurant.
public class Table {
    private final int tableId;
    private final int capacity;
    private TableStatus status;
    private Order currentOrder; // A table can have one active order.

    public Table(int id, int capacity) {}
    
    // Assigns a new order to the table.
    public void placeOrder(Order order) {}
    
    // Frees the table after payment.
    public void freeTable() {}
}

// Represents the restaurant's menu.
public class Menu {
    // A map allows for categorizing items.
    private final Map<String, List<MenuItem>> sections; // e.g., "Appetizers", "Main Courses"

    public Menu() {}
    public void addMenuItem(String section, MenuItem item) {}
    public void removeMenuItem(MenuItem item) {}
}

// Abstract class for an item on the menu. Could use Composite for sub-categories.
public abstract class MenuItem {
    private final String itemId;
    private String name;
    private String description;
    private double price;
    
    public MenuItem(String id, String name, double price) {}
}

// Concrete menu item types.
public class FoodItem extends MenuItem { private List<String> allergens; }
public class BeverageItem extends MenuItem { private boolean isAlcoholic; }

// Represents an order placed by a customer.
public class Order {
    private final String orderId;
    private final Table table;
    private final List<OrderItem> items;
    private OrderStatus status;
    
    public Order(Table table) {}

    // Adding an item to the order.
    public void addItem(MenuItem menuItem, int quantity) {}

    // When the order is confirmed, it notifies the kitchen.
    public void confirmOrder() {
        KitchenDisplaySystem.getInstance().newOrderPlaced(this);
    }
}

// An item within an order.
public class OrderItem {
    private final MenuItem menuItem;
    private int quantity;
    private OrderItemStatus status; // e.g., PREPARING, READY, SERVED
}

// Observer Pattern: The Kitchen is an observer of new orders.
// Implemented as a Singleton.
public class KitchenDisplaySystem {
    private static KitchenDisplaySystem instance;
    private final Queue<Order> pendingOrders;

    private KitchenDisplaySystem() {}
    public static synchronized KitchenDisplaySystem getInstance() {}

    // This method is called by Order's confirmOrder() method (the "notification").
    public void newOrderPlaced(Order order) {
        pendingOrders.add(order);
    }

    // A chef would call this to get the next order to prepare.
    public Order getNextOrder() {}
}

// Other essential classes and enums.
public class Bill { private final Order order; private double totalAmount; }
public abstract class Employee { private String employeeId; private String name; }
public class Waiter extends Employee {}
public class Chef extends Employee {}
public enum TableStatus { AVAILABLE, OCCUPIED, RESERVED; }
public enum OrderStatus { PENDING, CONFIRMED, PREPARING, COMPLETED, PAID; }
public enum OrderItemStatus { WAITING, PREPARING, READY, SERVED; }
```

### OOPS Concepts Applied

  * **Observer Pattern**: `Order` acts as the "subject" and `KitchenDisplaySystem` as the "observer." When an order is confirmed, the kitchen is automatically notified without a direct, hard-coded link.
  * **Singleton Pattern**: `KitchenDisplaySystem` is a single resource for the entire restaurant.
  * **Inheritance & Abstraction**: `Employee` is an abstract base class for `Waiter` and `Chef`. `MenuItem` is abstract for `FoodItem` and `BeverageItem`.
  * **Aggregation**: An `Order` aggregates `OrderItem`s.
  * **High Cohesion**: Classes have specific roles. `Table` manages its status, `Order` manages its items, and `KitchenDisplaySystem` manages the cooking queue.

-----

## 9\. Course Registration System

This design mirrors the complexity of a university's operations, clearly distinguishing between a general course and its specific offerings each semester.

### Core Classes & Design

```java
// Represents a student in the system.
public class Student {
    private final String studentId;
    private String name;
    private final Map<String, CourseOffering> registeredCourses; // Courses for the current semester.
    private final List<Course> completedCourses; // Transcript.

    public Student(String studentId, String name) {}

    // Logic to register for a specific course offering.
    public boolean registerForCourse(CourseOffering offering) {}
    
    // Logic to drop a course.
    public boolean dropCourse(CourseOffering offering) {}
}

// Represents a general course in the university catalog (e.g., "CS101 Intro to Programming").
public class Course {
    private final String courseCode; // e.g., "CS101"
    private String title;
    private String description;
    private int credits;
    private final Department department;
    private final List<Course> prerequisites; // A course can have prerequisites.
}

// Represents a specific instance of a Course offered in a particular semester.
public class CourseOffering {
    private final String offeringId;
    private final Course course; // Association to the general Course.
    private final String semester; // e.g., "Fall 2025"
    private Professor instructor;
    private int capacity;
    private final List<Student> registeredStudents; // List of enrolled students.
    private String schedule; // e.g., "Mon/Wed/Fri 10-11AM"

    public CourseOffering(Course course, String semester, Professor instructor, int capacity) {}

    // Adds a student if capacity is not full and prerequisites are met.
    public boolean addStudent(Student student) {}
    
    // Removes a student.
    public void removeStudent(Student student) {}
}

// Represents a registration action, linking a student and a course offering.
public class Registration {
    private final String registrationId;
    private final Student student;
    private final CourseOffering courseOffering;
    private final LocalDateTime registrationTime;
    private RegistrationStatus status;
}

// Represents a professor.
public class Professor {
    private final String employeeId;
    private String name;
    private Department department;
    private final List<CourseOffering> coursesTaught; // Courses they are teaching this semester.
}

// Represents a university department.
public class Department {
    private final String departmentId;
    private String name;
    private final List<Course> coursesOffered;
    private final List<Professor> faculty;
}

// Singleton class to manage the overall course catalog and offerings.
public class CourseCatalog {
    private static CourseCatalog instance;
    private final Map<String, Course> allCourses; // Master list of all courses.
    private final Map<String, List<CourseOffering>> offeringsBySemester;

    private CourseCatalog() {}
    public static synchronized CourseCatalog getInstance() {}
    
    // Methods to search for courses and offerings.
    public List<CourseOffering> findOfferings(String courseCode, String semester) {}
}

// Enums for status management.
public enum RegistrationStatus { REGISTERED, WAITLISTED, DROPPED; }
```

### OOPS Concepts Applied

  * **Clear Abstraction**: The distinction between `Course` (the abstract concept) and `CourseOffering` (the concrete instance) is a critical design choice that mirrors reality and avoids data redundancy.
  * **Singleton Pattern**: `CourseCatalog` acts as a single source of truth for all courses and offerings in the university.
  * **Association**: The model is rich with associations: `Student` to `CourseOffering`, `CourseOffering` to `Course` and `Professor`, `Course` to `Department`, etc.
  * **Encapsulation**: A `CourseOffering` encapsulates its own registration logic (checking capacity, adding students), hiding it from the `Student` class.
  * **Data Structures**: Using `Map` for `registeredCourses` provides efficient access. `Map`s in the `CourseCatalog` allow for fast lookups.

-----

## 10\. Online Bookstore

This design is a specialized version of a general e-commerce system, with classes tailored to the domain of books.

### Core Classes & Design

```java
// Represents the main bookstore, acting as a facade for the system.
public class Bookstore {
    private final Inventory inventory;
    
    public Bookstore() {
        this.inventory = new Inventory();
    }
    
    // Search functionality. Can use Method Overloading for different search criteria.
    public List<Book> searchByTitle(String title) {}
    public List<Book> searchByAuthor(Author author) {}
    public List<Book> searchByISBN(String isbn) {}
}

// Manages the stock of all books.
public class Inventory {
    // A map provides efficient O(1) lookup by ISBN.
    private final Map<String, Book> booksByIsbn; 

    public Inventory() {}
    
    public void addBook(Book book, int quantity) {}
    public void updateStock(String isbn, int quantity) {}
    public int getStock(String isbn) {}
}

// The core product class, representing a book.
public class Book {
    private final String ISBN; // International Standard Book Number, the unique identifier.
    private String title;
    private String description;
    private final List<Author> authors; // A book can have multiple authors.
    private Publisher publisher;
    private double price;
    private final List<Review> reviews;

    public Book(String isbn, String title) {}
    public void addReview(Review review) {}
}

// Represents the author of a book.
public class Author {
    private final String authorId;
    private String name;
    private String biography;
    // Overriding equals() and hashCode() is crucial for using Author in Maps or Sets.
    @Override public boolean equals(Object o) {}
    @Override public int hashCode() {}
}

// Represents the publisher of a book.
public class Publisher {
    private final String publisherId;
    private String name;
}

// A user's review of a book.
public class Review {
    private final String reviewId;
    private final User user;
    private final int rating; // e.g., 1-5 stars.
    private String comment;
    private final LocalDateTime timestamp;
}

// The following classes are similar to the general Shopping Cart System.
// Using an abstract User class for Guests and Members.
public abstract class User {
    private String sessionId;
    private ShoppingCart cart;

    public User() { this.cart = new ShoppingCart(this); }
    public void addBookToCart(Book book, int quantity) {}
    public Order checkout(Address shippingAddress) {}
}

public class Member extends User { /* ... orderHistory, savedAddresses ... */ }
public class Guest extends User { /* ... */ }

// The shopping cart.
public class ShoppingCart {
    private final User owner;
    private final Map<String, CartItem> items; // Map<ISBN, CartItem>
    
    public void addItem(Book book, int quantity) {}
}

// An item in the cart.
public class CartItem {
    private final Book book;
    private int quantity;
}

// A confirmed order.
public class Order {
    private final String orderId;
    private final List<OrderItem> items;
    private OrderStatus status;
}

// An item within a confirmed order. It's immutable.
public final class OrderItem {
    private final Book book;
    private final int quantity;
    private final double priceAtPurchase;
}

// Enums for status.
public enum OrderStatus { PENDING, SHIPPED, DELIVERED, CANCELLED; }
```

### OOPS Concepts Applied

  * **Domain-Specific Modeling**: The classes (`Book`, `Author`, `Publisher`, `ISBN`) are highly specific to the bookstore domain.
  * **Method Overloading**: `Bookstore`'s `search()` methods demonstrate how multiple methods can have the same name with different parameters for different search types.
  * **Aggregation**: A `Book` aggregates `Review`s. The reviews are part of the book's data but can also exist independently.
  * **`final` keyword**: `OrderItem` is a `final` class, meaning it cannot be subclassed, ensuring its immutability. The `ISBN` is a `final` field.
  * **`equals()` and `hashCode()`**: Mentioning their override in `Author` is important for correct behavior when authors are used as keys in maps or elements in sets.
  * **High Cohesion / Low Coupling**: The `Inventory` class is solely responsible for stock management, decoupling it from the `Bookstore`'s search logic or the `User`'s cart.







Of course. Here are the comprehensive, interview-ready LLDs for a generic Social Media Platform, Facebook, and Cricinfo, designed to be optimal and complete.

-----

## 11\. Social Media Platform (Generic)

This design serves as a flexible foundation for any social media service like Twitter, Instagram, or Facebook. It uses inheritance for content types and the **Factory Pattern** for content creation.

### Core Classes & Design

```java
// Represents a user of the platform.
public class User {
    private final String userId;
    private String username;
    private Profile profile;
    private final Map<String, User> followers; // Map<userId, User>
    private final Map<String, User> following; // Map<userId, User>

    public User(String username) {}

    // User actions. These methods would interact with other services.
    public void follow(User user) {}
    public void unfollow(User user) {}
    // Uses the PostFactory to create a post, decoupling User from concrete Post types.
    public Post createPost(PostType type, String content) {}
    public void commentOnPost(Post post, String text) {}
    public void likePost(Post post) {}
}

// User's profile details.
public class Profile {
    private String displayName;
    private String bio;
    private byte[] profilePicture; // byte array for image data.
}

// Abstract class for all post types. Core of content model.
public abstract class Post {
    private final String postId;
    private final User author;
    private final LocalDateTime timestamp;
    private final List<Comment> comments;
    private int likeCount;

    public Post(User author) {}

    // Common actions for all posts.
    public void addComment(Comment comment) {}
    public void addLike() {}
    public abstract void display(); // Abstract method for polymorphic display.
}

// Concrete post types inheriting from Post.
public class TextPost extends Post {
    private String textContent;
    @Override public void display() { /* logic to display text */ }
}
public class ImagePost extends Post {
    private byte[] imageData;
    private String caption;
    @Override public void display() { /* logic to display image and caption */ }
}
public class VideoPost extends Post {
    private byte[] videoData;
    private String caption;
    @Override public void display() { /* logic to display video and caption */ }
}

// Factory Pattern: A class dedicated to creating different types of posts.
public class PostFactory {
    // Static factory method.
    public static Post createPost(PostType type, User author, String content) {
        // ... logic to instantiate the correct Post subclass ...
    }
}

// Represents a comment on a post.
public class Comment {
    private final String commentId;
    private final User author;
    private final String text;
    private final LocalDateTime timestamp;
}

// Service responsible for generating a user's news feed.
public class NewsFeedService {
    // The algorithm for feed generation is complex and encapsulated here.
    public List<Post> getFeed(User user) {
        // Logic to fetch posts from followed users, rank, and sort them.
    }
}

// Observer Pattern: NotificationService observes actions and notifies users.
public class NotificationService {
    // Sends a notification to a user.
    public void sendNotification(User user, NotificationType type, String message) {}
}

// Other essential classes and enums.
public enum PostType { TEXT, IMAGE, VIDEO; }
public enum NotificationType { LIKE, COMMENT, FOLLOW; }
```

### OOPS Concepts Applied

  * **Inheritance & Abstraction**: `Post` is an `abstract` class providing a template for concrete types like `TextPost`, `ImagePost`, etc.
  * **Polymorphism**: The `display()` method can be called on any `Post` object, and the correct implementation for its type will be executed.
  * **Factory Pattern**: `PostFactory` decouples the `User` class (the client) from the process of creating specific `Post` objects.
  * **Encapsulation**: The complex logic for generating a news feed is hidden within the `NewsFeedService`.
  * **Data Structures**: Using `Map` for followers/following provides efficient O(1) lookups, crucial for social graphs.

-----

## 12\. Facebook (A Specific Social Media Platform)

This design extends the generic social media model with Facebook-specific features like Groups, Pages, Events, and complex Reactions.

### Core Classes & Design

```java
// User class, extended with Facebook-specific connections.
public class Member extends User { // Inherits from a base User class
    // In addition to followers/following (friends), a Member has other connections.
    private final Map<String, Group> groups; // Groups the member has joined.
    private final Map<String, Page> likedPages; // Pages the member follows.
    private final Map<String, Event> events; // Events the member is attending/interested in.
    
    // Facebook uses a friend request model.
    public void sendFriendRequest(Member recipient) {}
    public void acceptFriendRequest(FriendRequest request) {}
}

// The concept of a Post is extended to include reactions.
public abstract class Post { // Same as generic, but with more features
    private final String postId;
    private final User author;
    // A map is better than a simple count to track who reacted and how.
    private final Map<ReactionType, List<Member>> reactions; 
    
    public void addReaction(Member member, ReactionType type) {}
}

// A Group where members can share posts.
public class Group {
    private final String groupId;
    private String name;
    private final Map<String, Member> members;
    private final List<Post> posts;
    private GroupPrivacy privacy;
    
    public void addMember(Member member) {}
    public void createPost(Member author, String content) {}
}

// A Page for entities like businesses or celebrities.
public class Page {
    private final String pageId;
    private String name;
    private final List<Member> followers;
    private final List<Post> posts;
    
    public void addFollower(Member member) {}
}

// A planned social gathering.
public class Event {
    private final String eventId;
    private String title;
    private LocalDateTime startTime;
    private LocalDateTime endTime;
    private final Map<Member, EventRSVP> attendees;
    
    public void rsvp(Member member, EventRSVP status) {}
}

// Private messaging functionality.
public class Message {
    private final String messageId;
    private final Member sender;
    private final Member receiver;
    private String content;
    private final LocalDateTime timestamp;
}

// Represents a pending friend request, an association class.
public class FriendRequest {
    private final Member fromUser;
    private final Member toUser;
    private RequestStatus status;
}

// Enums to manage specific states and types.
public enum ReactionType { LIKE, LOVE, HAHA, WOW, SAD, ANGRY; }
public enum GroupPrivacy { PUBLIC, PRIVATE, SECRET; }
public enum RequestStatus { PENDING, ACCEPTED, REJECTED; }
public enum EventRSVP { ATTENDING, INTERESTED, NOT_ATTENDING; }
```

### OOPS Concepts Applied

  * **Specialization (Inheritance)**: `Member` is a specialized type of `User`, inheriting base functionality and adding its own.
  * **Complex Associations**: The design models the rich, graph-like relationships between `Member`, `Group`, `Page`, and `Event`.
  * **Data-Rich Enums**: Using enums like `ReactionType` and `GroupPrivacy` makes the code type-safe and self-documenting.
  * **High Cohesion**: Classes like `Group` and `Event` manage their own members and posts, encapsulating their specific logic and state.
  * **Low Coupling**: A `Member` doesn't need to know the internal logic of how an `Event` manages RSVPs; it just calls the `rsvp()` method.

-----

## 13\. Cricinfo (Sports Media Platform)

This design is highly domain-specific, focusing on the hierarchical structure of a cricket match. It uses composition extensively and an **Observer Pattern** for live updates.

### Core Classes & Design

```java
// The central class representing a cricket match.
public class Match {
    private final String matchId;
    private final Team teamA;
    private final Team teamB;
    private final Venue venue;
    private final LocalDateTime startTime;
    private MatchType type;
    private MatchStatus status;
    private Team tossWinner;
    // Composition: A match is composed of innings.
    private final List<Innings> innings;
    // Observer Pattern: List of viewers to notify on every ball.
    private final transient List<LiveScoreViewer> viewers;

    public Match(Team teamA, Team teamB, MatchType type, Venue venue) {}

    // When a ball is bowled, it updates the state and notifies all viewers.
    public void addBall(Innings innings, Ball ball) {
        // ... update scores ...
        this.notifyViewers(ball);
    }

    public void addViewer(LiveScoreViewer viewer) {}
    private void notifyViewers(Ball ball) {}
}

// Represents one team's turn to bat.
public class Innings {
    private final int inningsNumber; // 1st or 2nd innings
    private final Team battingTeam;
    private final Team bowlingTeam;
    // Composition: An innings is composed of overs.
    private final List<Over> overs;
    private int totalScore;
    private int wicketsLost;
}

// Represents an over, which consists of 6 balls.
public class Over {
    private final int overNumber;
    private final Player bowler;
    // Composition: An over is composed of balls.
    private final List<Ball> balls;
}

// Represents a single ball bowled. This is an immutable event object.
public final class Ball {
    private final int ballNumber;
    private final Player batsman;
    private final Player bowler;
    private final int runsScored;
    private final Wicket wicket; // Can be null if no wicket fell.
    private final String commentary; // Text commentary for this ball.

    public Ball(Player batsman, Player bowler, String commentary) {}
}

// Represents a wicket event.
public class Wicket {
    private final WicketType type;
    private final Player batsmanOut;
    private final Player dismissedBy; // Bowler or fielder
}

// Represents a player with their profile/statistics.
public class Player {
    private final String playerId;
    private String name;
    private final PlayerProfile profile;
    private PlayerRole role; // Batsman, Bowler, All-rounder
}

// Aggregates statistics for a player.
public class PlayerProfile {
    private int matchesPlayed;
    private int totalRuns;
    private int totalWickets;
    private double battingAverage;
    // ... many more stats
}

// Represents a team.
public class Team {
    private final String teamId;
    private String name;
    private final List<Player> players; // The squad for the team.
}

// Observer Pattern: Interface for anything that wants to receive live updates.
public interface LiveScoreViewer {
    void update(Ball ball);
}

// A concrete viewer, e.g., a UI component or a notification service.
public class WebClientViewer implements LiveScoreViewer {
    private final String sessionId;
    @Override public void update(Ball ball) {
        // Logic to push the update to a specific web client.
    }
}

// Other essential classes and enums.
public class Venue { /* name, city, country */ }
public class Tournament { private List<Match> matches; } // Composite Pattern can be used here.
public enum MatchType { TEST, ODI, T20; }
public enum MatchStatus { UPCOMING, LIVE, FINISHED; }
public enum WicketType { BOWLED, CAUGHT, LBW, RUN_OUT; }
public enum PlayerRole { BATSMAN, BOWLER, WICKET_KEEPER, ALL_ROUNDER; }

```

### OOPS Concepts Applied

  * **Composition**: The entire match structure is a deep composition: `Match` has `Innings`, which has `Overs`, which has `Balls`. This accurately models the real-world domain.
  * **Observer Pattern**: `LiveScoreViewer` provides a clean, decoupled way for clients (like web pages or apps) to receive real-time updates for every ball bowled without the `Match` class needing to know about specific clients.
  * **Immutability**: The `Ball` class is `final` and its fields are `final`, making it an immutable object. This is excellent for representing historical events that cannot be changed.
  * **Aggregation**: `Player` aggregates `PlayerProfile`. The profile is part of the player but is conceptually a separate collection of data.
  * **Domain-Specific Enums**: The use of enums like `MatchType` and `WicketType` makes the system robust and easy to understand.
