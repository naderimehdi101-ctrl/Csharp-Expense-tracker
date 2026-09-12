# Csharp-Expense-tracker

using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using System.Linq;
using System.Text.Json;
using System.IO;

// = = = = = = = = = = = = = = = = = = = = = = =//

static class ExpenseDataHelper
{
    static readonly string filePath = "expense.json";

    public static async Task SaveExpensesAsync(List<Expense> expenses)
    {
        string json = JsonSerializer.Serialize(expenses);
        await File.WriteAllTextAsync(filePath, json);
    }

    public static async Task<List<Expense>> LoadExpensesAsync()
    {
        if (!File.Exists(filePath))
        {
            return new List<Expense>();
        }

        string json = await File.ReadAllTextAsync(filePath);
        List<Expense> expenses = JsonSerializer.Deserialize<List<Expense>>(json);
        return expenses ?? new List<Expense>();
    }
}

// = = = = = = = = = = = = = = = = = = = = = = =//

class Expense
{
    public int ID { get; set; }
    public string Description { get; set; }
    public decimal Amount { get; set; }
    public string Category { get; set; }
    public DateTime Date { get; set; }

    public Expense(int id, string description, decimal amount, string category, DateTime date)
    {
        ID = id;
        Description = description;
        Amount = amount;
        Category = category;
        Date = date;
    }
}

// = = = = = = = = = = = = = = = = = = = = = = =//
//              CHOICE - METHOD                 //
// = = = = = = = = = = = = = = = = = = = = = = =//

