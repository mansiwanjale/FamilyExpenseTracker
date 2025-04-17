//package Family_Expence_Tracker;
import java.util.*;
import java.time.LocalDate;
import java.time.*;
import java.util.stream.Collectors;

class FamilyExpenseTracker {
    private static final Scanner scanner = new Scanner(System.in);
    private static final Map<String, String> users = new HashMap<>();
    private static final Map<String, FamilyMember> familyMembers = new HashMap<>();
    static final Map<String, ExpenseCategory> expenseCategories = new HashMap<>();
    private static final List<Expense> expenses = new ArrayList<>();
    private static final Map<String, Queue<String>> userAlerts = new HashMap<>();
    private static final TreeMap<String, ExpenseCategory> categoryTrie = new TreeMap<>();
    private static String loggedInUser = null;
    private static boolean isAdmin = false;
    static double totalFamilyIncome = 0;
    static double distributedBudget = 0;
    private static final Map<String, Map<String, Double>> debtGraph = new HashMap<>();
    private static final PriorityQueue<BudgetAlert> budgetAlerts = new PriorityQueue<>();
    private static final String[] MONTHS = {"Jan", "Feb", "Mar", "Apr", "May", "Jun", 
                                          "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"};
    private static final int MAX_CHART_WIDTH = 50;

    public static void main(String[] args) {
        initializeDefaultCategories();
        System.out.println("=== Family Expense Tracker ===");
        while (true) {
            System.out.println("1. Register\n2. Login\n3. Exit");
            System.out.print("Choose an option: ");
            int choice = scanner.nextInt();
            scanner.nextLine();
            switch (choice) {
                case 1 -> register();
                case 2 -> login();
                case 3 -> System.exit(0);
                default -> System.out.println("Invalid option, please try again.");
            }
        }
    }

    private static void initializeDefaultCategories() {
        String[] categories = {
            "Groceries", "Utilities", "Education", "Transport", "Health", 
            "Entertainment", "Rent", "Others", "Dining Out", "Shopping",
            "Insurance", "Taxes", "Savings", "Investments", "Gifts",
            "Donations", "Childcare", "Pets", "Travel", "Subscriptions",
            "Home Maintenance", "Vehicle Maintenance", "Clothing", "Personal Care", "Hobbies"
        };
        for (String cat : categories) {
            categoryTrie.put(cat.toLowerCase(), new ExpenseCategory(cat, 0));
        }
    }

    private static void register() {
        System.out.print("Enter Name: ");
        String name = scanner.nextLine();
        System.out.print("Enter Username: ");
        String username = scanner.nextLine();
        System.out.print("Enter Password: ");
        String password = scanner.nextLine();
        System.out.print("Choose role (admin/member): ");
        String role = scanner.nextLine().toLowerCase();

        users.put(username, password);
        userAlerts.put(username, new LinkedList<>());
        debtGraph.put(username, new HashMap<>());
        
        if (role.equals("admin")) {
            familyMembers.put(username, new FamilyMember(name, role, 0));
            System.out.println("Admin registered and added to family successfully!");
        } else {
            System.out.println("Member registered successfully! Admin needs to add you to the family.");
        }
    }

    private static void login() {
        System.out.print("Enter Username: ");
        String username = scanner.nextLine();
        System.out.print("Enter Password: ");
        String password = scanner.nextLine();

        if (users.containsKey(username) && users.get(username).equals(password)) {
            loggedInUser = username;
            isAdmin = familyMembers.containsKey(username) && familyMembers.get(username).getRole().equals("admin");

            System.out.println("Login successful! Welcome " + username);
            showAlerts();
            showDashboard();

            if (isAdmin) adminMenu();
            else memberMenu();
        } else {
            System.out.println("Invalid login credentials.");
        }
    }

    private static void showAlerts() {
        Queue<String> alerts = userAlerts.getOrDefault(loggedInUser, new LinkedList<>());
        if (!alerts.isEmpty()) {
            System.out.println("\n\uD83D\uDD14 You have new alerts:");
            while (!alerts.isEmpty()) {
                System.out.println("\uD83D\uDC49 " + alerts.poll());
            }
        } else {
            System.out.println("\u2705 No new alerts.");
        }
    }

