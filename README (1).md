# SQL Assignment III: Table Manipulation

## Section 1: Project Overview

| Item | Details |
|---|---|
| Project title | Library Management System |
| Student name | Maysara Dafaallh hassan
| Student ID | 202512952 |
| Business idea | Library Management System |
| DBMS | PostgreSQL 18 (pgAdmin 4) |
| Database name | `maysara_202512952_library_system` |

The system stores book categories, books, library members, and the loans that connect members to the books they borrow.

---

## Section 2: Database and Table Design

### Tables

| Table | Purpose |
|---|---|
| `categories` | Stores book categories (parent table) |
| `members` | Stores library members (parent table) |
| `books` | Stores books; each book belongs to one category (child of `categories`) |
| `loans` | Records which member borrowed which book (child of `books` and `members`) |

### Columns and data types

**categories**: `category_id` INT (PK, auto), `category_name` VARCHAR(60) UNIQUE NOT NULL, `description` TEXT

**members**: `member_id` INT (PK, auto), `full_name` VARCHAR(100) NOT NULL, `email` VARCHAR(120) UNIQUE NOT NULL, `phone` VARCHAR(20), `join_date` DATE DEFAULT CURRENT_DATE

**books**: `book_id` INT (PK, auto), `title` VARCHAR(150), `author` VARCHAR(100), `isbn` VARCHAR(20) UNIQUE, `published_year` INT, `price` NUMERIC(8,2) CHECK >= 0, `copies_available` INT CHECK >= 0, `category_id` INT (FK)

**loans**: `loan_id` INT (PK, auto), `book_id` INT (FK), `member_id` INT (FK), `loan_date` DATE, `due_date` DATE, `returned_date` DATE, `status` VARCHAR(15) CHECK IN ('Borrowed','Returned','Overdue')

### Primary keys and foreign keys

- **Primary keys:** every table has an auto-generated `GENERATED ALWAYS AS IDENTITY` key that uniquely identifies each row.
- **Foreign keys:**
  - `books.category_id` references `categories.category_id`
  - `loans.book_id` references `books.book_id`
  - `loans.member_id` references `members.member_id`
- A foreign key guarantees that a book always has a real category and a loan always refers to a real book and a real member.

### ERD

![ERD](erd.png)

### Relationships

- One category has many books (1:N).
- One member can have many loans (1:N).
- One book can appear in many loans over time (1:N).
- Members and books are therefore linked many-to-many through `loans`. This avoids repeating member or book data in every loan row.

---

## Section 3: Database and Table Creation

```sql
CREATE DATABASE maysara_202512952_library_system;
```

```sql
CREATE TABLE categories (
    category_id   INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category_name VARCHAR(60) NOT NULL UNIQUE,
    description   TEXT
);

CREATE TABLE members (
    member_id  INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name  VARCHAR(100) NOT NULL,
    email      VARCHAR(120) NOT NULL UNIQUE,
    phone      VARCHAR(20),
    join_date  DATE NOT NULL DEFAULT CURRENT_DATE
);

CREATE TABLE books (
    book_id          INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title            VARCHAR(150) NOT NULL,
    author           VARCHAR(100) NOT NULL,
    isbn             VARCHAR(20)  NOT NULL UNIQUE,
    published_year   INT CHECK (published_year BETWEEN 1400 AND 2100),
    price            NUMERIC(8,2) NOT NULL CHECK (price >= 0),
    copies_available INT NOT NULL DEFAULT 1 CHECK (copies_available >= 0),
    category_id      INT NOT NULL,
    CONSTRAINT fk_books_category
        FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

CREATE TABLE loans (
    loan_id       INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    book_id       INT NOT NULL,
    member_id     INT NOT NULL,
    loan_date     DATE NOT NULL,
    due_date      DATE NOT NULL,
    returned_date DATE,
    status        VARCHAR(15) NOT NULL DEFAULT 'Borrowed'
                  CHECK (status IN ('Borrowed', 'Returned', 'Overdue')),
    CONSTRAINT chk_loans_dates CHECK (due_date >= loan_date),
    CONSTRAINT fk_loans_book   FOREIGN KEY (book_id)   REFERENCES books(book_id),
    CONSTRAINT fk_loans_member FOREIGN KEY (member_id) REFERENCES members(member_id)
);
```

