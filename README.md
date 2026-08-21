# campus-lost-and-found-system

#include <iostream>
#include <string>
using namespace std;

struct Item {
    string itemID;
    string type;        // "Lost" or "Found"
    string category;
    string location;
    string date;
    string description;
    string status;      // "Available" or "Claimed"
};

// ===================== PAGE HISTORY STACK =====================
// Used to keep track of which page the user came from, so that
// pressing Enter takes them back to the correct previous page.
// Written in the same style as the course's Stack class example:
// dynamic array, constructor/destructor, Push/Pop/Top/Clear/Display.
class Stack
{
public:
    Stack();
    ~Stack();
    void push(string Value);
    string pop();
    string top();
    bool isEmpty();
    void Clear();
    void Display();

private:
    string *Array;
    int MaxCapacity;
    int CurrentPosition;
};

// Constructor (fixed size of 50 pages)
Stack::Stack()
{
    Array = new string[50];
    MaxCapacity = 50;
    CurrentPosition = -1;
}

// Destructor
Stack::~Stack()
{
    delete[] Array;
}

// Method Push
void Stack::push(string Value)
{
    if (CurrentPosition < MaxCapacity - 1)
    {
        CurrentPosition = CurrentPosition + 1;
        Array[CurrentPosition] = Value;
    }
    else
        cout << "Stack Overflow\n";
}

// Method Pop
string Stack::pop()
{
    string Result;

    if (CurrentPosition >= 0)
    {
        Result = Array[CurrentPosition];
        CurrentPosition = CurrentPosition - 1;

        return Result;
    }
    else
        cout << "Stack Underflow\n";

    return "Main Menu";   // safe fallback so pauseAndReturn always has a page name
}

// Method Top or Peek
string Stack::top()
{
    string Result;

    if (CurrentPosition >= 0)
    {
        Result = Array[CurrentPosition];

        return Result;
    }
    else
        cout << "Stack Underflow\n";

    return "Main Menu";
}

bool Stack::isEmpty()
{
    return CurrentPosition == -1;
}

// Method Clear or Reset
void Stack::Clear()
{
    CurrentPosition = -1;
}

// Method Display
void Stack::Display()
{
    int i;
    cout << "Display of Stack\n";
    for (i = 0; i < CurrentPosition + 1; i++)
        cout << Array[i] << endl;
}

Item* items = new Item[10];
int capacity = 10;
int countItem = 0;

int k = 0, w = 0, e = 0, b = 0, t = 0;

Stack pageStack;   // holds the page history

// ===================== PAUSE / RETURN FUNCTION =====================
// Pops the previous page off the stack, clears any leftover input,
// and waits for the user to press Enter (exactly once) before going
// back to it. cin.sync() discards anything already sitting in the
// buffer (e.g. a stray '\n' left behind by cin >>) without blocking,
// so the cin.get() that follows always waits for exactly one fresh
// key press, no matter whether the previous read used cin >> or
// getline().
void pauseAndReturn()
{
    string previousPage = pageStack.pop();

    cout << "\nPress Enter to return to " << previousPage << "...";
    cin.clear();
    cin.sync();
    cin.get();
}

void resizeArray() {
    capacity *= 2;

    Item* temp = new Item[capacity];

    for (int i = 0; i < countItem; i++)
        temp[i] = items[i];

    delete[] items;
    items = temp;
}

// ===================== ID GENERATOR =====================
string generateID(string cat)
{
    string prefix;
    int* counter;

    if (cat == "Key") { prefix = "K"; counter = &k; }
    else if (cat == "Wallet") { prefix = "W"; counter = &w; }
    else if (cat == "Electronics") { prefix = "E"; counter = &e; }
    else if (cat == "Water Bottle") { prefix = "B"; counter = &b; }
    else { prefix = "T"; counter = &t; }

    (*counter)++;

    string num;

    if (*counter < 10)
        num = "00" + to_string(*counter);
    else if (*counter < 100)
        num = "0" + to_string(*counter);
    else
        num = to_string(*counter);

    return prefix + num;
}

// Helper to draw horizontal lines for item table
void printLine()
{
    cout << "+------+---------+----------------+--------------------+------------+-----------+---------------------------+\n";
}

void printHeader()
{
    printLine();
    cout << "| ID   | Type    | Category       | Location           | Date       | Status    | Description               |\n";
    printLine();
}

// Manually left-aligns text within a fixed width by padding with
// spaces, replacing what setw()/left from <iomanip> used to do.
string padLeft(string text, int width)
{
    if ((int)text.length() >= width)
        return text.substr(0, width);   // truncate if too long

    return text + string(width - text.length(), ' ');
}

void printRow(Item& x)
{
    cout << "| "
        << padLeft(x.itemID, 4) << " | "
        << padLeft(x.type, 7) << " | "
        << padLeft(x.category, 14) << " | "
        << padLeft(x.location, 18) << " | "
        << padLeft(x.date, 10) << " | "
        << padLeft(x.status, 9) << " | "
        << padLeft(x.description, 25) << " |\n";
}