    private static void showDashboard() {
        System.out.println("\n=== DASHBOARD ===");
        FamilyMember member = familyMembers.get(loggedInUser);
        
        if (member == null) {
            System.out.println("You are not yet added to the family. Please ask admin to add you.");
            return;
        }
        
        // Calculate personal spending
        double personalSpent = expenses.stream()
            .filter(e -> e.getUsername().equals(loggedInUser))
            .mapToDouble(Expense::getAmount)
            .sum();
        
        // Calculate debts and credits
        double totalDebt = debtGraph.get(loggedInUser).values().stream().mapToDouble(Double::doubleValue).sum();
        double totalCredit = debtGraph.values().stream()
            .filter(m -> m.containsKey(loggedInUser))
            .mapToDouble(m -> m.get(loggedInUser))
            .sum();
        
        System.out.printf("Personal Income: ₹%.2f\n", member.getPersonalIncome());
        System.out.printf("Personal Spending: ₹%.2f\n", personalSpent);
        System.out.printf("Remaining Balance: ₹%.2f\n", member.getPersonalIncome() - personalSpent);
        System.out.printf("Total Debt: ₹%.2f\n", totalDebt);
        System.out.printf("Total Credit: ₹%.2f\n", totalCredit);
        
        if (isAdmin) {
            double familySpent = expenses.stream().mapToDouble(Expense::getAmount).sum();
            System.out.println("\nFamily Financial Status:");
            System.out.printf("Total Family Income: ₹%.2f\n", totalFamilyIncome);
            System.out.printf("Total Family Spending: ₹%.2f\n", familySpent);
            System.out.printf("Remaining Family Budget: ₹%.2f\n", totalFamilyIncome - familySpent);
            double utilization = (familySpent / totalFamilyIncome) * 100;
            System.out.printf("Budget Utilization: %.1f%%\n", utilization);
            
            // Add budget utilization alerts
            if (utilization >= 100) {
                System.out.println("\uD83D\uDD25 WARNING: Family budget fully utilized (100%)!");
            } else if (utilization >= 90) {
                System.out.println("\uD83D\uDD25 WARNING: Family budget nearly exhausted (90%)!");
            } else if (utilization >= 50) {
                System.out.println("\uD83D\uDD25 Notice: Family budget 50% utilized");
            }
        }
        
        // Show recent expenses
        System.out.println("\nRecent Expenses:");
        expenses.stream()
            .filter(e -> e.getUsername().equals(loggedInUser))
            .sorted(Comparator.comparing(Expense::getLocalDate).reversed())
            .limit(5)
            .forEach(e -> System.out.printf("%s: %s ₹%.2f\n", 
                e.getDate(), e.getCategory(), e.getAmount()));
    }

    private static void adminMenu() {
        while (true) {
            System.out.println("\n--- Admin Menu ---");
            System.out.println("1. Add Family Member\n2. Remove Family Member\n3. View Family Members");
            System.out.println("4. Add Personal Income\n5. Manage Expense Categories\n6. Add/View Expenses");
            System.out.println("7. View Expense Trends\n8. Debt Settlement\n9. Logout");
            System.out.print("Choose option: ");
            int choice = scanner.nextInt();
            scanner.nextLine();
            switch (choice) {
                case 1 -> addFamilyMember();
                case 2 -> removeFamilyMember();
                case 3 -> viewFamilyMembers();
                case 4 -> addPersonalIncome();
                case 5 -> manageExpenseCategories();
                case 6 -> addViewExpenses();
                case 7 -> showExpenseTrends();
                case 8 -> settleDebts();
                case 9 -> { loggedInUser = null; isAdmin = false; return; }
                default -> System.out.println("Invalid option, please try again.");
            }
        }
    }

    private static void memberMenu() {
        while (true) {
            System.out.println("\n--- Member Menu ---");
            System.out.println("1. Add Personal Income\n2. View Family Members\n3. Add/View Expenses");
            System.out.println("4. View Expense Trends\n5. View Alerts\n6. Dashboard\n7. Logout");
            System.out.print("Choose option: ");
            int choice = scanner.nextInt();
            scanner.nextLine();
            switch (choice) {
                case 1 -> addPersonalIncome();
                case 2 -> viewFamilyMembers();
                case 3 -> addViewExpenses();
                case 4 -> showExpenseTrends();
                case 5 -> showAlerts();
                case 6 -> showDashboard();
                case 7 -> { loggedInUser = null; return; }
                default -> System.out.println("Invalid option, please try again.");
            }
        }
    }

    private static void addFamilyMember() {
        System.out.print("Enter username of registered user to add: ");
        String username = scanner.nextLine();
        if (users.containsKey(username) && !familyMembers.containsKey(username)) {
            System.out.print("Enter name for family member: ");
            String name = scanner.nextLine();
            familyMembers.put(username, new FamilyMember(name, "member", 0));
            System.out.println(username + " added to family with role member.");
            
            // Initialize personal income to 0 and add to total family income
            totalFamilyIncome += 0;
        } else {
            System.out.println("User not found or already added.");
        }
    }

