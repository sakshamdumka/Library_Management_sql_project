# Library Management SQL Project

***Project Overview:***
<br>
This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.
<br>

***Project Title:*** Library Management System
<br>
***Database:*** ```library_management_db```

## Objectives
**1.** ***Set up the Library Management System Database:*** Create and populate the database with tables for branches, employees, members, books,   issued status, and return status.
<br>
**2.** ***CRUD Operations:*** Perform Create, Read, Update, and Delete operations on the data.
<br>
**3.** ***CTAS (Create Table As Select):*** Utilize CTAS to create new tables based on query results.
<br>
**4.** ***Advanced SQL Queries:*** Develop complex queries to analyze and retrieve specific data.

## Project Structure
**1.** ***Database Setup***
<br>

***Database Creation:*** Created a database named ```library_management_db```
```sql
CREATE DATABASE library_db;
```
***Table Creation:*** Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.
<br>

Create table "branch"
```sql
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);
```
Create table "employees"
```sql
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);
```
Create table "members"
```sql
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);
```
Create table "books"
```sql
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);
```
Create table "issued_status"
```sql
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);
```
Create table "return_status"
```sql
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```
<br>

***TASK 1. Create a New Book Record***
<br>
"('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```sql
INSERT INTO books
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```
***TASK 2: Update an Existing Member's Address***
```sql
UPDATE members
SET member_address = '125 main street'
WHERE member_id = 'C101';
```

***TASK 3: Delete a Record from the Issued Status Table***
<br>
***Objective: Delete the record with issued_id = 'IS121' from the issued_status table.***
```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```
***TASK 4: Retrieve All Books Issued by a Specific Employee***
<br>
***Objective: Select all books issued by the employee with emp_id = 'E101'.***
```sql
SELECT * 
FROM issued_status
WHERE issued_emp_id = 'E101'
```

***TASK 5: List Members Who Have Issued More Than One Book***
<br>
***Objective: Use GROUP BY to find members who have issued more than one***
```sql
SELECT issued_emp_id
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(*) >1
```

***TASK 6: Create Summary Tables:***
<br>
***Use CTAS to generate new tables based on query results - Find out each book with total book_issued_cnt***
```sql
CREATE TABLE books_count
AS
SELECT b.book_title,
	   COUNT(i.issued_id) as no_books_issued
FROM books b
LEFT JOIN issued_status i
ON b.isbn = i.issued_book_isbn
GROUP BY b.book_title
```

***TASK 7. Retrieve All Books in a Specific Category:***
```sql
SELECT *
FROM books
WHERE category = 'Classic'
```

***TASK 8: Find Total Rental Income by Category:***
```sql
SELECT 
    b.category,
    SUM(b.rental_price)
FROM issued_status as ist
JOIN
books as b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1
```

***TASK 9: List Members Who Registered in the Last 180 Days:***
```sql
SELECT *
FROM members
WHERE reg_date BETWEEN CURRENT_DATE - INTERVAL '180 DAYS' and CURRENT_DATE
```

***TASK 10: List Employees with Their Branch Manager's Name and their branch details:***
```sql
SELECT x.emp_name,
	   y.emp_name as manager_name,
	   x.branch_address,
	   x.contact_no
FROM 
(SELECT * FROM employees e
JOIN branch b
ON e.branch_id = b.branch_id) x
JOIN
(SELECT * FROM employees e
JOIN branch b
ON e.branch_id = b.branch_id) y
ON x.manager_id = y.emp_id
```
OR

```sql
SELECT e1.emp_id,
       e1.emp_name,
       e1.position,
       e1.salary,
       b.*,
       e2.emp_name as manager
FROM employees e1
JOIN branch b
ON e1.branch_id = b.branch_id
JOIN employees e2
ON e2.emp_id = b.manager_id
```

***TASK 11. Create a Table of Books with Rental Price Above a Certain Threshold:***
```sql
CREATE TABLE expensive_books
AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

***TASK 12: Retrieve the List of Books Not Yet Returned***
```sql
SELECT *
FROM issued_status ist
LEFT JOIN return_status rs
ON ist.issued_id = rs.issued_id
AND rs.return_id IS NULL
```