---

## Section 4: Initial Data Insertion

```sql
INSERT INTO categories (category_name, description) VALUES
('Computer Science', 'Programming, databases, networks and software engineering'),
('Islamic Studies',  'Theology, jurisprudence and Islamic thought'),
('Literature',       'Novels, poetry and essays'),
('Medicine',         'Health sciences and public health'),
('History',          'World and regional history');

INSERT INTO members (full_name, email, phone, join_date) VALUES
('Aisha Ibrahim',  'aisha.ibrahim@example.com',  '+250788000001', '2026-01-15'),
('Omar Hassan',    'omar.hassan@example.com',    '+250788000002', '2026-02-03'),
('Fatima Ali',     'fatima.ali@example.com',     '+250788000003', '2026-03-20'),
('Yusuf Mohamed',  'yusuf.mohamed@example.com',  '+250788000004', '2026-05-11'),
('Mariam Ahmed',   'mariam.ahmed@example.com',   '+250788000005', '2026-08-01');

INSERT INTO books (title, author, isbn, published_year, price, copies_available, category_id) VALUES
('Database System Concepts',  'Silberschatz',  '978-0078022159', 2019, 85.00, 4, 1),
('Clean Code',                'Robert Martin', '978-0132350884', 2008, 45.50, 3, 1),
('Season of Migration to the North', 'Tayeb Salih', '978-0141183855', 1966, 15.00, 2, 3),
('Where There Is No Doctor',  'David Werner',  '978-0942364156', 2013, 20.00, 5, 4),
('A History of the Arab Peoples', 'Albert Hourani', '978-0674017597', 2010, 30.00, 2, 5);

INSERT INTO loans (book_id, member_id, loan_date, due_date, returned_date, status) VALUES
(1, 1, '2026-09-20', '2026-10-04', '2026-10-02', 'Returned'),
(2, 2, '2026-10-01', '2026-10-15', NULL,         'Borrowed'),
(3, 3, '2026-10-10', '2026-10-24', NULL,         'Borrowed'),
(1, 4, '2026-10-20', '2026-11-03', NULL,         'Borrowed'),
(4, 1, '2026-10-22', '2026-11-05', NULL,         'Borrowed');
```

**Verification: every table has at least 5 rows**

```sql
SELECT 'categories' AS table_name, COUNT(*) AS total_rows FROM categories
UNION ALL SELECT 'members', COUNT(*) FROM members
UNION ALL SELECT 'books',   COUNT(*) FROM books
UNION ALL SELECT 'loans',   COUNT(*) FROM loans;
```

![Row counts](screenshots/count.png)
![categories](screenshots/categories.png)
![members](screenshots/members.png)
![books](screenshots/books.png)
![loans](screenshots/loans.png)

The primary keys were generated automatically (1 to 5) and every foreign key in `books` and `loans` points to an existing parent row.

---

## Section 5: SQL Practical Tasks

### Q1: SELECT

**Q1.1 Basic retrieval.** Objective: show the title, author and price of every book.
```sql
SELECT title, author, price FROM books;
```
![Q1.1](screenshots/q1_1.png)

Explanation: retrieves only three chosen columns from the `books` table.

**Q1.2 Conditional retrieval.** Objective: list books that cost more than 30.
```sql
SELECT title, author, price FROM books WHERE price > 30;
```
![Q1.2](screenshots/q1_2.png)

Explanation: `WHERE` filters the rows, so only the expensive books are returned.

**Q1.3 Sorted retrieval.** Objective: list books from the most expensive to the cheapest.
```sql
SELECT title, price FROM books ORDER BY price DESC;
```
![Q1.3](screenshots/q1_3.png)

