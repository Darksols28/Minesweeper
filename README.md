# Minesweeper in C#


using System;

/// <summary>
/// Console-based Minesweeper game in C# with a Konami Code cheat to reveal mines.
/// </summary>
class Minesweeper
{
    // Game field dimensions
    static int width = 10;
    static int height = 10;

    // Number of mines in the field
    static int minesCount = 15;

    // The game field array:
    // '*' for mines, '0'-'8' for number of adjacent mines
    static char[,] field = new char[width, height];

    // Tracks which cells are revealed
    static bool[,] revealed = new bool[width, height];

    // Tracks which cells are flagged
    static bool[,] flagged = new bool[width, height];

    // If cheat is activated, show mines on the board
    static bool cheatActive = false;

    // Random number generator for placing mines
    static Random rand = new Random();

    // Konami code sequence to activate cheat
    static string[] konamiCode = { "W", "W", "S", "S", "A", "D", "A", "D", "B", "A" };

    static void Main()
    {
        // Set console encoding to UTF-8 to display special characters
        Console.OutputEncoding = System.Text.Encoding.UTF8;

        bool useUTF = true; // Flag to choose UTF-8 or ASCII display

        // Initialize field with zeros
        InitializeField();

        // Randomly place mines and calculate numbers
        PlaceMines();

        // Main game loop
        while (true)
        {
            // Print the game board
            if (useUTF) PrintFieldUTF();
            else PrintFieldASCII();

            // Get user input
            Console.Write("Enter command (r x y - reveal, f x y - flag): ");
            string input = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(input)) continue;

            // Split input into parts
            string[] parts = input.ToUpper().Split();

            // Check if Konami code is entered
            if (parts.Length == konamiCode.Length)
            {
                bool match = true;
                for (int i = 0; i < konamiCode.Length; i++)
                    if (parts[i] != konamiCode[i])
                        match = false;

                if (match)
                {
                    cheatActive = true;
                    Console.WriteLine("Cheat activated: Mines are now visible!");
                }

                continue;
            }

            // Only accept commands with 3 parts
            if (parts.Length != 3) continue;

            char cmd = parts[0][0];

            // Parse coordinates
            if (!int.TryParse(parts[1], out int x) || !int.TryParse(parts[2], out int y))
                continue;

            // Check if coordinates are within bounds
            if (x < 0 || x >= width || y < 0 || y >= height) continue;

            // Handle reveal command
            if (cmd == 'R')
            {
                // Do not reveal flagged cells
                if (flagged[x, y]) continue;

                // If user clicks on a mine -> game over
                if (field[x, y] == '*')
                {
                    Console.WriteLine("You hit a mine! Game over.");
                    RevealAll();
                    if (useUTF) PrintFieldUTF(true);
                    else PrintFieldASCII(true);
                    break;
                }
                else
                {
                    // Reveal the cell and adjacent empty cells if necessary
                    Reveal(x, y);

                    // Check if player has won
                    if (CheckWin())
                    {
                        Console.WriteLine("Congratulations! You won!");
                        RevealAll();
                        if (useUTF) PrintFieldUTF(true);
                        else PrintFieldASCII(true);
                        break;
                    }
                }
            }
            // Handle flag command
            else if (cmd == 'F')
            {
                if (!revealed[x, y])
                    flagged[x, y] = !flagged[x, y]; // Toggle flag
            }
        }
    }

    /// <summary>
    /// Initialize the field with zeros.
    /// </summary>
    static void InitializeField()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                field[i, j] = '0';
    }

    /// <summary>
    /// Randomly place mines and calculate numbers for other cells.
    /// </summary>
    static void PlaceMines()
    {
        int placed = 0;
        while (placed < minesCount)
        {
            int x = rand.Next(width);
            int y = rand.Next(height);

            if (field[x, y] != '*')
            {
                field[x, y] = '*';
                placed++;
            }
        }

        // Calculate numbers for cells that are not mines
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                if (field[i, j] != '*')
                    field[i, j] = CountMinesAround(i, j).ToString()[0];
    }

    /// <summary>
    /// Count mines around a given cell.
    /// </summary>
    static int CountMinesAround(int x, int y)
    {
        int count = 0;

        for (int dx = -1; dx <= 1; dx++)
            for (int dy = -1; dy <= 1; dy++)
            {
                int nx = x + dx;
                int ny = y + dy;

                if (nx >= 0 && nx < width && ny >= 0 && ny < height)
                    if (field[nx, ny] == '*') count++;
            }

        return count;
    }

    /// <summary>
    /// Print the field using UTF symbols.
    /// </summary>
    static void PrintFieldUTF(bool revealAllFlags = false)
    {
        Console.Clear();

        // Print column numbers
        Console.Write("   ");
        for (int i = 0; i < width; i++) Console.Write(i + " ");
        Console.WriteLine();

        // Print each row
        for (int j = 0; j < height; j++)
        {
            Console.Write(j.ToString("D2") + " "); // Row number

            for (int i = 0; i < width; i++)
            {
                // Show flagged cells
                if (flagged[i, j] && (!revealed[i, j] || revealAllFlags))
                    Console.Write("➤ ");
                // Show revealed cells
                else if (revealed[i, j])
                {
                    if (field[i, j] == '0') Console.Write("○ ");
                    else if (field[i, j] == '*') Console.Write("● ");
                    else Console.Write(field[i, j] + " ");
                }
                // Show cheat view
                else if (cheatActive)
                {
                    if (field[i, j] != '*') Console.Write(field[i, j] + " ");
                    else Console.Write("■ ");
                }
                // Default hidden cell
                else
                    Console.Write("■ ");
            }
            Console.WriteLine();
        }
    }

    /// <summary>
    /// Print the field using ASCII symbols.
    /// </summary>
    static void PrintFieldASCII(bool revealAllFlags = false)
    {
        Console.Clear();

        // Print column numbers
        Console.Write("   ");
        for (int i = 0; i < width; i++) Console.Write(i + " ");
        Console.WriteLine();

        // Print each row
        for (int j = 0; j < height; j++)
        {
            Console.Write(j.ToString("D2") + " "); // Row number

            for (int i = 0; i < width; i++)
            {
                // Show flagged cells
                if (flagged[i, j] && (!revealed[i, j] || revealAllFlags))
                    Console.Write("F ");
                // Show revealed cells
                else if (revealed[i, j])
                {
                    if (field[i, j] == '0') Console.Write(". ");
                    else if (field[i, j] == '*') Console.Write("* ");
                    else Console.Write(field[i, j] + " ");
                }
                // Show cheat view
                else if (cheatActive)
                {
                    if (field[i, j] != '*') Console.Write(field[i, j] + " ");
                    else Console.Write("# ");
                }
                // Default hidden cell
                else
                    Console.Write("# ");
            }
            Console.WriteLine();
        }
    }

    /// <summary>
    /// Recursively reveal cells. Reveals adjacent empty cells automatically.
    /// </summary>
    static void Reveal(int x, int y)
    {
        if (x < 0 || x >= width || y < 0 || y >= height || revealed[x, y] || flagged[x, y])
            return;

        revealed[x, y] = true;

        // If cell has no adjacent mines, reveal surrounding cells
        if (field[x, y] == '0')
            for (int dx = -1; dx <= 1; dx++)
                for (int dy = -1; dy <= 1; dy++)
                    Reveal(x + dx, y + dy);
    }

    /// <summary>
    /// Reveal all cells (used at game over or win).
    /// </summary>
    static void RevealAll()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                revealed[i, j] = true;
    }

    /// <summary>
    /// Check if all non-mine cells are revealed -> player wins.
    /// </summary>
    static bool CheckWin()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                if (field[i, j] != '*' && !revealed[i, j])
                    return false;

        return true;
    }
}
