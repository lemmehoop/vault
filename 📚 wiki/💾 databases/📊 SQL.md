**select**
```sql
select name from passenger where name like '%man'

select company.name from trip join company on trip.company = company.id \
	where trip.plane = "Boeing" group by company.name

select * from trip where time_out 
	BETWEEN "1900-01-01 10:00:00" and "1900-01-01 14:00:00"

with temp(max_length) as (select max(LENGTH(name)) as max_length FROM passenger)
	select name from passenger, temp where max_length = LENGTH(name)

with grouped(name, count) as 
	(select name, count(id) as count from passenger group by name)
		select name from grouped where count > 1

select passenger.name, count(Pass_in_trip.id) as c from passenger 
	join Pass_in_trip on passenger.id = Pass_in_trip.passenger 
		group by passenger having c >= 1 order by count desc, passenger.name asc

select member_name from FamilyMembers 
	where birthday = (select min(birthday) from FamilyMembers)
```
**update**
```sql
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition;
```
**insert**
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);

INSERT INTO Customers (CustomerName, ContactName, Address, City, PostalCode, Country) VALUES
('Cardinal', 'Tom B. Erichsen', 'Skagen 21', 'Stavanger', '4006', 'Norway'),
('Greasy Burger', 'Per Olsen', 'Gateveien 15', 'Sandnes', '4306', 'Norway');
```
**DDL**
```sql
CREATE TABLE Orders (
    id int PRIMARY KEY,
    name varchar(255),
    address varchar(255),
    city varchar(255)
	FOREIGN KEY (PersonID) REFERENCES Persons(PersonID)
);
// or
ALTER TABLE users ADD CONSTRAINT fk_grade_id FOREIGN KEY (PersonID) REFERENCES Person(id);

ALTER TABLE Customers ADD / DROP Email varchar(255);
```
**joins**
```sql
-- inner join - only that are met
SELECT a.*, b.* FROM table_a a INNER JOIN table_b b ON a.id = b.a_id;

-- left join - all records from left table
SELECT a.*, b.* FROM table_a a LEFT JOIN table_b b ON a.id = b.a_id;

-- right join - all records from right table
SELECT a.*, b.* FROM table_a a RIGHT JOIN table_b b ON a.id = b.a_id;

-- full join - all records from both tables
SELECT a.*, b.* FROM table_a a FULL JOIN table_b b ON a.id = b.a_id;

-- cross join - every row mapped with every row
SELECT a.*, b.* FROM table_a a CROSS JOIN table_b b;

-- natural join - will join depending on same named column
SELECT a.*, b.* FROM table_a a NATURAL JOIN table_b b;
```