using System;

class Minesweeper
{
    static int width = 10;
    static int height = 10;
    static int minesCount = 15;
    static char[,] field = new char[width, height];
    static bool[,] revealed = new bool[width, height];
    static bool[,] flagged = new bool[width, height];
    static bool cheatActive = false;
    static Random rand = new Random();

    static string[] konamiCode = { "W", "W", "S", "S", "A", "D", "A", "D", "B", "A" };

    static void Main()
    {
        Console.OutputEncoding = System.Text.Encoding.UTF8;
        bool useUTF = true;

        InitializeField();
        PlaceMines();

        while (true)
        {
            if (useUTF) PrintFieldUTF();
            else PrintFieldASCII();

            Console.Write("Введите команду (r x y - открыть, f x y - флажок): ");
            string input = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(input)) continue;

            string[] parts = input.ToUpper().Split();

            if (parts.Length == konamiCode.Length)
            {
                bool match = true;
                for (int i = 0; i < konamiCode.Length; i++)
                    if (parts[i] != konamiCode[i])
                        match = false;

                if (match)
                    cheatActive = true;

                continue;
            }

            if (parts.Length != 3) continue;
            char cmd = parts[0][0];
            if (!int.TryParse(parts[1], out int x) || !int.TryParse(parts[2], out int y))
                continue;
            if (x < 0 || x >= width || y < 0 || y >= height) continue;

            if (cmd == 'R')
            {
                if (flagged[x, y]) continue;
                if (field[x, y] == '*')
                {
                    Console.WriteLine("Вы проиграли!");
                    RevealAll();
                    if (useUTF) PrintFieldUTF(true);
                    else PrintFieldASCII(true);
                    break;
                }
                else
                {
                    Reveal(x, y);
                    if (CheckWin())
                    {
                        Console.WriteLine("Вы выиграли!");
                        RevealAll();
                        if (useUTF) PrintFieldUTF(true);
                        else PrintFieldASCII(true);
                        break;
                    }
                }
            }
            else if (cmd == 'F')
            {
                if (!revealed[x, y])
                    flagged[x, y] = !flagged[x, y];
            }
        }
    }

    static void InitializeField()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                field[i, j] = '0';
    }

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

        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
            {
                if (field[i, j] == '*') continue;
                int minesAround = CountMinesAround(i, j);
                field[i, j] = minesAround.ToString()[0];
            }
    }

    static int CountMinesAround(int x, int y)
    {
        int count = 0;
        for (int dx = -1; dx <= 1; dx++)
            for (int dy = -1; dy <= 1; dy++)
            {
                int nx = x + dx, ny = y + dy;
                if (nx >= 0 && nx < width && ny >= 0 && ny < height)
                    if (field[nx, ny] == '*') count++;
            }
        return count;
    }

    static void PrintFieldUTF(bool revealAllFlags = false)
    {
        Console.Clear();
        Console.Write("   ");
        for (int i = 0; i < width; i++)
            Console.Write(i + " ");
        Console.WriteLine();

        for (int j = 0; j < height; j++)
        {
            Console.Write(j.ToString("D2") + " ");
            for (int i = 0; i < width; i++)
            {
                if (flagged[i, j] && (!revealed[i, j] || revealAllFlags))
                    Console.Write("➤ ");
                else if (revealed[i, j])
                {
                    if (field[i, j] == '0')
                        Console.Write("○ ");
                    else if (field[i, j] == '*')
                        Console.Write("● ");
                    else
                        Console.Write(field[i, j] + " ");
                }
                else if (cheatActive)
                {
                    if (field[i, j] != '*')
                        Console.Write(field[i, j] + " ");
                    else
                        Console.Write("■ ");
                }
                else
                    Console.Write("■ ");
            }
            Console.WriteLine();
        }
    }

    static void PrintFieldASCII(bool revealAllFlags = false)
    {
        Console.Clear();
        Console.Write("   ");
        for (int i = 0; i < width; i++)
            Console.Write(i + " ");
        Console.WriteLine();

        for (int j = 0; j < height; j++)
        {
            Console.Write(j.ToString("D2") + " ");
            for (int i = 0; i < width; i++)
            {
                if (flagged[i, j] && (!revealed[i, j] || revealAllFlags))
                    Console.Write("F ");
                else if (revealed[i, j])
                {
                    if (field[i, j] == '0')
                        Console.Write(". ");
                    else if (field[i, j] == '*')
                        Console.Write("* ");
                    else
                        Console.Write(field[i, j] + " ");
                }
                else if (cheatActive)
                {
                    if (field[i, j] != '*')
                        Console.Write(field[i, j] + " ");
                    else
                        Console.Write("# ");
                }
                else
                    Console.Write("# ");
            }
            Console.WriteLine();
        }
    }

    static void Reveal(int x, int y)
    {
        if (x < 0 || x >= width || y < 0 || y >= height || revealed[x, y] || flagged[x, y])
            return;

        revealed[x, y] = true;

        if (field[x, y] == '0')
        {
            for (int dx = -1; dx <= 1; dx++)
                for (int dy = -1; dy <= 1; dy++)
                    Reveal(x + dx, y + dy);
        }
    }

    static void RevealAll()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                revealed[i, j] = true;
    }

    static bool CheckWin()
    {
        for (int i = 0; i < width; i++)
            for (int j = 0; j < height; j++)
                if (field[i, j] != '*' && !revealed[i, j])
                    return false;
        return true;
    }
}
