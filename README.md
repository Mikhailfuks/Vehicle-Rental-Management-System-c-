using System;
using System.Collections.Generic;
using System.Linq;

// Namespace: Defines a container to hold related classes.
namespace VehicleRentalSystem
{
    // **Model Layer:** Represents the data and business logic.

    // Class: Vehicle - Represents a generic vehicle.
    public class Vehicle
    {
        // Properties: Characteristics of a vehicle.
        public int VehicleId { get; set; } // Unique identifier for the vehicle.
        public string Make { get; set; } // Manufacturer of the vehicle (e.g., Toyota, Ford).
        public string Model { get; set; } // Specific model of the vehicle (e.g., Camry, F-150).
        public string LicensePlate { get; set; } // The vehicle's license plate number.
        public decimal RentalRatePerDay { get; set; } // The cost to rent the vehicle per day.
        public bool IsAvailable { get; set; } // Indicates if the vehicle is currently available for rent.
        public VehicleType Type { get; set; } // Enum representing the type of vehicle.

        // Constructor: Initializes a new instance of the Vehicle class.
        public Vehicle(int vehicleId, string make, string model, string licensePlate, decimal rentalRatePerDay, VehicleType type)
        {
            VehicleId = vehicleId;
            Make = make;
            Model = model;
            LicensePlate = licensePlate;
            RentalRatePerDay = rentalRatePerDay;
            IsAvailable = true; // Initially, vehicles are assumed to be available.
            Type = type;
        }

        // Method: Override ToString() for easy display of vehicle information.
        public override string ToString()
        {
            return $"{Make} {Model} ({LicensePlate}) - ${RentalRatePerDay}/day - Available: {IsAvailable}";
        }
    }

    // Enum: VehicleType - Represents the different types of vehicles.
    public enum VehicleType
    {
        Car,
        Truck,
        SUV,
        Motorcycle,
        Van
    }

    // Class: Rental - Represents a rental agreement.
    public class Rental
    {
        // Properties: Details of a rental agreement.
        public int RentalId { get; set; } // Unique identifier for the rental.
        public int VehicleId { get; set; } // Foreign key referencing the rented vehicle.
        public int CustomerId { get; set; } // Foreign key referencing the customer renting the vehicle.
        public DateTime StartDate { get; set; } // The date the rental period begins.
        public DateTime EndDate { get; set; } // The date the rental period ends.
        public decimal TotalCost { get; set; } // The total cost of the rental.

        // Constructor: Initializes a new instance of the Rental class.
        public Rental(int rentalId, int vehicleId, int customerId, DateTime startDate, DateTime endDate)
        {
            RentalId = rentalId;
            VehicleId = vehicleId;
            CustomerId = customerId;
            StartDate = startDate;
            EndDate = endDate;
        }

        // Method: CalculateRentalCost - Calculates the total cost of the rental based on the rental rate and duration.
        public decimal CalculateRentalCost(decimal rentalRatePerDay)
        {
            TimeSpan rentalDuration = EndDate - StartDate;
            int numberOfDays = rentalDuration.Days;

            TotalCost = numberOfDays * rentalRatePerDay;
            return TotalCost;
        }

        // Method: Override ToString() for easy display of rental information.
        public override string ToString()
        {
            return $"Rental ID: {RentalId}, Vehicle ID: {VehicleId}, Start: {StartDate:d}, End: {EndDate:d}, Cost: ${TotalCost}";
        }
    }

    // Class: Customer - Represents a customer.
    public class Customer
    {
        // Properties: Information about a customer.
        public int CustomerId { get; set; } // Unique identifier for the customer.
        public string FirstName { get; set; } // Customer's first name.
        public string LastName { get; set; } // Customer's last name.
        public string Email { get; set; } // Customer's email address.
        public string PhoneNumber { get; set; } // Customer's phone number.

        // Constructor: Initializes a new instance of the Customer class.
        public Customer(int customerId, string firstName, string lastName, string email, string phoneNumber)
        {
            CustomerId = customerId;
            FirstName = firstName;
            LastName = lastName;
            Email = email;
            PhoneNumber = phoneNumber;
        }

        // Method: Override ToString() for easy display of customer information.
        public override string ToString()
        {
            return $"{FirstName} {LastName} ({Email}) - {PhoneNumber}";
        }
    }

    // **Data Access Layer:**  Handles data storage and retrieval.

