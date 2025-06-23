# HR Payroll System - SQL Schema

## Tables

### 1. `departments`
```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY AUTO_INCREMENT,
    department_name VARCHAR(100) NOT NULL
);
```

### 2. `salary_structure`
```sql
CREATE TABLE salary_structure (
    structure_id INT PRIMARY KEY AUTO_INCREMENT,
    basic DECIMAL(10, 2),
    hra DECIMAL(10, 2),
    bonus DECIMAL(10, 2),
    deductions DECIMAL(10, 2)
);
```

### 3. `employees`
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    dob DATE,
    doj DATE,
    department_id INT,
    designation VARCHAR(100),
    structure_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id),
    FOREIGN KEY (structure_id) REFERENCES salary_structure(structure_id)
);
```

### 4. `attendance`
```sql
CREATE TABLE attendance (
    attendance_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    att_date DATE,
    check_in TIME,
    check_out TIME,
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

### 5. `timesheets`
```sql
CREATE TABLE timesheets (
    timesheet_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    work_date DATE,
    hours_worked DECIMAL(4,2),
    overtime_hours DECIMAL(4,2),
    shift_type ENUM('Morning', 'Evening', 'Night'),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

### 6. `leaves`
```sql
CREATE TABLE leaves (
    leave_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    leave_type VARCHAR(50),
    start_date DATE,
    end_date DATE,
    status ENUM('Applied', 'Approved', 'Rejected'),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

### 7. `deductions`
```sql
CREATE TABLE deductions (
    deduction_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    month VARCHAR(7),
    deduction_type VARCHAR(100),
    amount DECIMAL(10, 2),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

### 8. `payslips`
```sql
CREATE TABLE payslips (
    slip_id INT PRIMARY KEY AUTO_INCREMENT,
    emp_id INT,
    salary_month VARCHAR(7),
    gross_salary DECIMAL(10,2),
    total_deductions DECIMAL(10,2),
    net_salary DECIMAL(10,2),
    generated_on DATE,
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);
```

### 9. `audit_logs`
```sql
CREATE TABLE audit_logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    table_name VARCHAR(50),
    action_type VARCHAR(50),
    action_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    performed_by VARCHAR(100)
);
```

## Sample Data

### Insert Departments
```sql
INSERT INTO departments (department_name) VALUES
('HR'), ('IT'), ('Finance'), ('Marketing');
```

### Insert Salary Structure
```sql
INSERT INTO salary_structure (basic, hra, bonus, deductions) VALUES
(30000, 10000, 5000, 2000),
(40000, 12000, 6000, 3000),
(25000, 8000, 3000, 1500);
```

### Insert Employees
```sql
INSERT INTO employees (first_name, last_name, dob, doj, department_id, designation, structure_id) VALUES
('Alice', 'Smith', '1990-01-01', '2020-05-01', 1, 'HR Executive', 1),
('Bob', 'Johnson', '1985-03-12', '2018-03-15', 2, 'Software Engineer', 2),
('Carol', 'White', '1992-06-23', '2021-07-01', 2, 'Developer', 3),
('David', 'Brown', '1988-08-15', '2019-11-20', 3, 'Accountant', 1),
('Eve', 'Green', '1991-12-02', '2022-01-10', 4, 'Marketing Lead', 2),
('Frank', 'Hall', '1994-10-05', '2023-02-25', 3, 'Auditor', 2),
('Grace', 'Miller', '1993-04-10', '2017-08-12', 1, 'HR Manager', 3),
('Hank', 'Wilson', '1995-07-30', '2020-09-01', 4, 'SEO Specialist', 1),
('Ivy', 'Taylor', '1996-11-11', '2021-04-21', 2, 'QA Engineer', 3),
('Jack', 'Anderson', '1987-02-27', '2016-12-01', 1, 'Recruiter', 1);
```

## Views

### Salary Slip View
```sql
CREATE VIEW v_salary_slips AS
SELECT
    e.emp_id,
    CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
    p.salary_month,
    ss.basic + ss.hra + ss.bonus AS gross_salary,
    p.total_deductions,
    p.net_salary
FROM
    payslips p
JOIN employees e ON p.emp_id = e.emp_id
JOIN salary_structure ss ON e.structure_id = ss.structure_id;
```

## Stored Procedure

### Generate Salary
```sql
DELIMITER $$
CREATE PROCEDURE generate_salary(IN eid INT, IN month_val VARCHAR(7))
BEGIN
    DECLARE basic DECIMAL(10,2);
    DECLARE hra DECIMAL(10,2);
    DECLARE bonus DECIMAL(10,2);
    DECLARE deduction DECIMAL(10,2);
    DECLARE gross DECIMAL(10,2);
    DECLARE net DECIMAL(10,2);

    SELECT s.basic, s.hra, s.bonus, s.deductions
    INTO basic, hra, bonus, deduction
    FROM employees e
    JOIN salary_structure s ON e.structure_id = s.structure_id
    WHERE e.emp_id = eid;

    SET gross = basic + hra + bonus;
    SET net = gross - deduction;

    INSERT INTO payslips(emp_id, salary_month, gross_salary, total_deductions, net_salary, generated_on)
    VALUES(eid, month_val, gross, deduction, net, CURDATE());
END $$
DELIMITER ;
```

## Trigger

### Log Payslip Insertions
```sql
DELIMITER $$
CREATE TRIGGER trg_insert_payslip
AFTER INSERT ON payslips
FOR EACH ROW
BEGIN
    INSERT INTO audit_logs(table_name, action_type, performed_by)
    VALUES('payslips', 'INSERT', USER());
END $$
DELIMITER ;
```