    private static void removeFamilyMember() {
        System.out.print("Enter username of registered user to remove: ");
        String username = scanner.nextLine();
        if (familyMembers.containsKey(username)) {
            // Subtract the member's income from total family income
            totalFamilyIncome -= familyMembers.get(username).getPersonalIncome();
            familyMembers.remove(username);
            System.out.println(username + " removed from family.");
        } else {
            System.out.println("User not found in family.");
        }
    }

    private static void viewFamilyMembers() {
        if (familyMembers.isEmpty()) {
            System.out.println("No family members added yet.");
            return;
        }
        
        System.out.println("\nFamily Members:");
        System.out.printf("%-15s %-10s %-15s\n", "Username", "Role", "Personal Income");
        familyMembers.forEach((username, member) -> 
            System.out.printf("%-15s %-10s ₹%-15.2f\n", 
                username, member.getRole(), member.getPersonalIncome()));
        
        System.out.printf("\nTotal Family Income: ₹%.2f\n", totalFamilyIncome);
    }

    private static void addPersonalIncome() {
        FamilyMember member = familyMembers.get(loggedInUser);
        if (member == null) {
            System.out.println("You are not yet added to the family. Please ask admin to add you.");
            return;
        }
        
        System.out.print("Enter your monthly income (₹): ");
        double income = scanner.nextDouble();
        scanner.nextLine();
        
        // Update total family income by subtracting old income and adding new income
        totalFamilyIncome -= member.getPersonalIncome();
        member.setPersonalIncome(income);
        totalFamilyIncome += income;
        
        System.out.printf("Personal income set to ₹%.2f\n", income);
        System.out.printf("Total family income is now ₹%.2f\n", totalFamilyIncome);
    }

    private static void manageExpenseCategories() {
        System.out.print("Admin, enter the monthly total budget: ");
        double totalBudget = scanner.nextDouble();
        scanner.nextLine();
        ManageExpense obj = new ManageExpense(scanner, categoryTrie, expenseCategories, totalBudget, distributedBudget);
        obj.manageCategories(expenseCategories, totalBudget);
    }

    private static void addViewExpenses() {
        while (true) {
            System.out.println("\n--- Expense Menu ---");
            System.out.println("1. Add Expense\n2. View Expense Summary\n3. Back");
            System.out.print("Choose option: ");
            int choice = scanner.nextInt();
            scanner.nextLine();

            switch (choice) {
                case 1 -> {
                    if (expenseCategories.isEmpty()) {
                        System.out.println("No categories available. Please ask admin to add categories first.");
                        return;
                    }
                    
                    System.out.println("Available categories:");
                    expenseCategories.forEach((cat, category) -> {
                        double monthlySpent = getMonthlySpending(cat);
                        double remaining = category.getLimit() - monthlySpent;
                        System.out.printf("- %s (Limit: ₹%.2f, Remaining: ₹%.2f)\n", 
                            cat, category.getLimit(), remaining > 0 ? remaining : 0);
                    });
                    
                    System.out.print("Enter category (or type 'more' to see all): ");
                    String categoryInput = scanner.nextLine();
                    
                    String category;
                    if (categoryInput.equalsIgnoreCase("more")) {
                        System.out.println("All available categories:");
                        expenseCategories.keySet().forEach(cat -> 
                            System.out.println("- " + cat + " (Limit: ₹" + expenseCategories.get(cat).getLimit() + ")"));
                        System.out.print("Enter category: ");
                        category = scanner.nextLine();
                    } else {
                        category = categoryInput;
                    }
                    
                    if (!expenseCategories.containsKey(category)) {
                        System.out.println("Invalid category.");
                        return;
                    }
                    
                    System.out.print("Enter amount (₹): ");
                    double amount;
                    try {
                        amount = scanner.nextDouble();
                        scanner.nextLine();
                    } catch (InputMismatchException e) {
                        System.out.println("Invalid amount. Please enter a valid number.");
                        scanner.nextLine();
                        continue;
                    }

                    System.out.print("Split with members (comma-separated usernames or 'none'): ");
                    String splitWith = scanner.nextLine();

                    // Check budget limits
                    checkBudgetLimits(category, amount);

                    System.out.print("Enter expense date (yyyy-MM-dd): ");
                    String dateStr = scanner.nextLine();
                    LocalDate date;
                    try {
                        date = LocalDate.parse(dateStr);
                    } catch (Exception e) {
                        System.out.println("Invalid date format. Using today's date.");
                        date = LocalDate.now();
                    }
                    
                    // Add expense for the payer
                    expenses.add(new Expense(loggedInUser, category, amount, dateStr, date));
                    
                    // Handle expense splitting
                    if (!splitWith.equalsIgnoreCase("none")) {
                        splitExpense(loggedInUser, splitWith, amount, category, dateStr, date);
                    }
                    
                    System.out.println("✅ Expense added successfully!");
                }
                case 2 -> viewFilteredExpenses();
                case 3 -> { return; }
                default -> System.out.println("Invalid option.");
            }
        }
    }

