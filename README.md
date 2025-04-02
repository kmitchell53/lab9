# lab9

food

public class Food {
    private String name;
    private String foodGroup;
    private int calories;
    private double dailyPercentage;
    private Food next; // For linked list

    public Food(String name, String foodGroup, int calories, double dailyPercentage) {
        this.name = name;
        this.foodGroup = foodGroup;
        this.calories = calories;
        this.dailyPercentage = dailyPercentage;
        this.next = null;
    }

    public String getName() { return name; }
    public String getFoodGroup() { return foodGroup; }
    public int getCalories() { return calories; }
    public double getDailyPercentage() { return dailyPercentage; }
    public Food getNext() { return next; }
    public void setNext(Food next) { this.next = next; }

    @Override
    public String toString() {
        return String.format("%-20s %-15s %-10d %.2f", name, foodGroup, calories, dailyPercentage);
    }
}



foodlist

package assignments;

import java.io.*;
import java.util.Random;

public class FoodList {
    private Food head;

    public void addFood(String name, String foodGroup, int calories, double dailyPercentage) {
        Food newFood = new Food(name, foodGroup, calories, dailyPercentage);
        if (head == null) {
            head = newFood;
        } else {
            Food temp = head;
            while (temp.getNext() != null) {
                temp = temp.getNext();
            }
            temp.setNext(newFood);
        }
    }

    public void loadFromFile(String filename) {
        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(" ");
                addFood(parts[0], parts[1], Integer.parseInt(parts[2]), Double.parseDouble(parts[3]));
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
    }

    public void displayFoods() {
        System.out.println("================================================================");
        System.out.printf("%-20s %-15s %-10s %s%n", "Name", "Food Group", "Calories", "Daily %");
        System.out.println("================================================================");

        Food temp = head;
        while (temp != null) {
            System.out.println(temp);
            temp = temp.getNext();
        }
    }

    public Food findFood(String name) {
        Food temp = head;
        while (temp != null) {
            if (temp.getName().equalsIgnoreCase(name)) {
                return temp;
            }
            temp = temp.getNext();
        }
        return null;
    }

    public Food[] getRandomMeal() {
        Food[] meal = new Food[3];
        Random rand = new Random();
        int size = getSize();
        for (int i = 0; i < 3; i++) {
            int index = rand.nextInt(size);
            meal[i] = getFoodAt(index);
        }
        return meal;
    }

    public void removeHighCalorieFoods(int limit) {
        while (head != null && head.getCalories() > limit) {
            head = head.getNext();
        }
        if (head == null) return;

        Food temp = head;
        while (temp.getNext() != null) {
            if (temp.getNext().getCalories() > limit) {
                temp.setNext(temp.getNext().getNext());
            } else {
                temp = temp.getNext();
            }
        }
    }

    private int getSize() {
        int count = 0;
        Food temp = head;
        while (temp != null) {
            count++;
            temp = temp.getNext();
        }
        return count;
    }

    private Food getFoodAt(int index) {
        Food temp = head;
        for (int i = 0; i < index && temp != null; i++) {
            temp = temp.getNext();
        }
        return temp;
    }
}


mealplanner


package assignments;

import java.util.Scanner;

public class MealPlanner {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        FoodList foodList = new FoodList();
        foodList.loadFromFile("foods.txt");

        while (true) {
            System.out.println("\n---------------------------------");
            System.out.println("Welcome to Parkland Meal Selector");
            System.out.println("---------------------------------");
            System.out.println("1 - List food database");
            System.out.println("2 - Create meal by manual selection");
            System.out.println("3 - Create meal by random selection");
            System.out.println("4 - Remove foods high in calorie");
            System.out.println("5 - Exit");
            System.out.print("Enter choice: ");
            
            String choice = scanner.nextLine();
            switch (choice) {
                case "1":
                    foodList.displayFoods();
                    break;
                case "2":
                    manualMealSelection(scanner, foodList);
                    break;
                case "3":
                    randomMealSelection(foodList);
                    break;
                case "4":
                    removeHighCalorieFoods(scanner, foodList);
                    break;
                case "5":
                    System.out.println("Exiting program.");
                    scanner.close();
                    return;
                default:
                    System.out.println("Invalid choice, try again.");
            }
        }
    }

    private static void manualMealSelection(Scanner scanner, FoodList foodList) {
        Food[] meal = new Food[3];
        for (int i = 0; i < 3; i++) {
            while (true) {
                System.out.print("Enter food name: ");
                String name = scanner.nextLine();
                Food food = foodList.findFood(name);
                if (food != null) {
                    meal[i] = food;
                    break;
                } else {
                    System.out.println("Food not in database, try again.");
                }
            }
        }
        displayMeal(meal);
    }

    private static void randomMealSelection(FoodList foodList) {
        Food[] meal = foodList.getRandomMeal();
        displayMeal(meal);
    }

    private static void removeHighCalorieFoods(Scanner scanner, FoodList foodList) {
        try {
            System.out.print("Enter calorie limit: ");
            int limit = Integer.parseInt(scanner.nextLine());
            foodList.removeHighCalorieFoods(limit);
            System.out.println("Foods above " + limit + " calories removed.");
        } catch (NumberFormatException e) {
            System.out.println("Invalid input. Please enter a valid number.");
        }
    }

    private static void displayMeal(Food[] meal) {
        int totalCalories = 0;
        double totalPercentage = 0.0;
        System.out.println("\n===============================");
        System.out.print("Your selected meal:\nFoods: ");
        for (Food food : meal) {
            if (food != null) {
                System.out.print(food.getName() + " ");
                totalCalories += food.getCalories();
                totalPercentage += food.getDailyPercentage();
            }
        }
        System.out.printf("\nTotal calorie count: %d\nTotal daily percentage: %.2f%%\n", totalCalories, totalPercentage * 100);
        System.out.println("===============================");
    }
}
