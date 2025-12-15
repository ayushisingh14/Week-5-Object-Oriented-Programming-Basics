from datetime import datetime, timedelta
import json
import os

# -------------------- Book Class --------------------
class Book:
    def __init__(self, title, author, isbn, year=None):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.year = year
        self.available = True
        self.borrowed_by = None
        self.due_date = None

    def check_out(self, member_id, days=14):
        if not self.available:
            return False
        self.available = False
        self.borrowed_by = member_id
        self.due_date = (datetime.now() + timedelta(days=days)).strftime('%Y-%m-%d')
        return True

    def return_book(self):
        self.available = True
        self.borrowed_by = None
        self.due_date = None

    def to_dict(self):
        return self.__dict__

    @classmethod
    def from_dict(cls, data):
        book = cls(data['title'], data['author'], data['isbn'], data['year'])
        book.__dict__.update(data)
        return book

    def __str__(self):
        if self.available:
            return f"{self.title} ({self.isbn}) - Available"
        return f"{self.title} ({self.isbn}) - Borrowed by {self.borrowed_by}"


# -------------------- Member Class --------------------
class Member:
    MAX_BOOKS = 5

    def __init__(self, name, member_id):
        self.name = name
        self.member_id = member_id
        self.borrowed_books = []

    def borrow_book(self, isbn):
        if len(self.borrowed_books) >= self.MAX_BOOKS:
            return False
        self.borrowed_books.append(isbn)
        return True

    def return_book(self, isbn):
        if isbn in self.borrowed_books:
            self.borrowed_books.remove(isbn)
            return True
        return False

    def to_dict(self):
        return self.__dict__

    @classmethod
    def from_dict(cls, data):
        member = cls(data['name'], data['member_id'])
        member.borrowed_books = data['borrowed_books']
        return member


# -------------------- Library Class --------------------
class Library:
    def __init__(self):
        self.books = {}
        self.members = {}
        self.load_data()

    def add_book(self, book):
        self.books[book.isbn] = book

    def register_member(self, member):
        self.members[member.member_id] = member

    def borrow_book(self, isbn, member_id):
        book = self.books.get(isbn)
        member = self.members.get(member_id)
        if not book or not member:
            return "Invalid book or member"
        if not member.borrow_book(isbn):
            return "Borrow limit reached"
        if not book.check_out(member_id):
            return "Book not available"
        return "Book borrowed successfully"

    def return_book(self, isbn, member_id):
        book = self.books.get(isbn)
        member = self.members.get(member_id)
        if book and member and member.return_book(isbn):
            book.return_book()
            return "Book returned successfully"
        return "Return failed"

    def search_books(self, keyword):
        return [b for b in self.books.values() if keyword.lower() in b.title.lower() or keyword.lower() in b.author.lower() or keyword == b.isbn]

    def save_data(self):
        with open('books.json', 'w') as f:
            json.dump({k: v.to_dict() for k, v in self.books.items()}, f, indent=4)
        with open('members.json', 'w') as f:
            json.dump({k: v.to_dict() for k, v in self.members.items()}, f, indent=4)

    def load_data(self):
        if os.path.exists('books.json'):
            with open('books.json', 'r') as f:
                data = json.load(f)
                self.books = {k: Book.from_dict(v) for k, v in data.items()}
        if os.path.exists('members.json'):
            with open('members.json', 'r') as f:
                data = json.load(f)
                self.members = {k: Member.from_dict(v) for k, v in data.items()}


# -------------------- Menu --------------------
def main():
    library = Library()

    while True:
        print("\n--- Library Management System ---")
        print("1. Add Book")
        print("2. Register Member")
        print("3. Borrow Book")
        print("4. Return Book")
        print("5. Search Books")
        print("6. Save & Exit")

        choice = input("Enter choice: ")

        if choice == '1':
            library.add_book(Book(input('Title: '), input('Author: '), input('ISBN: '), input('Year: ')))
            print("Book added")

        elif choice == '2':
            library.register_member(Member(input('Name: '), input('Member ID: ')))
            print("Member registered")

        elif choice == '3':
            print(library.borrow_book(input('ISBN: '), input('Member ID: ')))

        elif choice == '4':
            print(library.return_book(input('ISBN: '), input('Member ID: ')))

        elif choice == '5':
            results = library.search_books(input('Search keyword: '))
            for book in results:
                print(book)
            print(f"Found {len(results)} books")

        elif choice == '6':
            library.save_data()
            print("Saved. Exiting...")
            break

        else:
            print("Invalid choice")


if __name__ == '__main__':
    main()