    private static void checkBudgetLimits(String category, double newAmount) {
        Queue<String> alerts = userAlerts.getOrDefault(loggedInUser, new LinkedList<>());
        
        // Check category limits
        if (expenseCategories.containsKey(category)) {
            double limit = expenseCategories.get(category).getLimit();
            if (limit > 0) {
                double monthlySpent = getMonthlySpending(category);
                double newTotal = monthlySpent + newAmount;
                double utilization = (newTotal / limit) * 100;
                
                if (utilization >= 80) {
                    BudgetAlert alert = new BudgetAlert(category, utilization);
                    budgetAlerts.removeIf(a -> a.category().equals(category));
                    budgetAlerts.add(alert);
                    
                    if (utilization >= 100) {
                        double exceeded = newTotal - limit;
                        alerts.add("⚠ WARNING: Exceeded " + category + " budget by " + 
                            String.format("%.1f%% (₹%.2f over limit)", utilization - 100, exceeded));
                    } else if (utilization >= 90) {
                        alerts.add("⚠ WARNING: Nearing " + category + " budget (90% reached)");
                    } else {
                        alerts.add("Notice: " + category + " budget 80% reached");
                    }
                }
            }
        }
        
        // Check for high expense
        if (newAmount > 10000) {
            alerts.add("⚠ High expense of ₹" + String.format("%.2f", newAmount) + " in " + category);
        }
        
        // Check total budget
        double familySpent = expenses.stream().mapToDouble(Expense::getAmount).sum();
        double familyUtilization = ((familySpent + newAmount) / totalFamilyIncome) * 100;
        
        // Add alerts for overall budget utilization
        if (familyUtilization >= 100) {
            alerts.add("⚠ CRITICAL: Family budget fully utilized (100%)!");
        } else if (familyUtilization >= 90) {
            alerts.add("⚠ WARNING: Family budget nearly exhausted (90%)!");
        } else if (familyUtilization >= 50) {
            alerts.add("Notice: Family budget 50% utilized");
        }
        
        userAlerts.put(loggedInUser, alerts);
    }

    private static double getMonthlySpending(String category) {
        return expenses.stream()
            .filter(e -> e.getCategory().startsWith(category)) // Include shared expenses
            .filter(e -> e.getLocalDate().getMonth() == LocalDate.now().getMonth())
            .filter(e -> e.getLocalDate().getYear() == LocalDate.now().getYear())
            .mapToDouble(Expense::getAmount)
            .sum();
    }