Explanation: `ORDER BY price DESC` sorts the result in descending order of price.

### Q2: JOIN

**Q2.1 INNER JOIN.** Objective: show each book with its category.
```sql
SELECT b.title, b.author, c.category_name
FROM books b
INNER JOIN categories c ON b.category_id = c.category_id;
```
![Q2.1](screenshots/q2_1.png)

Explanation: INNER JOIN returns only rows that have a match in both tables.

**Q2.2 LEFT JOIN.** Objective: show all members, including those with no loans.
```sql
SELECT m.full_name, l.loan_id, l.loan_date, l.status
FROM members m
LEFT JOIN loans l ON m.member_id = l.member_id;
```
![Q2.2](screenshots/q2_2.png)

Explanation: LEFT JOIN keeps every member. Mariam Ahmed has no loan, so her loan columns are NULL.

**Q2.3 Multi-table JOIN.** Objective: a full loan report across four tables.
```sql
SELECT l.loan_id, m.full_name, b.title, c.category_name,
       l.loan_date, l.due_date, l.status
FROM loans l
JOIN members    m ON l.member_id   = m.member_id
JOIN books      b ON l.book_id     = b.book_id
JOIN categories c ON b.category_id = c.category_id
ORDER BY l.loan_id;
```
![Q2.3](screenshots/q2_3.png)

Explanation: combines loans, members, books and categories to show who borrowed which book and from which category.

### Q3: UPDATE

**Q3.1 Update one record.** Objective: change the phone number of Omar Hassan.
```sql
SELECT member_id, full_name, phone FROM members WHERE email = 'omar.hassan@example.com';
UPDATE members SET phone = '+250788999999' WHERE email = 'omar.hassan@example.com';
SELECT member_id, full_name, phone FROM members WHERE email = 'omar.hassan@example.com';
```
![Q3.1 before](screenshots/q3_1_before.png)
![Q3.1 update](screenshots/q3_1_update.png)
![Q3.1 after](screenshots/q3_1_after.png)

Explanation: the unique email identifies exactly one member, and the final SELECT confirms the new number.

**Q3.2 Update multiple records.** Objective: raise Computer Science book prices by 10%.
```sql
SELECT book_id, title, price FROM books WHERE category_id = 1;
UPDATE books SET price = ROUND(price * 1.10, 2) WHERE category_id = 1;
SELECT book_id, title, price FROM books WHERE category_id = 1;
```
![Q3.2 before](screenshots/q3_2_before.png)
![Q3.2 update](screenshots/q3_2_update.png)
![Q3.2 after](screenshots/q3_2_after.png)

Explanation: the rows were identified first, then all of them were updated by one statement.

**Q3.3 Conditional update.** Objective: mark unreturned loans past their due date as Overdue.
```sql
SELECT loan_id, due_date, returned_date, status FROM loans
WHERE returned_date IS NULL AND due_date < CURRENT_DATE AND status = 'Borrowed';

UPDATE loans SET status = 'Overdue'
WHERE returned_date IS NULL AND due_date < CURRENT_DATE AND status = 'Borrowed';

SELECT loan_id, due_date, returned_date, status FROM loans ORDER BY loan_id;
```
![Q3.3 before](screenshots/q3_3_before.png)
![Q3.3 update](screenshots/q3_3_update.png)
![Q3.3 after](screenshots/q3_3_after.png)

Explanation: this applies the business rule that a late, unreturned loan must be flagged as Overdue.

### Q4: ALTER TABLE

**Q4.1 Rename a column.**
```sql
ALTER TABLE members RENAME COLUMN phone TO phone_number;
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'members';
SELECT member_id, full_name, phone_number FROM members;
```
![Q4.1 alter](screenshots/q4_1_alter.png)
![Q4.1 structure](screenshots/q4_1_structure.png)
![Q4.1 data](screenshots/q4_1_data.png)

Explanation: `phone` is now `phone_number`, and the existing numbers are still accessible.

