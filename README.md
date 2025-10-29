import csv
import os

BOOKS_FILE = 'books.csv'
ISSUED_FILE = 'issued.csv'

def load_books():
    books = {}
    if os.path.exists(BOOKS_FILE):
        with open(BOOKS_FILE, 'r', newline='') as f:
            reader = csv.DictReader(f)
            for row in reader:
                books[row['BookID']] = row
    return books

def save_books(books):
    with open(BOOKS_FILE, 'w', newline='') as f:
        fieldnames = ['BookID', 'Title', 'Author', 'Quantity']
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for book in books.values():
            writer.writerow(book)

def add_book():
    books = load_books()
    book_id = input("Enter Book ID: ")
    if book_id in books:
        print("Book ID already exists.")
        return
    title = input("Enter Title: ")
    author = input("Enter Author: ")
    quantity = input("Enter Quantity: ")
    books[book_id] = {
        'BookID': book_id,
        'Title': title,
        'Author': author,
        'Quantity': quantity
    }
    save_books(books)
    print("Book added successfully.")

def view_books():
    books = load_books()
    print("\nLibrary Books:")
    print("{:<10} {:<30} {:<20} {:<10}".format('BookID', 'Title', 'Author', 'Quantity'))
    for book in books.values():
        print("{:<10} {:<30} {:<20} {:<10}".format(book['BookID'], book['Title'], book['Author'], book['Quantity']))

def search_books():
    books = load_books()
    keyword = input("Enter title or author to search: ").lower()
    found = False
    for book in books.values():
        if keyword in book['Title'].lower() or keyword in book['Author'].lower():
            print(book)
            found = True
    if not found:
        print("No matching books found.")

def issue_book():
    books = load_books()
    book_id = input("Enter Book ID to issue: ")
    if book_id not in books:
        print("Book ID does not exist.")
        return
    if int(books[book_id]['Quantity']) <= 0:
        print("Book not available.")
        return
    borrower = input("Enter borrower name: ")
    books[book_id]['Quantity'] = str(int(books[book_id]['Quantity']) - 1)
    save_books(books)
    # Record issued book
    issued = []
    if os.path.exists(ISSUED_FILE):
        with open(ISSUED_FILE, 'r', newline='') as f:
            reader = csv.DictReader(f)
            issued = list(reader)
    issued.append({'BookID': book_id, 'Borrower': borrower})
    with open(ISSUED_FILE, 'w', newline='') as f:
        fieldnames = ['BookID', 'Borrower']
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for row in issued:
            writer.writerow(row)
    print("Book issued successfully.")

def return_book():
    books = load_books()
    book_id = input("Enter Book ID to return: ")
    borrower = input("Enter borrower name: ")
    # Remove from issued
    issued = []
    found = False
    if os.path.exists(ISSUED_FILE):
        with open(ISSUED_FILE, 'r', newline='') as f:
            reader = csv.DictReader(f)
            for row in reader:
                if row['BookID'] == book_id and row['Borrower'] == borrower and not found:
                    found = True
                    continue
                issued.append(row)
    if found:
        books[book_id]['Quantity'] = str(int(books[book_id]['Quantity']) + 1)
        save_books(books)
        with open(ISSUED_FILE, 'w', newline='') as f:
            fieldnames = ['BookID', 'Borrower']
            writer = csv.DictWriter(f, fieldnames=fieldnames)
            writer.writeheader()
            for row in issued:
                writer.writerow(row)
        print("Book returned successfully.")
    else:
        print("Record not found.")

def delete_book():
    books = load_books()
    book_id = input("Enter Book ID to delete: ")
    if book_id in books:
        del books[book_id]
        save_books(books)
        print("Book deleted successfully.")
    else:
        print("Book ID not found.")

def menu():
    while True:
        print("\n------ Library Management System ------")
        print("1. Add Book")
        print("2. View Books")
        print("3. Search Books")
        print("4. Issue Book")
        print("5. Return Book")
        print("6. Delete Book")
        print("7. Exit")
        choice = input("Enter your choice: ")
        if choice == '1':
            add_book()
        elif choice == '2':
            view_books()
        elif choice == '3':
            search_books()
        elif choice == '4':
            issue_book()
        elif choice == '5':
            return_book()
        elif choice == '6':
            delete_book()
        elif choice == '7':
            print("Exiting the system.")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == '__main__':
    menu()