***TASK 13: Identify Members with Overdue Books
Write a query to identify members who have overdue books (assume a 30-day return period).
Display the member's_id, member's name, book title, issue date, and days overdue.***
```sql
SELECT ist.issued_member_id,
       m.member_name,
       b.book_title,
       ist.issued_date,
       (CURRENT_DATE - ist.issued_date) - 30 as days_overdue
FROM issued_status ist 
JOIN
members m
ON ist.issued_member_id = m.member_id
JOIN
books b
ON ist.issued_book_isbn = b.isbn
LEFT JOIN
return_status rs
ON ist.issued_id = rs.issued_id
WHERE return_date IS NULL
AND (CURRENT_DATE - ist.issued_date) - 30 >=1
ORDER BY 1
```

***TASK 14: Create Stored Procedure for inserting record in return_status, when an employee returns a books,
and update the status in the books table***
```sql
CREATE OR REPLACE PROCEDURE add_return_status(p_return_id VARCHAR(10), p_issued_id VARCHAR(10))
LANGUAGE plpgsql
AS $$
DECLARE
		v_isbn VARCHAR(25);
		v_book_name VARCHAR(100);
BEGIN
		INSERT INTO return_status(return_id, issued_id, return_date)
		VALUES
		(p_return_id, p_issued_id, CURRENT_DATE);

		SELECT issued_book_isbn,
			   issued_book_name
		INTO v_isbn,
			 v_book_name
	    FROM issued_status
		WHERE issued_id = p_issued_id;

		UPDATE books
		SET status = 'yes'
		WHERE isbn = v_isbn;

		RAISE NOTICE 'Thank You for returning the book: %', v_book_name;

END;
$$;
```
Call Stored Procedure
```sql
CALL add_return_status('RS125', 'IS137');
```

***TASK 15: Branch Performance Report.***
<br>
***Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned,
and the total revenue generated from book rentals.***
```sql
CREATE TABLE branch_report
AS
SELECT b.branch_id,
	   COUNT(ist.issued_id) as no_books_issued,
	   COUNT(rs.return_id) as no_books_returned,
	   SUM(bk.rental_price) as total_revenue
FROM branch b
JOIN
 employees e
ON bbranch_id = e.branch_id
LEFT JOIN
issued_status ist
ON e.emp_id = ist.issued_emp_id
LEFT JOIN
return_status rs
ON ist.issued_id = rs.issued_id
LEFT JOIN
books bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY b.branch_id
ORDER BY 1;
```

***TASK 16: CTAS: Create a Table of Active Members.***
<br>
***Use the CREATE TABLE AS (CTAS) statement to create a new table active_members
containing members who have issued at least one book in the last 2 months.***
```sql
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (
		    SELECT DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE issued_date >= CURRENT_DATE - INTERVAL '2 month'
                    )
```

***TASK 17: Find Employees with the Most Book Issues Processed***
<br>
***Write a query to find the top 3 employees who have processed the most book issues.***
<br>
***Display the employee name, number of books processed, and their branch.***
```sql
SELECT emp_id,
	   emp_name,
	   COUNT(issued_id) as no_books_processed,
	   branch_id
FROM employees e
LEFT JOIN
issued_status ist
ON e.emp_id = ist.issued_emp_id
GROUP BY emp_id, emp_name, branch_id
ORDER BY no_books_processed DESC
LIMIT 3
```

***TASK 18: Stored Procedure Objective:***
<br>
***Create a stored procedure to manage the status of books in a library system.***
<br>
***Description: Write a stored procedure that updates the status of a book in the library based on its issuance.***
<br>
***The procedure should function as follows:***
<br>
***The stored procedure should take the book_id as an input parameter.***
<br>
***The procedure should first check if the book is available (status = 'yes'). If the book is available,
it should be issued, and the status in the books table should be updated to 'no'.***
<br>
***If the book is not available (status = 'no'), the procedure should return an error message indicating
that the book is currently not available.***
```sql
CREATE OR REPLACE PROCEDURE select_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(10),
p_issued_book_isbn VARCHAR(25), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$
DECLARE
	v_status VARCHAR(10);
		v_book_title VARCHAR(100);
BEGIN
	SELECT status,
	       book_title
	       INTO v_status,
	       v_book_title
	FROM books
	WHERE isbn = p_issued_book_isbn;

	IF v_status = 'yes' THEN 
			    INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn,issued_emp_id)
			    VALUES
			    (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

			    UPDATE books
			    SET status = 'no'
			    WHERE isbn = p_issued_book_isbn;
		
			    RAISE NOTICE 'The issued book record is successfull for book: %', v_book_title;
		
	ELSE
			    RAISE NOTICE 'The book is not available with the book name: %', v_book_title;
	END IF;
END;
$$;
```
Call the Stored Procedure.
```sql
CALL select_book('IS145', 'C109', '978-0-14-118776-1', 'E104');
```