    private static void splitExpense(String payer, String participants, double amount, String category, String dateStr, LocalDate date) {
        String[] members = participants.split(",");
        int totalParticipants = members.length + 1; // Including payer
        double share = amount / totalParticipants;
        
        // Update the payer's expense to reflect only their share
        Expense payerExpense = expenses.stream()
            .filter(e -> e.getUsername().equals(payer))
            .filter(e -> e.getCategory().equals(category))
            .filter(e -> e.getDate().equals(dateStr))
            .findFirst()
            .orElse(null);
        
        if (payerExpense != null) {
            payerExpense.setAmount(share);
        }
        
        for (String member : members) {
            member = member.trim();
            if (!familyMembers.containsKey(member)) {
                System.out.println("Member " + member + " not found in family!");
                continue;
            }
            
            // Update debt graph
            debtGraph.get(member).merge(payer, share, Double::sum);
            
            // Add expense for the member (their share)
            expenses.add(new Expense(member, category + " (shared)", share, dateStr, date));
            
            // Add alert for the member
            Queue<String> alerts = userAlerts.getOrDefault(member, new LinkedList<>());
            alerts.add("⚡ You owe " + payer + " ₹" + String.format("%.2f", share) + " for " + category);
            userAlerts.put(member, alerts);
        }
    }
private static void viewFilteredExpenses() {
    System.out.println("\n--- View Filtered Expense Summary ---");
    System.out.println("1. Filter by Date Range");
    System.out.println("2. Filter by Category");
    System.out.println("3. Filter by Member");
    System.out.println("4. View All Expenses");
    System.out.print("Choose option: ");
    int choice = scanner.nextInt();
    scanner.nextLine();

    List<Expense> filtered = new ArrayList<>();

    switch (choice) {
        case 1 -> {
            System.out.print("Enter start date (YYYY-MM-DD): ");
            LocalDate startDate = LocalDate.parse(scanner.nextLine());
            System.out.print("Enter end date (YYYY-MM-DD): ");
            LocalDate endDate = LocalDate.parse(scanner.nextLine());

            filtered = expenses.stream()
                .filter(e -> !e.getLocalDate().isBefore(startDate) && !e.getLocalDate().isAfter(endDate))
                .sorted(Comparator.comparing(Expense::getLocalDate))
                .collect(Collectors.toList());
        }
        case 2 -> {
            System.out.print("Enter category: ");
            String category = scanner.nextLine();
            // Filter by category AND logged-in user only
            filtered = expenses.stream()
                .filter(e -> e.getCategory().equalsIgnoreCase(category))
                .filter(e -> e.getUsername().equals(loggedInUser))  // Only show logged-in user's expenses
                .sorted(Comparator.comparing(Expense::getLocalDate))  // Sort by date
                .collect(Collectors.toList());
        }
        case 3 -> {
            System.out.print("Enter member username: ");
            String member = scanner.nextLine();
            // Group by category for the selected member (all time, not filtered by date)
            Map<String, Double> categoryTotals = expenses.stream()
                .filter(e -> e.getUsername().equalsIgnoreCase(member))
                .collect(Collectors.groupingBy(
                    e -> e.getCategory().replace(" (shared)", ""),
                    Collectors.summingDouble(Expense::getAmount)
                ));

            // Display category-wise totals for the member
            System.out.println("\nExpense Summary for " + member + " by Category:");
            System.out.println("+----------------+------------+");
            System.out.println("| Category       | Amount     |");
            System.out.println("+----------------+------------+");
            
            categoryTotals.forEach((category, total) -> 
                System.out.printf("| %-14s | ₹%-9.2f |\n", category, total));
            
            System.out.println("+----------------+------------+");
            
            double memberTotal = categoryTotals.values().stream().mapToDouble(Double::doubleValue).sum();
            System.out.printf("| %-14s | ₹%-9.2f |\n", "TOTAL", memberTotal);
            System.out.println("+----------------+------------+");
            return;
        }
        case 4 -> {
            filtered = new ArrayList<>(expenses);
        }
        default -> {
            System.out.println("Invalid choice.");
            return;
        }
    }

    if (filtered.isEmpty()) {
        System.out.println("No expenses found for the selected filter.");
    } else {
        // Rest of the display logic remains the same...
        // Calculate total spending for utilization percentage
        double totalSpent = filtered.stream().mapToDouble(Expense::getAmount).sum();
        
        // Display expenses with utilization percentage
        System.out.println("\nExpense Summary:");
        System.out.println("+------------+------------+------------+------------+----------------+");
        System.out.println("| Date       | Member     | Category   | Amount     | Utilization    |");
        System.out.println("+------------+------------+------------+------------+----------------+");
        
        for (Expense e : filtered) {
            String baseCategory = e.getCategory().replace(" (shared)", "");
            double limit = expenseCategories.getOrDefault(baseCategory, new ExpenseCategory("", 0)).getLimit();
            double utilization = limit > 0 ? (e.getAmount() / limit) * 100 : 0;
            String utilizationStr;
            
            if (limit == 0) {
                utilizationStr = "No limit";
            } else if (utilization > 100) {
                utilizationStr = String.format("%.1f%% (Exceeded by %.1f%%)", utilization, utilization - 100);
            } else {
                utilizationStr = String.format("%.1f%%", utilization);
            }
            
            System.out.printf("| %-10s | %-10s | %-10s | ₹%-9.2f | %-14s |\n",
                e.getDate(),
                e.getUsername(),
                e.getCategory(),
                e.getAmount(),
                utilizationStr);
        }
        
        System.out.println("+------------+------------+------------+------------+----------------+");
        System.out.printf("| %-10s | %-10s | %-10s | ₹%-9.2f | %-14s |\n",
            "TOTAL", "", "", totalSpent, "");
        System.out.println("+------------+------------+------------+------------+----------------+");
        
        // Show category breakdown
        System.out.println("\nCategory-wise Breakdown:");
        filtered.stream()
            .collect(Collectors.groupingBy(e -> e.getCategory().replace(" (shared)", ""), 
                Collectors.summingDouble(Expense::getAmount)))
            .forEach((cat, total) -> {
                double limit = expenseCategories.getOrDefault(cat, new ExpenseCategory("", 0)).getLimit();
                double utilization = limit > 0 ? (total / limit) * 100 : 0;
                String status;
                
                if (limit == 0) {
                    status = "No limit set";
                } else if (utilization > 100) {
                    status = String.format("Exceeded by %.1f%%", utilization - 100);
                } else {
                    status = String.format("%.1f%% of limit", utilization);
                }
                
                System.out.printf("- %s: ₹%.2f (%s)\n", 
                    cat, total, status);
            });
            
        // Show overall budget utilization
        if (isAdmin) {
            double familyUtilization = (totalSpent / totalFamilyIncome) * 100;
            System.out.printf("\nOverall Budget Utilization: %.1f%% of total family income\n", familyUtilization);
            
            if (familyUtilization >= 100) {
                System.out.println("\uD83D\uDD25 WARNING: Family budget fully utilized (100%)!");
            } else if (familyUtilization >= 90) {
                System.out.println("\uD83D\uDD25 WARNING: Family budget nearly exhausted (90%)!");
            } else if (familyUtilization >= 50) {
                System.out.println("\uD83D\uDD25 Notice: Family budget 50% utilized");
            }
        }
    }
}