    // Class:  RentalDatabase - Simulates a database using lists in memory.
    public class RentalDatabase
    {
        // Properties: Lists to store data.
        public List<Vehicle> Vehicles { get; set; } // List of all vehicles.
        public List<Rental> Rentals { get; set; } // List of all rentals.
        public List<Customer> Customers { get; set; } // List of all customers.

        // Constructor: Initializes the database and populates it with sample data.
        public RentalDatabase()
        {
            Vehicles = new List<Vehicle>();
            Rentals = new List<Rental>();
            Customers = new List<Customer>();

            // Sample Data: Adding some initial vehicles, customers, and rentals.
            // Vehicles
            Vehicles.Add(new Vehicle(1, "Toyota", "Camry", "ABC-123", 35.00m, VehicleType.Car));
            Vehicles.Add(new Vehicle(2, "Ford", "F-150", "DEF-456", 50.00m, VehicleType.Truck));
            Vehicles.Add(new Vehicle(3, "Honda", "CR-V", "GHI-789", 40.00m, VehicleType.SUV));

            // Customers
            Customers.Add(new Customer(101, "John", "Doe", "john.doe@example.com", "555-123-4567"));
            Customers.Add(new Customer(102, "Jane", "Smith", "jane.smith@example.com", "555-987-6543"));

            // Rentals
            Rentals.Add(new Rental(201, 1, 101, DateTime.Now.AddDays(1), DateTime.Now.AddDays(5))); // John Doe rents the Camry
            Rentals.Add(new Rental(202, 2, 102, DateTime.Now.AddDays(3), DateTime.Now.AddDays(7))); // Jane Smith rents the F-150

            // Now that there are rentals, calculate the costs and update availability
            foreach (var rental in Rentals)
            {
                Vehicle rentedVehicle = Vehicles.FirstOrDefault(v => v.VehicleId == rental.VehicleId);
                if (rentedVehicle != null)
                {
                    rental.CalculateRentalCost(rentedVehicle.RentalRatePerDay);
                    rentedVehicle.IsAvailable = false; // Mark the vehicle as unavailable
                }
            }
        }
    }

    // **Business Logic Layer:** Contains the core logic of the application.

    // Class: RentalManager - Provides methods for managing rentals, vehicles, and customers.
    public class RentalManager
    {
        // Property:  Reference to the data access layer.
        public RentalDatabase Database { get; set; }

        // Constructor: Initializes the RentalManager with a RentalDatabase.
        public RentalManager(RentalDatabase database)
        {
            Database = database;
        }

        // Method: FindAvailableVehicles - Returns a list of vehicles that are currently available for rent.
        public List<Vehicle> FindAvailableVehicles()
        {
            return Database.Vehicles.Where(v => v.IsAvailable).ToList();
        }

        // Method: RentVehicle - Creates a new rental agreement.
        public Rental RentVehicle(int vehicleId, int customerId, DateTime startDate, DateTime endDate)
        {
            // Validation: Check if the vehicle and customer exist.
            Vehicle vehicle = Database.Vehicles.FirstOrDefault(v => v.VehicleId == vehicleId);
            Customer customer = Database.Customers.FirstOrDefault(c => c.CustomerId == customerId);

            if (vehicle == null)
            {
                throw new ArgumentException($"Vehicle with ID {vehicleId} not found.");
            }
            if (customer == null)
            {
                throw new ArgumentException($"Customer with ID {customerId} not found.");
            }
            if (!vehicle.IsAvailable)
            {
                throw new InvalidOperationException($"Vehicle {vehicle.Make} {vehicle.Model} is not available.");
            }

            // Create Rental: Create a new rental object.
            int newRentalId = Database.Rentals.Any() ? Database.Rentals.Max(r => r.RentalId) + 1 : 1;
            Rental rental = new Rental(newRentalId, vehicleId, customerId, startDate, endDate);

            // Calculate Cost: Calculate the rental cost.
            rental.CalculateRentalCost(vehicle.RentalRatePerDay);

            // Update State: Mark the vehicle as unavailable and add the rental to the database.
            vehicle.IsAvailable = false;
            Database.Rentals.Add(rental);

            return rental;
        }

