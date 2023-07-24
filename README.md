# WGU-Advanced-Date-Management-Postgres
Project completed for WGU Adv data management class
PostgresSQL was the primary focus of this project 
Everything was used from the DVD Rental Database tar file

DROP TABLE IF EXISTS customer_details;                                   --incase there is an existing one to prevent future problems we must drop any existing...
CREATE TABLE customer_details (                                          --Creating our data types...
	customer_id int
	,first_name varchar(60)
	,last_name varchar(60)
	,email varchar(50)
	,payment_date timestamp
	,payment_id int 
	,amount numeric (5,2)
);
-----------------------------------------------------------
DROP TABLE IF EXISTS summary;                                           --incase there is an existing one to prevent future problems we must drop any existing... 
CREATE TABLE summary(                                                    --Creating our data types...
	full_name varchar(90)
	,email varchar(50)
	,total_rentals int
	,total_spent numeric (5,2)
);
-----------------------------------------------------------
SELECT * FROM summary;                --just testing out our newly created tables...
SELECT * FROM customer_details;        --just testing out our newly created tables...
-----------------------------------------------------------
INSERT INTO customer_details (customer_id, first_name, last_name, email, payment_id, payment_date,amount)        --We must populate our detailed table by using join (Whatever join statement you use)...
SELECT C.customer_id, C.first_name, C.last_name, C.email, P.payment_id, P.payment_date, P.amount
FROM customer C
INNER JOIN payment P ON C.customer_id = P.customer_id;
-----------------------------------------------------------
CREATE OR REPLACE PROCEDURE table_refresh()                                                                      --This will delete all of our data and repopulate it with the new data...
LANGUAGE plpgsql 
AS
$$
BEGIN
	DELETE FROM customer_details;
	INSERT INTO customer_details (customer_id, first_name,last_name, email , payment_id, payment_date, amount)
	SELECT C.customer_id, C.first_name, C.last_name, C.email, P.payment_id, P.payment_date, P.amount
	FROM customer C
	INNER JOIN payment P ON C.customer_id = P.customer_id;
END;
$$
-----------------------------------------------------------
CREATE OR REPLACE FUNCTION refresh_summary()                                     --a TRIGGER FUNCTION that will delete data from the summary table and then repopulate it with the data from the detailed table, so that they are now both "in-sync"...
RETURNS TRIGGER
LANGUAGE plpgsql
AS
$refresh_summary_trigger$
BEGIN
	DELETE FROM summary;
	INSERT INTO summary (
		SELECT Concat(first_name, ' , ', last_name) AS full_name, email, Count(customer_id), Sum(amount)
		FROM customer_details
		GROUP BY full_name, email
		ORDER BY Count(customer_id) DESC
		LIMIT 100
	);
RETURN NEW;
END;
$refresh_summary_trigger$
-----------------------------------------------------------
CREATE TRIGGER refresh_summary_trigger                                  --upon the INSERT performed by the PROCEDURE, the TRIGGER will activate the TRIGGER FUNCTION and all that is inside the TRIGGER FUNCTION...
AFTER INSERT ON customer_details
FOR EACH STATEMENT
EXECUTE PROCEDURE refresh_summary();
-----------------------------------------------------------
CALL table_refresh();                                                  --Now we must call our PROCEDURE so that everything can work as intended....
-----------------------------------------------------------
 --again,just testing out some stuff. Ignore everything below if you wish...
 --Special note at the bottom...
SELECT * FROM summary;                                   

select * 
from customer_details
order by payment_date desc;
-----------------------------------------------------------
select first_name,customer_id
FROM customer_details
WHERE last_name = 'Seal';
-----------------------------------------------------------
INSERT INTO customer_details (customer_id,first_name,last_name,email,payment_date,payment_id,amount)
VALUES (526,'Karl','Seal','karl.seal@sakilacustomer.org','2025-07-16 22:25:46.996577',32099,75.99)
,(526,'Karl','Seal','karl.seal@sakilacustomer.org','2025-07-17 22:25:46.996577',32100,75.99)
,(526,'Karl','Seal','karl.seal@sakilacustomer.org','2025-07-17 22:25:46.996577',32101,12.99)
,(526,'Karl','Seal','karl.seal@sakilacustomer.org','2025-07-17 22:25:46.996577',32102,2.99);

-----------------------------------------------------------
INSERT INTO customer(customer_id,first_name,last_name,email,payment_date,payment_id,amount)
VALUES (526,'Karl','Seal','karl.seal@sakilacustomer.org')
		,(526,'Karl','Seal','karl.seal@sakilacustomer.org')
		,(526,'Karl','Seal','karl.seal@sakilacustomer.org')
		,(526,'Karl','Seal','karl.seal@sakilacustomer.org');

INSERT INTO (payment_date,payment_id,amount)
VALUES ('2025-07-16 22:25:46.996577',32099,75.99)
		,('2025-07-17 22:25:46.996577',32100,75.99)
		,('2025-07-17 22:25:46.996577',32101,12.99)
		,('2025-07-17 22:25:46.996577',32102,2.99);
---------------------
------------------
------------------------
------------------------

select *
FROM payment
ORDER BY payment_id DESC;

select *
FROM payment;

INSERT INTO payment
Values (32099,355,2,5000,4.66,'2007-12-20 17:31:48.996577');

------------------------------------------------- # END OF CODE---------------------------------------------------------

-------------------------------------------------------FINAL THOUGHTS---------------------------------------------------

-- If both summary and details table are the same based on test queries above then the code is correct --
-- please if you are student DO NOT copy and paste this code for easy answers.. it takes away from you actually learning, instead use this as inspiration! --

-- I hope this will be inspiration to some. Thank you! --
--Noah Butler--


		
