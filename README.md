# codeAlpha_-Stock-Trading-Platform

import java.util.*;
import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

class Stock {
    private String symbol;
    private String name;
    private double price;

    public Stock(String symbol, String name, double price) {
        this.symbol = symbol;
        this.name = name;
        this.price = price;
    }

    public String getSymbol() {
        return symbol;
    }

    public String getName() {
        return name;
    }

    public double getPrice() {
        return price;
    }

    public void updatePrice(double price) {
        this.price = price;
    }

    public void display() {
        System.out.printf("%-10s %-20s Rs. %.2f%n",
                symbol, name, price);
    }
}

class Transaction {
    String type;
    String symbol;
    int quantity;
    double price;
    String time;

    public Transaction(String type, String symbol,
                       int quantity, double price) {
        this.type = type;
        this.symbol = symbol;
        this.quantity = quantity;
        this.price = price;
        this.time = LocalDateTime.now().format(
                DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss"));
    }

    public void display() {
        System.out.printf("%-6s %-8s Qty: %-4d Price: Rs. %.2f | %s%n",
                type, symbol, quantity, price, time);
    }
}

class Trader {
    private String name;
    private double cash;
    private double invested = 0;

    private Map<String, Integer> portfolio = new LinkedHashMap<>();
    private List<Transaction> transactions = new ArrayList<>();

    public Trader(String name, double cash) {
        this.name = name;
        this.cash = cash;
    }

    public void buy(Stock stock, int quantity) {
        if (quantity <= 0) {
            System.out.println("Quantity must be positive.");
            return;
        }

        double cost = stock.getPrice() * quantity;

        if (cost > cash) {
            System.out.println("Insufficient cash balance!");
            return;
        }

        cash -= cost;
        invested += cost;

        portfolio.put(stock.getSymbol(),
                portfolio.getOrDefault(stock.getSymbol(), 0) + quantity);

        transactions.add(new Transaction(
                "BUY", stock.getSymbol(), quantity, stock.getPrice()));

        System.out.println("Stock purchased successfully!");
    }

    public void sell(Stock stock, int quantity) {
        if (quantity <= 0) {
            System.out.println("Quantity must be positive.");
            return;
        }

        int owned = portfolio.getOrDefault(stock.getSymbol(), 0);

        if (quantity > owned) {
            System.out.println("You do not own enough shares!");
            return;
        }

        double revenue = stock.getPrice() * quantity;
        cash += revenue;

        portfolio.put(stock.getSymbol(), owned - quantity);

        transactions.add(new Transaction(
                "SELL", stock.getSymbol(), quantity, stock.getPrice()));

        System.out.println("Stock sold successfully!");
    }

    public void showPortfolio(Map<String, Stock> market) {
        double marketValue = 0;

        System.out.println("\n===== YOUR PORTFOLIO =====");
        System.out.printf("%-10s %-10s %-15s%n",
                "Symbol", "Quantity", "Market Value");

        for (String symbol : portfolio.keySet()) {
            int quantity = portfolio.get(symbol);

            if (quantity > 0) {
                Stock stock = market.get(symbol);
                double value = stock.getPrice() * quantity;
                marketValue += value;

                System.out.printf("%-10s %-10d Rs. %.2f%n",
                        symbol, quantity, value);
            }
        }

        System.out.printf("%nCash Balance : Rs. %.2f%n", cash);
        System.out.printf("Stock Value  : Rs. %.2f%n", marketValue);
        System.out.printf("Total Value  : Rs. %.2f%n", cash + marketValue);
        System.out.printf("Realized P/L : Rs. %.2f%n", cash - 100000);

        System.out.println(
                "Note: Realized P/L above assumes initial cash was Rs. 100000.");
    }

    public void showTransactions() {
        System.out.println("\n===== TRANSACTION HISTORY =====");

        if (transactions.isEmpty()) {
            System.out.println("No transactions yet.");
            return;
        }

        for (Transaction t : transactions) {
            t.display();
        }
    }

    public void saveToFile() {
        try (PrintWriter writer =
                     new PrintWriter(new FileWriter("trading_history.txt", true))) {
            for (Transaction t : transactions) {
                writer.println(t.type + "," + t.symbol + "," +
                        t.quantity + "," + t.price + "," + t.time);
            }
            System.out.println("Transaction history saved to trading_history.txt");
        } catch (IOException e) {
            System.out.println("Unable to save file: " + e.getMessage());
        }
    }

    public double getCash() {
        return cash;
    }
}

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        Map<String, Stock> market = new LinkedHashMap<>();
        market.put("TCS", new Stock("TCS", "Tata Consultancy", 3500));
        market.put("INFY", new Stock("INFY", "Infosys", 1800));
        market.put("RELIANCE", new Stock("RELIANCE", "Reliance", 2900));
        market.put("WIPRO", new Stock("WIPRO", "Wipro", 550));

        System.out.print("Enter trader name: ");
        String name = sc.nextLine();

        Trader trader = new Trader(name, 100000);

        int choice;

        do {
            System.out.println("\n===== STOCK TRADING SIMULATOR =====");
            System.out.println("Trader: " + name);
            System.out.println("1. View Market Data");
            System.out.println("2. Buy Stocks");
            System.out.println("3. Sell Stocks");
            System.out.println("4. View Portfolio");
            System.out.println("5. View Transaction History");
            System.out.println("6. Simulate Market Price Change");
            System.out.println("7. Save Transaction History");
            System.out.println("0. Exit");
            System.out.print("Enter choice: ");

            if (!sc.hasNextInt()) {
                System.out.println("Please enter a valid number.");
                sc.nextLine();
                continue;
            }

            choice = sc.nextInt();

            switch (choice) {
                case 1:
                    System.out.println("\n===== MARKET DATA =====");
                    System.out.printf("%-10s %-20s %s%n",
                            "Symbol", "Company", "Price");
                    for (Stock s : market.values()) {
                        s.display();
                    }
                    break;

                case 2:
                case 3:
                    System.out.print("Enter stock symbol: ");
                    String symbol = sc.next().toUpperCase();

                    Stock stock = market.get(symbol);

                    if (stock == null) {
                        System.out.println("Stock not found!");
                        break;
                    }

                    System.out.print("Enter quantity: ");
                    if (!sc.hasNextInt()) {
                        System.out.println("Invalid quantity.");
                        sc.nextLine();
                        break;
                    }

                    int quantity = sc.nextInt();

                    if (choice == 2) {
                        trader.buy(stock, quantity);
                    } else {
                        trader.sell(stock, quantity);
                    }
                    break;

                case 4:
                    trader.showPortfolio(market);
                    break;

                case 5:
                    trader.showTransactions();
                    break;

                case 6:
                    Random random = new Random();

                    for (Stock s : market.values()) {
                        double change = (random.nextDouble() * 0.10) - 0.05;
                        double newPrice = s.getPrice() * (1 + change);
                        s.updatePrice(Math.round(newPrice * 100.0) / 100.0);
                    }

                    System.out.println("Market prices updated!");
                    break;

                case 7:
                    trader.saveToFile();
                    break;

                case 0:
                    System.out.println("Thank you for using the simulator!");
                    break;

                default:
                    System.out.println("Invalid choice!");
            }

        } while (choice != 0);

        sc.close();
    }
}