    private static void showExpenseTrends() {
        System.out.println("\n--- Expense Trend Analysis ---");
        System.out.print("Enter year (yyyy): ");
        int year = scanner.nextInt();
        scanner.nextLine();
        
        System.out.print("Enter category (or 'all'): ");
        String category = scanner.nextLine();

        Map<String, Double> monthlySpending = new LinkedHashMap<>();
        for (int i = 1; i <= 12; i++) {
            final int month = i;
            double total = expenses.stream()
                .filter(e -> e.getLocalDate().getYear() == year)
                .filter(e -> e.getLocalDate().getMonthValue() == month)
                .filter(e -> category.equals("all") || e.getCategory().replace(" (shared)", "").equals(category))
                .mapToDouble(Expense::getAmount)
                .sum();
            monthlySpending.put(MONTHS[i-1], total);
        }

        System.out.println("\nMonthly Spending Trend:");
        drawBarChart(monthlySpending);
    }

    private static void drawBarChart(Map<String, Double> data) {
        double max = data.values().stream().max(Double::compare).orElse(1.0);
        data.forEach((month, amount) -> {
            int bars = (int) (amount / max * MAX_CHART_WIDTH);
            System.out.printf("%-5s | %s ₹%.2f%n", 
                month, 
                "=".repeat(bars), 
                amount);
        });
    }

    private static void settleDebts() {
        System.out.println("\n--- Debt Settlement ---");
        MinimumCashFlow solver = new MinimumCashFlow(debtGraph);
        List<Transaction> transactions = solver.settle();
        
        if (transactions.isEmpty()) {
            System.out.println("No debts to settle!");
            return;
        }
        
        System.out.println("Optimal Transactions:");
        transactions.forEach(t -> 
            System.out.printf("%s pays %s ₹%.2f%n", 
                t.from(), t.to(), t.amount()));
    }
}

// ========== Data Classes ==========

class FamilyMember {
    private final String name;
    private final String role;
    private double personalIncome;

    public FamilyMember(String name, String role, double personalIncome) {
        this.name = name;
        this.role = role;
        this.personalIncome = personalIncome;
    }

    public String getRole() {
        return role;
    }

    public String getName() {
        return name;
    }

    public double getPersonalIncome() {
        return personalIncome;
    }

    public void setPersonalIncome(double income) {
        this.personalIncome = income;
    }
}

class ExpenseCategory {
    private final String name;
    private final double limit;

    public ExpenseCategory(String name, double limit) {
        this.name = name;
        this.limit = limit;
    }

    public double getLimit() {
        return limit;
    }

    public String getName() {
        return name;
    }
}

class Expense {
    private String username;
    private final String category;
    private double amount;
    private final String date;
    private final LocalDate localDate;

    public Expense(String username, String category, double amount, String date, LocalDate localDate) {
        this.username = username;
        this.category = category;
        this.amount = amount;
        this.date = date;
        this.localDate = localDate;
    }

    public String getUsername() {
        return username;
    }

    public String getCategory() {
        return category;
    }

    public double getAmount() {
        return amount;
    }

    public void setAmount(double amount) {
        this.amount = amount;
    }

    public String getDate() {
        return date;
    }

    public LocalDate getLocalDate() {
        return localDate;
    }
}

record BudgetAlert(String category, double percentage) implements Comparable<BudgetAlert> {
    public int compareTo(BudgetAlert o) {
        return Double.compare(o.percentage(), this.percentage());
    }
}

record Transaction(String from, String to, double amount) {}

// ========== Minimum Cash Flow Algorithm ==========

class MinimumCashFlow {
    private final Map<String, Map<String, Double>> graph;
    private final Map<String, Double> net = new HashMap<>();

    public MinimumCashFlow(Map<String, Map<String, Double>> graph) {
        this.graph = graph;
        calculateNet();
    }

