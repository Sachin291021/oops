# oops
 Project of Library management system 
#include <iostream>
#include <fstream>
#include <string>
#include <iomanip>
using namespace std;

class Library {
public:
    int bookID;
    string title, author;
    bool issued;

    void getBookData() {
        cout << "\nEnter Book ID: ";
        cin >> bookID;
        cin.ignore();
        cout << "Enter Book Title: ";
        getline(cin, title);
        cout << "Enter Author Name: ";
        getline(cin, author);
        issued = false;
    }

    void showBookData() const {
        cout << left << setw(10) << bookID
             << setw(30) << title
             << setw(20) << author
             << setw(10) << (issued ? "Issued" : "Available") << endl;
    }

    int getBookID() const { return bookID; }
    bool isIssued() const { return issued; }
    void issueBook() { issued = true; }
    void returnBook() { issued = false; }
};

fstream file;
Library book;

// --- Function Prototypes ---
void addBook();
void displayAllBooks();
void searchBook(int);
void issueBook(int);
void returnBook(int);
void deleteBook(int);

int main() {
    int choice, id;

    do {
        cout << "\n========= LIBRARY MANAGEMENT SYSTEM =========\n";
        cout << "1. Add Book\n";
        cout << "2. Display All Books\n";
        cout << "3. Search Book\n";
        cout << "4. Issue Book\n";
        cout << "5. Return Book\n";
        cout << "6. Delete Book\n";
        cout << "7. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1: addBook(); break;
            case 2: displayAllBooks(); break;
            case 3:
                cout << "Enter Book ID to search: ";
                cin >> id;
                searchBook(id);
                break;
            case 4:
                cout << "Enter Book ID to issue: ";
                cin >> id;
                issueBook(id);
                break;
            case 5:
                cout << "Enter Book ID to return: ";
                cin >> id;
                returnBook(id);
                break;
            case 6:
                cout << "Enter Book ID to delete: ";
                cin >> id;
                deleteBook(id);
                break;
            case 7:
                cout << "Exiting the system... Thank you!\n";
                break;
            default:
                cout << "Invalid Choice! Try again.\n";
        }
    } while (choice != 7);

    return 0;
}

// --- Function Definitions ---

void addBook() {
    file.open("library.dat", ios::binary | ios::app);
    book.getBookData();
    file.write(reinterpret_cast<char*>(&book), sizeof(book));
    file.close();
    cout << "\nBook Added Successfully!\n";
}

void displayAllBooks() {
    file.open("library.dat", ios::binary | ios::in);
    if (!file) {
        cout << "\nFile not found!\n";
        return;
    }

    cout << "\n" << left << setw(10) << "BookID" << setw(30) << "Title"
         << setw(20) << "Author" << setw(10) << "Status" << endl;
    cout << "---------------------------------------------------------------\n";

    while (file.read(reinterpret_cast<char*>(&book), sizeof(book))) {
        book.showBookData();
    }
    file.close();
}

void searchBook(int n) {
    bool found = false;
    file.open("library.dat", ios::binary | ios::in);
    if (!file) {
        cout << "\nFile not found!\n";
        return;
    }

    while (file.read(reinterpret_cast<char*>(&book), sizeof(book))) {
        if (book.getBookID() == n) {
            cout << "\nBook Found:\n";
            book.showBookData();
            found = true;
            break;
        }
    }

    file.close();
    if (!found)
        cout << "\nBook not found!\n";
}

void issueBook(int n) {
    fstream temp;
    bool found = false;

    file.open("library.dat", ios::binary | ios::in);
    temp.open("temp.dat", ios::binary | ios::out);

    while (file.read(reinterpret_cast<char*>(&book), sizeof(book))) {
        if (book.getBookID() == n) {
            if (!book.isIssued()) {
                book.issueBook();
                cout << "\nBook issued successfully!\n";
            } else {
                cout << "\nBook is already issued!\n";
            }
            found = true;
        }
        temp.write(reinterpret_cast<char*>(&book), sizeof(book));
    }

    file.close();
    temp.close();
    remove("library.dat");
    rename("temp.dat", "library.dat");

    if (!found)
        cout << "\nBook not found!\n";
}

void returnBook(int n) {
    fstream temp;
    bool found = false;

    file.open("library.dat", ios::binary | ios::in);
    temp.open("temp.dat", ios::binary | ios::out);

    while (file.read(reinterpret_cast<char*>(&book), sizeof(book))) {
        if (book.getBookID() == n) {
            if (book.isIssued()) {
                book.returnBook();
                cout << "\nBook returned successfully!\n";
            } else {
                cout << "\nBook was not issued!\n";
            }
            found = true;
        }
        temp.write(reinterpret_cast<char*>(&book), sizeof(book));
    }

    file.close();
    temp.close();
    remove("library.dat");
    rename("temp.dat", "library.dat");

    if (!found)
        cout << "\nBook not found!\n";
}

void deleteBook(int n) {
    fstream temp;
    bool found = false;

    file.open("library.dat", ios::binary | ios::in);
    temp.open("temp.dat", ios::binary | ios::out);

    while (file.read(reinterpret_cast<char*>(&book), sizeof(book))) {
        if (book.getBookID() != n) {
            temp.write(reinterpret_cast<char*>(&book), sizeof(book));
        } else {
            found = true;
        }
    }

    file.close();
    temp.close();
    remove("library.dat");
    rename("temp.dat", "library.dat");

    if (found)
        cout << "\nBook deleted successfully!\n";
    else
        cout << "\nBook not found!\n";
}
