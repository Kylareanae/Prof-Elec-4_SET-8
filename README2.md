using System;
using System.IO;

class Program
{
    static void Main()
    {
        string filePath = "users.txt";

        // Check if the users file exists; if not, create it
        if (!File.Exists(filePath))
        {
            File.Create(filePath).Close();
        }

        Console.WriteLine("Welcome!");
        Console.WriteLine("1. Login");
        Console.WriteLine("2. Register");
        Console.Write("Choose an option: ");
        
        // Read user choice and handle input
        if (int.TryParse(Console.ReadLine(), out int choice))
        {
            switch (choice)
            {
                case 1:
                    Login(filePath);
                    break;
                case 2:
                    Register(filePath);
                    break;
                default:
                    Console.WriteLine("Invalid option.");
                    break;
            }
        }
        else
        {
            Console.WriteLine("Invalid input. Please enter a number.");
        }
    }

    static void Register(string filePath)
    {
        Console.Write("Enter a username: ");
        string username = Console.ReadLine();
        Console.Write("Enter a password: ");
        string password = Console.ReadLine();
        string language = SetLanguagePreference();

        // Append new user to the file
        using (StreamWriter sw = File.AppendText(filePath))
        {
            sw.WriteLine($"{username},{password},{language}");
        }

        Console.WriteLine("Registration successful. You can now login.");
    }

    static void Login(string filePath)
    {
        Console.Write("Enter your username: ");
        string username = Console.ReadLine();
        Console.Write("Enter your password: ");
        string password = Console.ReadLine();

        string[] users = File.ReadAllLines(filePath);
        string preferredLanguage = "english"; // default language
        bool isValidUser = false;

        foreach (string user in users)
        {
            string[] userDetails = user.Split(',');
            if (userDetails[0] == username && userDetails[1] == password)
            {
                isValidUser = true;
                preferredLanguage = userDetails[2];
                break;
            }
        }

        if (isValidUser)
        {
            DisplayWelcomeMessage(preferredLanguage);
            UpdateLanguagePreference(filePath, username);
        }
        else
        {
            Console.WriteLine("Invalid username or password.");
        }
    }

    static void DisplayWelcomeMessage(string language)
    {
        string message = GetTimeBasedGreeting(language);
        Console.WriteLine(message);
    }

    static string GetTimeBasedGreeting(string language)
    {
        int hour = DateTime.Now.Hour;
        return hour switch
        {
            < 12 => language == "spanish" ? "¡Buenos Días!" : "Good Morning!",
            < 18 => language == "spanish" ? "¡Buenas Tardes!" : "Good Afternoon!",
            _ => language == "spanish" ? "¡Buenas Noches!" : "Good Evening!",
        };
    }

    static string SetLanguagePreference()
    {
        Console.WriteLine("Choose your preferred language:");
        Console.WriteLine("1. English");
        Console.WriteLine("2. Spanish");
        Console.Write("Select an option: ");
        
        if (int.TryParse(Console.ReadLine(), out int languageChoice))
        {
            return languageChoice == 2 ? "spanish" : "english";
        }

        return "english"; // default if invalid input
    }

    static void UpdateLanguagePreference(string filePath, string username)
    {
        Console.WriteLine("Would you like to change your preferred language? (yes/no)");
        string response = Console.ReadLine().ToLower();

        if (response == "yes")
        {
            string newLanguage = SetLanguagePreference();
            UpdateUserFile(filePath, username, newLanguage);
            Console.WriteLine("Language preference updated.");
        }
    }

    static void UpdateUserFile(string filePath, string username, string newLanguage)
    {
        string[] users = File.ReadAllLines(filePath);
        
        for (int i = 0; i < users.Length; i++)
        {
            string[] userDetails = users[i].Split(',');
            if (userDetails[0] == username)
            {
                users[i] = $"{username},{userDetails[1]},{newLanguage}"; // Update language
            }
        }

        File.WriteAllLines(filePath, users);
    }
}