        // Method: ReturnVehicle - Ends a rental agreement and makes the vehicle available again.
        public void ReturnVehicle(int rentalId)
        {
            // Find Rental: Locate the rental agreement.
            Rental rental = Database.Rentals.FirstOrDefault(r => r.RentalId == rentalId);
            if (rental == null)
            {
                throw new ArgumentException($"Rental with ID {rentalId} not found.");
            }

            // Find Vehicle:  Locate the vehicle associated with the rental.
            Vehicle vehicle = Database.Vehicles.FirstOrDefault(v => v.VehicleId == rental.VehicleId);
            if (vehicle == null)
            {
                throw new InvalidOperationException($"Vehicle with ID {rental.VehicleId} not found.");
            }

            // Update State: Mark the vehicle as available.
            vehicle.IsAvailable = true;
        }

        // Method: GetRentalDetails - Retrieves rental details based on rentalId.
        public Rental GetRentalDetails(int rentalId)
        {
            return Database.Rentals.FirstOrDefault(r => r.RentalId == rentalId);
        }

        // Method: ListRentals - Lists all rental agreements.
        public List<Rental> ListRentals()
        {
            return Database.Rentals;
        }

        // Method: AddNewVehicle - Adds a new vehicle to the system
        public void AddNewVehicle(Vehicle newVehicle)
        {
            // Generate a unique ID
            newVehicle.VehicleId = Database.Vehicles.Any() ? Database.Vehicles.Max(v => v.VehicleId) + 1 : 1;
            Database.Vehicles.Add(newVehicle);
        }

        // Method: FindCustomerById - finds customer based on Id.
        public Customer FindCustomerById(int id)
        {
            return Database.Customers.FirstOrDefault(r => r.CustomerId == id);
        }

        // Method: ListCustomers - Lists all customers
        public List<Customer> ListCustomers()
        {
            return Database.Customers;
        }
    }

    // **Presentation Layer:** (Illustrative - No actual UI code) -  Would handle user interaction.

    // Class:  Program -  Illustrates how the system might be used.
    public class Program
    {
        // Method: Main - Entry point of the application.
        public static void Main(string[] args)
        {
            // Initialization: Create the database and rental manager.
            RentalDatabase database = new RentalDatabase();
            RentalManager manager = new RentalManager(database);

            // Usage Example: Find available vehicles.
            Console.WriteLine("Available Vehicles:");
            List<Vehicle> availableVehicles = manager.FindAvailableVehicles();
            foreach (var vehicle in availableVehicles)
            {
                Console.WriteLine(vehicle);
            }

            // Usage Example: List Customers
            Console.WriteLine("\nList of Customers:");
            List<Customer> customers = manager.ListCustomers();
            foreach (var customer in customers)
            {
                Console.WriteLine(customer);
            }

            // Usage Example: Find a customer
            Console.WriteLine("\nLooking for Customer ID 101");
            Customer foundCustomer = manager.FindCustomerById(101);
            if (foundCustomer != null)
                Console.WriteLine($"Found Customer: {foundCustomer.FirstName} {foundCustomer.LastName}");
            else
                Console.WriteLine("Customer not found");

            // Usage Example: Rent a vehicle.
            try
            {
                Rental newRental = manager.RentVehicle(3, 101, DateTime.Now.AddDays(10), DateTime.Now.AddDays(14));
                Console.WriteLine("\nNew Rental Created:");
                Console.WriteLine(newRental);

                Console.WriteLine("\nAvailable Vehicles After Rental:");
                availableVehicles = manager.FindAvailableVehicles();
                foreach (var vehicle in availableVehicles)
                {
                    Console.WriteLine(vehicle);
                }
            }
            catch (ArgumentException ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }
            catch (InvalidOperationException ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }

            // Usage Example: List all rentals.
            Console.WriteLine("\nAll Rentals:");
            List<Rental> allRentals = manager.ListRentals();
            foreach (var rental in allRentals)
            {
                Console.WriteLine(rental);
            }

            // Usage Example: Return a vehicle.
            try
            {
                manager.ReturnVehicle(201); //Return the rental with Id 201
                Console.WriteLine("\nVehicle Returned:");
                Console.WriteLine("\nAvailable Vehicles After Return:");
                availableVehicles = manager.FindAvailableVehicles();
                foreach (var vehicle in availableVehicles)
                {
                    Console.WriteLine(vehicle);
                }
            }
            catch (ArgumentException ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
            }

            // Usage Example: Display rental details.
            Console.WriteLine("\nRental Details for Rental ID 202:");
            Rental rentalDetails = manager.GetRentalDetails(202);
            if (rentalDetails != null)
            {
                Console.WriteLine(rentalDetails);
            }
            else
            {
                Console.WriteLine("Rental not found.");
            }

            Console.ReadKey(); // Pause to view the output.
        }
    }
}