    private void calculateNet() {
        graph.forEach((person, debts) -> {
            net.putIfAbsent(person, 0.0);
            debts.forEach((creditor, amount) -> {
                net.put(person, net.get(person) - amount);
                net.put(creditor, net.getOrDefault(creditor, 0.0) + amount);
            });
        });
    }

    public List<Transaction> settle() {
        List<Transaction> transactions = new ArrayList<>();
        PriorityQueue<Map.Entry<String, Double>> debtors = new PriorityQueue<>(
            (a, b) -> Double.compare(b.getValue(), a.getValue()));
        PriorityQueue<Map.Entry<String, Double>> creditors = new PriorityQueue<>(
            Comparator.comparingDouble(Map.Entry::getValue));

        net.entrySet().stream()
            .filter(e -> e.getValue() < 0)
            .forEach(debtors::offer);
        net.entrySet().stream()
            .filter(e -> e.getValue() > 0)
            .forEach(creditors::offer);

        while (!debtors.isEmpty() && !creditors.isEmpty()) {
            Map.Entry<String, Double> debtor = debtors.poll();
            Map.Entry<String, Double> creditor = creditors.poll();

            double amount = Math.min(-debtor.getValue(), creditor.getValue());
            transactions.add(new Transaction(debtor.getKey(), creditor.getKey(), amount));

            double newDebt = debtor.getValue() + amount;
            double newCredit = creditor.getValue() - amount;

            if (newDebt < 0) debtors.offer(Map.entry(debtor.getKey(), newDebt));
            if (newCredit > 0) creditors.offer(Map.entry(creditor.getKey(), newCredit));
        }

        return transactions;
    }
}

class ManageExpense {
    private final Scanner scanner;
    private final TreeMap<String, ExpenseCategory> categoryTrie;
    private final Map<String, ExpenseCategory> expenseCategories;
    private double distributedBudget;
    private final double totalBudget;

    public ManageExpense(Scanner scanner, TreeMap<String, ExpenseCategory> categoryTrie,
                       Map<String, ExpenseCategory> expenseCategories,
                       double totalBudget, double distributedBudget) {
        this.scanner = scanner;
        this.categoryTrie = categoryTrie;
        this.expenseCategories = expenseCategories;
        this.totalBudget = totalBudget;
        this.distributedBudget = distributedBudget;
    }

    public void manageCategories(Map<String, ExpenseCategory> expenseCategories, double totalBudget) {
        int choice;
        do {
            System.out.println("\nManage Categories");
            System.out.println("1. Add Category");
            System.out.println("2. Remove Category");
            System.out.println("3. Modify Category Limit");
            System.out.println("4. List Categories");
            System.out.println("5. Back to Main Menu");
            System.out.print("Enter your choice: ");
            try {
                choice = scanner.nextInt();
                scanner.nextLine();
            } catch (InputMismatchException e) {
                System.out.println("Invalid input. Please enter a number.");
                scanner.nextLine();
                choice = 0;
                continue;
            }

            switch (choice) {
                case 1 -> addCategory();
                case 2 -> removeCategory();
                case 3 -> modifyCategoryLimit();
                case 4 -> listCategories();
                case 5 -> System.out.println("Returning to Main Menu...");
                default -> System.out.println("Invalid choice! Please try again.");
            }
        } while (choice != 5);
    }

    private void addCategory() {
        do {
            System.out.println("\nMost common categories:");
            categoryTrie.keySet().stream()
                .limit(8)
                .forEach(cat -> System.out.println("- " + cat));
            
            System.out.print("\nEnter starting letters of category name (or 'done' to finish): ");
            String prefix = scanner.nextLine().toLowerCase();
            
            if (prefix.equals("done")) {
                break;
            }

            List<String> matchedCategories = new ArrayList<>();
            for (Map.Entry<String, ExpenseCategory> entry : categoryTrie.entrySet()) {
                if (entry.getKey().startsWith(prefix) && !expenseCategories.containsKey(entry.getKey())) {
                    matchedCategories.add(entry.getKey());
                }
            }

            if (matchedCategories.isEmpty()) {
                System.out.println("❌ No matching categories found or all matching categories are already added.");
                continue;
            }

            System.out.println("Matching categories:");
            for (int i = 0; i < matchedCategories.size(); i++) {
                System.out.println((i+1) + ". " + matchedCategories.get(i));
            }

            System.out.print("Enter number of category to add (or 0 to search again): ");
            int selection;
            try {
                selection = scanner.nextInt();
                scanner.nextLine();
            } catch (InputMismatchException e) {
                System.out.println("Invalid input. Please enter a number.");
                scanner.nextLine();
                continue;
            }

            if (selection == 0) {
                continue;
            }

            if (selection < 1 || selection > matchedCategories.size()) {
                System.out.println("Invalid selection.");
                continue;
            }

            String categoryName = matchedCategories.get(selection - 1);

            System.out.print("Set monthly limit for " + categoryName + ": ₹");
            double limit;
            try {
                limit = scanner.nextDouble();
                scanner.nextLine();
            } catch (InputMismatchException e) {
                System.out.println("Invalid amount. Please enter a valid number.");
                scanner.nextLine();
                continue;
            }

            if (distributedBudget + limit > totalBudget) {
                System.out.println("❌ Amount exceeded total budget. Cannot assign ₹" + limit + " to " + categoryName);
                System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
                continue;
            }

            distributedBudget += limit;
            expenseCategories.put(categoryName, new ExpenseCategory(categoryName, limit));
            FamilyExpenseTracker.distributedBudget = distributedBudget;
            
            System.out.println("\n✅ Category added successfully!");
            System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
            
            System.out.print("Add another category? (y/n): ");
            String another = scanner.nextLine();
            if (!another.equalsIgnoreCase("y")) {
                break;
            }
        } while (true);
    }

