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
```