// ===================== ADD ITEM =====================
void addItem(string type)
{
    pageStack.push("Main Menu");

    if (countItem == capacity)
        resizeArray();

    Item item;

    int choice;

    cout << "\n================= SELECT CATEGORY =================" << endl;
    cout << "1. Key\n";
    cout << "2. Wallet\n";
    cout << "3. Electronics\n";
    cout << "4. Water Bottle\n";
    cout << "5. Others\n";
    cout << "0. Return to Main Menu\n";
    cout << "Enter choice: ";
    
    cin >> choice;

    while (cin.fail())      // 防止用户输入 ABC导致死机
    {
        cin.clear();
        cin.ignore(1000, '\n');

        cout << "Invalid input. Enter again: ";
        cin >> choice;
    }

    if (choice == 0)
    {
        cout << "Returning to Main Menu...\n";
        pauseAndReturn();
        return;
    }

    switch (choice)
    {
    case 1: 
        item.category = "Key"; 
        break;
    case 2: 
        item.category = "Wallet"; 
        break;
    case 3: 
        item.category = "Electronics"; 
        break;
    case 4: 
        item.category = "Water Bottle"; 
        break;
    default: 
        item.category = "Others";
    }

    cin.ignore();       // clear buffer

    cout << "Location: ";
    getline(cin, item.location);

    cout << "Date (DD-MM-YYYY): ";
    getline(cin, item.date);

    cout << "Description: ";
    getline(cin, item.description);

    item.type = type;
    item.itemID = generateID(item.category);
    item.status = "Available";

    items[countItem++] = item;

    cout << "Item Added. ID: " << item.itemID << endl;
    pauseAndReturn();
}

void displayItems()
{
    pageStack.push("Main Menu");

    if (countItem == 0)
    {
        cout << "No records.\n";
        pauseAndReturn();
        return;
    }

    printHeader();

    for (int i = 0; i < countItem; i++)
        printRow(items[i]);

    printLine();
    pauseAndReturn();
}

// ===================== SEARCH =====================
void searchCategory()
{
    pageStack.push("Main Menu");

    int choice;
    string cat;

    cout << "\n================= SELECT CATEGORY =================" << endl;
    cout << "1. Key\n";
    cout << "2. Wallet\n";
    cout << "3. Electronics\n";
    cout << "4. Water Bottle\n";
    cout << "5. Others\n";
    cout << "0. Return to Main Menu\n";
    cout << "Enter choice: ";
    cin >> choice;

    if (choice == 0)
    {
        cout << "Returning to Main Menu...\n";
        pauseAndReturn();
        return;
    }

    switch (choice)
    {
    case 1: cat = "Key"; break;
    case 2: cat = "Wallet"; break;
    case 3: cat = "Electronics"; break;
    case 4: cat = "Water Bottle"; break;
    default: cat = "Others";
    }

    bool found = false;

    for (int i = 0; i < countItem; i++)
    {
        if (items[i].category == cat)
        {
            if (!found)
            {
                printHeader();
                found = true;
            }

            printRow(items[i]);
        }
    }

    if (found) printLine();
    else cout << "No items found.\n";

    pauseAndReturn();
}

// ===================== SORT =====================
void sortItems()
{
    pageStack.push("Main Menu");

    for (int i = 0; i < countItem - 1; i++)
    {
        int minIndex = i;

        for (int j = i + 1; j < countItem; j++)
        {
            if (items[j].itemID < items[minIndex].itemID)
                minIndex = j;
        }

        Item temp = items[i];
        items[i] = items[minIndex];
        items[minIndex] = temp;
    }

    cout << "Selection Sort completed.\n";
    pauseAndReturn();
}

// ===================== CLAIM =====================
void claim()
{
    pageStack.push("Main Menu");

    string id;

    cout << "Enter Item ID: ";
    cin >> id;

    for (int i = 0; i < countItem; i++)
    {
        if (items[i].itemID == id)
        {
            if (items[i].status == "Claimed")
            {
                cout << "Already claimed.\n";
                pauseAndReturn();
                return;
            }

            items[i].status = "Claimed";
            cout << "Claim processed. Item " << id << " is now marked as Claimed.\n";
            pauseAndReturn();
            return;
        }
    }

    cout << "Item not found.\n";
    pauseAndReturn();
}

// ===================== MAIN MENU =====================
int main()
{
    int choice;

    do
    {
        cout << "\n======================================================\n";
        cout << "              CAMPUS LOST & FOUND SYSTEM              \n";
        cout << "======================================================\n";

        cout << "1. Add Lost Item\n";
        cout << "2. Add Found Item\n";
        cout << "3. Display All Items\n";
        cout << "4. Search by Category\n";
        cout << "5. Sort Items by ID\n";
        cout << "6. Claim Item\n";
        cout << "7. Exit\n";
        cout << "======================================================\n";
        
        cout << "Enter your choice: ";
        cin >> choice;

        while (cin.fail())
        {
            cin.clear();
            cin.ignore(1000, '\n');
            cout << "Invalid menu input! Please enter a number from 1 to 7.\n";
            cin >> choice;
        }

        switch (choice)
        {
        case 1: 
            addItem("Lost"); 
            break;
        case 2: 
            addItem("Found"); 
            break;
        case 3:
            displayItems(); 
            break;
        case 4: 
            searchCategory(); 
            break;
        case 5: 
            sortItems(); 
            break;
        case 6: 
            claim(); 
            break;
        case 7: 
            cout << "Exiting system... Thank you!\n"; 
            break;
        default: 
            cout << "Invalid choice! Please select between 1 and 7.\n";
            pageStack.push("Main Menu");
            pauseAndReturn();
        }

    } while (choice != 7);

    delete[] items;

    return 0;
}
