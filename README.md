# Library-System-Management-using-SQL
# Project Overview
This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries.

**1. Database Setup**
<img width="1101" height="631" alt="library_erd" src="https://github.com/user-attachments/assets/cded4b76-d4e2-49bf-84d3-cba3a913e8dd" />

**2. Table Creation**
```sql
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(20)
);

CREATE TABLE employee (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(10),
    salary INT,
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch (branch_id)
);

CREATE TABLE books (
    isbn VARCHAR(30) PRIMARY KEY,
    book_title VARCHAR(70),
    category VARCHAR(20),
    rental_price FLOAT,
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

CREATE TABLE issued_status (
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(10),
    issued_book_name VARCHAR(70),
    issued_date DATE,
    issued_book_isbn VARCHAR(20),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_emp_id) REFERENCES employee (emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES books (isbn),
    FOREIGN KEY (issued_member_id) REFERENCES members (member_id)
);

CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(70),
    return_date DATE,
    return_book_isbn VARCHAR(30),
    FOREIGN KEY (return_book_isbn) REFERENCES books (isbn)
);

**3. SQL Operations**
**Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"**
```sql
INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher) 
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');```


**Task 2: Update an Existing Member's Address**
```sql
UPDATE members
SET member_address= '234 Main St'
WHERE member_id= 'C101';```


**Task 3: Delete a Record from the Issued Status Table
Objective: Delete the record with issued_id = 'IS104' from the issued_status table.**
```sql
SELECT * FROM issued_status
WHERE issued_id= 'IS121';

DELETE FROM issued_status
WHERE issued_id= 'IS121';```


**Task 4: Retrieve All Books Issued by a Specific Employee
Objective: Select all books issued by the employee with emp_id = 'E101'.**
```sql
SELECT * FROM issued_status
WHERE issued_emp_id='E101';
```

**Task 5: List Members Who Have Issued More Than One Book
Objective: Use GROUP BY to find members who have issued more than one book.**
```sql
SELECT ist.issued_member_id, mem.member_name, COUNT(ist.issued_id) AS no_of_books
FROM issued_status AS ist
JOIN members AS mem 
ON ist.issued_member_id = mem.member_id
GROUP BY ist.issued_member_id
HAVING COUNT(ist.issued_id) > 1
;
```

**Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**
```sql
CREATE TABLE book_counts AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS no_issued
FROM books AS b
JOIN issued_status AS ist ON b.isbn = ist.issued_book_isbn
GROUP BY b.isbn;

SELECT * FROM book_counts;
```

**Data Analysis & Findings**
**Task 7. Retrieve All Books in a Specific Category:**
```sql
select * from books
where category='Classic';
```

**Task 8: Find Total Rental Income by Category:**
```sql
SELECT b.category, COUNT(*) AS Rented_Count, SUM(rental_price) AS Total_Rental_Income
FROM books AS b
JOIN issued_status AS ist 
ON b.isbn = ist.issued_book_isbn
GROUP BY category;
```

**Task 9. List Members Who Registered in the Last 180 Days**:
```sql
SELECT *
FROM members
WHERE reg_date >= CURDATE() - INTERVAL 800 DAY;
```

-- Task 10: List Employees with Their Branch Manager's Name and their branch details**:
SELECT e1.*, b.manager_id, e2.emp_name AS manager
FROM employee AS e1
JOIN branch AS b ON b.branch_id = e1.branch_id
JOIN employee AS e2 ON b.manager_id = e2.emp_id;


-- Task 11. Create a Table of Books with Rental Price Above a Certain Threshold
CREATE TABLE ex_book AS 
SELECT * FROM books
WHERE rental_price > 5;

SELECT * FROM ex_book;


-- Task 12: Retrieve the List of Books Not Yet Returned
SELECT DISTINCT i.issued_id, i.issued_book_name
FROM issued_status AS i 
LEFT JOIN return_status AS r 
ON i.issued_id = r.issued_id
WHERE r.return_date IS NULL;


-- Adding new records
INSERT INTO issued_status(issued_id, issued_member_id, issued_book_name, issued_date, issued_book_isbn, issued_emp_id)
VALUES
('IS151', 'C118', 'The Catcher in the Rye', CURRENT_DATE - INTERVAL 24 day,  '978-0-553-29698-2', 'E108'),
('IS152', 'C119', 'The Catcher in the Rye', CURRENT_DATE - INTERVAL 13 day,  '978-0-553-29698-2', 'E109'),
('IS153', 'C106', 'Pride and Prejudice', CURRENT_DATE - INTERVAL 7 day,  '978-0-14-143951-8', 'E107'),
('IS154', 'C105', 'The Road', CURRENT_DATE - INTERVAL 32 day,  '978-0-375-50167-0', 'E101');

-- Adding new column in return_status
ALTER TABLE return_status
ADD Column book_quality VARCHAR(15) DEFAULT('Good');

SET SQL_SAFE_UPDATES = 0;

UPDATE return_status
SET book_quality = 'Damaged'
WHERE issued_id  IN ('IS112', 'IS117', 'IS118');



/* Task 13: Write a query to identify members who have overdue books (assume a 30-day return period). 
Display the member's_id, member's name, book title, issue date, and days overdue. */

SELECT m.member_id, m.member_name, b.book_title, i.issued_date, 
	CURDATE() - i.issued_date AS overdue_days --  r.return_date,