class Program
{
    static async Task Choice()
    {
        List<Expense> expenses = await ExpenseDataHelper.LoadExpensesAsync();

        int IDCount = expenses.Count == 0 ? 0 : expenses.Max(e => e.ID);
        bool isRunning = true;

        while (isRunning)
        {
            Console.Clear();

            Console.WriteLine($"1- Add expense\n2- View Expenses\n3- View Monthly expense\n4- Remove expense\n5- Update expense");

            if (!int.TryParse(Console.ReadLine(), out int choice))
            {
                WriteColor("Invalid choice input!", ConsoleColor.Red);
                Thread.Sleep(1000);
                continue;
            }
            switch (choice)
            {
                case 1:
                    await AddExpenseAsync(expenses, IDCount); break;
                case 2:
                    ViewExpense(expenses); break;
                case 3:
                    MonthlySum(expenses); break;
                case 4:
                    await RemoveExpenseAsync(expenses); break;
                case 5:
                    await UpdateExpenseAsync(expenses); break;
                default:
                    WriteColor("Invalid choice input!", ConsoleColor.Red);
                    Thread.Sleep(1000);
                    break;
            }
        }
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //              ADD-EXPENSE-METHOD              //       
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static async Task AddExpenseAsync(List<Expense> expenses, int idCount)
    {
        Console.Clear();

        PrintTitel("ADD EXPENSES");

        WriteColor("-Please type the Description of the item: ", ConsoleColor.Green);
        string description = Console.ReadLine().ToLower();

        if (string.IsNullOrWhiteSpace(description))
        {
            WriteColor("Description cannot be empty!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        WriteColor("-Enter Amount: ", ConsoleColor.Green);
        if (!decimal.TryParse(Console.ReadLine(), out decimal amount))
        {
            WriteColor("Invaild input!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        WriteColor("-Enter Category (e.g. Food/Travel): ", ConsoleColor.Green);
        string category = Console.ReadLine().ToUpper();
        if (string.IsNullOrWhiteSpace(category))
        {
            WriteColor("Category cannot be empty!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        WriteColor("-Date time: (e.g. 01/01/2001)", ConsoleColor.Green);
        if (!DateTime.TryParse(Console.ReadLine(), out DateTime expenseDate))
        {
            WriteColor("Invalid Date format!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
        DateTime earliestAllowed = DateTime.Today.AddDays(-7);

        if (expenseDate > DateTime.Today)
        {
            WriteColor("Date cannot be in the future!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
        else if (expenseDate < earliestAllowed)
        {
            WriteColor("Date cannot be 7 days old!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        idCount++;
        Expense E = new Expense(idCount, description, amount, category, expenseDate);

        WriteColor(" [1.Save] Or [2.Reset]", ConsoleColor.Yellow);
        if (int.TryParse(Console.ReadLine(), out int SaveOeReset) && SaveOeReset == 1)
        {
            expenses.Add(E);
            await ExpenseDataHelper.SaveExpensesAsync(expenses);
            WriteColor("Expense saved ✅ ", ConsoleColor.Green);
            Thread.Sleep(1000);
            return;
        }
        else
        {
            WriteColor("Reseted🚫 ", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //              VIEW-EXPENSE-METHOD             //     
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static void ViewExpense(List<Expense> expenses)
    {
        Console.Clear();
        PrintTitel("VIEW ALL EXPENSES");

        if (expenses.Count == 0)
        {
            WriteColor("No expense has been added yet!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
        foreach (Expense e in expenses)
        {
            WriteColor($"Expense ID:       #{e.ID}", ConsoleColor.Yellow);
            WriteColor($"Category:         {(e.Category)}", ConsoleColor.Green);
            WriteColor($"Description:      {e.Description}", ConsoleColor.Yellow);
            WriteColor($"Amount:           {e.Amount}€", ConsoleColor.Yellow);
            WriteColor($"Expense date:     {e.Date:dd/MM/yyyy}", ConsoleColor.Cyan);
            Line();
        }
        Console.ReadKey(true);
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //             MONTHLY-SUM-METHOD               //     
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static void MonthlySum(List<Expense> expenses)
    {
        if (expenses.Count == 0)
        {
            WriteColor("No Expense added yet!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        Console.Clear();

        DateTime today = DateTime.Today;
        var thisMonthExpenses = expenses.Where(e => e.Date.Month == today.Month && e.Date.Year == today.Year);
        if (!thisMonthExpenses.Any())
        {
            WriteColor("No expenses this month!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        var total = thisMonthExpenses.Sum(e => e.Amount);

        PrintTitel("VIEW MONTHLY EXPENSE");

        WriteColor($"Total monthly expense:    {total}€", ConsoleColor.Cyan);
        Line();
        foreach (Expense e in thisMonthExpenses)
        {
            WriteColor($"Item:     {e.Description}", ConsoleColor.Yellow);
            WriteColor($"Amount:   {e.Amount}€", ConsoleColor.Green);
            Line();
        }
        Console.ReadKey(true);
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //            REMOVE-EXPENSE-METHOD             //     
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static async Task RemoveExpenseAsync(List<Expense> expenses)
    {
        if (expenses.Count == 0)
        {
            WriteColor("No expenses added yet!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        WriteColor("Enter Expense ID to remove: ", ConsoleColor.Green);
        if (!int.TryParse(Console.ReadLine(), out int findID))
        {
            WriteColor("No expense found!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        Console.Clear();

        var found = expenses.FirstOrDefault(r => r.ID == findID);

        if (found == null)
        {
            WriteColor("No Expense found with that ID!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
        else
        {
            WriteColor($"Expense ID:       #{found.ID}", ConsoleColor.Yellow);
            WriteColor($"Category:         {(found.Category)}", ConsoleColor.Green);
            WriteColor($"Description:      {found.Description}", ConsoleColor.Yellow);
            WriteColor($"Amount:           {found.Amount}€", ConsoleColor.Yellow);
            WriteColor($"Expense date:     {found.Date:dd/MM/yyyy}", ConsoleColor.Cyan);
        }

        Line();
        WriteColor("[1.Remove] or tap Enter to [Cancel]", ConsoleColor.Yellow);
        if (int.TryParse(Console.ReadLine(), out int removeOrCancel) && removeOrCancel == 1)
        {
            expenses.Remove(found);
            await ExpenseDataHelper.SaveExpensesAsync(expenses);
            WriteColor("Expense has been successfully removed!", ConsoleColor.Green);
            Thread.Sleep(1000);
            return;
        }
        else
        {
            WriteColor("Cancelling!...", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //              UPDATE-EXPENSE-METHOD           //     
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static async Task UpdateExpenseAsync(List<Expense> expenses)
    {

        if (expenses.Count == 0)
        {
            WriteColor("No expenses added yet!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        WriteColor("Enter Expense ID to Update: ", ConsoleColor.Green);
        if (!int.TryParse(Console.ReadLine(), out int findID))
        {
            WriteColor("No expense found!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }

        Console.Clear();

        var found = expenses.FirstOrDefault(r => r.ID == findID);

        if (found == null)
        {
            WriteColor("No Expense found with that ID!", ConsoleColor.Red);
            Thread.Sleep(1000);
            return;
        }
        else
        {
            WriteColor("What do you want to update: ", ConsoleColor.Green);
            WriteColor($"1- Category\n2- Description\n3- Amount\n4- Date", ConsoleColor.Yellow);

            if (!int.TryParse(Console.ReadLine(), out int updateChoice))
            {
                WriteColor("Invalid Input!", ConsoleColor.Red);
                Thread.Sleep(1000);
                return;
            }
            else
            {
                switch (updateChoice)
                {
                    case 1:
                        found.Category = Console.ReadLine();
                        WriteColor("successfully updated!", ConsoleColor.Green);
                        await ExpenseDataHelper.SaveExpensesAsync(expenses);
                        Thread.Sleep(1000); break;
                    case 2:
                        found.Description = Console.ReadLine();
                        WriteColor("successfully updated!", ConsoleColor.Green);
                        await ExpenseDataHelper.SaveExpensesAsync(expenses);
                        Thread.Sleep(1000); break;
                    case 3:
                        if (decimal.TryParse(Console.ReadLine(), out decimal newAmount))
                        {
                            found.Amount = newAmount;
                            WriteColor("successfully updated!", ConsoleColor.Green);
                            await ExpenseDataHelper.SaveExpensesAsync(expenses);
                            Thread.Sleep(1000);
                        }
                        else
                        {
                            WriteColor("Invalid amount!", ConsoleColor.Red);
                            Thread.Sleep(1000);
                        }
                        break;
                    case 4:
                        if (DateTime.TryParse(Console.ReadLine(), out DateTime newDate))
                        {
                            found.Date = newDate;
                            WriteColor("successfully updated!", ConsoleColor.Green);
                            await ExpenseDataHelper.SaveExpensesAsync(expenses);
                            Thread.Sleep(1000);
                        }
                        else
                        {
                            WriteColor("Invalid date format!", ConsoleColor.Red);
                        }
                        break;
                    default:
                        WriteColor("Invalid choice input!", ConsoleColor.Red);
                        Thread.Sleep(1000);
                        break;
                }
            }
        }
    }

    // = = = = = = = = = = = = = = = = = = = = = = =//
    //                  MAIN-METHOD                 //       
    // = = = = = = = = = = = = = = = = = = = = = = =//

    static async Task Main()
    {
        await Choice();
    }
    static void WriteColor(string message, ConsoleColor color)
    {
        Console.ForegroundColor = color;
        Console.WriteLine(message);
        Console.ResetColor();
    }
    static void Line()
    {
        Console.ForegroundColor = ConsoleColor.Yellow;
        Console.Write("=================================================");
        Console.WriteLine();
        Console.ResetColor();
    }
    static void PrintTitel(string title)
    {
        int screenWidth = 50;
        int spacesCount = (screenWidth - title.Length) / 2;
        if (spacesCount < 0) spacesCount = 0;
        string spaces = new string(' ', spacesCount);

        Line();
        Console.ForegroundColor = ConsoleColor.Green;
        Console.WriteLine(spaces + title);
        Console.ResetColor();
        Line();
    }
}