**Q4.2 Change a data type.**
```sql
ALTER TABLE books
ALTER COLUMN published_year TYPE SMALLINT USING published_year::SMALLINT;
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'books' AND column_name = 'published_year';
SELECT title, published_year FROM books;
```
![Q4.2 alter](screenshots/q4_2_alter.png)
![Q4.2 structure](screenshots/q4_2_structure.png)
![Q4.2 data](screenshots/q4_2_data.png)

Explanation: years (1400 to 2100) fit inside SMALLINT, so the conversion is safe and no data was lost.

**Q4.3 Add a column.**
```sql
ALTER TABLE members ADD COLUMN address VARCHAR(150);
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'members' ORDER BY ordinal_position;
SELECT member_id, full_name, phone_number, address FROM members;
```
![Q4.3 alter](screenshots/q4_3_alter.png)
![Q4.3 structure](screenshots/q4_3_structure.png)
![Q4.3 data](screenshots/q4_3_data.png)

Explanation: an address is useful member information. Existing rows have NULL in the new column.

### Q5: INSERT

**Q5.1 Insert a single record.**
```sql
INSERT INTO members (full_name, email, phone_number, address)
VALUES ('Khalid Osman', 'khalid.osman@example.com', '+250788000006', 'Kigali, Gasabo');
SELECT * FROM members WHERE email = 'khalid.osman@example.com';
```
![Q5.1 insert](screenshots/q5_1_insert.png)
![Q5.1 check](screenshots/q5_1_check.png)

**Q5.2 Insert multiple records.**
```sql
INSERT INTO books (title, author, isbn, published_year, price, copies_available, category_id) VALUES
('Introduction to Algorithms', 'Cormen',        '978-0262046305', 2022, 90.00, 3, 1),
('The Study Quran',            'S. H. Nasr',    '978-0062227621', 2015, 40.00, 4, 2),
('Things Fall Apart',          'Chinua Achebe', '978-0385474542', 1958, 12.00, 6, 3);
SELECT book_id, title, author, price FROM books
WHERE isbn IN ('978-0262046305', '978-0062227621', '978-0385474542');
```
![Q5.2 insert](screenshots/q5_2_insert.png)
![Q5.2 check](screenshots/q5_2_check.png)

**Q5.3 Insert a related record.**
```sql
INSERT INTO loans (book_id, member_id, loan_date, due_date)
VALUES (
    (SELECT book_id   FROM books   WHERE title = 'Clean Code'),
    (SELECT member_id FROM members WHERE email = 'khalid.osman@example.com'),
    CURRENT_DATE,
    CURRENT_DATE + 14
);
SELECT l.loan_id, m.full_name, b.title, l.loan_date, l.due_date, l.status
FROM loans l
JOIN members m ON l.member_id = m.member_id
JOIN books   b ON l.book_id   = b.book_id
WHERE m.email = 'khalid.osman@example.com';
```
![Q5.3 insert](screenshots/q5_3_insert.png)
![Q5.3 check](screenshots/q5_3_check.png)

Explanation (Q5): a single INSERT adds one row, a multi-row INSERT adds several at once, and the child-table INSERT works only because the referenced book and member already exist.

---

## Section 6: Conclusion

In this assignment I built a small relational database for a library system using PostgreSQL. I learned how to create tables with suitable data types and constraints such as NOT NULL, UNIQUE and CHECK, and how to create parent tables before child tables. Primary keys uniquely identify each record, and foreign keys connect tables and prevent invalid data, for example a loan that refers to a book that does not exist. I practiced inserting records without manually setting the auto-generated keys, and retrieving data with SELECT, WHERE and ORDER BY. I also learned that INNER JOIN returns only matching rows, while LEFT JOIN keeps all rows from the left table, as shown by the member who has no loans. When updating data, I learned to check the target rows first and always use a precise WHERE condition to avoid unintended changes. Finally, I learned that ALTER TABLE changes the structure of a table (renaming a column, changing a data type, adding a column), while UPDATE only changes the data inside it.