FROM members AS m
JOIN issued_status AS i ON m.member_id = i.issued_member_id
JOIN books AS b ON b.isbn = i.issued_book_isbn
LEFT JOIN return_status AS r ON i.issued_id = r.issued_id
WHERE r.return_date IS NULL
AND (CURDATE() - i.issued_date) > 1000
ORDER BY m.member_id;


/* Task 14: Update Book Status on Return
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table). */

-- book is issued
select * from issued_status
where issued_book_isbn='978-0-307-58837-1';

-- not entry in return- not returned yet
select * from return_status
where issued_id='IS135';

-- status is no 
select * from books
where isbn='978-0-307-58837-1';


-- manual way
insert into return_status (return_id, issued_id, return_date, book_quality)
values('RS120', 'IS135', curdate(), 'Good');

update books
set status= 'yes'
where isbn='978-0-307-58837-1';


-- stored procudeure for Update Book Status on Return
DELIMITER $$
CREATE PROCEDURE add_return_records(
	p_return_id varchar(10), 
    p_issued_id varchar(30), 
    p_book_quality varchar(30)
)

BEGIN
	DECLARE v_isbn VARCHAR(50);
    DECLARE v_book_name VARCHAR(80);

    -- Insert into return_status
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURDATE(), p_book_quality);

    -- Fetch book details
    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id
    LIMIT 1;

    -- Update book status
    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;
	
    -- Display message
    SELECT concat('Thank you for returning : ', v_book_name) AS message;
END $$
DELIMITER ;

-- calling procedure
CALL add_return_records('RS120', 'IS135', 'Good');


/* Task 15: Branch Performance Report
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals. */

SELECT 
    b.branch_id,
    COUNT(i.issued_id) AS total_books_issue,
    COUNT(r.return_id) AS total_books_return,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status AS i
JOIN employee AS e ON i.issued_emp_id = e.emp_id
JOIN branch AS b ON e.branch_id = b.branch_id
JOIN books AS bk ON i.issued_book_isbn = bk.isbn
LEFT JOIN return_status AS r ON i.issued_id = r.issued_id
GROUP BY b.branch_id;


/* Task 16: CTAS: Create a Table of Active Members
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months. */

CREATE TABLE active_members AS 
SELECT issued_member_id, issued_date as last_issued
FROM issued_status
WHERE issued_date >= CURDATE() - INTERVAL 2 MONTH;

SELECT * FROM active_members;


/* Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch. */

SELECT e.emp_id, e.emp_name, e.branch_id, COUNT(i.issued_emp_id) AS book_processed
FROM employee AS e
JOIN issued_status AS i ON e.emp_id = i.issued_emp_id
GROUP BY e.emp_id
ORDER BY COUNT(i.issued_emp_id) DESC
LIMIT 3;


/* Task 18: Identify Members Issuing High-Risk Books
Write a query to identify members who have issued books with the status "damaged" in the return_status table. */

SELECT i.issued_id, m.member_name, b.book_title, r.book_quality
FROM issued_status AS i
JOIN books AS b ON i.issued_book_isbn = b.isbn
JOIN members AS m ON i.issued_member_id = m.member_id
LEFT JOIN return_status AS r ON i.issued_id = r.issued_id
WHERE r.book_quality = 'Damaged';


/* Task 19: Stored Procedure Objective: Create a stored procedure to manage the status of books in a library system. Description: Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows: The stored procedure should take the book_id as an input parameter. The procedure should first check if the book is available (status = 'yes'). If the book is available, it should be issued, and the status in the books table should be updated to 'no'. If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available. */

DELIMITER $$
CREATE PROCEDURE issue_book(
	p_issued_id varchar(10), 
    p_issued_member_id varchar(10), 
    p_issued_book_isbn varchar(20), 
    p_issued_emp_id varchar(10)
)

BEGIN
	declare v_status varchar(10);

	-- first check if the book is available
    select status into v_status from books where isbn= p_issued_book_isbn;
    
    IF v_status= 'yes' 
    THEN
    insert into issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
    values(p_issued_id, p_issued_member_id, curdate(), p_issued_book_isbn, p_issued_emp_id);
    
    -- update status in books
    update books
    set status = 'no'
    where isbn= p_issued_book_isbn;
    
    SELECT CONCAT('Book records added successfully for book isbn: ', p_issued_book_isbn) AS message;

    ELSE
        SELECT CONCAT('Sorry, the book is unavailable. isbn: ', p_issued_book_isbn) AS message;
    END IF;
END $$
DELIMITER ;

-- for status yes
CALL issue_book('IS155', 'C108', '978-0-06-025492-6', 'E109');

-- for status no
CALL issue_book('IS156', 'C109', '978-0-06-025492-6', 'E105');

select * from books
where isbn= '978-0-06-025492-6';


/* Task 20: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.
Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include: The number of overdue books. The total fines, with each day's fine calculated at $0.50. The number of books issued by each member. The resulting table should show: Member ID Number of overdue books Total fines */

CREATE TABLE overdue_fines AS
SELECT 
    i.issued_member_id,
    COUNT(*) AS overdue_books,
    SUM(DATEDIFF(CURDATE(), i.issued_date)) AS total_overdue_days,
    SUM(DATEDIFF(CURDATE(), i.issued_date) * 0.50) AS total_fines
FROM issued_status AS i
LEFT JOIN return_status AS r ON i.issued_id = r.issued_id
WHERE return_date IS NULL
GROUP BY i.issued_member_id
ORDER BY total_fines DESC;

SELECT * FROM overdue_fines;
```
