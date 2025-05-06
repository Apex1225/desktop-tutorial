# Welcome to GitHub Desktop!

This is your README. READMEs are where you can communicate what your project is and how to use it.

Write your name on line 6, save it, and then head back to GitHub Desktop.
Karan Kumawat

import java.io.*;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Scanner;

class Transaction implements Serializable {
    private final Date date;
    private final String type;
    private final double amount;
    private final double balanceAfter;

    public Transaction(String type, double amount, double balanceAfter) {
        this.date = new Date();
        this.type = type;
        this.amount = amount;
        this.balanceAfter = balanceAfter;
    }

    public String getFormattedDate() {
        return new SimpleDateFormat("dd-MM-yyyy HH:mm:ss").format(date);
    }

    @Override
    public String toString() {
        return String.format("%-20s %-10s %12.2f %14.2f",
                getFormattedDate(),
                type,
                amount,
                balanceAfter);
    }
}

class User implements Serializable {
    private final String userID;
    private String userPIN;
    private double balance;
    private final List<Transaction> transactions;

    public User(String userID, String userPIN, double balance) {
        this.userID = userID;
        this.userPIN = userPIN;
        this.balance = balance;
        this.transactions = new ArrayList<>();
        // Initial balance transaction
        addTransaction("Account Open", balance);
    }

    public void addTransaction(String type, double amount) {
        transactions.add(new Transaction(type, amount, balance));
    }

    // Getters and setters
    public String getUserID() { return userID; }
    public String getUserPIN() { return userPIN; }
    public double getBalance() { return balance; }
    public List<Transaction> getTransactions() { return transactions; }
    
    public void setUserPIN(String pin) { userPIN = pin; }
    public void setBalance(double balance) { this.balance = balance; }
}

class ATM {
    private static final String DATA_FILE = "atm_users.dat";
    private User currentUser;
    private final Scanner scanner;
    private List<User> users;

    public ATM() {
        this.scanner = new Scanner(System.in);
        loadUsers();
    }

    public void start() {
        showWelcomeScreen();
        scanner.close();
    }

    private void showWelcomeScreen() {
        while(true) {
            System.out.println("\n=== Welcome to MyBank ATM ===");
            System.out.println("1. Login");
            System.out.println("2. Exit");
            System.out.print("Enter choice: ");
            
            String choice = scanner.nextLine();
            
            if(choice.equals("1")) {
                if(authenticateUser()) {
                    showMainMenu();
                }
            } else if(choice.equals("2")) {
                System.out.println("Thank you for using MyBank ATM!");
                saveUsers();
                System.exit(0);
            } else {
                System.out.println("Invalid choice!");
            }
        }
    }

    private boolean authenticateUser() {
        System.out.print("\nEnter User ID: ");
        String userId = scanner.nextLine();
        System.out.print("Enter PIN: ");
        String pin = scanner.nextLine();

        for(User user : users) {
            if(user.getUserID().equals(userId) && user.getUserPIN().equals(pin)) {
                currentUser = user;
                System.out.println("\nLogin successful!");
                return true;
            }
        }
        System.out.println("Invalid credentials!");
        return false;
    }

    private void showMainMenu() {
        while(true) {
            System.out.println("\n=== Main Menu ===");
            System.out.println("1. Check Balance");
            System.out.println("2. Withdraw Cash");
            System.out.println("3. Deposit Cash");
            System.out.println("4. Change PIN");
            System.out.println("5. View Statement");
            System.out.println("6. Logout");
            System.out.print("Enter choice: ");

            String choice = scanner.nextLine();

            switch(choice) {
                case "1": checkBalance(); break;
                case "2": withdraw(); break;
                case "3": deposit(); break;
                case "4": changePIN(); break;
                case "5": viewStatement(); break;
                case "6": 
                    System.out.println("Logging out...");
                    saveUsers();
                    return;
                default: System.out.println("Invalid choice!");
            }
        }
    }

    private void checkBalance() {
        System.out.printf("\nCurrent Balance: ₹%.2f%n", currentUser.getBalance());
    }

    private void withdraw() {
        System.out.print("\nEnter withdrawal amount: ");
        try {
            double amount = Double.parseDouble(scanner.nextLine());
            if(amount <= 0) {
                System.out.println("Amount must be positive!");
                return;
            }
            
            if(amount > currentUser.getBalance()) {
                System.out.println("Insufficient funds!");
                return;
            }
            
            currentUser.setBalance(currentUser.getBalance() - amount);
            currentUser.addTransaction("Withdrawal", -amount);
            saveUsers();
            
            System.out.println("Please collect your cash!");
            System.out.printf("New balance: ₹%.2f%n", currentUser.getBalance());
        } catch(NumberFormatException e) {
            System.out.println("Invalid amount!");
        }
    }

    private void deposit() {
        System.out.print("\nEnter deposit amount: ");
        try {
            double amount = Double.parseDouble(scanner.nextLine());
            if(amount <= 0) {
                System.out.println("Amount must be positive!");
                return;
            }
            
            currentUser.setBalance(currentUser.getBalance() + amount);
            currentUser.addTransaction("Deposit", amount);
            saveUsers();
            
            System.out.println("Amount deposited successfully!");
            System.out.printf("New balance: ₹%.2f%n", currentUser.getBalance());
        } catch(NumberFormatException e) {
            System.out.println("Invalid amount!");
        }
    }

    private void changePIN() {
        System.out.print("\nEnter current PIN: ");
        String currentPin = scanner.nextLine();
        
        if(!currentPin.equals(currentUser.getUserPIN())) {
            System.out.println("Incorrect current PIN!");
            return;
        }
        
        System.out.print("Enter new PIN: ");
        String newPin = scanner.nextLine();
        System.out.print("Confirm new PIN: ");
        String confirmPin = scanner.nextLine();
        
        if(newPin.equals(confirmPin)) {
            currentUser.setUserPIN(newPin);
            saveUsers();
            System.out.println("PIN changed successfully!");
        } else {
            System.out.println("PINs don't match!");
        }
    }

    private void viewStatement() {
        List<Transaction> transactions = currentUser.getTransactions();
        System.out.println("\n=== Account Statement ===");
        System.out.printf("%-20s %-10s %12s %14s%n",
                "Date/Time", "Type", "Amount", "Balance After");
        
        for(Transaction t : transactions) {
            System.out.println(t);
        }
    }

    @SuppressWarnings("unchecked")
    private void loadUsers() {
        try(ObjectInputStream ois = new ObjectInputStream(new FileInputStream(DATA_FILE))) {
            users = (List<User>) ois.readObject();
        } catch(Exception e) {
            // Create default admin if no data file exists
            System.out.println("Creating new user database...");
            users = new ArrayList<>();
            User admin = new User("Admin", "1111", 100000);
            users.add(admin);
            saveUsers();
        }
    }

    private void saveUsers() {
        try(ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(DATA_FILE))) {
            oos.writeObject(users);
        } catch(IOException e) {
            System.out.println("Error saving user data!");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        ATM atm = new ATM();
        atm.start();
    }
}
