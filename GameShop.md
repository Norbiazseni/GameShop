# Játékkereskedés WPF Projekt

![image](https://github.com/user-attachments/assets/4fd5c064-f444-4868-a38d-f6ac0c4f09d3)


Ez a projekt egy WPF alkalmazás, amely bemutatja a ListView és ComboBox vezérlők használatát egy játékbolt kontextusában. A program lehetővé teszi játékok és vásárlók kezelését, valamint rendelések létrehozását.
## Funkcionalitás

- **ListView:** Játékok és vásárlók táblázatos megjelenítése különálló fülön.
- **ComboBox:** Vásárlók és játékok kiválasztása rendelések létrehozásához.
- **Rendeléskezelés:** Vásárló-játék párosítások mentése és megjelenítése.

## Felépítés

### XAML (MainWindow.xaml)
A felület két fő sorra oszlik egy `Grid` segítségével:
- **Első sor:** TabIndex, melyben 4 TabItem található(Availible Games, Customers, Pairing, Orders).
- **Második sor:** Menüelemek, melyek eltérőek minden kiválasztott TabIndexnél:
  - Availible Games: Egy `ListView` kiírja a játékokat tartalmazó beolvasott fájlból az adatokat("games.txt")
  - Customers: Egy `ListView` kiírja a vásárlókat tartalmazó beolvasott fájlból az adatokat("customers".txt")
  - Pairing: A vásárlókat és a játékokat össze lehet párosítani, és elmenteni egy "orders.txt" fájlba.
  - Orders: Az "orders.txt" fájlban lévő adatokat kiírja a program. Választani lehet játékot, ezesetben kiírja a vásárlóit az adott játéknak, vagy vásárlót, ezesetben kiírja, hogy mennyi játékot vett a vásárló.

#### Teljes ablak struktúrája
```c#
<Window x:Class="Wpf_1_GameShop.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_GameShop"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="auto" />
            <RowDefinition/>
        </Grid.RowDefinitions>
```
1. A Grid.RowDefinitons két sort hoz létre.
2. A Height="450" Width="800" az ablak méretét állítja be pixelben.

### Párosítás definíciója:
```c#
<TabItem Header="Pairing">
    <StackPanel Margin="10">
        <TextBlock Text="Customer's name:" FontWeight="Bold"/>
        <ComboBox x:Name="CustomerCombo" DisplayMemberPath="CustomerName" Width="200"/>

        <TextBlock Text="Game name:" FontWeight="Bold"/>
        <ComboBox x:Name="GameCombo" DisplayMemberPath="GameName" Width="200"/>

        <Button Content="Save" Click="SaveOrderToFile" Width="150" Margin="20"/>

        <TextBlock x:Name="StatusText" FontWeight="Bold" Foreground="Green"/>
    </StackPanel>
</TabItem>
```
1. StackPanel: Rendezi a mezőket.
2. ComboBox: Választási lehetőséget kínáló menü. 
3. x:Name="StatusText": Azonosító a TextBlock-nak, hogy a C# kódban elérjük a beírt szöveget.
4. Click="SaveOrderToFile": A gomb megnyomásakor futó eseménykezelő neve.

### Példa a ListView definíciója
```c#
<ListView x:Name="GameList">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="Name" Width="150" DisplayMemberBinding="{Binding GameName}"/>
            <GridViewColumn Header="Stile" Width="80" DisplayMemberBinding="{Binding Style}"/>
            <GridViewColumn Header="Price" Width="120" DisplayMemberBinding="{Binding Price}"/>
            <GridViewColumn Header="Ratings" Width="100" DisplayMemberBinding="{Binding Ratings}"/>
            <GridViewColumn Header="PEG" Width="100" DisplayMemberBinding="{Binding PEG}"/>
        </GridView>
    </ListView.View>
</ListView>
```
1. x:Name="GamelList": Azonosító a C# kód számára.
2. <"GridView">: Táblázatos nézetet biztosít oszlopokkal.
3. DisplayMemberBinding="{Binding GameName}": Az oszlop a GameInfo osztály GameName tulajdonságát jeleníti meg, hasonlóan a többi oszlop.

### C# (MainWindow.xaml.cs)
Adatmodellek: Három osztály definiálja az adatstruktúrát:
```c#
public class GameInfo
{
    public string GameName { get; set; }
    public string Style { get; set; }
    public int Price { get; set; }
    public string Ratings { get; set; }
    public string PEGI { get; set; }

}

public class CustomerInfo
{
    public string CustomerName { get; set; }
    public int Year { get; set; }
    public double CreditCard { get; set; }

}

public class OrderInfo
{
    public string CustomerName { get; set; }
    public string GameName { get; set; }
}

```
1. GameInfo: Egy játékadatbázist reprezentál, a GameName a neve, a Style a műfaja, a Price az ára, a Ratings az értékelése, a PEGI pedig a korhatáros besorolása az adott játéknak.
2. CustomerInfo: Egy vásárló adatait jeleníti meg, CustomerName a neve, Year, a születési éve, a CreditCard pedig a bankkártyaszáma.
3. OrderInfo: Ez a GameInfo és CustomerInfo név tulajdonságait kapcsolja össze.
4. A { get; set; } lehetővé teszi az adatkapcsolást a WPF-ben.

### Fájlbeolvasás("games.txt")
```c#
private void LoadGames(string filePath)
{
    if (!File.Exists(filePath))
        return;

    foreach (string line in File.ReadAllLines(filePath))
    {
        string[] parts = line.Split(';');
        if (parts.Length == 5)
        {
            int price = 0;
            if (int.TryParse(parts[2], out var parsedPrice))
            {
                price = parsedPrice;
            }

            Games.Add(new GameInfo
            {
                GameName = parts[0],
                Style = parts[1],
                Price = price,
                Ratings = parts[3],
                PEGI = parts[4]
            });
        }
    }
}
```
1. Hibaellenőrzés: Ha létezik a fájl, csak akkor járja végig.
2. ";" karakterenként szétválassza a beolvasott adatokat, és eltárolja a parts string tömbbe.
3. Ár int változóvá alakítása. 
4. A Games gyűjteményhez ad egy GameInfo típusú adatot, mely feltöltésre kerül a megfelelő adattípusokkal.

### Párosítás elmentése "orders.txt" fájlba
```c#
 private void SaveOrderToFile(object sender, RoutedEventArgs e)
 {
     var selectedCustomer = CustomerCombo.SelectedItem as CustomerInfo;
     var selectedGame = GameCombo.SelectedItem as GameInfo;

     if (selectedCustomer == null || selectedGame == null)
     {
         StatusText.Text = "Please select both of them!";
         StatusText.Foreground = Brushes.Red;
         return;
     }

     string line = $"{selectedCustomer.CustomerName};{selectedGame.GameName}";
     string path = System.IO.Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "orders.txt");

     try
     {
         File.AppendAllText(path, line + Environment.NewLine);
         StatusText.Text = "Save successful!";
         StatusText.Foreground = Brushes.Green;
     }
     catch (Exception ex)
     {
         StatusText.Text = "Error";
         StatusText.Foreground = Brushes.Red;
     }
 }
        
```
1. A ComboBoxban kiválasztott vásárlót és játékot felvesszük 2 változóba, ezeknek jelenlétét ellenőrzi. 
2. Formátumát, illetve a mentés helyét megadjuk(nem a felhasználó, hanem a program ezt egy bizonyos előre beállított helyre menti).
3. Hibakezelés: Ha hiba adódik a mentés közben, akkor jelzést ad.

### Felhasználó-Játékok/Játék-Felhasználók táblázása
```c#
private void ShowOrders_Click(object sender, RoutedEventArgs e)
{
    var selectedCustomer = OrderCustomerCombo.SelectedItem as CustomerInfo;
    var selectedGame = OrderGameCombo.SelectedItem as GameInfo;

    List<OrderInfo> allOrders = new List<OrderInfo>();

    // Fájl beolvasás
    if (File.Exists("orders.txt"))
    {
        foreach (string line in File.ReadAllLines("orders.txt"))
        {
            string[] parts = line.Split(';');
            if (parts.Length == 2)
            {
                allOrders.Add(new OrderInfo
                {
                    CustomerName = parts[0],
                    GameName = parts[1]
                });
            }
        }
    }

    List<OrderInfo> filteredOrders = new List<OrderInfo>();

    if (selectedCustomer != null)
    {
        filteredOrders = allOrders
            .Where(order => order.CustomerName == selectedCustomer.CustomerName)
            .ToList();
    }
    else if (selectedGame != null)
    {
        filteredOrders = allOrders
            .Where(order => order.GameName == selectedGame.GameName)
            .ToList();
    }
    else
    {
        filteredOrders.Clear();
    }

    OrderResultsListView.ItemsSource = filteredOrders;
}
```
1. A Párosítást követően mentésre került "orders.txt" fájlból beolvassuk az adatokat.
2. Létrehozzuk a filteredOrders listát, melynek OrderInfo a típusa.
3. LINQ kifejezések használata a hatékony és olvasható kód érdekében. 
4. Ha az egyik ComboBox már tartalmaz adatot, a másik nem tartalmazhat. Ha választani akarunk mindkettőből, az egyik adat értéke null.
5. Az OrderResultsListView táblázza a megfelelő adatokat.

### Vásárló és Játék működése
```c#

private void OrderCustomerCombo_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (OrderCustomerCombo.SelectedItem != null)
    {
        OrderGameCombo.SelectedItem = null;
    }
}

private void OrderGameCombo_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (OrderGameCombo.SelectedItem != null)
    {
        OrderCustomerCombo.SelectedItem = null;
    }
}
 
```
1. Megtiltja, hogy mindkettő ComboBox tartalmazzon kiválasztott adatot a rendeléseknél.

### Új felhasználó/játék hozzáadása
-Játék hozzáadása:
```c#
private void AddGameButton_Click(object sender, RoutedEventArgs e)
{
    if (string.IsNullOrWhiteSpace(GameNameInput.Text) ||
            string.IsNullOrWhiteSpace(StyleInput.Text) ||
            string.IsNullOrWhiteSpace(PriceInput.Text) ||
            string.IsNullOrWhiteSpace(RatingInput.Text) ||
            string.IsNullOrWhiteSpace(PEGIInput.Text))
    {
        MessageBox.Show("Don't leave blank spaces!");
        return;
    }

    if (!int.TryParse(PriceInput.Text, out int price))
    {
        MessageBox.Show("Invalid Price!");
        return;
    }

    GameInfo game = new GameInfo
    {
        GameName = GameNameInput.Text,
        Style = StyleInput.Text,
        Price = price,
        Ratings = RatingInput.Text,
        PEGI = PEGIInput.Text
    };

    Games.Add(game);

    string newLine = $"{game.GameName};{game.Style};{game.Price};{game.Ratings};{game.PEGI}";
    File.AppendAllText("games.txt", newLine + Environment.NewLine);

    GameNameInput.Text = "";
    StyleInput.Text = "";
    PriceInput.Text = "";
    RatingInput.Text = "";
    PEGIInput.Text = "";

    MessageBox.Show("New game added!");
}
```
-Vásárló hozzáadása:
```c#
private void AddCustomerButton_Click(object sender, RoutedEventArgs e)
{
    if (string.IsNullOrWhiteSpace(CustomerNameInput.Text) ||
        string.IsNullOrWhiteSpace(CustomerYearInput.Text) ||
        string.IsNullOrWhiteSpace(CustomerCardInput.Text))
    {
        MessageBox.Show("Please fill all fields!");
        return;
    }

    if (!int.TryParse(CustomerYearInput.Text, out int year))
    {
        MessageBox.Show("Invalid year!");
        return;
    }

    if (!double.TryParse(CustomerCardInput.Text, out double creditCard))
    {
        MessageBox.Show("Invalid credit card number!");
        return;
    }

    CustomerInfo customer = new CustomerInfo
    {
        CustomerName = CustomerNameInput.Text,
        Year = year,
        CreditCard = creditCard
    };

    Customers.Add(customer);

    string newLine = $"{customer.CustomerName};{customer.Year};{customer.CreditCard}";
    File.AppendAllText("customers.txt", newLine + Environment.NewLine);

    CustomerNameInput.Text = "";
    CustomerYearInput.Text = "";
    CustomerCardInput.Text = "";

    MessageBox.Show("New customer added!");
}

```

1. string.IsNullOrWhiteSpace(...): Ellenőrzi, hogy minden mező ki van-e töltve.
2. int.TryParse(...): Átalakítja a szöveget számmá, és hibát jelez, ha nem sikerül.
3. CustomerInfo customer = new CustomerInfo { ... }: Új vásárló objektum létrehozása a bevitt adatokkal.
4. Customers.Add(customer): Hozzáadja a vásárlót a gyűjteményhez.



### Használat
1. Indítás: A program betölti az előre definiált játékokat és vásárlókat.
2. Navigáció: TabItemek között lehet váltogatni.
3. Játékok és vásárlók párosítása:
   - Válassz ki egy játék nevét.
   -Válassz ki egy vásárló nevét.
   -Kattints a "Save" gombra.
4. Rendelések kiírása:
  -Válassz ki egy vásárló vagy egy játék nevét
  -Kilistázza a kiválasztott adathoz való megfelelő adatot.


<details> <summary>MainWindow.xaml</summary>
  
```xml
<Window x:Class="Wpf_1_GameShop.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_GameShop"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="auto" />
            <RowDefinition/>
        </Grid.RowDefinitions>

        <TabControl Grid.Row="0">
            <TabItem Header="Availible Games">
                <ListView x:Name="GameList">
                    <ListView.View>
                        <GridView>
                            <GridViewColumn Header="Name" Width="150" DisplayMemberBinding="{Binding GameName}"/>
                            <GridViewColumn Header="Stile" Width="80" DisplayMemberBinding="{Binding Style}"/>
                            <GridViewColumn Header="Price" Width="120" DisplayMemberBinding="{Binding Price}"/>
                            <GridViewColumn Header="Ratings" Width="100" DisplayMemberBinding="{Binding Ratings}"/>
                            <GridViewColumn Header="PEGI" Width="100" DisplayMemberBinding="{Binding PEGI}"/>
                        </GridView>
                    </ListView.View>
                </ListView>
            </TabItem>
            <TabItem Header="Customers">
                <ListView x:Name="CustomerList" Grid.Row="1">
                    <ListView.View>
                        <GridView>
                            <GridViewColumn Header="Name" Width="150" DisplayMemberBinding="{Binding CustomerName}"/>
                            <GridViewColumn Header="Year of Birth" Width="80" DisplayMemberBinding="{Binding Year}"/>
                            <GridViewColumn Header="Credit card number" Width="120" DisplayMemberBinding="{Binding CreditCard}"/>

                        </GridView>
                    </ListView.View>
                </ListView>
            </TabItem>
            <TabItem Header="Pairing">
                <StackPanel Margin="10">
                    <TextBlock Text="Customer's name:" FontWeight="Bold"/>
                    <ComboBox x:Name="CustomerCombo" DisplayMemberPath="CustomerName" Width="200"/>

                    <TextBlock Text="Game name:" FontWeight="Bold"/>
                    <ComboBox x:Name="GameCombo" DisplayMemberPath="GameName" Width="200"/>

                    <Button Content="Save" Click="SaveOrderToFile" Width="150"/>

                    <TextBlock x:Name="StatusText" FontWeight="Bold" Foreground="Green"/>
                </StackPanel>
            </TabItem>
            <TabItem Header="Orders" Height="20" VerticalAlignment="Bottom">
                <StackPanel Margin="10">
                    <TextBlock Text="Select customer:" FontWeight="Bold"/>
                    <ComboBox x:Name="OrderCustomerCombo" DisplayMemberPath="CustomerName" Width="200" SelectionChanged="OrderCustomerCombo_SelectionChanged"/>

                    <TextBlock Text="Select game:" FontWeight="Bold" Margin="10,10,0,0"/>
                    <ComboBox x:Name="OrderGameCombo" DisplayMemberPath="GameName" Width="200" SelectionChanged="OrderGameCombo_SelectionChanged"/>
                    <Button Content="Show orders" Width="150" Margin="0,10,0,0" Click="ShowOrders_Click"/>

                    <TextBlock Text="Results:" FontWeight="Bold" Margin="10,10,0,5"/>
                    <ListView x:Name="OrderResultsListView" Width="400" Height="200">
                        <ListView.View>
                            <GridView>
                                <GridViewColumn Header="Customer Name" Width="200" DisplayMemberBinding="{Binding CustomerName}"/>
                                <GridViewColumn Header="Game Name" Width="200" DisplayMemberBinding="{Binding GameName}"/>
                            </GridView>
                        </ListView.View>
                    </ListView>
                </StackPanel>
            </TabItem>
        </TabControl>

    </Grid>
</Window>


``` 
</details><details> <summary>MainWindow.xaml.cs</summary>
  
```c# 
using System.Collections.ObjectModel;
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.IO;

namespace Wpf_1_GameShop
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public ObservableCollection<GameInfo> Games { get; set; } = new();
        public ObservableCollection<CustomerInfo> Customers { get; set; } = new();



        public MainWindow()
        {
            InitializeComponent();

            LoadGames("games.txt");
            LoadCustomers("customers.txt");

            GameList.ItemsSource = Games;
            CustomerList.ItemsSource = Customers;

            CustomerCombo.ItemsSource = Customers;
            GameCombo.ItemsSource = Games;

            OrderCustomerCombo.ItemsSource = Customers;
            OrderGameCombo.ItemsSource = Games;


        }

        public class GameInfo
        {
            public string GameName { get; set; }
            public string Style { get; set; }
            public int Price { get; set; }
            public string Ratings { get; set; }
            public string PEGI { get; set; }

        }

        public class CustomerInfo
        {
            public string CustomerName { get; set; }
            public int Year { get; set; }
            public double CreditCard { get; set; }

        }

        public class OrderInfo
        {
            public string CustomerName { get; set; }
            public string GameName { get; set; }
        }

        private void LoadGames(string filePath)
        {
            if (!File.Exists(filePath))
                return;

            foreach (string line in File.ReadAllLines(filePath))
            {
                string[] parts = line.Split(';');
                if (parts.Length == 5)
                {
                    int price = 0;
                    if (int.TryParse(parts[2], out var parsedPrice))
                    {
                        price = parsedPrice;
                    }

                    Games.Add(new GameInfo
                    {
                        GameName = parts[0],
                        Style = parts[1],
                        Price = price,
                        Ratings = parts[3],
                        PEGI = parts[4]
                    });
                }
            }
        }


        private void LoadCustomers(string filePath)
        {
            if (!File.Exists(filePath))
                return;

            foreach (string line in File.ReadAllLines(filePath))
            {
                string[] parts = line.Split(';');
                if (parts.Length == 3)
                {
                    int year = 0;
                    if (int.TryParse(parts[1], out var parsedYear))
                    {
                        year = parsedYear;
                    }

                    double creditCard = 0;
                    if (double.TryParse(parts[2], out var parsedCard))
                    {
                        creditCard = parsedCard;
                    }

                    Customers.Add(new CustomerInfo
                    {
                        CustomerName = parts[0],
                        Year = year,
                        CreditCard = creditCard
                    });
                }
            }
        }

        private void SaveOrderToFile(object sender, RoutedEventArgs e)
        {
            var selectedCustomer = CustomerCombo.SelectedItem as CustomerInfo;
            var selectedGame = GameCombo.SelectedItem as GameInfo;

            if (selectedCustomer == null || selectedGame == null)
            {
                StatusText.Text = "Please select both of them!";
                StatusText.Foreground = Brushes.Red;
                return;
            }

            string line = $"{selectedCustomer.CustomerName};{selectedGame.GameName}";
            string path = System.IO.Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "orders.txt");

            try
            {
                File.AppendAllText(path, line + Environment.NewLine);
                StatusText.Text = "Save successful!";
                StatusText.Foreground = Brushes.Green;
            }
            catch (Exception ex)
            {
                StatusText.Text = "Error";
                StatusText.Foreground = Brushes.Red;
            }
        }

        private void OrderCustomerCombo_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (OrderCustomerCombo.SelectedItem != null)
            {
                OrderGameCombo.SelectedItem = null;
            }
        }

        private void OrderGameCombo_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (OrderGameCombo.SelectedItem != null)
            {
                OrderCustomerCombo.SelectedItem = null;
            }
        }

        private void ShowOrders_Click(object sender, RoutedEventArgs e)
        {
            var selectedCustomer = OrderCustomerCombo.SelectedItem as CustomerInfo;
            var selectedGame = OrderGameCombo.SelectedItem as GameInfo;

            List<OrderInfo> allOrders = new List<OrderInfo>();

            // Fájl beolvasás
            if (File.Exists("orders.txt"))
            {
                foreach (string line in File.ReadAllLines("orders.txt"))
                {
                    string[] parts = line.Split(';');
                    if (parts.Length == 2)
                    {
                        allOrders.Add(new OrderInfo
                        {
                            CustomerName = parts[0],
                            GameName = parts[1]
                        });
                    }
                }
            }

            // Szűrés
            List<OrderInfo> filteredOrders = new List<OrderInfo>();

            if (selectedCustomer != null)
            {
                filteredOrders = allOrders
                    .Where(order => order.CustomerName == selectedCustomer.CustomerName)
                    .ToList();
            }
            else if (selectedGame != null)
            {
                filteredOrders = allOrders
                    .Where(order => order.GameName == selectedGame.GameName)
                    .ToList();
            }
            else
            {
                filteredOrders.Clear();
            }

            // ListView frissítése
            OrderResultsListView.ItemsSource = filteredOrders;
        }
    }
}

``` 
</details> 
