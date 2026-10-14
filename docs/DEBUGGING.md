\# Felsökningsrapport



Jag undersökte fel 4: programmet kraschar vid felaktigt deltagarantal. När användaren skriver text i stället för ett tal vid frågan om antal deltagare avslutas applikationen oväntat.



Jag startade applikationen och valde att registrera en ny bokning. Jag fyllde i kundnamn, mötesrum, datum, starttid och sluttid som vanligt. När programmet frågade efter antal deltagare skrev jag `two` i stället för en siffra.



Programmet kraschade med felet `System.FormatException: The input string 'two' was not in a correct format.` Undantaget uppstod i metoden `CreateBooking()` i `BookingConsole.cs`, och applikationen avslutades.



Programmet skulle i stället visa ett begripligt felmeddelande och låta användaren försöka igen eller återgå till menyn, utan att krascha.



Felet fanns i metoden `CreateBooking()` i `BookingConsole.cs`, på raden:





var participants = int.Parse(Console.ReadLine() ?? string.Empty);



`Console.ReadLine()` returnerar alltid text, även när användaren skriver en siffra. Metoden `int.Parse` försöker omvandla texten till ett heltal, men om texten inte är ett giltigt tal kastar den ett `FormatException`. Eftersom undantaget inte fångades någonstans i koden avbröts hela programmet. Samma sak hände om användaren lämnade fältet tomt, eftersom en tom sträng inte heller kan omvandlas till ett tal. Koden förutsatte alltså att användaren alltid skriver ett korrekt tal, vilket man inte kan lita på.



Jag ersatte `int.Parse` med `int.TryParse`. `TryParse` försöker omvandla texten och returnerar `true` eller `false` i stället för att kasta ett undantag. Om omvandlingen misslyckas visas ett meddelande och användaren skickas tillbaka till menyn:



if (!int.TryParse(Console.ReadLine(), out var participants))

{

&#x20;   Console.WriteLine("Enter a valid participant number.");

&#x20;   return;

}



Jag startade applikationen igen och skrev `two` som antal deltagare. Programmet visade nu meddelandet "Enter a valid participant number." och återgick till menyn utan att krascha. Jag testade också med ett giltigt tal för att kontrollera att bokningar fortfarande kan registreras som vanligt. Därefter körde jag alla tester i Test Explorer, och samtliga lyckades.



Felet fanns i konsolgränssnittet, som läser inmatning från användaren. Det finns inget automatiserat test för just inmatningen, eftersom testerna testar bokningslogiken i BookingService. Jag kontrollerade därför rättningen manuellt.



En alternativ lösning hade varit att använda en loop som frågar efter antal deltagare igen tills användaren skriver ett giltigt tal. Jag valde att visa ett meddelande och återgå till menyn, eftersom lösningen är enkel, tydlig och följer samma mönster som redan används i koden för avbokning, där `int.TryParse` används på samma sätt.