    private void removeCategory() {
        if (expenseCategories.isEmpty()) {
            System.out.println("No categories to remove.");
            return;
        }
        
        System.out.println("Current categories:");
        int i = 1;
        for (String cat : expenseCategories.keySet()) {
            System.out.println(i++ + ". " + cat + " (Limit: ₹" + expenseCategories.get(cat).getLimit() + ")");
        }
        
        System.out.print("Enter number of category to remove: ");
        int selection;
        try {
            selection = scanner.nextInt();
            scanner.nextLine();
        } catch (InputMismatchException e) {
            System.out.println("Invalid input. Please enter a number.");
            scanner.nextLine();
            return;
        }
        
        if (selection < 1 || selection > expenseCategories.size()) {
            System.out.println("Invalid selection.");
            return;
        }
        
        String categoryName = expenseCategories.keySet().toArray(new String[0])[selection - 1];
        double limit = expenseCategories.get(categoryName).getLimit();
        expenseCategories.remove(categoryName);
        distributedBudget -= limit;
        FamilyExpenseTracker.distributedBudget = distributedBudget;
        
        System.out.println("Category removed successfully.");
        System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
    }

    private void modifyCategoryLimit() {
        if (expenseCategories.isEmpty()) {
            System.out.println("No categories to modify.");
            return;
        }
        
        System.out.println("Current categories:");
        int i = 1;
        for (String cat : expenseCategories.keySet()) {
            System.out.println(i++ + ". " + cat + " (Limit: ₹" + expenseCategories.get(cat).getLimit() + ")");
        }
        
        System.out.print("Enter number of category to modify: ");
        int selection;
        try {
            selection = scanner.nextInt();
            scanner.nextLine();
        } catch (InputMismatchException e) {
            System.out.println("Invalid input. Please enter a number.");
            scanner.nextLine();
            return;
        }
        
        if (selection < 1 || selection > expenseCategories.size()) {
            System.out.println("Invalid selection.");
            return;
        }
        
        String categoryName = expenseCategories.keySet().toArray(new String[0])[selection - 1];
        double currentLimit = expenseCategories.get(categoryName).getLimit();
        
        System.out.print("Current limit: ₹" + currentLimit + ". Enter new limit: ₹");
        double newLimit;
        try {
            newLimit = scanner.nextDouble();
            scanner.nextLine();
        } catch (InputMismatchException e) {
            System.out.println("Invalid amount. Please enter a valid number.");
            scanner.nextLine();
            return;
        }
        
        double difference = newLimit - currentLimit;
        
        if (distributedBudget + difference > totalBudget) {
            System.out.println("❌ This change would exceed total budget. Cannot increase by ₹" + difference);
            System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
            return;
        }
        
        expenseCategories.put(categoryName, new ExpenseCategory(categoryName, newLimit));
        distributedBudget += difference;
        FamilyExpenseTracker.distributedBudget = distributedBudget;
        
        System.out.println("Category limit updated successfully.");
        System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
    }

    private void listCategories() {
        if (expenseCategories.isEmpty()) {
            System.out.println("No categories added yet.");
        } else {
            System.out.println("\nCategories and Limits:");
            expenseCategories.forEach((name, category) -> 
                System.out.printf("- %s: ₹%.2f (%.1f%% of total budget)\n", 
                    name, 
                    category.getLimit(), 
                    (category.getLimit() / totalBudget) * 100));
            
            System.out.printf("\nTotal budget: ₹%.2f\n", totalBudget);
            System.out.printf("Distributed budget: ₹%.2f\n", distributedBudget);
            System.out.printf("Remaining budget: ₹%.2f\n", totalBudget - distributedBudget);
        }
    }
}
