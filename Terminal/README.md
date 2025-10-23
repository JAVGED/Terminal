# Terminal
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Terminal.Gui;

class Program
{
    static string csvFile = "produkty.csv";
    static List<Produkt> produkty = new List<Produkt>();

    static void Main()
    { 
        WczytajZCsv();

        while (true)
        {
            Console.WriteLine("\n=== MENU ===");
            Console.WriteLine("1) Dodaj produkt");
            Console.WriteLine("2) Pokaż produkty");
            Console.WriteLine("3) Edytuj produkt");
            Console.WriteLine("4) Usuń produkt");
            Console.WriteLine("5) Zapisz dane do pliku CSV");
            Console.WriteLine("0) Wyjście");
            Console.Write("> ");

            if (!int.TryParse(Console.ReadLine(), out int opcja))
                opcja = -1;

            switch (opcja)
            {
                case 1:
                    DodajLubEdytuj();
                    break;
                case 2:
                    PokazProdukty();
                    break;
                case 3:
                    DodajLubEdytuj(true);
                    break;
                case 4:
                    UsunProdukt();
                    break;
                case 5:
                    ZapiszDaneDoPliku();
                    break;
                case 0:
                    return;
                default:
                    Console.WriteLine("Nieprawidłowa opcja");
                    break;
            }
        }
    }

    static void DodajLubEdytuj(bool edycja = false)
    {
        Produkt produkt;

        if (edycja)
        {
            PokazProdukty();
            Console.Write("\nPodaj ID produktu do edycji: ");
            if (!int.TryParse(Console.ReadLine(), out int id) ||
                (produkt = produkty.FirstOrDefault(p => p.Id == id)) == null)
            {
                Console.WriteLine("Nie znaleziono produktu");
                return;
            }
        }
        else
        {
            produkt = new Produkt
            {
                Id = produkty.Any() ? produkty.Max(p => p.Id) + 1 : 1
            };
            produkty.Add(produkt);
        }

        Console.Write($"Nazwa ({produkt.Nazwa}): ");
        string nazwa = Console.ReadLine();
        if (!string.IsNullOrWhiteSpace(nazwa)) produkt.Nazwa = nazwa;

        Console.Write($"Cena ({produkt.Cena}): ");
        if (decimal.TryParse(Console.ReadLine(), out decimal cena))
            produkt.Cena = cena;

        Console.Write($"Kategoria ({produkt.Kategoria}): ");
        string kat = Console.ReadLine();
        if (!string.IsNullOrWhiteSpace(kat)) produkt.Kategoria = kat;

        Console.WriteLine(edycja ? "Zaktualizowano produkt": "Dodano nowy produkt");
    }

    static void PokazProdukty()
    {
        if (!produkty.Any())
        {
            Console.WriteLine("Brak produktów");
            return;
        }

        Console.WriteLine("\nLista produktów:");
        foreach (var p in produkty)
            Console.WriteLine($"{p.Id}. {p.Nazwa} | {p.Cena} zł | {p.Kategoria}");
    }

    static void UsunProdukt()
    {
        PokazProdukty();
        Console.Write("\nPodaj ID produktu do usunięcia: ");
        if (!int.TryParse(Console.ReadLine(), out int id) || produkty.All(p => p.Id != id))
        {
            Console.WriteLine("Nie znaleziono produktu.");
            return;
        }

        produkty.RemoveAll(p => p.Id == id);
        Console.WriteLine("Produkt usunięty (nie zapisano jeszcze do pliku)");
    }

    static void WczytajZCsv()
{
    if (!File.Exists(csvFile))
        return;

    produkty.Clear();
    
    foreach (var linia in File.ReadLines(csvFile).Skip(1))
    {
        var dane = linia.Split(';');
        if (dane.Length < 4) 
            continue;

        if (int.TryParse(dane[0], out int id) && 
            decimal.TryParse(dane[2], out decimal cena))
        {
            produkty.Add(new Produkt
            {
                Id = id,
                Nazwa = dane[1],
                Cena = cena,
                Kategoria = dane[3]
            });
        }
    }
}

    static void ZapiszDoCsv()
    {
        File.WriteAllLines(csvFile, new[] { "Id;Nazwa;Cena;Kategoria" }
            .Concat(produkty.Select(p => $"{p.Id};{p.Nazwa};{p.Cena};{p.Kategoria}")));
    }

    static void ZapiszDaneDoPliku()
    {
        ZapiszDoCsv();
        Console.WriteLine("Dane zostały zapisane do pliku CSV");
    }
}

class Produkt
{
    public int Id { get; set; }
    public string Nazwa { get; set; }
    public decimal Cena { get; set; }
    public string Kategoria { get; set; }
}
 