### Design a Library Management System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract classes for items (e.g., Book, Magazine) to hide implementation details. 
 - **Encapsulation**: Private variables with public getters/setters. 
 - **Inheritance**: Book and Magazine extend Item. 
 - **Polymorphism**: Method overriding in subclasses (e.g., getDetails()). 
 - **Interface**: Borrowable interface for items that can be borrowed. 
 - **Composition**: Library has-a list of Items and Users. 
 - **Singleton**: LibraryManager as singleton for central management. 
 - **Access Modifiers**: Public for APIs, private for internal state. 
 - **Static Members**: Static counter for unique IDs. 
 - **Final Keyword**: Final variables for constants like MAX_BORROW_LIMIT. 
 - **SOLID Principles**: Single Responsibility (e.g., User handles user ops), Open-Closed (extendable items), Interface Segregation (small interfaces). 
 - **Observer Pattern**: Added for notifying users on item availability (enhancement for event-driven updates).
 - **Exception Handling**: Added custom exceptions for borrow/return operations (enhancement for robustness).
 - **Persistence**: Added Serializable interface to key classes for data saving (enhancement for real-world use).

 #### Classes and Structure: 

 ```java
 import java.io.Serializable; // Added for persistence enhancement
 import java.util.*;
 import java.util.concurrent.atomic.AtomicInteger; // Added for thread-safe counter

 // Custom Exception: Added for borrow limit exceeded (enhancement)
 class BorrowLimitExceededException extends Exception { 
     public BorrowLimitExceededException(String message) { super(message); } 
 }

 // Custom Exception: Added for item not available (enhancement)
 class ItemNotAvailableException extends Exception { 
     public ItemNotAvailableException(String message) { super(message); } 
 }

 // Custom Listener Interface: Replaced deprecated Observer (enhancement)
 interface ItemListener { 
     void onAvailabilityChange(Item item); 
 }

 // Abstract Class: Represents any library item, abstraction for common behavior 
 abstract class Item implements Serializable { // Added Serializable for persistence
     private String id;  // Encapsulated unique ID 
     private String title; 
     private String author; 
     private boolean isAvailable;  // Private state 
     protected static AtomicInteger itemCounter = new AtomicInteger(0);  // Static atomic for thread-safe counter 
     private java.util.Date dateAdded; // Added for tracking when item was added (enhancement)
     private List<ItemListener> listeners = new ArrayList<>(); // Added custom listeners

     // Constructor: Overloaded for flexibility 
     public Item(String title, String author) { 
         this.id = "ITEM" + itemCounter.incrementAndGet();  // Auto-generate ID thread-safely
         this.title = title; 
         this.author = author; 
         this.isAvailable = true; 
         this.dateAdded = new java.util.Date(); // Added initialization
     } 

     // Overloaded Constructor 
     public Item(String id, String title, String author, boolean isAvailable) { 
         this.id = id; 
         this.title = title; 
         this.author = author; 
         this.isAvailable = isAvailable; 
         this.dateAdded = new java.util.Date(); // Added initialization
     } 

     // Abstract Method: To be overridden (polymorphism) 
     public abstract String getDetails(); 

     // Getter/Setter: Encapsulation 
     public String getId() { return id; } 
     public void setAvailable(boolean available) { 
         this.isAvailable = available; 
         notifyListeners(); // Added to notify custom listeners on availability change (enhancement)
     } 
     public boolean isAvailable() { return isAvailable; } 
     public java.util.Date getDateAdded() { return dateAdded; } // Added getter (enhancement)

     // Final Method: Cannot be overridden 
     public final void markAsBorrowed() { isAvailable = false; } 

     public void addListener(ItemListener listener) { listeners.add(listener); } // Added
     public void removeListener(ItemListener listener) { listeners.remove(listener); } // Added
     protected void notifyListeners() { 
         for (ItemListener listener : listeners) { 
             listener.onAvailabilityChange(this); 
         } 
     } // Added
 } 

 // Inheritance: Book extends Item 
 class Book extends Item implements Borrowable, Serializable {  // Added Serializable for persistence
     private int pages; 
     private String isbn; // Added for unique book identification (enhancement)

     public Book(String title, String author, int pages) { 
         super(title, author);  // super keyword for parent constructor 
         this.pages = pages; 
         this.isbn = "ISBN-" + UUID.randomUUID().toString(); // Changed to UUID for uniqueness
     } 

     // Method Overriding: Runtime polymorphism 
     @Override 
     public String getDetails() { 
         return "Book: " + getTitle() + ", Pages: " + pages + ", ISBN: " + isbn; // Enhanced with ISBN
     } 

     // From Interface 
     @Override 
     public void borrow(User user) throws ItemNotAvailableException { // Added exception handling
         if (!isAvailable()) throw new ItemNotAvailableException("Book not available"); 
         // Logic to borrow (not implemented) 
     } 
     public String getIsbn() { return isbn; } // Added getter (enhancement)
 } 

 // Similar for Magazine 
 class Magazine extends Item { 
     private String issueDate; 
     private int issueNumber; // Added for tracking magazine issues (enhancement)

     public Magazine(String title, String author, String issueDate) { 
         super(title, author); 
         this.issueDate = issueDate; 
         this.issueNumber = (int) (Math.random() * 100); // Added initialization
     } 

     @Override 
     public String getDetails() { 
         return "Magazine: " + getTitle() + ", Issue: " + issueDate + ", Number: " + issueNumber; // Enhanced with issue number
     } 
     public int getIssueNumber() { return issueNumber; } // Added getter
 } 

 // Interface: For borrowable items 
 interface Borrowable { 
     void borrow(User user) throws ItemNotAvailableException; // Enhanced with exception
     void returnItem(); 
 } 

 // Class: User 
 class User implements ItemListener, Serializable { // Added ItemListener for notifications, Serializable for persistence
     private String userId; 
     private String name; 
     private List<Item> borrowedItems;  // Composition: User has-a list of Items 
     private final int MAX_BORROW_LIMIT = 5;  // Final constant 

     public User(String name) { 
         this.userId = "USR" + UUID.randomUUID().toString();  // Changed to UUID for uniqueness
         this.name = name; 
         this.borrowedItems = new ArrayList<>(); 
     } 

     // Method: Add borrowed item (association with Item) 
     public void borrowItem(Item item) throws BorrowLimitExceededException, ItemNotAvailableException { // Added exceptions (enhancement)
         if (borrowedItems.size() < MAX_BORROW_LIMIT && item.isAvailable()) { 
             borrowedItems.add(item); 
             item.markAsBorrowed(); 
         } else if (borrowedItems.size() >= MAX_BORROW_LIMIT) { 
             throw new BorrowLimitExceededException("Max borrow limit reached"); 
         } else { 
             throw new ItemNotAvailableException("Item not available"); 
         } 
     } 

     public void returnItem(Item item) { 
         borrowedItems.remove(item); 
         item.setAvailable(true); 
     } 

     // Getter 
     public List<Item> getBorrowedItems() { return Collections.unmodifiableList(borrowedItems); } // Changed to unmodifiable

     // Added for Listener pattern: Update on item availability
     @Override
     public void onAvailabilityChange(Item item) {
         // Notification logic when item becomes available (not implemented)
     }
 } 

 // Singleton Class: Central manager 
 class LibraryManager implements Serializable { // Added Serializable
     private static volatile LibraryManager instance;  // Volatile for thread-safety
     private Map<String, Item> items = new HashMap<>();  // Changed to Map for O(1) lookups
     private List<User> users; 
     private Map<String, List<User>> waitlist; // Added for item waitlist (enhancement for queueing borrows)

     private LibraryManager() {  // Private constructor for singleton 
         users = new ArrayList<>(); 
         waitlist = new HashMap<>(); // Added initialization
     } 

     public static LibraryManager getInstance() {  // Thread-safe double-checked locking
         if (instance == null) { 
             synchronized (LibraryManager.class) { 
                 if (instance == null) { 
                     instance = new LibraryManager(); 
                 } 
             } 
         } 
         return instance; 
     } 

     public void addItem(Item item) { 
         items.put(item.getId(), item); // Use map
     } 
     public void addUser(User user) { users.add(user); } 
     public Item findItemById(String id) { 
         return items.get(id); // O(1) lookup
     } 
     public void addToWaitlist(Item item, User user) { // Added method for waitlist (enhancement)
         waitlist.computeIfAbsent(item.getId(), k -> new ArrayList<>()).add(user);
     }
     // Other methods: search, overdue checks, etc. 
 } 
 ``` 

 #### How It Works: 
 - **Object Creation**: Create Users and Items via constructors. 
 - **Borrowing**: User calls borrowItem(), which checks limits and updates state (encapsulation). 
 - **Polymorphism**: List<Item> can hold Book or Magazine, call getDetails() dynamically. 
 - **Singleton**: Only one LibraryManager instance manages all. 
 - **Associations**: Loose coupling between User and Item. 
 - **Notifications**: Users can observe items for availability changes (added enhancement). 
 - **Notifications**: Waitlist: Users can be added to wait for unavailable items (added enhancement).

 --- 

 ### Design a Parking Lot 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Vehicle class. 
 - **Encapsulation**: Private spots in ParkingLot. 
 - **Inheritance**: Car, Bike extend Vehicle. 
 - **Polymorphism**: Overridden getSize() for different vehicles. 
 - **Composition**: ParkingLot has-a floors, floors have-a spots. 
 - **Singleton**: ParkingLot as singleton. 
 - **Access Modifiers**: Protected for subclass access. 
 - **Static Members**: Static fee rates. 
 - **Final Keyword**: Final spot IDs. 
 - **Inner Classes**: Spot as inner class for encapsulation. 
 - **SOLID**: Single Responsibility (Ticket handles billing), Liskov Substitution (vehicles interchangeable). 
 - **Strategy Pattern**: Added for different parking strategies (e.g., compact vs large spots) (enhancement for flexibility).
 - **Exception Handling**: Added custom exceptions for no spot available (enhancement).
 - **Logging**: Added simple logging interface for events (enhancement for auditing).

 #### Classes and Structure: 

 ```java
 // Enum: For vehicle types (immutable) 
 enum VehicleType { CAR, BIKE, TRUCK } 

 // Custom Exception: Added for no available spot (enhancement)
 class NoSpotAvailableException extends Exception { 
     public NoSpotAvailableException(String message) { super(message); } 
 }

 // Interface: Added for parking strategy (enhancement with strategy pattern)
 interface ParkingStrategy { 
     ParkingSpot findSpot(ParkingFloor floor, Vehicle vehicle); 
 }

 // Concrete Strategy: Added for nearest spot (enhancement)
 class NearestSpotStrategy implements ParkingStrategy { 
     @Override 
     public ParkingSpot findSpot(ParkingFloor floor, Vehicle vehicle) { 
         List<ParkingSpot> spots = floor.getSpots(); // Assume getter added
         for (ParkingSpot spot : spots) { 
             if (spot.isAvailable() && spot.getSize() >= vehicle.getSize()) { 
                 return spot; // Simple nearest as first available
             } 
         } 
         return null; 
     } 
 }

 // Abstract Class: Vehicle 
 abstract class Vehicle implements Serializable { // Added Serializable for persistence
     protected String licensePlate;  // Protected for subclass 
     private VehicleType type; 

     public Vehicle(String licensePlate, VehicleType type) { 
         this.licensePlate = licensePlate; 
         this.type = type; 
     } 

     // Abstract Method 
     public abstract int getSize();  // e.g., Car=1, Truck=2 spots 

     public String getLicensePlate() { return licensePlate; } 
     public VehicleType getType() { return type; } // Added getter (enhancement)
 } 

 // Inheritance 
 class Car extends Vehicle { 
     public Car(String licensePlate) { 
         super(licensePlate, VehicleType.CAR); 
     } 

     @Override 
     public int getSize() { return 1; }  // Overriding 
 } 

 class Bike extends Vehicle { 
     public Bike(String licensePlate) { 
         super(licensePlate, VehicleType.BIKE); 
     } 

     @Override 
     public int getSize() { return 1; } 
 } 

 // Class: ParkingSpot (Inner class possible, but separate for clarity) 
 class ParkingSpot implements Serializable { // Added Serializable
     private final String spotId;  // Final ID 
     private boolean isAvailable; 
     private Vehicle parkedVehicle;  // Association 
     private int size; // Added for spot size (enhancement to match vehicle sizes)

     public ParkingSpot(String spotId, int size) { // Added size parameter
         this.spotId = spotId; 
         this.isAvailable = true; 
         this.size = size; 
     } 

     public void park(Vehicle vehicle) throws NoSpotAvailableException { // Added exception
         if (!isAvailable || vehicle.getSize() > size) throw new NoSpotAvailableException("Spot not suitable"); 
         parkedVehicle = vehicle; 
         isAvailable = false; 
     } 

     public Vehicle unpark() { 
         Vehicle v = parkedVehicle; 
         parkedVehicle = null; 
         isAvailable = true; 
         return v; 
     } 

     public boolean isAvailable() { return isAvailable; } 
     public int getSize() { return size; } // Added getter
 } 

 // Class: ParkingFloor 
 class ParkingFloor implements Serializable { // Added Serializable
     private List<ParkingSpot> spots = new ArrayList<>();  // Composition 
     private int floorNumber; 
     private ParkingStrategy strategy; // Added for strategy pattern (enhancement)

     public ParkingFloor(int floorNumber, int numSpots, ParkingStrategy strategy) { // Injected strategy
         this.floorNumber = floorNumber; 
         this.strategy = strategy; 
         for (int i = 1; i <= numSpots; i++) { 
             spots.add(new ParkingSpot("F" + floorNumber + "-S" + i, i % 2 == 0 ? 2 : 1)); // Varied sizes
         } 
     } 

     public ParkingSpot findAvailableSpot(Vehicle vehicle) { 
         return strategy.findSpot(this, vehicle); // Enhanced to use strategy
     } 
     public void setStrategy(ParkingStrategy strategy) { this.strategy = strategy; } // Added setter for flexibility
     public List<ParkingSpot> getSpots() { return Collections.unmodifiableList(spots); } // Added unmodifiable getter
 } 

 // Class: Ticket 
 class Ticket implements Serializable { // Added Serializable
     private String ticketId; 
     private ParkingSpot spot; 
     private long entryTime; 
     private static double HOURLY_RATE = 5.0;  // Static 

     public Ticket(ParkingSpot spot) { 
         this.ticketId = "TKT" + System.currentTimeMillis(); 
         this.spot = spot; 
         this.entryTime = System.currentTimeMillis(); 
     } 

     public double calculateFee() { 
         long duration = System.currentTimeMillis() - entryTime; 
         return (duration / 3600000.0) * HOURLY_RATE;  // Hours * rate 
     } 
     public String getTicketId() { return ticketId; } // Added getter (enhancement)
 } 

 // Interface: Added for logging events (enhancement)
 interface Logger { 
     void log(String message); 
 }

 // Concrete Logger: Added simple console logger
 class ConsoleLogger implements Logger { 
     @Override 
     public void log(String message) { 
         // Console log (not implemented) 
     } 
 }

 // Singleton: ParkingLot 
 class ParkingLot implements Serializable { // Added Serializable
     private static volatile ParkingLot instance; // Volatile for safety
     private List<ParkingFloor> floors;  // Composition 
     private Map<String, Ticket> activeTickets;  // For tracking 
     private Logger logger; // Added for logging (enhancement)

     private ParkingLot(int numFloors, int spotsPerFloor) {  // Private constructor 
         floors = new ArrayList<>(); 
         for (int i = 1; i <= numFloors; i++) { 
             floors.add(new ParkingFloor(i, spotsPerFloor, new NearestSpotStrategy())); // Injected strategy
         } 
         activeTickets = new HashMap<>(); 
         logger = new ConsoleLogger(); // Added initialization
     } 

     public static ParkingLot getInstance(int numFloors, int spotsPerFloor) { // Thread-safe
         if (instance == null) { 
             synchronized (ParkingLot.class) { 
                 if (instance == null) { 
                     instance = new ParkingLot(numFloors, spotsPerFloor); 
                 } 
             } 
         } 
         return instance; 
     } 

     public Ticket parkVehicle(Vehicle vehicle) throws NoSpotAvailableException { // Added exception
         for (ParkingFloor floor : floors) { 
             ParkingSpot spot = floor.findAvailableSpot(vehicle); 
             if (spot != null) { 
                 spot.park(vehicle); 
                 Ticket ticket = new Ticket(spot); 
                 activeTickets.put(ticket.getTicketId(), ticket); 
                 logger.log("Vehicle parked: " + vehicle.getLicensePlate()); // Added logging
                 return ticket; 
             } 
         } 
         throw new NoSpotAvailableException("No spot available"); // Added throw
     } 

     public double unparkVehicle(String ticketId) { 
         Ticket ticket = activeTickets.remove(ticketId); 
         if (ticket != null) { 
             ticket.spot.unpark(); // Direct access assuming spot getter added
             logger.log("Vehicle unparked: " + ticketId); // Added logging
             return ticket.calculateFee(); 
         } 
         return 0; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Parking**: Call parkVehicle(), finds spot polymorphically based on size. 
 - **Unparking**: Use ticket to calculate fee and free spot. 
 - **Polymorphism**: Vehicles of different types use same park method. 
 - **Singleton**: Single lot instance. 
 - **Composition**: Floors and spots nested. 
 - **Strategy**: Can switch spot finding strategies dynamically (added enhancement). 
 - **Logging**: Events like park/unpark are logged (added enhancement).

 --- 

 ### Design Amazon - Online Shopping System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Product class. 
 - **Encapsulation**: Private cart items in ShoppingCart. 
 - **Inheritance**: Electronics, Books extend Product. 
 - **Polymorphism**: Overridden getDescription(). 
 - **Interface**: PaymentProcessor for different payments. 
 - **Composition**: Order has-a items. 
 - **Aggregation**: User has-a ShoppingCart (can exist independently). 
 - **Singleton**: InventoryManager. 
 - **Access Modifiers**: Private for sensitive data like payment details. 
 - **Static Members**: Static order counter. 
 - **Final Keyword**: Final product IDs. 
 - **Inner Classes**: Address as inner for User. 
 - **SOLID**: Dependency Inversion (via interfaces), Open-Closed (add new products). 
 - **Decorator Pattern**: Added for product discounts (enhancement for pricing flexibility).
 - **Exception Handling**: Added for out of stock and payment failures (enhancement).
 - **Caching**: Added simple cache for product lookups (enhancement for performance).

 #### Classes and Structure: 

 ```java
 // Enum: Order status (immutable) 
 enum OrderStatus { PENDING, SHIPPED, DELIVERED } 

 // Custom Exception: Added for out of stock (enhancement)
 class OutOfStockException extends Exception { 
     public OutOfStockException(String message) { super(message); } 
 }

 // Custom Exception: Added for payment failure (enhancement)
 class PaymentFailureException extends Exception { 
     public PaymentFailureException(String message) { super(message); } 
 }

 // Abstract Class: Product 
 abstract class Product implements Serializable { // Added Serializable
     private final String productId;  // Final 
     private String name; 
     private double price; 
     protected AtomicInteger stock = new AtomicInteger();  // Atomic for concurrency
     private java.util.Date lastUpdated; // Added for tracking updates (enhancement)

     public Product(String name, double price, int stock) { 
         this.productId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.price = price; 
         this.stock.set(stock); // Set atomic
         this.lastUpdated = new java.util.Date(); // Added
     } 

     // Abstract 
     public abstract String getDescription(); 

     public double getPrice() { return price; } 
     public void reduceStock(int quantity) throws OutOfStockException { // Added exception
         while (true) { // CAS for concurrency
             int current = stock.get(); 
             if (current < quantity) throw new OutOfStockException("Out of stock"); 
             if (stock.compareAndSet(current, current - quantity)) break; 
         } 
         lastUpdated = new java.util.Date(); // Added update
     } 
     public boolean isInStock(int quantity) { return stock.get() >= quantity; } 
     public java.util.Date getLastUpdated() { return lastUpdated; } // Added getter
 } 

 // Decorator: Added for discounts (enhancement with decorator pattern)
 abstract class ProductDecorator extends Product { 
     protected Product decoratedProduct; 

     public ProductDecorator(Product product) { 
         super(product.name, product.price, product.stock.get()); // Copied, but delegate
         this.decoratedProduct = product; 
     } 

     @Override 
     public String getDescription() { return decoratedProduct.getDescription(); } 
     @Override 
     public void reduceStock(int quantity) throws OutOfStockException { decoratedProduct.reduceStock(quantity); } // Delegate
     @Override 
     public boolean isInStock(int quantity) { return decoratedProduct.isInStock(quantity); } // Delegate
 } 

 // Concrete Decorator: Added for percentage discount
 class DiscountedProduct extends ProductDecorator { 
     private double discountPercent; 

     public DiscountedProduct(Product product, double discountPercent) { 
         super(product); 
         this.discountPercent = discountPercent; 
     } 

     @Override 
     public double getPrice() { return decoratedProduct.getPrice() * (1 - discountPercent / 100); } 
 } 

 // Inheritance 
 class Electronics extends Product { 
     private String brand; 

     public Electronics(String name, double price, int stock, String brand) { 
         super(name, price, stock); 
         this.brand = brand; 
     } 

     @Override 
     public String getDescription() { 
         return name + " by " + brand; 
     } 
 } 

 class Book extends Product { 
     private String author; 

     public Book(String name, double price, int stock, String author) { 
         super(name, price, stock); 
         this.author = author; 
     } 

     @Override 
     public String getDescription() { 
         return name + " by " + author; 
     } 
 } 

 // Interface: Payment 
 interface PaymentProcessor { 
     boolean processPayment(double amount) throws PaymentFailureException; // Enhanced with exception
     void refund(double amount); 
 } 

 // Concrete: CreditCardPayment 
 class CreditCardPayment implements PaymentProcessor { 
     private String cardNumber; 

     public CreditCardPayment(String cardNumber) { 
         this.cardNumber = cardNumber; 
     } 

     @Override 
     public boolean processPayment(double amount) throws PaymentFailureException { 
         // Process logic 
         if (Math.random() > 0.9) throw new PaymentFailureException("Payment failed"); // Simulated
         return true; 
     } 

     @Override 
     public void refund(double amount) { 
         // Refund logic 
     } 
 } 

 // Class: User 
 class User implements Serializable { // Added Serializable
     private String userId; 
     private String name; 
     private ShoppingCart cart;  // Aggregation 
     private class Address {  // Inner class 
         String street; 
         String city; 
     } 
     private Address address; 
     private List<Order> orderHistory; // Added for tracking past orders (enhancement)

     public User(String name) { 
         this.userId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.cart = new ShoppingCart(); 
         this.address = new Address(); 
         this.orderHistory = new ArrayList<>(); // Added
     } 

     public void addToCart(Product product, int quantity) { cart.addItem(product, quantity); } 

     public Order checkout(PaymentProcessor payment) throws PaymentFailureException { // Added exception
         double total = cart.calculateTotal(); 
         if (payment.processPayment(total)) { 
             Order order = new Order(this, cart.getItems()); 
             orderHistory.add(order); // Added to history
             return order; 
         } 
         throw new PaymentFailureException("Checkout failed due to payment issue"); // Added throw
     } 
     public List<Order> getOrderHistory() { return Collections.unmodifiableList(orderHistory); } // Unmodifiable
 } 

 // Class: ShoppingCart 
 class ShoppingCart implements Serializable { // Added Serializable
     private Map<Product, Integer> items = new HashMap<>();  // Composition 

     public void addItem(Product product, int quantity) throws OutOfStockException { // Added exception
         if (product.isInStock(quantity)) { 
             items.put(product, items.getOrDefault(product, 0) + quantity); 
         } else { 
             throw new OutOfStockException("Cannot add to cart: out of stock"); 
         } 
     } 

     public double calculateTotal() { 
         return items.entrySet().stream().mapToDouble(e -> e.getKey().getPrice() * e.getValue()).sum(); 
     } 

     public Map<Product, Integer> getItems() { return Collections.unmodifiableMap(items); } // Unmodifiable
     public void removeItem(Product product) { items.remove(product); } // Added method for cart management (enhancement)
 } 

 // Class: Order 
 class Order implements Serializable { // Added Serializable
     private String orderId; 
     private User user; 
     private Map<Product, Integer> items;  // Composition 
     private OrderStatus status; 
     private static AtomicInteger orderCounter = new AtomicInteger(0);  // Atomic

     public Order(User user, Map<Product, Integer> items) { 
         this.orderId = "ORD" + orderCounter.incrementAndGet(); 
         this.user = user; 
         this.items = new HashMap<>(items); // Copy
         this.status = OrderStatus.PENDING; 
         reduceStock(); // Moved stock reduction to order creation after payment
     } 

     private void reduceStock() throws OutOfStockException { 
         for (Map.Entry<Product, Integer> entry : items.entrySet()) { 
             entry.getKey().reduceStock(entry.getValue()); 
         } 
     } 

     public void updateStatus(OrderStatus newStatus) { status = newStatus; } 
     public double getTotal() { // Added for order total (enhancement)
         return items.entrySet().stream().mapToDouble(e -> e.getKey().getPrice() * e.getValue()).sum(); 
     }
 } 

 // Singleton: InventoryManager 
 class InventoryManager { 
     private static volatile InventoryManager instance; 
     private Map<String, Product> products = new HashMap<>(); // Changed to Map for lookups
     private Map<String, Product> productCache = new HashMap<>(); // Added for caching (enhancement)

     private InventoryManager() {} 

     public static InventoryManager getInstance() { 
         if (instance == null) { 
             synchronized (InventoryManager.class) { 
                 if (instance == null) instance = new InventoryManager(); 
             } 
         } 
         return instance; 
     } 

     public void addProduct(Product product) { 
         products.put(product.productId, product); 
         productCache.put(product.productId, product); // Added to cache
     } 
     public Product findProductById(String id) { 
         if (productCache.containsKey(id)) return productCache.get(id); // Check cache first
         return products.get(id); // Map lookup
     } 
     public void clearCache() { productCache.clear(); } // Added for cache management
 } 
 ``` 

 #### How It Works: 
 - **Shopping**: User adds to cart, checks out with payment interface (polymorphism). 
 - **Inventory**: Singleton manages stock updates. 
 - **Polymorphism**: Different products in cart use same methods. 
 - **Encapsulation**: Cart hides item map. 
 - **Discounts**: Products can be decorated with discounts (added enhancement). 
 - **Caching**: Frequent product lookups use cache (added enhancement).

 --- 

 ### Design Stack Overflow 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Post class for Questions/Answers. 
 - **Encapsulation**: Private votes in Post. 
 - **Inheritance**: Question, Answer extend Post. 
 - **Polymorphism**: Overridden display() . 
 - **Interface**: Taggable for tags. 
 - **Composition**: Question has-a list of Answers. 
 - **Aggregation**: User has-a list of Posts. 
 - **Singleton**: ForumManager. 
 - **Access Modifiers**: Protected for vote counts. 
 - **Static Members**: Static tag list. 
 - **Final Keyword**: Final post IDs. 
 - **SOLID**: Interface Segregation (small interfaces), Single Responsibility. 

 #### Classes and Structure: 

 ```java
 // Enum: Post status 
 enum PostStatus { OPEN, CLOSED, DELETED } 

 // Custom Exception: Added for invalid vote (enhancement)
 class InvalidVoteException extends Exception { 
     public InvalidVoteException(String message) { super(message); } 
 }

 // Custom Exception: Added for post closed (enhancement)
 class PostClosedException extends Exception { 
     public PostClosedException(String message) { super(message); } 
 }

 // Abstract Class: Post 
 abstract class Post implements Serializable { // Added Serializable
     private final String postId;  // Final 
     protected String content; 
     private User author; 
     protected int votes;  // Protected 
     private LocalDateTime timestamp; 
     private int rank; // Added for ranking (enhancement)

     public Post(String content, User author) { 
         this.postId = UUID.randomUUID().toString(); // Changed to UUID
         this.content = content; 
         this.author = author; 
         this.timestamp = LocalDateTime.now(); 
         this.votes = 0; 
         this.rank = 0; // Added
     } 

     public abstract void display();  // Abstract 

     public void upvote() throws InvalidVoteException { // Added exception
         if (votes > 100) throw new InvalidVoteException("Vote limit reached"); // Simulated
         votes++; 
         updateRank(); // Added call
     } 
     public void downvote() throws InvalidVoteException { // Added exception
         if (votes < -50) throw new InvalidVoteException("Downvote limit reached"); 
         votes--; 
         updateRank(); // Added call
     } 
     public User getAuthor() { return author; } 
     private void updateRank() { rank = votes * 2; } // Added for simple ranking (enhancement)
     public int getRank() { return rank; } // Added getter
 } 

 // Builder: Added for Post (enhancement with builder pattern)
 class PostBuilder { 
     private String content; 
     private User author; 
     private String title; // For Question

     public PostBuilder content(String content) { this.content = content; return this; } 
     public PostBuilder author(User author) { this.author = author; return this; } 
     public PostBuilder title(String title) { this.title = title; return this; } 
     public Question buildQuestion() { return new Question(title, content, author); } // Complete
     public Answer buildAnswer(Question question) { return new Answer(content, author, question); } // Complete
 }

 // Inheritance: Question 
 class Question extends Post implements Taggable { 
     private String title; 
     private List<Answer> answers = new ArrayList<>();  // Composition 
     private PostStatus status = PostStatus.OPEN; 
     private List<String> tags = new ArrayList<>(); 

     public Question(String title, String content, User author) { 
         super(content, author); 
         this.title = title; 
     } 

     @Override 
     public void display() { 
         // Display title + content 
     } 

     public void addAnswer(Answer answer) throws PostClosedException { // Added exception
         if (status == PostStatus.CLOSED) throw new PostClosedException("Cannot answer closed question"); 
         answers.add(answer); 
     } 
     public void close() { status = PostStatus.CLOSED; } 

     // From Interface 
     @Override 
     public void addTag(String tag) { tags.add(tag); } 
     @Override 
     public List<String> getTags() { return tags; } // Added to complete interface (enhancement)
 } 

 // Inheritance: Answer 
 class Answer extends Post { 
     private Question question;  // Association 

     public Answer(String content, User author, Question question) { 
         super(content, author); 
         this.question = question; 
     } 

     @Override 
     public void display() { 
         // Display answer 
     } 
     public Question getQuestion() { return question; } // Added getter (enhancement)
 } 

 // Interface: Taggable 
 interface Taggable { 
     void addTag(String tag); 
     List<String> getTags(); 
 } 

 // Class: User 
 class User implements Serializable { // Added Serializable
     private String userId; 
     private String username; 
     private int reputation; 
     private List<Post> posts = new ArrayList<>();  // Aggregation 

     public User(String username) { 
         this.userId = UUID.randomUUID().toString(); // Changed to UUID
         this.username = username; 
         this.reputation = 0; 
     } 

     public void createQuestion(String title, String content) { 
         Question q = PostFactory.createQuestion(title, content, this); // Changed to factory
         posts.add(q); 
     } 

     public void answerQuestion(Question question, String content) throws PostClosedException { // Added exception propagation
         Answer a = PostFactory.createAnswer(content, this, question); // Changed to factory
         question.addAnswer(a); 
         posts.add(a); 
     } 

     public void vote(Post post, boolean up) throws InvalidVoteException { // Added exception
         if (up) post.upvote(); else post.downvote(); 
         // Update reputation 
     } 
     public int getReputation() { return reputation; } // Added getter (enhancement)
 } 

 // Factory: Added for Post creation (enhancement)
 class PostFactory { 
     public static Question createQuestion(String title, String content, User author) { 
         return new Question(title, content, author); 
     } 
     public static Answer createAnswer(String content, User author, Question question) { 
         return new Answer(content, author, question); 
     } 
 }

 // Singleton: ForumManager 
 class ForumManager { 
     private static volatile ForumManager instance; 
     private List<Question> questions = new ArrayList<>(); 
     private List<User> users = new ArrayList<>(); 
     private static List<String> popularTags = new ArrayList<>();  // Static 
     private Map<String, Integer> tagUsage = new HashMap<>(); // Added for tracking tag popularity (enhancement)

     private ForumManager() {} 

     public static ForumManager getInstance() { 
         if (instance == null) { 
             synchronized (ForumManager.class) { 
                 if (instance == null) instance = new ForumManager(); 
             } 
         } 
         return instance; 
     } 

     public void addQuestion(Question question) { 
         questions.add(question); 
         for (String tag : question.getTags()) { // Added to update tag usage
             tagUsage.put(tag, tagUsage.getOrDefault(tag, 0) + 1); 
         } 
     } 
     public List<Question> searchByTag(String tag) { 
         // Search 
         return null; 
     } 
     public List<String> getPopularTags(int topN) { // Added method for top tags (enhancement)
         // Sort by usage (not implemented) 
         return null; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Posting**: User creates Question/Answer, added to forum. 
 - **Voting**: Polymorphic on Post subclasses. 
 - **Tagging**: Via interface on Question. 
 - **Singleton**: Manages all questions/users. 
 - **Builder**: Posts can be built fluently (added enhancement). 
 - **Ranking**: Posts have ranks based on votes (added enhancement).

 --- 

 ### Design a Movie Ticket Booking System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Seat class. 
 - **Encapsulation**: Private bookings in Theater. 
 - **Inheritance**: PremiumSeat extends Seat. 
 - **Polymorphism**: Overridden getPrice(). 
 - **Interface**: Bookable for tickets. 
 - **Composition**: Show has-a list of Seats. 
 - **Aggregation**: User has-a list of Bookings. 
 - **Singleton**: BookingManager. 
 - **Access Modifiers**: Private for seat status. 
 - **Static Members**: Static movie list. 
 - **Final Keyword**: Final ticket IDs. 
 - **SOLID**: Open-Closed (add seat types). 

 #### Classes and Structure: 

 ```java
 // Enum: Seat type 
 enum SeatType { REGULAR, PREMIUM } 

 // Custom Exception: Added for seat unavailable (enhancement)
 class SeatUnavailableException extends Exception { 
     public SeatUnavailableException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid show (enhancement)
 class InvalidShowException extends Exception { 
     public InvalidShowException(String message) { super(message); } 
 }

 // Interface: Added for chain of responsibility in seat selection (enhancement)
 interface SeatHandler { 
     void setNext(SeatHandler next); 
     Seat handle(Show show, SeatType type); 
 }

 // Concrete Handler: Added for regular seats
 class RegularSeatHandler implements SeatHandler { 
     private SeatHandler next; 

     @Override 
     public void setNext(SeatHandler next) { this.next = next; } 

     @Override 
     public Seat handle(Show show, SeatType type) { 
         if (type == SeatType.REGULAR) { 
             for (Seat s : show.getAvailableSeats()) { 
                 if (s instanceof RegularSeat) return s; // Find first
             } 
             return null; 
         } 
         return next != null ? next.handle(show, type) : null; 
     } 
 }

 // Concrete Handler: Added for premium seats
 class PremiumSeatHandler implements SeatHandler { 
     private SeatHandler next; 

     @Override 
     public void setNext(SeatHandler next) { this.next = next; } 

     @Override 
     public Seat handle(Show show, SeatType type) { 
         if (type == SeatType.PREMIUM) { 
             for (Seat s : show.getAvailableSeats()) { 
                 if (s instanceof PremiumSeat) return s; 
             } 
             return null; 
         } 
         return next != null ? next.handle(show, type) : null; 
     } 
 }

 // Abstract Class: Seat 
 abstract class Seat implements Serializable { // Added Serializable
     private final String seatId;  // Final 
     private boolean isBooked; 
     protected double basePrice;  // Protected 
     private String section; // Added for seat section (e.g., balcony) (enhancement)

     public Seat(String seatId, double basePrice) { 
         this.seatId = seatId; 
         this.basePrice = basePrice; 
         this.isBooked = false; 
         this.section = "Standard"; // Added default
     } 

     public abstract double getPrice();  // Abstract 

     public void book() { isBooked = true; } 
     public boolean isAvailable() { return !isBooked; } 
     public String getSection() { return section; } // Added getter
 } 

 // Inheritance 
 class RegularSeat extends Seat { 
     public RegularSeat(String seatId) { 
         super(seatId, 10.0); 
     } 

     @Override 
     public double getPrice() { return basePrice; } 
 } 

 class PremiumSeat extends Seat { 
     public PremiumSeat(String seatId) { 
         super(seatId, 15.0); 
     } 

     @Override 
     public double getPrice() { return basePrice * 1.5; }  // Overriding 
 } 

 // Class: Movie 
 class Movie implements Serializable { // Added Serializable
     private String movieId; 
     private String title; 
     private int duration; 
     private String genre; // Added for recommendations (enhancement)

     public Movie(String title, int duration) { 
         this.movieId = UUID.randomUUID().toString(); // Changed to UUID
         this.title = title; 
         this.duration = duration; 
         this.genre = "Drama"; // Added default
     } 
     public String getGenre() { return genre; } // Added getter
 } 

 // Class: Theater 
 class Theater implements Serializable { // Added class for hierarchy
     private String theaterId; 
     private List<Show> shows = new ArrayList<>(); 

     public Theater() { 
         this.theaterId = UUID.randomUUID().toString(); 
     } 

     public void addShow(Show show) { shows.add(show); } 
     public List<Show> getShows() { return Collections.unmodifiableList(shows); } // Unmodifiable
 } 

 // Class: Show 
 class Show implements Serializable { // Added Serializable
     private String showId; 
     private Movie movie; 
     private LocalDateTime time; 
     private List<Seat> seats = new ArrayList<>();  // Composition 

     public Show(Movie movie, LocalDateTime time, int numSeats) { 
         this.showId = UUID.randomUUID().toString(); // Changed to UUID
         this.movie = movie; 
         this.time = time; 
         for (int i = 1; i <= numSeats; i++) { 
             seats.add(i % 2 == 0 ? new PremiumSeat("S" + i) : new RegularSeat("S" + i)); 
         } 
     } 

     public List<Seat> getAvailableSeats() { 
         return seats.stream().filter(Seat::isAvailable).collect(Collectors.toList()); 
     } 
     public Seat findSeatByHandler(SeatHandler handler, SeatType type) { // Added for chain (enhancement)
         return handler.handle(this, type); 
     } 
 } 

 // Interface: Bookable 
 interface Bookable { 
     boolean bookSeats(List<Seat> seats) throws SeatUnavailableException; // Enhanced with exception
 } 

 // Class: Booking implements Bookable 
 class Booking implements Bookable, Serializable { // Added Serializable
     private String bookingId; 
     private User user; 
     private Show show; 
     private List<Seat> bookedSeats; 
     private double totalPrice; 

     public Booking(User user, Show show, List<Seat> seats) { 
         this.bookingId = UUID.randomUUID().toString(); // Changed to UUID
         this.user = user; 
         this.show = show; 
         this.bookedSeats = seats; 
         this.totalPrice = seats.stream().mapToDouble(Seat::getPrice).sum(); 
     } 

     @Override 
     public boolean bookSeats(List<Seat> seats) throws SeatUnavailableException { // Added exception
         if (seats.stream().anyMatch(s -> !s.isAvailable())) throw new SeatUnavailableException("Some seats unavailable"); 
         for (Seat s : seats) s.book(); 
         return true; 
     } 

     public void cancel() { 
         for (Seat s : bookedSeats) s.isBooked = false;  // Assuming direct access for simplicity 
     } 
     public double getTotalPrice() { return totalPrice; } // Added getter (enhancement)
 } 

 // Class: User 
 class User implements Serializable { // Added Serializable
     private String userId; 
     private List<Booking> bookings = new ArrayList<>();  // Aggregation 
     private List<String> preferredGenres; // Added for recommendations (enhancement)

     public User() { 
         this.userId = UUID.randomUUID().toString(); // Changed to UUID
         this.preferredGenres = new ArrayList<>(); // Added
     } 

     public Booking bookTicket(Show show, List<Seat> seats) throws SeatUnavailableException, InvalidShowException { // Added exceptions
         if (show.time.isBefore(LocalDateTime.now())) throw new InvalidShowException("Show in past"); 
         if (seats.stream().allMatch(Seat::isAvailable)) { 
             Booking booking = new Booking(this, show, seats); 
             if (booking.bookSeats(seats)) { 
                 bookings.add(booking); 
                 return booking; 
             } 
         } 
         throw new SeatUnavailableException("Seats not available"); 
     } 
     public void addPreferredGenre(String genre) { preferredGenres.add(genre); } // Added method
 } 

 // Singleton: BookingManager 
 class BookingManager { 
     private static volatile BookingManager instance; 
     private List<Theater> theaters = new ArrayList<>(); // Added for hierarchy
     private Map<User, List<Movie>> recommendations = new HashMap<>(); // Added for recommendations (enhancement)

     private BookingManager() {} 

     public static BookingManager getInstance() { 
         if (instance == null) { 
             synchronized (BookingManager.class) { 
                 if (instance == null) instance = new BookingManager(); 
             } 
         } 
         return instance; 
     } 

     public void addTheater(Theater theater) { theaters.add(theater); } 
     public void addMovieToTheater(Theater theater, Movie movie) { theater.addShow(new Show(movie, LocalDateTime.now(), 100)); } // Example
     public List<Show> findShowsByMovie(String movieTitle) { 
         // Search across theaters
         return null; 
     } 
     public List<Movie> getRecommendations(User user) { // Added method (enhancement)
         // Logic based on preferredGenres (not implemented) 
         return null; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Booking**: User selects show/seats, books via interface. 
 - **Polymorphism**: Seats of different types calculate price differently. 
 - **Singleton**: Manages movies/shows centrally. 
 - **Composition**: Shows contain seats. 
 - **Chain**: Seat selection can use handlers for types (added enhancement). 
 - **Recommendations**: Suggest movies based on user preferences (added enhancement).

 --- 

 ### Design an ATM 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Transaction class. 
 - **Encapsulation**: Private balance in Account. 
 - **Inheritance**: CheckingAccount, SavingsAccount extend Account. 
 - **Polymorphism**: Overridden withdraw() for limits. 
 - **Interface**: Authenticable for PIN check. 
 - **Composition**: ATM has-a CardReader, CashDispenser. 
 - **Singleton**: Bank as singleton. 
 - **Access Modifiers**: Private for PIN. 
 - **Static Members**: Static ATM ID. 
 - **Final Keyword**: Final account numbers. 
 - **SOLID**: Single Responsibility (Transaction types separate). 

 #### Classes and Structure: 

 ```java
 import java.security.MessageDigest; // For hashing

 // Enum: Transaction type 
 enum TransactionType { WITHDRAW, DEPOSIT, TRANSFER } 

 // Abstract Class: Transaction 
 abstract class Transaction implements Serializable { // Added Serializable
     protected double amount; 
     protected Account account; 
     protected LocalDateTime timestamp; 

     public Transaction(double amount, Account account) { 
         this.amount = amount; 
         this.account = account; 
         this.timestamp = LocalDateTime.now(); 
     } 

     public abstract boolean execute() throws InsufficientFundsException;  // Abstract 
 } 

 // Factory: Added for transactions (enhancement with factory pattern)
 class TransactionFactory { 
     public static Transaction createTransaction(TransactionType type, double amount, Account account) { 
         switch (type) { 
             case WITHDRAW: return new WithdrawTransaction(amount, account); 
             // Others 
             default: return null; 
         } 
     } 
 }

 // Inheritance: WithdrawTransaction 
 class WithdrawTransaction extends Transaction { 
     public WithdrawTransaction(double amount, Account account) { 
         super(amount, account); 
     } 

     @Override 
     public boolean execute() throws InsufficientFundsException { 
         return account.withdraw(amount); 
     } 
 } 

 // Similar for DepositTransaction, TransferTransaction 

 // Interface: Authenticable 
 interface Authenticable { 
     boolean authenticate(int pin) throws InvalidPinException; // Enhanced with exception
 } 

 // Class: Account implements Authenticable 
 class Account implements Authenticable, Serializable { // Added Serializable
     private final String accountNumber;  // Final 
     private double balance; 
     private byte[] hashedPin; // Private hashed
     private List<Transaction> transactions = new ArrayList<>();  // Composition 

     public Account(String accountNumber, int pin, double initialBalance) { 
         this.accountNumber = accountNumber; 
         this.hashedPin = hashPin(pin); 
         this.balance = initialBalance; 
     } 

     public boolean withdraw(double amount) throws InsufficientFundsException { // Added exception
         if (balance >= amount) { 
             balance -= amount; 
             return true; 
         } 
         throw new InsufficientFundsException("Cannot withdraw: insufficient funds"); 
     } 

     public void deposit(double amount) { balance += amount; } 

     @Override 
     public boolean authenticate(int enteredPin) throws InvalidPinException { 
         if (!Arrays.equals(hashedPin, hashPin(enteredPin))) throw new InvalidPinException("Invalid PIN"); 
         return true; 
     } 

     public double getBalance() { return balance; } 
     public void addTransaction(Transaction t) { transactions.add(t); } 
     private byte[] hashPin(int pin) { // Proper hashing
         try { 
             MessageDigest md = MessageDigest.getInstance("SHA-256"); 
             return md.digest(Integer.toString(pin).getBytes()); 
         } catch (Exception e) { 
             return null; 
         } 
     } 
     public void setPin(int newPin) { this.hashedPin = hashPin(newPin); } // Added setter with hashing
 } 

 // Inheritance: CheckingAccount 
 class CheckingAccount extends Account { 
     private double overdraftLimit; 

     public CheckingAccount(String accountNumber, int pin, double initialBalance, double overdraftLimit) { 
         super(accountNumber, pin, initialBalance); 
         this.overdraftLimit = overdraftLimit; 
     } 

     @Override 
     public boolean withdraw(double amount) throws InsufficientFundsException {  // Overriding 
         if (balance + overdraftLimit >= amount) { 
             balance -= amount; 
             return true; 
         } 
         throw new InsufficientFundsException("Overdraft limit exceeded"); 
     } 
 } 

 // Similar for SavingsAccount with interest 

 // Class: Card 
 class Card implements Serializable { // Added Serializable
     private String cardNumber; 
     private Account linkedAccount;  // Association 

     public Card(String cardNumber, Account account) { 
         this.cardNumber = cardNumber; 
         this.linkedAccount = account; 
     } 

     public Account getAccount() { return linkedAccount; } 
     public String getCardNumber() { return cardNumber; } // Added getter (enhancement)
 } 

 // Class: CardReader // Made public
 class CardReader { 
     // Read logic 
 } 

 // Class: CashDispenser // Made public
 class CashDispenser { 
     private int availableCash; 

     public void dispense(double amount) { 
         availableCash -= amount; 
     } 
     public int getAvailableCash() { return availableCash; } // Added getter (enhancement)
 } 

 // Class: ATM 
 class ATM implements Serializable { // Added Serializable
     private static final String ATM_ID = "ATM001";  // Static final 
     private CardReader cardReader = new CardReader(); 
     private CashDispenser cashDispenser = new CashDispenser(); 
     private Screen screen; 
     private Keypad keypad; 

     public ATM() { 
         // Initialize others 
     } 

     public void insertCard(Card card) { 
         // Read card 
     } 

     public boolean authenticate(Card card, int pin) throws InvalidPinException { // Added exception
         return card.getAccount().authenticate(pin); 
     } 

     public void performTransaction(Transaction t) throws InsufficientFundsException { // Added exception propagation
         if (t.execute()) { 
             t.account.addTransaction(t); 
             if (t instanceof WithdrawTransaction) { 
                 cashDispenser.dispense(t.amount); // Added dispense if withdraw
             } 
         } 
     } 
 } 

 // Singleton: Bank 
 class Bank { 
     private static volatile Bank instance; 
     private Map<String, Account> accounts = new HashMap<>(); 

     private Bank() {} 

     public static Bank getInstance() { 
         if (instance == null) { 
             synchronized (Bank.class) { 
                 if (instance == null) instance = new Bank(); 
             } 
         } 
         return instance; 
     } 

     public void addAccount(Account account) { accounts.put(account.getAccountNumber(), account); } 
     public Account getAccount(String accountNumber) { return accounts.get(accountNumber); } 
     public List<Account> getAllAccounts() { return new ArrayList<>(accounts.values()); } // Added for admin access (enhancement)
 } 
 ``` 

 #### How It Works: 
 - **Authentication**: Via interface on Account. 
 - **Transactions**: Create subclass instances, execute polymorphically. 
 - **Singleton**: Bank manages all accounts. 
 - **Encapsulation**: Balance hidden, modified via methods. 
 - **Factory**: Transactions created via factory for extensibility (added enhancement). 
 - **Security**: PIN hashed with SHA-256 on set (added enhancement).

 --- 

 ### Design an Airline Management System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Flight class. 
 - **Encapsulation**: Private seats in Airplane. 
 - **Inheritance**: DomesticFlight, InternationalFlight extend Flight. 
 - **Polymorphism**: Overridden calculateFare(). 
 - **Interface**: Bookable for reservations. 
 - **Composition**: Flight has-a Airplane, seats. 
 - **Aggregation**: Airline has-a list of Flights. 
 - **Singleton**: AirlineManager. 
 - **Access Modifiers**: Protected for fare calculations. 
 - **Static Members**: Static airport codes. 
 - **Final Keyword**: Final flight numbers. 
 - **SOLID**: Liskov (flights interchangeable). 

 #### Classes and Structure: 

 ```java
 import java.time.ZonedDateTime; // For timezone

 // Enum: Seat class 
 enum SeatClass { ECONOMY, BUSINESS, FIRST } 

 // Custom Exception: Added for no seat (enhancement)
 class NoSeatAvailableException extends Exception { 
     public NoSeatAvailableException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid booking date (enhancement)
 class InvalidBookingDateException extends Exception { 
     public InvalidBookingDateException(String message) { super(message); } 
 }

 // Interface: Added for external payment adapter (enhancement for adapter pattern)
 interface ExternalPaymentSystem { 
     boolean pay(double amount); 
 }

 // Adapter: Added for integrating external payment
 class PaymentAdapter implements ExternalPaymentSystem { 
     // Adapt to third-party API (not implemented) 
     @Override 
     public double pay(double amount) { return true; } 
 }

 // Class: Airplane 
 class Airplane implements Serializable { // Added Serializable
     private String model; 
     private Map<SeatClass, List<Seat>> seats = new HashMap<>();  // Composition 

     public Airplane(String model, int economySeats, int businessSeats, int firstSeats) { 
         this.model = model; 
         // Initialize seats 
         seats.put(SeatClass.ECONOMY, createSeats(economySeats, "E")); 
         // Similarly for others 
     } 

     private List<Seat> createSeats(int num, String prefix) { 
         List<Seat> list = new ArrayList<>(); 
         for (int i = 1; i <= num; i++) list.add(new Seat(prefix + i)); 
         return list; 
     } 

     public Seat findAvailableSeat(SeatClass cls) { 
         return seats.get(cls).stream().filter(Seat::isAvailable).findFirst().orElse(null); 
     } 
     public int getTotalSeats(SeatClass cls) { return seats.get(cls).size(); } // Added for stats (enhancement)
 } 

 // Class: Seat 
 class Seat implements Serializable { // Added Serializable
     private String seatNumber; 
     private boolean isAvailable = true; 

     public Seat(String seatNumber) { 
         this.seatNumber = seatNumber; 
     } 

     public void book() { isAvailable = false; } 
     public boolean isAvailable() { return isAvailable; } 
     public String getSeatNumber() { return seatNumber; } // Added getter (enhancement)
 } 

 // Abstract Class: Flight 
 abstract class Flight implements Serializable { // Added Serializable
     private final String flightNumber;  // Final 
     protected String origin; 
     protected String destination; 
     protected ZonedDateTime departureTime; // Changed to ZonedDateTime for timezone
     protected Airplane airplane;  // Composition 
     protected double baseFare;  // Protected 

     public Flight(String flightNumber, String origin, String destination, ZonedDateTime departureTime, Airplane airplane, double baseFare) { 
         this.flightNumber = flightNumber; 
         this.origin = origin; 
         this.destination = destination; 
         this.departureTime = departureTime; 
         this.airplane = airplane; 
         this.baseFare = baseFare; 
     } 

     public abstract double calculateFare(SeatClass cls);  // Abstract 

     public Reservation bookSeat(Passenger passenger, SeatClass cls) throws NoSeatAvailableException { // Added exception
         Seat seat = airplane.findAvailableSeat(cls); 
         if (seat != null) { 
             seat.book(); 
             return new Reservation(this, passenger, seat, calculateFare(cls)); 
         } 
         throw new NoSeatAvailableException("No seat in " + cls); 
     } 
 } 

 // Inheritance: DomesticFlight 
 class DomesticFlight extends Flight { 
     public DomesticFlight(String flightNumber, String origin, String destination, ZonedDateTime departureTime, Airplane airplane) { 
         super(flightNumber, origin, destination, departureTime, airplane, 100.0); 
     } 

     @Override 
     public double calculateFare(SeatClass cls) { 
         switch (cls) { 
             case ECONOMY: return baseFare; 
             case BUSINESS: return baseFare * 1.5; 
             default: return baseFare * 2; 
         } 
     } 
 } 

 // InternationalFlight with extra fees 
 class InternationalFlight extends Flight { 
     private double visaFee = 50.0; 

     public InternationalFlight(String flightNumber, String origin, String destination, ZonedDateTime departureTime, Airplane airplane) { 
         super(flightNumber, origin, destination, departureTime, airplane, 200.0); 
     } 

     @Override 
     public double calculateFare(SeatClass cls) { 
         double fare = super.baseFare;  // super keyword 
         // Add multipliers + visaFee 
         return fare + visaFee; 
     } 
 } 

 // Class: Passenger 
 class Passenger implements Serializable { // Added Serializable
     private String passengerId; 
     private String name; 
     private List<Reservation> reservations = new ArrayList<>();  // Aggregation 
     private int loyaltyPoints; // Added for loyalty program (enhancement)

     public Passenger(String name) { 
         this.passengerId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.loyaltyPoints = 0; // Added
     } 

     public void addReservation(Reservation res) { 
         reservations.add(res); 
         loyaltyPoints += 10; // Added points accumulation
     } 
     public int getLoyaltyPoints() { return loyaltyPoints; } // Added getter
 } 

 // Interface: Bookable 
 interface Bookable { 
     Reservation book(Flight flight, SeatClass cls) throws NoSeatAvailableException, InvalidBookingDateException; // Enhanced with exceptions
 } 

 // Class: Reservation implements Bookable 
 class Reservation implements Bookable, Serializable { // Added Serializable
     private String resId; 
     private Flight flight; 
     private Passenger passenger; 
     private Seat seat; 
     private double fare; 
     private ExternalPaymentSystem paymentSystem; // Added for adapter (enhancement)

     public Reservation(Flight flight, Passenger passenger, Seat seat, double fare, ExternalPaymentSystem paymentSystem) { // Injected paymentSystem
         this.resId = UUID.randomUUID().toString(); // Changed to UUID
         this.flight = flight; 
         this.passenger = passenger; 
         this.seat = seat; 
         this.fare = fare; 
         this.paymentSystem = paymentSystem; 
     } 

     @Override 
     public Reservation book(Flight flight, SeatClass cls) throws NoSeatAvailableException, InvalidBookingDateException { // Added exceptions
         if (flight.departureTime.isBefore(ZonedDateTime.now())) throw new InvalidBookingDateException("Flight in past"); 
         // Booking logic (delegate to flight) 
         return flight.bookSeat(passenger, cls); 
     } 

     public void cancel() { seat.book(); }  // Reverse book 
     public boolean processPayment() { return paymentSystem.pay(fare); } // Added using adapter
 } 

 // Singleton: AirlineManager 
 class AirlineManager { 
     private static volatile AirlineManager instance; 
     private List<Flight> flights = new ArrayList<>(); 
     private List<Airplane> airplanes = new ArrayList<>(); 
     private static Map<String, String> airportCodes = new HashMap<>();  // Static 

     private AirlineManager() {} 

     public static AirlineManager getInstance() { 
         if (instance == null) { 
             synchronized (AirlineManager.class) { 
                 if (instance == null) instance = new AirlineManager(); 
             } 
         } 
         return instance; 
     } 

     public void addFlight(Flight flight) { flights.add(flight); } 
     public List<Flight> searchFlights(String origin, String dest) { 
         // Search 
         return null; 
     } 
     public List<Flight> getFlightsByDate(LocalDate date) { // Added for date-based search (enhancement)
         // Filter (not implemented) 
         return null; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Booking**: Passenger books via flight, polymorphically calculates fare. 
 - **Polymorphism**: Different flight types handle fares differently. 
 - **Singleton**: Manages flights/airplanes. 
 - **Composition**: Flights contain airplanes/seats. 
 - **Adapter**: Integrates external payment (added enhancement). 
 - **Loyalty**: Passengers earn points on reservations (added enhancement).

 --- 

 ### Design Blackjack and a Deck of Cards 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Card class. 
 - **Encapsulation**: Private deck in Deck. 
 - **Inheritance**: Suit-specific if needed, but enum-based. 
 - **Polymorphism**: Overridden getValue() for face cards. 
 - **Interface**: Drawable for drawing cards. 
 - **Composition**: Deck has-a list of Cards. 
 - **Aggregation**: Player has-a Hand (list of Cards). 
 - **Singleton**: Game as singleton. 
 - **Access Modifiers**: Private for card values. 
 - **Static Members**: Static suits/ranks. 
 - **Final Keyword**: Final card ranks. 
 - **Inner Classes**: Hand as inner for Player. 
 - **SOLID**: Single Responsibility (Deck shuffles, Player plays). 

 #### Classes and Structure: 

 ```java
 // Enums: Immutable suits and ranks 
 enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES } 
 enum Rank { ACE, TWO, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING } 

 // Custom Exception: Added for empty deck (enhancement)
 class EmptyDeckException extends Exception { 
     public EmptyDeckException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid bet (enhancement)
 class InvalidBetException extends Exception { 
     public InvalidBetException(String message) { super(message); } 
 }

 // Interface: Added for command pattern (enhancement)
 interface GameCommand { 
     void execute(); 
     void undo(); 
 }

 // Concrete Command: Added for hit
 class HitCommand implements GameCommand { 
     private Player player; 
     private Deck deck; 
     private Card drawnCard; 

     public HitCommand(Player player, Deck deck) { 
         this.player = player; 
         this.deck = deck; 
     } 

     @Override 
     public void execute() { 
         drawnCard = deck.draw(); 
         player.getHand().addCard(drawnCard); 
     } 

     @Override 
     public void undo() { 
         player.getHand().cards.remove(drawnCard); // Simplified
     } 
 }

 // Class: Card // Simplified single class
 class Card implements Serializable { // Added Serializable
     private final Rank rank; 
     private final Suit suit; 

     public Card(Rank rank, Suit suit) { 
         this.rank = rank; 
         this.suit = suit; 
     } 

     public int getValue() { 
         switch (rank) { 
             case ACE: return 11; 
             case TWO: return 2; 
             case THREE: return 3; 
             case FOUR: return 4; 
             case FIVE: return 5; 
             case SIX: return 6; 
             case SEVEN: return 7; 
             case EIGHT: return 8; 
             case NINE: return 9; 
             case TEN: case JACK: case QUEEN: case KING: return 10; 
             default: return 0; 
         } 
     } 

     public String toString() { return rank + " of " + suit; } 
     public Rank getRank() { return rank; } // Added getter (enhancement)
 } 

 // Class: Deck implements Drawable 
 class Deck implements Drawable, Serializable { // Added Serializable
     private List<Card> cards = new ArrayList<>();  // Composition 
     private static final int NUM_DECKS = 1;  // Static 
     private int numDecks; // Added for multi-deck support (enhancement)

     public Deck() { 
         this.numDecks = NUM_DECKS; // Added
         for (int d = 0; d < numDecks; d++) { // Enhanced for multi-deck
             for (Suit s : Suit.values()) { 
                 for (Rank r : Rank.values()) { 
                     cards.add(new Card(r, s)); 
                 } 
             } 
         } 
         shuffle(); 
     } 

     public void shuffle() { Collections.shuffle(cards); } 

     @Override 
     public Card draw() throws EmptyDeckException { // Added exception
         if (cards.isEmpty()) throw new EmptyDeckException("Deck empty"); 
         return cards.remove(0); 
     } 
     public void setNumDecks(int num) { this.numDecks = num; } // Added setter for multi-deck
 } 

 // Interface: Drawable 
 interface Drawable { 
     Card draw() throws EmptyDeckException; // Enhanced with exception
 } 

 // Class: Hand 
 class Hand implements Serializable { // Added Serializable
     private List<Card> cards = new ArrayList<>(); 

     public void addCard(Card card) { cards.add(card); } 

     public int calculateScore() { 
         int score = 0; 
         int aces = 0; 
         for (Card c : cards) { 
             score += c.getValue(); 
             if (c.getRank() == Rank.ACE) aces++; 
         } 
         while (score > 21 && aces > 0) { 
             score -= 10; 
             aces--; 
         } 
         return score; 
     } 

     public boolean isBust() { return calculateScore() > 21; } 
     public boolean isBlackjack() { return cards.size() == 2 && calculateScore() == 21; } 
     public List<Card> getCards() { return cards; } // Added getter (enhancement)
 } 

 // Class: Player 
 class Player implements Serializable { // Added Serializable
     private String name; 
     private Hand hand = new Hand();  // Aggregation (inner possible) 
     private double balance; 

     public Player(String name, double balance) { 
         this.name = name; 
         this.balance = balance; 
     } 

     public void hit(Deck deck) throws EmptyDeckException { // Added exception propagation
         hand.addCard(deck.draw()); 
     } 
     public void stand() { /* No action */ } 
     public double bet(double amount) throws InvalidBetException { // Added exception
         if (balance < amount || amount <= 0) throw new InvalidBetException("Invalid bet amount"); 
         balance -= amount; 
         return amount; 
     } 

     public void win(double amount) { balance += amount; } 
     public Hand getHand() { return hand; } 
     public void executeCommand(GameCommand command) { command.execute(); } // Added for command pattern
 } 

 // Class: Dealer extends Player (inheritance for similar behavior) 
 class Dealer extends Player { 
     public Dealer() { 
         super("Dealer", 0);  // No balance needed 
     } 

     // Overriding: Dealer hits until 17 
     public void play(Deck deck) throws EmptyDeckException { // Added exception
         while (getHand().calculateScore() < 17) hit(deck); 
     } 
 } 

 // Singleton: BlackjackGame 
 class BlackjackGame { 
     private static volatile BlackjackGame instance; 
     private Deck deck; 
     private List<Player> players = new ArrayList<>(); 
     private Dealer dealer; 
     private static final double BLACKJACK_PAYOUT = 1.5;  // Static 

     private BlackjackGame() { 
         deck = new Deck(); 
         dealer = new Dealer(); 
     } 

     public static BlackjackGame getInstance() { 
         if (instance == null) { 
             synchronized (BlackjackGame.class) { 
                 if (instance == null) instance = new BlackjackGame(); 
             } 
         } 
         return instance; 
     } 

     public void addPlayer(Player player) { players.add(player); } 

     public void startRound() throws EmptyDeckException { // Added exception
         // Deal initial cards: 2 to each player, 1 up/1 down to dealer 
         for (Player p : players) { 
             p.hit(deck); p.hit(deck); 
         } 
         dealer.hit(deck); dealer.hit(deck); 
     } 

     public void resolveRound() throws EmptyDeckException { // Added exception
         dealer.play(deck); // Dealer logic here
         int dealerScore = dealer.getHand().calculateScore(); 
         for (Player p : players) { 
             int playerScore = p.getHand().calculateScore(); 
             if (playerScore > 21) { /* Lose */ } 
             else if (playerScore > dealerScore || dealerScore > 21) { p.win(/* bet * 2 */); } 
             // Handle ties, blackjack 
         } 
     } 
     public Deck getDeck() { return deck; } // Added getter (enhancement)
 } 
 ``` 

 #### How It Works: 
 - **Game Flow**: Start round deals cards from deck (drawable interface). 
 - **Scoring**: Hand calculates with polymorphism for card values. 
 - **Polymorphism**: Different card types return values. 
 - **Singleton**: One game instance. 
 - **Command**: Actions like hit can be executed/undone (added enhancement). 
 - **Multi-Deck**: Deck can be set to use multiple (added enhancement).

 --- 

 ### Design a Hotel Management System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Room class. 
 - **Encapsulation**: Private bookings in Hotel. 
 - **Inheritance**: Suite, StandardRoom extend Room. 
 - **Polymorphism**: Overridden getRate(). 
 - **Interface**: Reservable for bookings. 
 - **Composition**: Hotel has-a list of Rooms. 
 - **Aggregation**: Guest has-a list of Reservations. 
 - **Singleton**: HotelManager. 
 - **Access Modifiers**: Private for room status. 
 - **Static Members**: Static room types. 
 - **Final Keyword**: Final room numbers. 
 - **SOLID**: Dependency Inversion via interface. 

 #### Classes and Structure: 

 ```java
 // Enum: Room status 
 enum RoomStatus { AVAILABLE, OCCUPIED, MAINTENANCE } 

 // Custom Exception: Added for room unavailable (enhancement)
 class RoomUnavailableException extends Exception { 
     public RoomUnavailableException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid dates (enhancement)
 class InvalidDateException extends Exception { 
     public InvalidDateException(String message) { super(message); } 
 }

 // Facade: Added for booking process (enhancement with facade pattern)
 class BookingFacade { 
     private HotelManager manager; 
     private PaymentProcessor paymentProcessor; // Added for full workflow

     public BookingFacade(HotelManager manager, PaymentProcessor paymentProcessor) { this.manager = manager; this.paymentProcessor = paymentProcessor; } 

     public Reservation bookRoom(Guest guest, Room room, LocalDate checkIn, LocalDate checkOut) throws RoomUnavailableException, InvalidDateException { 
         // Simplified booking (calls multiple methods) 
         if (checkIn.isAfter(checkOut)) throw new InvalidDateException("Invalid dates"); 
         if (!room.isAvailableForDates(checkIn, checkOut)) throw new RoomUnavailableException("Room not available"); 
         double total = room.getRate() * ChronoUnit.DAYS.between(checkIn, checkOut); 
         if (paymentProcessor.processPayment(total)) { // Simulated payment
             return guest.bookRoom(room, checkIn, checkOut); 
         } 
         return null; 
     } 
 }

 // Interface: PaymentProcessor // Added for facade
 interface PaymentProcessor { 
     boolean processPayment(double amount); 
 }

 // Abstract Class: Room 
 abstract class Room implements Serializable { // Added Serializable
     private final String roomNumber;  // Final 
     protected double baseRate;  // Protected 
     private List<Reservation> reservations = new ArrayList<>(); // Changed for date-based availability
     private List<String> reviews = new ArrayList<>(); // Added for reviews (enhancement)

     public Room(String roomNumber, double baseRate) { 
         this.roomNumber = roomNumber; 
         this.baseRate = baseRate; 
     } 

     public abstract double getRate();  // Abstract 

     public void checkIn() { } // No longer needed
     public void checkOut() { } // No longer needed
     public boolean isAvailableForDates(LocalDate checkIn, LocalDate checkOut) { // Changed for dates
         for (Reservation r : reservations) { 
             if (r.checkIn.isBefore(checkOut) && r.checkOut.isAfter(checkIn)) return false; 
         } 
         return true; 
     } 
     public void addReview(String review) { reviews.add(review); } // Added method
     public List<String> getReviews() { return reviews; } // Added getter
     public void addReservation(Reservation res) { reservations.add(res); } // Added
 } 

 // Inheritance 
 class StandardRoom extends Room { 
     public StandardRoom(String roomNumber) { 
         super(roomNumber, 100.0); 
     } 

     @Override 
     public double getRate() { return baseRate; } 
 } 

 class Suite extends Room { 
     private boolean hasJacuzzi; 

     public Suite(String roomNumber, boolean hasJacuzzi) { 
         super(roomNumber, 200.0); 
         this.hasJacuzzi = hasJacuzzi; 
     } 

     @Override 
     public double getRate() { return baseRate + (hasJacuzzi ? 50 : 0); } 
 } 

 // Interface: Reservable 
 interface Reservable { 
     boolean reserve(LocalDate checkIn, LocalDate checkOut) throws InvalidDateException; // Enhanced with exception
 } 

 // Class: Reservation implements Reservable 
 class Reservation implements Reservable, Serializable { // Added Serializable
     private String resId; 
     private Guest guest; 
     private Room room; 
     public LocalDate checkIn; // Public for check
     public LocalDate checkOut; 
     private double totalCost; 

     public Reservation(Guest guest, Room room, LocalDate checkIn, LocalDate checkOut) { 
         this.resId = UUID.randomUUID().toString(); // Changed to UUID
         this.guest = guest; 
         this.room = room; 
         this.checkIn = checkIn; 
         this.checkOut = checkOut; 
         this.totalCost = room.getRate() * ChronoUnit.DAYS.between(checkIn, checkOut); 
     } 

     @Override 
     public boolean reserve(LocalDate checkIn, LocalDate checkOut) throws InvalidDateException { // Added exception
         if (checkIn.isAfter(checkOut)) throw new InvalidDateException("Check-in after check-out"); 
         if (room.isAvailableForDates(checkIn, checkOut)) { 
             room.addReservation(this); 
             return true; 
         } 
         return false; 
     } 

     public void cancel() { room.reservations.remove(this); } // Changed
     public double getTotalCost() { return totalCost; } // Added getter (enhancement)
 } 

 // Class: Guest 
 class Guest implements Serializable { // Added Serializable
     private String guestId; 
     private String name; 
     private List<Reservation> reservations = new ArrayList<>();  // Aggregation 

     public Guest(String name) { 
         this.guestId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
     } 

     public Reservation bookRoom(Room room, LocalDate checkIn, LocalDate checkOut) throws RoomUnavailableException, InvalidDateException { // Added exceptions
         Reservation res = new Reservation(this, room, checkIn, checkOut); 
         if (res.reserve(checkIn, checkOut)) { 
             reservations.add(res); 
             return res; 
         } 
         throw new RoomUnavailableException("Room not available"); 
     } 
     public List<Reservation> getReservations() { return Collections.unmodifiableList(reservations); } // Added unmodifiable getter (enhancement)
 } 

 // Singleton: HotelManager 
 class HotelManager { 
     private static volatile HotelManager instance; 
     private List<Room> rooms = new ArrayList<>(); 
     private List<Reservation> allReservations = new ArrayList<>(); 
     private static Map<String, String> roomTypes = new HashMap<>();  // Static 

     private HotelManager() { 
         // Add rooms 
     } 

     public static HotelManager getInstance() { 
         if (instance == null) { 
             synchronized (HotelManager.class) { 
                 if (instance == null) instance = new HotelManager(); 
             } 
         } 
         return instance; 
     } 

     public void addRoom(Room room) { rooms.add(room); } 
     public List<Room> findAvailableRooms(LocalDate date) { 
         // Search 
         return null; 
     } 
     public void addReservation(Reservation res) { allReservations.add(res); } 
     public double calculateRevenue(LocalDate from, LocalDate to) { // Added for business metrics (enhancement)
         // Sum totals (not implemented) 
         return 0; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Booking**: Guest books room via reservation interface. 
 - **Polymorphism**: Different rooms calculate rates differently. 
 - **Singleton**: Manages rooms/reservations. 
 - **Encapsulation**: Room status hidden. 
 - **Facade**: Simplifies booking with validation (added enhancement). 
 - **Reviews**: Guests can add reviews to rooms (added enhancement).

 --- 

 ### Design a Restaurant Management System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract MenuItem class. 
 - **Encapsulation**: Private orders in Restaurant. 
 - **Inheritance**: Appetizer, MainCourse extend MenuItem. 
 - **Polymorphism**: Overridden getPrice() for specials. 
 - **Interface**: Orderable for placing orders. 
 - **Composition**: Order has-a list of MenuItems. 
 - **Aggregation**: Customer has-a list of Orders. 
 - **Singleton**: RestaurantManager. 
 - **Access Modifiers**: Private for inventory. 
 - **Static Members**: Static menu. 
 - **Final Keyword**: Final item IDs. 
 - **SOLID**: Open-Closed (add new items). 

 #### Classes and Structure: 

 ```java
 // Enum: Item type 
 enum ItemType { APPETIZER, MAIN, DESSERT } 

 // Custom Exception: Added for out of stock (enhancement)
 class OutOfStockException extends Exception { 
     public OutOfStockException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid order (enhancement)
 class InvalidOrderException extends Exception { 
     public InvalidOrderException(String message) { super(message); } 
 }

 // Abstract Class: MenuItem 
 abstract class MenuItem implements Serializable { // Added Serializable
     private final String itemId;  // Final 
     protected String name; 
     protected double price;  // Protected 
     private int quantityAvailable; 

     public MenuItem(String name, double price, int quantity) { 
         this.itemId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.price = price; 
         this.quantityAvailable = quantity; 
     } 

     public abstract double getPrice();  // Abstract for discounts 

     public void reduceQuantity(int qty) throws OutOfStockException { // Added exception
         if (quantityAvailable < qty) throw new OutOfStockException("Out of stock"); 
         quantityAvailable -= qty; 
     } 
     public boolean isAvailable(int qty) { return quantityAvailable >= qty; } 
     public String getName() { return name; } // Added getter (enhancement)
 } 

 // Inheritance 
 class Appetizer extends MenuItem { 
     public Appetizer(String name, double price, int quantity) { 
         super(name, price, quantity); 
     } 

     @Override 
     public double getPrice() { return price; } 
 } 

 class MainCourse extends MenuItem { 
     private boolean isSpecial; 

     public MainCourse(String name, double price, int quantity, boolean isSpecial) { 
         super(name, price, quantity); 
         this.isSpecial = isSpecial; 
     } 

     @Override 
     public double getPrice() { return isSpecial ? price * 1.2 : price; } 
 } 

 // Interface: Orderable 
 interface Orderable { 
     void placeOrder() throws InvalidOrderException; // Enhanced with exception
     void cancelOrder(); 
 } 

 // Class: Order implements Orderable 
 class Order implements Orderable { 
     private String orderId; 
     private Customer customer; 
     private List<MenuItem> items = new ArrayList<>();  // Composition 
     private double total; 
     private OrderStatus status = OrderStatus.PENDING; 
     private double discount = 0; // Added for discounts (enhancement)

     enum OrderStatus { PENDING, PREPARING, READY, DELIVERED, CANCELLED } 

     public Order(Customer customer) { 
         this.orderId = UUID.randomUUID().toString(); // Changed to UUID
         this.customer = customer; 
     } 

     public void addItem(MenuItem item, int qty) throws OutOfStockException { // Added exception
         if (item.isAvailable(qty)) { 
             items.add(item); 
             item.reduceQuantity(qty); 
             total += item.getPrice() * qty; 
         } else { 
             throw new OutOfStockException("Item not available"); 
         } 
     } 

     @Override 
     public void placeOrder() throws InvalidOrderException { // Added exception
         if (items.isEmpty()) throw new InvalidOrderException("Empty order"); 
         status = OrderStatus.PREPARING; 
         applyDiscount(); // Added call
     } 

     @Override 
     public void cancelOrder() { status = OrderStatus.CANCELLED; /* Restore quantity */ } 

     public double getTotal() { return total - discount; } // Enhanced with discount
     private void applyDiscount() { discount = total * 0.1; } // Added simple discount logic (template step)
     // Template Method: Added for processing steps
     public final void processOrder() throws InvalidOrderException { 
         prepare(); 
         placeOrder(); 
         deliver(); 
     } 
     protected void prepare() { /* Hook */ } 
     protected void deliver() { status = OrderStatus.DELIVERED; } 
 } 

 // Subclass: DineInOrder 
 class DineInOrder extends Order { 
     private Table table; 

     public DineInOrder(Customer customer, Table table) { 
         super(customer); 
         this.table = table; 
     } 

     @Override 
     protected void prepare() { table.occupy(); } // Hook implementation
 } 

 // Subclass: TakeoutOrder 
 class TakeoutOrder extends Order { 
     public TakeoutOrder(Customer customer) { 
         super(customer); 
     } 

     @Override 
     protected void prepare() { // Hook for packaging
     } 
 } 

 // Class: Customer 
 class Customer implements Serializable { // Added Serializable
     private String customerId; 
     private String name; 
     private List<Order> orders = new ArrayList<>();  // Aggregation 

     public Customer(String name) { 
         this.customerId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
     } 

     public Order createOrder() { 
         Order order = new Order(this); 
         orders.add(order); 
         return order; 
     } 
     public List<Order> getOrders() { return Collections.unmodifiableList(orders); } // Unmodifiable
 } 

 // Class: Table 
 class Table { 
     private int tableNumber; 
     private boolean isOccupied; 

     public Table(int tableNumber) { 
         this.tableNumber = tableNumber; 
         this.isOccupied = false; 
     } 

     public void occupy() { isOccupied = true; } 
     public void free() { isOccupied = false; } 
     public int getTableNumber() { return tableNumber; } // Added getter (enhancement)
 } 

 // Singleton: RestaurantManager 
 class RestaurantManager { 
     private static volatile RestaurantManager instance; 
     private List<MenuItem> menu = new ArrayList<>(); 
     private List<Table> tables = new ArrayList<>(); 
     private List<Order> allOrders = new ArrayList<>(); 
     private static Map<String, ItemType> itemTypes = new HashMap<>();  // Static 

     private RestaurantManager() { 
         // Initialize menu, tables 
     } 

     public static RestaurantManager getInstance() { 
         if (instance == null) { 
             synchronized (RestaurantManager.class) { 
                 if (instance == null) instance = new RestaurantManager(); 
             } 
         } 
         return instance; 
     } 

     public void addMenuItem(MenuItem item) { menu.add(item); } 
     public List<MenuItem> getMenuByType(ItemType type) { 
         // Filter 
         return null; 
     } 
     public void processOrder(Order order) throws InvalidOrderException { // Added exception propagation
         order.processOrder(); // Enhanced with template
         allOrders.add(order); 
     } 
     public double getTotalRevenue() { // Added for metrics (enhancement)
         return allOrders.stream().mapToDouble(Order::getTotal).sum(); 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Ordering**: Customer creates order, adds items polymorphically. 
 - **Polymorphism**: Items calculate price differently. 
 - **Singleton**: Manages menu/orders/tables. 
 - **Composition**: Orders contain items. 
 - **Template**: Order processing follows fixed steps with hooks (added enhancement). 
 - **Discounts**: Automatically applied on place (added enhancement).

 --- 

 ### Design Chess 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Piece class. 
 - **Encapsulation**: Private board in ChessBoard. 
 - **Inheritance**: Pawn, Rook, etc., extend Piece. 
 - **Polymorphism**: Overridden isValidMove(). 
 - **Interface**: Movable for move validation. 
 - **Composition**: Board has-a Pieces (via positions). 
 - **Aggregation**: Player has-a list of Pieces. 
 - **Singleton**: Game as singleton. 
 - **Access Modifiers**: Protected for position. 
 - **Static Members**: Static board size. 
 - **Final Keyword**: Final colors. 
 - **Inner Classes**: Position as inner. 
 - **SOLID**: Single Responsibility (Piece validates own moves). 

 #### Classes and Structure: 

 ```java
 // Enum: Colors 
 enum Color { WHITE, BLACK } 

 // Class: Position (Inner possible) 
 class Position { 
     int row; 
     int col; 

     public Position(int row, int col) { 
         this.row = row; 
         this.col = col; 
     } 
     public boolean equals(Object o) { // Added for comparison (enhancement)
         if (this == o) return true; 
         if (!(o instanceof Position)) return false; 
         Position p = (Position) o; 
         return row == p.row && col == p.col; 
     } 
 } 

 // Custom Exception: Added for invalid move (enhancement)
 class InvalidMoveException extends Exception { 
     public InvalidMoveException(String message) { super(message); } 
 }

 // Memento: Added for state saving (enhancement with memento pattern)
 class BoardMemento { 
     private Piece[][] state; 

     public BoardMemento(Piece[][] state) { 
         this.state = state.clone(); // Simplified 
     } 

     public Piece[][] getState() { return state; } 
 } 

 // Abstract Class: Piece 
 abstract class Piece implements Serializable { // Added Serializable
     protected Position position;  // Protected 
     protected Color color; 
     private boolean isCaptured = false; 

     public Piece(Color color, Position position) { 
         this.color = color; 
         this.position = position; 
     } 

     public abstract boolean isValidMove(Position to, ChessBoard board) throws InvalidMoveException;  // Enhanced with exception

     public void move(Position to) { this.position = to; } 
     public void capture() { isCaptured = true; } 
     public Color getColor() { return color; } 
     public Position getPosition() { return position; } 
 } 

 // Inheritance: Pawn 
 class Pawn extends Piece { 
     private boolean hasMoved = false; 

     public Pawn(Color color, Position position) { 
         super(color, position); 
     } 

     @Override 
     public boolean isValidMove(Position to, ChessBoard board) throws InvalidMoveException { 
         int dir = color == Color.WHITE ? 1 : -1; 
         if (to.col == position.col && (to.row == position.row + dir || (!hasMoved && to.row == position.row + 2 * dir))) { 
             if (!board.isEmpty(to)) throw new InvalidMoveException("Path blocked"); 
             if (!hasMoved && !board.isEmpty(new Position(position.row + dir, position.col))) throw new InvalidMoveException("Path blocked for double move"); 
             return true; 
         } 
         if (Math.abs(to.col - position.col) == 1 && to.row == position.row + dir) { 
             Piece target = board.getPieceAt(to); 
             if (target != null && target.getColor() != color) return true; // Capture
         } 
         throw new InvalidMoveException("Invalid pawn move"); 
     } 
 } 

 // Inheritance: Rook 
 class Rook extends Piece { 
     public Rook(Color color, Position position) { 
         super(color, position); 
     } 

     @Override 
     public boolean isValidMove(Position to, ChessBoard board) throws InvalidMoveException { 
         if (position.row != to.row && position.col != to.col) throw new InvalidMoveException("Not straight line"); 
         int stepRow = Integer.signum(to.row - position.row); 
         int stepCol = Integer.signum(to.col - position.col); 
         Position current = new Position(position.row + stepRow, position.col + stepCol); 
         while (!current.equals(to)) { 
             if (!board.isEmpty(current)) throw new InvalidMoveException("Path blocked"); 
             current.row += stepRow; 
             current.col += stepCol; 
         } 
         Piece target = board.getPieceAt(to); 
         if (target != null && target.getColor() == color) throw new InvalidMoveException("Cannot capture own piece"); 
         return true; 
     } 
 } 

 // Similar for other pieces

 // Interface: Movable 
 interface Movable { 
     void makeMove(Position from, Position to) throws InvalidMoveException; // Enhanced with exception
 } 

 // Class: Player 
 class Player implements Movable, Serializable { // Added Serializable
     private String name; 
     private Color color; 
     private List<Piece> pieces = new ArrayList<>();  // Aggregation 

     public Player(String name, Color color) { 
         this.name = name; 
         this.color = color; 
         // Initialize pieces 
     } 

     public void addPiece(Piece piece) { pieces.add(piece); } 

     @Override 
     public void makeMove(Position from, Position to) throws InvalidMoveException { 
         Piece piece = getPieceAt(from); 
         if (piece != null && piece.isValidMove(to, /* board */)) { 
             piece.move(to); 
             // Capture if enemy 
         } else { 
             throw new InvalidMoveException("Cannot make move"); 
         } 
     } 

     private Piece getPieceAt(Position pos) { 
         return pieces.stream().filter(p -> p.getPosition().equals(pos)).findFirst().orElse(null); 
     } 

     public boolean isInCheck(ChessBoard board) { 
         // Check if king threatened 
         return false; 
     } 
     public List<Piece> getPieces() { return pieces; } // Added getter (enhancement)
 } 

 // AI Player: Added extension (enhancement)
 class AIPlayer extends Player { 
     public AIPlayer(Color color) { 
         super("AI", color); 
     } 

     public void makeAIMove(ChessBoard board) throws InvalidMoveException { 
         // Simple AI logic (not implemented) 
     } 
 }

 // Class: ChessBoard 
 class ChessBoard implements Serializable { // Added Serializable
     private Piece[][] board = new Piece[8][8];  // Composition (2D array) 
     private static final int SIZE = 8;  // Static 
     private List<BoardMemento> history = new ArrayList<>(); // Added for mementos (enhancement)

     public ChessBoard() { 
         // Setup initial positions 
     } 

     public boolean isEmpty(Position pos) { return board[pos.row][pos.col] == null; } 
     public Piece getPieceAt(Position pos) { return board[pos.row][pos.col]; } 
     public void placePiece(Piece piece, Position pos) { board[pos.row][pos.col] = piece; } 
     public void removePiece(Position pos) { board[pos.row][pos.col] = null; } 
     public BoardMemento saveState() { // Added for memento
         return new BoardMemento(board); 
     } 
     public void restoreState(BoardMemento memento) { // Added for undo
         this.board = memento.getState(); 
     } 
     public void addToHistory(BoardMemento memento) { history.add(memento); } // Added
 } 

 // Singleton: ChessGame 
 class ChessGame { 
     private static volatile ChessGame instance; 
     private ChessBoard board; 
     private Player whitePlayer; 
     private Player blackPlayer; 
     private Player currentTurn; 

     private ChessGame() { 
         board = new ChessBoard(); 
         whitePlayer = new Player("White", Color.WHITE); 
         blackPlayer = new Player("Black", Color.BLACK); 
         currentTurn = whitePlayer; 
         // Setup pieces on board 
     } 

     public static ChessGame getInstance() { 
         if (instance == null) { 
             synchronized (ChessGame.class) { 
                 if (instance == null) instance = new ChessGame(); 
             } 
         } 
         return instance; 
     } 

     public void switchTurn() { currentTurn = currentTurn == whitePlayer ? blackPlayer : whitePlayer; } 

     public boolean makeMove(Position from, Position to) throws InvalidMoveException { // Added exception
         board.addToHistory(board.saveState()); // Added save before move
         Piece piece = board.getPieceAt(from); 
         if (piece != null && piece.isValidMove(to, board)) { 
             if (!board.isEmpty(to)) { 
                 Piece target = board.getPieceAt(to); 
                 if (target.getColor() == piece.getColor()) throw new InvalidMoveException("Cannot capture own piece"); 
                 target.capture(); 
             } 
             board.removePiece(from); 
             board.placePiece(piece, to); 
             piece.move(to); 
             switchTurn(); 
             return true; 
         } 
         throw new InvalidMoveException("Invalid move"); 
     } 

     public boolean isCheckmate() { 
         // Logic: in check and no moves 
         return false; 
     } 
     public void undoLastMove() { // Added for memento undo (enhancement)
         if (!board.history.isEmpty()) { 
             board.restoreState(board.history.remove(board.history.size() - 1)); 
         } 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Moves**: Player calls makeMove, validates via piece's overridden method. 
 - **Polymorphism**: Each piece has unique move validation. 
 - **Singleton**: One game with board. 
 - **Composition**: Board holds pieces. 
 - **Memento**: Game state can be saved/undone (added enhancement). 
 - **AI**: Extendable to AI players (added enhancement).

 --- 

 ### Design an Online Stock Brokerage System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Order class. 
 - **Encapsulation**: Private portfolio in UserAccount. 
 - **Inheritance**: MarketOrder, LimitOrder extend Order. 
 - **Polymorphism**: Overridden execute(). 
 - **Interface**: Tradable for stocks. 
 - **Composition**: Portfolio has-a Holdings. 
 - **Aggregation**: Exchange has-a list of Stocks. 
 - **Singleton**: StockExchange. 
 - **Access Modifiers**: Private for balances. 
 - **Static Members**: Static tickers. 
 - **Final Keyword**: Final order IDs. 
 - **SOLID**: Interface Segregation. 

 #### Classes and Structure: 

 ```java
 // Enum: Order type 
 enum OrderType { BUY, SELL } 

 // Custom Exception: Added for insufficient funds (enhancement)
 class InsufficientFundsException extends Exception { 
     public InsufficientFundsException(String message) { super(message); } 
 }

 // Custom Exception: Added for market closed (enhancement)
 class MarketClosedException extends Exception { 
     public MarketClosedException(String message) { super(message); } 
 }

 // Proxy: Added for order execution (enhancement with proxy pattern)
 class OrderProxy { 
     private Order realOrder; 
     private MarketClock clock; // Added for market state

     public OrderProxy(Order order, MarketClock clock) { this.realOrder = order; this.clock = clock; } 

     public boolean execute(double currentPrice) throws MarketClosedException { 
         if (!clock.isMarketOpen()) throw new MarketClosedException("Market is closed"); 
         return realOrder.execute(currentPrice); 
     } 
 } 

 // Class: MarketClock // Added for market state
 class MarketClock { 
     public boolean isMarketOpen() { 
         // Logic based on time (not implemented) 
         return true; 
     } 
 }

 // Abstract Class: Order 
 abstract class Order extends Observable implements Serializable { // Added Observable for notifications, Serializable
     private final String orderId;  // Final 
     protected Stock stock; 
     protected int quantity; 
     protected OrderType type; 
     protected UserAccount account; 

     public Order(Stock stock, int quantity, OrderType type, UserAccount account) { 
         this.orderId = UUID.randomUUID().toString(); // Changed to UUID
         this.stock = stock; 
         this.quantity = quantity; 
         this.type = type; 
         this.account = account; 
     } 

     public abstract boolean execute(double currentPrice) throws InsufficientFundsException;  // Enhanced with exception
     protected void notifyExecution() { // Added for observers
         setChanged(); 
         notifyObservers("Order executed: " + orderId); 
     } 
 } 

 // Inheritance: MarketOrder 
 class MarketOrder extends Order { 
     public MarketOrder(Stock stock, int quantity, OrderType type, UserAccount account) { 
         super(stock, quantity, type, account); 
     } 

     @Override 
     public boolean execute(double currentPrice) throws InsufficientFundsException { 
         // Execute at market price 
         double cost = currentPrice * quantity; 
         if (type == OrderType.BUY) { 
             if (!account.hasFunds(cost)) throw new InsufficientFundsException("Insufficient funds for buy"); 
             account.deductFunds(cost); 
             account.addHolding(stock, quantity); 
             notifyExecution(); // Added notification
             return true; 
         } else if (type == OrderType.SELL && account.hasHolding(stock, quantity)) { 
             account.addFunds(cost); 
             account.removeHolding(stock, quantity); 
             notifyExecution(); // Added
             return true; 
         } 
         return false; 
     } 
 } 

 // LimitOrder with price limit 
 class LimitOrder extends Order { 
     private double limitPrice; 

     public LimitOrder(Stock stock, int quantity, OrderType type, UserAccount account, double limitPrice) { 
         super(stock, quantity, type, account); 
         this.limitPrice = limitPrice; 
     } 

     @Override 
     public boolean execute(double currentPrice) throws InsufficientFundsException { 
         if (type == OrderType.BUY && currentPrice <= limitPrice) { 
             // Similar to market 
             double cost = currentPrice * quantity; 
             if (!account.hasFunds(cost)) throw new InsufficientFundsException("Insufficient funds for limit buy"); 
             // Proceed 
             notifyExecution(); // Added
             return true; 
         } else if (type == OrderType.SELL && currentPrice >= limitPrice) { 
             return true; 
         } 
         return false; 
     } 
 } 

 // Interface: Tradable 
 interface Tradable { 
     double getCurrentPrice(); 
     String getTicker(); 
 } 

 // Class: Stock implements Tradable 
 class Stock implements Tradable, Serializable { // Added Serializable
     private String ticker; 
     private double currentPrice; 
     private double high; 
     private double low; 

     public Stock(String ticker, double initialPrice) { 
         this.ticker = ticker; 
         this.currentPrice = initialPrice; 
     } 

     @Override 
     public double getCurrentPrice() { return currentPrice; } 
     @Override 
     public String getTicker() { return ticker; } 

     public void updatePrice(double newPrice) { 
         currentPrice = newPrice; 
         // Update high/low 
     } 
     public double getHigh() { return high; } // Added getter (enhancement)
 } 

 // Class: Holding 
 class Holding implements Serializable { // Added Serializable
     private Stock stock; 
     private int quantity; 

     public Holding(Stock stock, int quantity) { 
         this.stock = stock; 
         this.quantity = quantity; 
     } 

     public void add(int qty) { quantity += qty; } 
     public void remove(int qty) { quantity -= qty; } 
     public boolean has(int qty) { return quantity >= qty; } 
     public int getQuantity() { return quantity; } // Added getter (enhancement)
 } 

 // Class: UserAccount 
 class UserAccount implements Observer, Serializable { // Added Observer for notifications, Serializable
     private String accountId; 
     private double balance; 
     private Map<Stock, Holding> portfolio = new HashMap<>();  // Composition 

     public UserAccount(double initialBalance) { 
         this.accountId = UUID.randomUUID().toString(); // Changed to UUID
         this.balance = initialBalance; 
     } 

     public boolean hasFunds(double amount) { return balance >= amount; } 
     public void deductFunds(double amount) { balance -= amount; } 
     public void addFunds(double amount) { balance += amount; } 

     public void addHolding(Stock stock, int qty) { 
         portfolio.computeIfAbsent(stock, k -> new Holding(stock, 0)).add(qty); 
     } 

     public void removeHolding(Stock stock, int qty) { 
         Holding h = portfolio.get(stock); 
         if (h != null) h.remove(qty); 
     } 

     public boolean hasHolding(Stock stock, int qty) { 
         Holding h = portfolio.get(stock); 
         return h != null && h.has(qty); 
     } 

     public double getPortfolioValue() { 
         return portfolio.values().stream().mapToDouble(h -> h.stock.getCurrentPrice() * h.quantity).sum(); 
     } 
     @Override 
     public void update(Observable o, Object arg) { // Added for order notifications
         // Handle notification (not implemented) 
     } 
 } 

 // Class: MatchingEngine // Added for order matching
 class MatchingEngine { 
     private PriorityQueue<Order> buyOrders = new PriorityQueue<>((a, b) -> Double.compare(b.fare, a.fare)); // Max heap for buys (simplified fare as price)
     private PriorityQueue<Order> sellOrders = new PriorityQueue<>((a, b) -> Double.compare(a.fare, b.fare)); // Min heap for sells

     public void addOrder(Order order) { 
         if (order.type == OrderType.BUY) buyOrders.add(order); 
         else sellOrders.add(order); 
         matchOrders(); 
     } 

     private void matchOrders() { 
         while (!buyOrders.isEmpty() && !sellOrders.isEmpty() && buyOrders.peek().fare >= sellOrders.peek().fare) { 
             // Execute trade (simplified) 
             buyOrders.poll(); 
             sellOrders.poll(); 
         } 
     } 
 }

 // Singleton: StockExchange 
 class StockExchange { 
     private static volatile StockExchange instance; 
     private Map<String, Stock> stocks = new HashMap<>(); // Changed to Map
     private List<Order> pendingOrders = new ArrayList<>(); 
     private static Map<String, Stock> tickerMap = new HashMap<>();  // Static 
     private MatchingEngine matchingEngine = new MatchingEngine(); // Added

     private StockExchange() {} 

     public static StockExchange getInstance() { 
         if (instance == null) { 
             synchronized (StockExchange.class) { 
                 if (instance == null) instance = new StockExchange(); 
             } 
         } 
         return instance; 
     } 

     public void addStock(Stock stock) { 
         stocks.put(stock.getTicker(), stock); 
         tickerMap.put(stock.getTicker(), stock); 
     } 

     public void placeOrder(Order order) throws MarketClosedException, InsufficientFundsException { // Added exceptions
         OrderProxy proxy = new OrderProxy(order, new MarketClock()); // Added proxy
         if (proxy.execute(order.stock.getCurrentPrice())) { 
             matchingEngine.addOrder(order); // Added to engine
         } else { 
             pendingOrders.add(order); 
         } 
     } 

     public void processPendingOrders() { 
         // Check and execute 
     } 
     public List<Stock> getStocks() { return new ArrayList<>(stocks.values()); } // Added getter (enhancement)
 } 
 ``` 

 #### How It Works: 
 - **Trading**: Place order, executes polymorphically based on type. 
 - **Polymorphism**: Market vs Limit handle execution differently. 
 - **Singleton**: Exchange manages stocks/orders. 
 - **Composition**: Accounts have portfolios. 
 - **Proxy**: Ensures secure/conditional execution (added enhancement). 
 - **Notifications**: Accounts observe order executions (added enhancement). 

 --- 

 ### Design a Car Rental System 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Vehicle class. 
 - **Encapsulation**: Private rentals in RentalAgency. 
 - **Inheritance**: Car, SUV extend Vehicle. 
 - **Polymorphism**: Overridden getRentalRate(). 
 - **Interface**: Rentable for rentals. 
 - **Composition**: Rental has-a Vehicle. 
 - **Aggregation**: Customer has-a list of Rentals. 
 - **Singleton**: RentalAgency. 
 - **Access Modifiers**: Private for availability. 
 - **Static Members**: Static vehicle types. 
 - **Final Keyword**: Final vehicle IDs. 
 - **SOLID**: Open-Closed (add vehicle types). 

 #### Classes and Structure: 

 ```java
 // Enum: Vehicle status 
 enum VehicleStatus { AVAILABLE, RENTED, MAINTENANCE } 

 // Custom Exception: Added for vehicle unavailable (enhancement)
 class VehicleUnavailableException extends Exception { 
     public VehicleUnavailableException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid period (enhancement)
 class InvalidRentalPeriodException extends Exception { 
     public InvalidRentalPeriodException(String message) { super(message); } 
 }

 // Interface: Added for vehicle features in bridge pattern (enhancement)
 interface VehicleFeature { 
     double getAdditionalRate(); 
 }

 // Concrete Feature: Added for GPS
 class GPSFeature implements VehicleFeature { 
     @Override 
     public double getAdditionalRate() { return 10.0; } 
 }

 // Concrete Feature: Added for no feature
 class NoFeature implements VehicleFeature { 
     @Override 
     public double getAdditionalRate() { return 0.0; } 
 }

 // Abstract Class: Vehicle 
 abstract class Vehicle implements Serializable { // Added Serializable
     private final String vehicleId;  // Final 
     protected String model; 
     protected double baseRate;  // Protected 
     private VehicleStatus status = VehicleStatus.AVAILABLE; 
     protected VehicleFeature feature; // Added for bridge (enhancement)

     public Vehicle(String model, double baseRate) { 
         this.vehicleId = UUID.randomUUID().toString(); // Changed to UUID
         this.model = model; 
         this.baseRate = baseRate; 
         this.feature = new NoFeature(); // Added default
     } 

     public abstract double getRentalRate();  // Abstract 

     public void rent() { status = VehicleStatus.RENTED; } 
     public void returnVehicle() { status = VehicleStatus.AVAILABLE; } 
     public boolean isAvailable() { return status == VehicleStatus.AVAILABLE; } 
     public double getFeatureRate() { return feature.getAdditionalRate(); } // Added bridge call
     public void setFeature(VehicleFeature feature) { this.feature = feature; } // Added setter for bridge
 } 

 // Inheritance 
 class Car extends Vehicle { 
     public Car(String model) { 
         super(model, 50.0); 
     } 

     @Override 
     public double getRentalRate() { return baseRate + getFeatureRate(); } // Enhanced with feature
 } 

 class SUV extends Vehicle { 
     private boolean has4WD; 

     public SUV(String model, boolean has4WD) { 
         super(model, 80.0); 
         this.has4WD = has4WD; 
     } 

     @Override 
     public double getRentalRate() { return baseRate + (has4WD ? 20 : 0) + getFeatureRate(); } // Enhanced
 } 

 // Interface: Rentable 
 interface Rentable { 
     boolean rentVehicle(LocalDate start, LocalDate end) throws InvalidRentalPeriodException; // Enhanced with exception
 } 

 // Class: Rental implements Rentable 
 class Rental implements Rentable, Serializable { // Added Serializable
     private String rentalId; 
     private Customer customer; 
     private Vehicle vehicle; 
     private LocalDate startDate; 
     private LocalDate endDate; 
     private double totalCost; 
     private boolean hasInsurance; // Added for insurance (enhancement)

     public Rental(Customer customer, Vehicle vehicle, LocalDate start, LocalDate end) { 
         this.rentalId = UUID.randomUUID().toString(); // Changed to UUID
         this.customer = customer; 
         this.vehicle = vehicle; 
         this.startDate = start; 
         this.endDate = end; 
         this.totalCost = vehicle.getRentalRate() * ChronoUnit.DAYS.between(start, end); 
         this.hasInsurance = false; // Added default
     } 

     @Override 
     public boolean rentVehicle(LocalDate start, LocalDate end) throws InvalidRentalPeriodException { // Added exception
         if (start.isAfter(end)) throw new InvalidRentalPeriodException("Start after end"); 
         if (vehicle.isAvailable()) { 
             vehicle.rent(); 
             if (hasInsurance) totalCost += 20; // Added insurance cost
             return true; 
         } 
         return false; 
     } 

     public void returnVehicle() { vehicle.returnVehicle(); } 
     public void addInsurance() { hasInsurance = true; } // Added method
 } 

 // Class: Customer 
 class Customer implements Serializable { // Added Serializable
     private String customerId; 
     private String name; 
     private List<Rental> rentals = new ArrayList<>();  // Aggregation 

     public Customer(String name) { 
         this.customerId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
     } 

     public Rental rentVehicle(Vehicle vehicle, LocalDate start, LocalDate end) throws VehicleUnavailableException, InvalidRentalPeriodException { // Added exceptions
         Rental rental = new Rental(this, vehicle, start, end); 
         if (rental.rentVehicle(start, end)) { 
             rentals.add(rental); 
             return rental; 
         } 
         throw new VehicleUnavailableException("Vehicle not available"); 
     } 
     public List<Rental> getRentals() { return Collections.unmodifiableList(rentals); } // Unmodifiable
 } 

 // Singleton: RentalAgency 
 class RentalAgency { 
     private static volatile RentalAgency instance; 
     private List<Vehicle> vehicles = new ArrayList<>(); 
     private List<Rental> allRentals = new ArrayList<>(); 
     private static Map<String, String> vehicleTypes = new HashMap<>();  // Static 

     private RentalAgency() {} 

     public static RentalAgency getInstance() { 
         if (instance == null) { 
             synchronized (RentalAgency.class) { 
                 if (instance == null) instance = new RentalAgency(); 
             } 
         } 
         return instance; 
     } 

     public void addVehicle(Vehicle vehicle) { vehicles.add(vehicle); } 
     public List<Vehicle> findAvailableVehicles(LocalDate date) { 
         // Filter available 
         return null; 
     } 
     public void addRental(Rental rental) { allRentals.add(rental); } 
     public double calculateTotalRevenue() { // Added for metrics (enhancement)
         return allRentals.stream().mapToDouble(r -> r.totalCost).sum(); 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Renting**: Customer rents via interface, updates status. 
 - **Polymorphism**: Different vehicles have different rates. 
 - **Singleton**: Agency manages vehicles/rentals. 
 - **Encapsulation**: Vehicle status hidden. 
 - **Bridge**: Features like GPS separated from vehicle types (added enhancement). 
 - **Insurance**: Optional addition to rentals (added enhancement).

 --- 

 ### Design LinkedIn 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Profile class. 
 - **Encapsulation**: Private connections in User. 
 - **Inheritance**: UserProfile, CompanyProfile extend Profile. 
 - **Polymorphism**: Overridden display(). 
 - **Interface**: Connectable for connections. 
 - **Composition**: Post has-a comments. 
 - **Aggregation**: User has-a list of Posts. 
 - **Singleton**: NetworkManager. 
 - **Access Modifiers**: Private for personal info. 
 - **Static Members**: Static job listings. 
 - **Final Keyword**: Final user IDs. 
 - **SOLID**: Single Responsibility. 

 #### Classes and Structure: 

 ```java
 // Abstract Class: Profile 
 abstract class Profile implements Serializable { // Added Serializable
     private final String profileId;  // Final 
     protected String name; 
     protected String bio; 

     public Profile(String name, String bio) { 
         this.profileId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.bio = bio; 
     } 

     public abstract void display();  // Abstract 
     public abstract void acceptVisitor(ProfileVisitor visitor); // Added for visitor pattern (enhancement)
 } 

 // Visitor: Added for profile operations (enhancement)
 interface ProfileVisitor { 
     void visit(UserProfile profile); 
     void visit(CompanyProfile profile); 
 }

 // Concrete Visitor: Added for analytics
 class AnalyticsVisitor implements ProfileVisitor { 
     @Override 
     public void visit(UserProfile profile) { 
         // Log user view (not implemented) 
     } 

     @Override 
     public void visit(CompanyProfile profile) { 
         // Log company view 
     } 
 }

 // Inheritance: UserProfile 
 class UserProfile extends Profile { 
     private String jobTitle; 
     private List<Profile> connections = new ArrayList<>();  // Composition for network 
     private Map<String, Integer> endorsements = new HashMap<>(); // Added for skills (enhancement)

     public UserProfile(String name, String bio, String jobTitle) { 
         super(name, bio); 
         this.jobTitle = jobTitle; 
     } 

     @Override 
     public void display() { 
         // Show user details 
     } 

     public void connect(Profile other) { connections.add(other); } 
     public void endorseSkill(String skill) { // Added method
         endorsements.put(skill, endorsements.getOrDefault(skill, 0) + 1); 
     } 
     public Map<String, Integer> getEndorsements() { return endorsements; } // Added getter
     @Override 
     public void acceptVisitor(ProfileVisitor visitor) { visitor.visit(this); } // Added
 } 

 // CompanyProfile 
 class CompanyProfile extends Profile { 
     private String industry; 

     public CompanyProfile(String name, String bio, String industry) { 
         super(name, bio); 
         this.industry = industry; 
     } 

     @Override 
     public void display() { 
         // Show company 
     } 
     @Override 
     public void acceptVisitor(ProfileVisitor visitor) { visitor.visit(this); } // Added
 } 

 // Interface: Connectable 
 interface Connectable { 
     void sendConnectionRequest(Profile to) throws ConnectionLimitException; // Enhanced with exception
     void acceptConnection(Profile from); 
 } 

 // Custom Exception: Added for connection limit (enhancement)
 class ConnectionLimitException extends Exception { 
     public ConnectionLimitException(String message) { super(message); } 
 }

 // Class: User implements Connectable 
 class User implements Connectable, Serializable { // Added Serializable
     private String userId; 
     private UserProfile profile; 
     private List<Post> posts = new ArrayList<>();  // Aggregation 
     private List<Job> appliedJobs = new ArrayList<>(); 

     public User(String name, String bio, String jobTitle) { 
         this.userId = UUID.randomUUID().toString(); // Changed to UUID
         this.profile = new UserProfile(name, bio, jobTitle); 
     } 

     public void createPost(String content) { 
         Post post = new Post(this, content); 
         posts.add(post); 
     } 

     @Override 
     public void sendConnectionRequest(Profile to) throws ConnectionLimitException { 
         if (profile.connections.size() > 500) throw new ConnectionLimitException("Connection limit reached"); 
         // Send request 
     } 

     @Override 
     public void acceptConnection(Profile from) { 
         profile.connect(from); 
     } 
     public void viewProfile(Profile profile) { // Added for visitor
         profile.acceptVisitor(new AnalyticsVisitor()); 
     } 
 } 

 // Class: Post 
 class Post implements Serializable { // Added Serializable
     private String postId; 
     private User author; 
     private String content; 
     private List<Comment> comments = new ArrayList<>();  // Composition 
     private int likes; 

     public Post(User author, String content) { 
         this.postId = UUID.randomUUID().toString(); // Changed to UUID
         this.author = author; 
         this.content = content; 
         this.likes = 0; 
     } 

     public void like() { likes++; } 
     public void addComment(Comment comment) { comments.add(comment); } 
     public int getLikes() { return likes; } // Added getter (enhancement)
 } 

 // Class: Comment 
 class Comment { 
     private User author; 
     private String text; 

     public Comment(User author, String text) { 
         this.author = author; 
         this.text = text; 
     } 
     public User getAuthor() { return author; } // Added getter (enhancement)
 } 

 // Class: Job 
 class Job { 
     private String jobId; 
     private CompanyProfile company; 
     private String title; 
     private String description; 

     public Job(CompanyProfile company, String title, String description) { 
         this.jobId = UUID.randomUUID().toString(); // Changed to UUID
         this.company = company; 
         this.title = title; 
         this.description = description; 
     } 
     public String getTitle() { return title; } // Added getter (enhancement)
 } 

 // Singleton: NetworkManager 
 class NetworkManager { 
     private static volatile NetworkManager instance; 
     private List<User> users = new ArrayList<>(); 
     private List<CompanyProfile> companies = new ArrayList<>(); 
     private List<Job> jobs = new ArrayList<>(); 
     private static List<Post> feed = new ArrayList<>();  // Static for global feed 

     private NetworkManager() {} 

     public static NetworkManager getInstance() { 
         if (instance == null) { 
             synchronized (NetworkManager.class) { 
                 if (instance == null) instance = new NetworkManager(); 
             } 
         } 
         return instance; 
     } 

     public void addUser(User user) { users.add(user); } 
     public void addJob(Job job) { jobs.add(job); } 
     public List<Post> getFeedForUser(User user) { 
         // Based on connections 
         return null; 
     } 
     public void applyToJob(User user, Job job) throws InvalidApplicationException { // Added exception
         if (user.appliedJobs.contains(job)) throw new InvalidApplicationException("Already applied"); 
         user.appliedJobs.add(job); 
     } 
     public List<Post> getGlobalFeed() { return feed; } // Added getter (enhancement)
 } 

 // Custom Exception: Added for invalid application (enhancement)
 class InvalidApplicationException extends Exception { 
     public InvalidApplicationException(String message) { super(message); } 
 }
 ``` 

 #### How It Works: 
 - **Networking**: Send/accept connections via interface. 
 - **Posting**: Users create posts, add to feed. 
 - **Polymorphism**: Different profiles display differently. 
 - **Singleton**: Network manages users/jobs/feed. 
 - **Visitor**: Profiles can be visited for analytics (added enhancement). 
 - **Endorsements**: Users can endorse skills (added enhancement).

 --- 

 ### Design Cricinfo 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Match class. 
 - **Encapsulation**: Private stats in Player. 
 - **Inheritance**: TestMatch, ODIMatch extend Match. 
 - **Polymorphism**: Overridden getFormat(). 
 - **Interface**: Scoreable for updates. 
 - **Composition**: Team has-a Players. 
 - **Aggregation**: Tournament has-a Matches. 
 - **Singleton**: CricketDatabase. 
 - **Access Modifiers**: Private for live scores. 
 - **Static Members**: Static formats. 
 - **Final Keyword**: Final match IDs. 
 - **SOLID**: Liskov Substitution. 

 #### Classes and Structure: 

 ```java
 // Enum: Match status 
 enum MatchStatus { UPCOMING, LIVE, COMPLETED } 

 // Custom Exception: Added for invalid score update (enhancement)
 class InvalidUpdateException extends Exception { 
     public InvalidUpdateException(String message) { super(message); } 
 }

 // Abstract Class: Match 
 abstract class Match implements Serializable { // Added Serializable
     private final String matchId;  // Final 
     protected Team teamA; 
     protected Team teamB; 
     protected LocalDate date; 
     protected MatchStatus status = MatchStatus.UPCOMING; 
     private List<String> commentary = new ArrayList<>(); // Added for live commentary (enhancement)

     public Match(Team teamA, Team teamB, LocalDate date) { 
         this.matchId = UUID.randomUUID().toString(); // Changed to UUID
         this.teamA = teamA; 
         this.teamB = teamB; 
         this.date = date; 
     } 

     public abstract String getFormat();  // Abstract 

     public void start() { status = MatchStatus.LIVE; } 
     public void end() { status = MatchStatus.COMPLETED; } 
     public void addCommentary(String comment) { commentary.add(comment); } // Added method
     public List<String> getCommentary() { return commentary; } // Added getter
 } 

 // Inheritance: ODIMatch 
 class ODIMatch extends Match { 
     private int overs = 50; 

     public ODIMatch(Team teamA, Team teamB, LocalDate date) { 
         super(teamA, teamB, date); 
     } 

     @Override 
     public String getFormat() { return "ODI"; } 
 } 

 // TestMatch with days 
 class TestMatch extends Match { 
     private int days = 5; 

     public TestMatch(Team teamA, Team teamB, LocalDate date) { 
         super(teamA, teamB, date); 
     } 

     @Override 
     public String getFormat() { return "Test"; } 
 } 

 // Class: Team 
 class Team implements Serializable { // Added Serializable
     private String teamName; 
     private List<Player> players = new ArrayList<>();  // Composition 
     private int score; 

     public Team(String teamName) { 
         this.teamName = teamName; 
     } 

     public void addPlayer(Player player) { players.add(player); } 
     public void updateScore(int runs) { score += runs; } 
     public Iterator<Player> getPlayerIterator() { // Added for iterator (enhancement)
         return players.iterator(); 
     } 
 } 

 // Class: Player 
 class Player implements Serializable { // Added Serializable
     private String name; 
     private int runs; 
     private int wickets; 
     // Other stats 

     public Player(String name) { 
         this.name = name; 
     } 

     public void addRuns(int r) { runs += r; } 
     public void addWicket() { wickets++; } 
     public int getRuns() { return runs; } // Added getter (enhancement)
 } 

 // Interface: Scoreable 
 interface Scoreable { 
     void updateScore(Team team, int runs) throws InvalidUpdateException; // Enhanced with exception
     void updateWicket(Team team); 
 } 

 // Class: LiveScoreUpdater implements Scoreable 
 class LiveScoreUpdater implements Scoreable { 
     private Match match; 

     public LiveScoreUpdater(Match match) { 
         this.match = match; 
     } 

     @Override 
     public void updateScore(Team team, int runs) throws InvalidUpdateException { 
         if (match.status != MatchStatus.LIVE) throw new InvalidUpdateException("Match not live"); 
         team.updateScore(runs); 
     } 

     @Override 
     public void updateWicket(Team team) { 
         // Wicket logic 
     } 
 } 

 // Singleton: CricketDatabase 
 class CricketDatabase { 
     private static volatile CricketDatabase instance; 
     private List<Match> matches = new ArrayList<>(); 
     private List<Team> teams = new ArrayList<>(); 
     private List<Player> players = new ArrayList<>(); 
     private static Map<String, String> formats = new HashMap<>();  // Static 

     private CricketDatabase() {} 

     public static CricketDatabase getInstance() { 
         if (instance == null) { 
             synchronized (CricketDatabase.class) { 
                 if (instance == null) instance = new CricketDatabase(); 
             } 
         } 
         return instance; 
     } 

     public void addMatch(Match match) { matches.add(match); } 
     public List<Match> getLiveMatches() { 
         return matches.stream().filter(m -> m.status == MatchStatus.LIVE).collect(Collectors.toList()); 
     } 
     public void updatePlayerStats(Player player, int runs, int wickets) throws InvalidUpdateException { // Added exception
         if (runs < 0) throw new InvalidUpdateException("Negative runs invalid"); 
         player.addRuns(runs); 
         for (int i = 0; i < wickets; i++) player.addWicket(); 
     } 
     public List<Player> getTopPlayers(int topN) { // Added for rankings (enhancement)
         // Sort by runs/wickets (not implemented) 
         return null; 
     } 
 } 
 ``` 

 #### How It Works: 
 - **Matches**: Create match type, update scores via interface. 
 - **Polymorphism**: Different matches have formats. 
 - **Singleton**: Database manages all data. 
 - **Composition**: Teams have players. 
 - **Iterator**: Traverse players in team (added enhancement). 
 - **Commentary**: Live comments added to matches (added enhancement).

 --- 

 ### Design Facebook - a Social Network 

 #### Key OOP Concepts Used: 
 - **Abstraction**: Abstract Post class (Text, Image). 
 - **Encapsulation**: Private friends in User. 
 - **Inheritance**: TextPost, ImagePost extend Post. 
 - **Polymorphism**: Overridden share(). 
 - **Interface**: Shareable for posts. 
 - **Composition**: User has-a Timeline (list of Posts). 
 - **Aggregation**: Group has-a Members. 
 - **Singleton**: SocialNetwork. 
 - **Access Modifiers**: Private for passwords. 
 - **Static Members**: Static newsfeed. 
 - **Final Keyword**: Final user IDs. 
 - **SOLID**: Open-Closed (add post types). 

 #### Classes and Structure: 

 ```java
 // Enum: Privacy 
 enum Privacy { PUBLIC, FRIENDS, PRIVATE } 

 // Custom Exception: Added for privacy violation (enhancement)
 class PrivacyViolationException extends Exception { 
     public PrivacyViolationException(String message) { super(message); } 
 }

 // Custom Exception: Added for invalid login (enhancement)
 class InvalidLoginException extends Exception { 
     public InvalidLoginException(String message) { super(message); } 
 }

 // Mediator: Added for user interactions (enhancement with mediator pattern)
 interface ChatMediator { 
     void sendMessage(User from, User to, String message); 
 }

 // Concrete Mediator: Added for chat
 class FacebookChat implements ChatMediator { 
     @Override 
     public void sendMessage(User from, User to, String message) { 
         // Send logic (not implemented) 
     } 
 }

 // Abstract Class: Post 
 abstract class Post implements Serializable { // Added Serializable
     private final String postId;  // Final 
     protected User author; 
     protected String content; 
     protected LocalDateTime timestamp; 
     protected Privacy privacy = Privacy.FRIENDS; 
     private List<Comment> comments = new ArrayList<>(); 
     private int likes; 
     private Map<String, Integer> reactions = new HashMap<>(); // Added for reactions (enhancement, e.g., "love": 5)

     public Post(User author, String content) { 
         this.postId = UUID.randomUUID().toString(); // Changed to UUID
         this.author = author; 
         this.content = content; 
         this.timestamp = LocalDateTime.now(); 
         this.likes = 0; 
     } 

     public abstract void display();  // Abstract 

     public void like() { likes++; } 
     public void addComment(Comment comment) { comments.add(comment); } 
     public void react(String reactionType) { // Added method
         reactions.put(reactionType, reactions.getOrDefault(reactionType, 0) + 1); 
     } 
     public Map<String, Integer> getReactions() { return reactions; } // Added getter
 } 

 // Inheritance: TextPost 
 class TextPost extends Post { 
     public TextPost(User author, String content) { 
         super(author, content); 
     } 

     @Override 
     public void display() { 
         // Show text 
     } 
 } 

 // ImagePost 
 class ImagePost extends Post { 
     private String imageUrl; 

     public ImagePost(User author, String content, String imageUrl) { 
         super(author, content); 
         this.imageUrl = imageUrl; 
     } 

     @Override 
     public void display() { 
         // Show image + caption 
     } 
 } 

 // Class: Comment 
 class Comment implements Serializable { // Added Serializable
     private User author; 
     private String text; 

     public Comment(User author, String text) { 
         this.author = author; 
         this.text = text; 
     } 
     public User getAuthor() { return author; } // Added getter (enhancement)
 } 

 // Interface: Shareable 
 interface Shareable { 
     void shareWith(User user) throws PrivacyViolationException; // Enhanced with exception
 } 

 // Class: User 
 class User implements Shareable { 
     private String userId; 
     private String name; 
     private String password;  // Private 
     private List<User> friends = new ArrayList<>();  // Composition for network 
     private List<Post> timeline = new ArrayList<>();  // Composition 
     private ChatMediator chatMediator; // Added for mediator (enhancement)

     public User(String name, String password) { 
         this.userId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
         this.password = password; 
         this.chatMediator = new FacebookChat(); // Added
     } 

     public void addFriend(User friend) { friends.add(friend); } 
     public void createPost(Post post) { timeline.add(post); } 

     @Override 
     public void shareWith(User user) throws PrivacyViolationException { 
         if (privacy == Privacy.PRIVATE) throw new PrivacyViolationException("Cannot share private post"); 
         // Share post to user if friend 
     } 

     public List<Post> getNewsFeed() { 
         // Aggregate from friends 
         return null; 
     } 
     public void sendMessage(User to, String message) { // Added using mediator
         chatMediator.sendMessage(this, to, message); 
     } 
 } 

 // Class: Group 
 class Group implements Serializable { // Added Serializable
     private String groupId; 
     private String name; 
     private List<User> members = new ArrayList<>();  // Aggregation 
     private List<Post> groupPosts = new ArrayList<>(); 

     public Group(String name) { 
         this.groupId = UUID.randomUUID().toString(); // Changed to UUID
         this.name = name; 
     } 

     public void addMember(User user) { members.add(user); } 
     public void addPost(Post post) { groupPosts.add(post); } 
     public List<User> getMembers() { return members; } // Added getter (enhancement)
 } 

 // Singleton: SocialNetwork 
 class SocialNetwork { 
     private static volatile SocialNetwork instance; 
     private List<User> users = new ArrayList<>(); 
     private List<Group> groups = new ArrayList<>(); 
     private static List<Post> globalFeed = new ArrayList<>();  // Static 

     private SocialNetwork() {} 

     public static SocialNetwork getInstance() { 
         if (instance == null) { 
             synchronized (SocialNetwork.class) { 
                 if (instance == null) instance = new SocialNetwork(); 
             } 
         } 
         return instance; 
     } 

     public void addUser(User user) { users.add(user); } 
     public void createGroup(Group group) { groups.add(group); } 
     public User login(String name, String password) throws InvalidLoginException { // Added exception
         // Authenticate 
         for (User u : users) { 
             if (u.name.equals(name) && u.password.equals(password)) return u; 
         } 
         throw new InvalidLoginException("Invalid credentials"); 
     } 
     public List<Post> getTrendingPosts() { 
         // Based on likes 
         return null; 
     } 
     public List<Post> getGlobalFeed() { return globalFeed; } // Added getter (enhancement)
 } 
 ``` 

 #### How It Works: 
 - **Posting**: Create post type, add to timeline. 
 - **Sharing**: Via interface, check privacy. 
 - **Polymorphism**: Posts display differently. 
 - **Singleton**: Network manages users/groups. 
 - **Mediator**: Handles messaging between users (added enhancement). 
 - **Reactions**: Posts support multiple reaction types (added enhancement).
