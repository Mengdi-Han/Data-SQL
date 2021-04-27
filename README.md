# Data-SQL
Using SQL to manage hotel orders and provide services
# SQL application
## Using SQL to create table and calculate different variables

### AMENITY table

```{r}
CREATE TABLE AMENITY ( 
Amenity_id INT PRIMARY KEY 
NOT NULL 
UNIQUE, 
Name VARCHAR (255) NOT NULL, 
Description VARCHAR (255) NOT NULL 
); 
```
### AMENITY_BOOKING table
```
CREATE TABLE AMENITY_BOOKING ( 
Booking_id INT NOT NULL 
PRIMARY KEY 
UNIQUE, 
Amenity_id INT NOT NULL 
REFERENCES AMENITY (Amenity_id), 
Hotel_id INT NOT NULL, 
Start_time DATETIME NOT NULL, 
Guest_id INT NOT NULL, 
Start_date DATE NOT NULL, 
End_time DATETIME NOT NULL, 
End_date DATE NOT NULL, 
Status VARCHAR (255) NOT NULL 
DEFAULT Ongoing, 
Channel_id INT NOT NULL 
); 
```
### BOOKING_CHANNEL table
```
CREATE TABLE BOOKING_CHANNEL ( 
Channel_id INT NOT NULL 
UNIQUE 
PRIMARY KEY, 
Name VARCHAR (255) NOT NULL, 
Booking_fee_percentage NUMERIC NOT NULL 
); 
```
### CHARACTERISTIC table
```
CREATE TABLE CHARACTERISTIC ( 
Characteristic_id INT NOT NULL 
PRIMARY KEY 
UNIQUE, 
Name VARCHAR (255) NOT NULL 
); 
```
### GUEST table
```
CREATE TABLE GUEST ( 
Guest_id INT PRIMARY KEY 
NOT NULL 
UNIQUE, 
First_Name VARCHAR (255) NOT NULL, 
Middle_name VARCHAR (255), 
Last_name VARCHAR (255) NOT NULL, 
Street_address VARCHAR (255) NOT NULL, 
Country VARCHAR (255) NOT NULL, 
Email VARCHAR (255), 
Phone_number_home VARCHAR (255), 
Phone_number_work VARCHAR (255), 
Phone_number_cell VARCHAR (255) 
); 
```

### HOTEL table
```
CREATE TABLE HOTEL ( 
Hotel_id INT NOT NULL 
UNIQUE 
PRIMARY KEY, 
Name VARCHAR (255) NOT NULL, 
Street_number VARCHAR (255) NOT NULL, 
Street_name INT NOT NULL, 
City VARCHAR (255) NOT NULL, 
State VARCHAR (255) NOT NULL, 
Post_code VARCHAR (255) NOT NULL, 
URL VARCHAR (255) NOT NULL, 
Phone_number VARCHAR (255) NOT NULL 
); 
```

### HOTEL_AMENITY table
```
CREATE TABLE HOTEL_AMENITY ( 
Hotel_id NOT NULL, 
Amenity_id NOT NULL, 
FOREIGN KEY ( 
Hotel_id 
) 
REFERENCES HOTEL (Hotel_id) ON DELETE CASCADE, 
FOREIGN KEY ( 
Amenity_id 
) 
REFERENCES AMENITY (Amenity_id) ON DELETE CASCADE 
); 
```
### INVOICE table
```
CREATE TABLE INVOICE ( 
Invoice_id INT UNIQUE 
PRIMARY KEY, 
Guest_id INT NOT NULL, 
Payment_time DATETIME NOT NULL, 
Card_Type VARCHAR (255) NOT NULL, 
Card_number VARCHAR (255) NOT NULL, 
Expirate_date VARCHAR (255) NOT NULL, 
Payment_amount NUMERIC NOT NULL, 
Total_balance NUMERIC NOT NULL, 
Hotel_id INT NOT NULL, 
Channel_id INT NOT NULL 
REFERENCES BOOKING_CHANNEL 
(Channel_id), 
Booking_fee_percentage NUMERIC NOT NULL 
); 
```
### PAYMENT_AMENITY_BOOKING table
```
CREATE TABLE PAYMENT_AMENITY_BOOKING ( 
Invoice_id INT NOT NULL 
UNIQUE, 
Booking_id INT NOT NULL 
UNIQUE, 
Amenity_payment_amount NUMERIC 
); 
```
### PAYMENT_ROOM_RESERVATION table
```
CREATE TABLE PAYMENT_ROOM_RESERVATION ( 
Inovice_id INT NOT NULL 
UNIQUE 
REFERENCES INVOICE (Invoice_id), 
Reservation_id INT NOT NULL 
UNIQUE, 
Room_payment_amount NUMERIC NOT NULL 
); 
```
### PREFERENCE table
```
CREATE TABLE PREFERENCE ( 
Reservation_id INT NOT NULL, 
Characteristic_id INT NOT NULL, 
Description VARCHAR (255) NOT NULL, 
FOREIGN KEY (
Characteristic_id 
) 
REFERENCES CHARACTERISTIC (Characteristic_id) 
); 
```
### ROOM table
```
CREATE TABLE ROOM (
Hotel_id INT REFERENCES Hotel (Hotel_id) 
NOT NULL, 
Room_id INT NOT NULL, 
Price NUMERIC NOT NULL, 
Type VARCHAR (255) NOT NULL, 
Floor_number NUMERIC NOT NULL, 
PRIMARY KEY ( 
Hotel_id, 
Room_id 
), 
FOREIGN KEY ( 
Hotel_id 
) 
REFERENCES HOTEL (Hotel_id) ON DELETE CASCADE 
); 
```
### ROOM_CHARACTERISTIC table
```
CREATE TABLE ROOM_CHARACTERISTIC ( 
Hotel_id INT NOT NULL 
REFERENCES HOTEL (Hotel_id), 
Room_id INT NOT NULL, 
Characteristic_id INT NOT NULL, 
Description VARCHAR (255) NOT NULL 
); 
```
### ROOM_RESERVATION table
```
CREATE TABLE ROOM_RESERVATION ( 
Reservation_id INT PRIMARY KEY 
UNIQUE, 
Hotel_id INT NOT NULL, 
Room_id INT NOT NULL, 
Guest_id INT NOT NULL, 
Desired_arrival_date DATE NOT NULL, 
Departure_date DATE NOT NULL, 
Card_number VARCHAR (255) NOT NULL, 
Expire_date TEXT NOT NULL, 
Status VARCHAR (255) NOT NULL, 
Channel_id INT NOT NULL, 
FOREIGN KEY ( 
Channel_id 
) 
REFERENCES BOOKING_CHANNEL (Channel_id) ON DELETE CASCADE 
); 
```

### 1.1 The total spent for the customer for a particular stay
(checkout invoice)
```
select PAYMENT_ROOM_RESERVATION.Inovice_id,Reservation_id,Room_payment_a 
mount,INVOICE.Guest_id,GUEST.First_name,Middle_name,Last_name 
from PAYMENT_ROOM_RESERVATION 
left join INVOICE 
on PAYMENT_ROOM_RESERVATION.Inovice_id=INVOICE.Invoice_id 
left join GUEST 
on INVOICE.Guest_id=GUEST.Guest_id 
```

### 1.2 The most valuable customers in (a) the last two months,(b) past year and (c) from the beginning of the records.
```
(a).In the last two months
Select sum(Payment_amount) as Value,Guest_id 
from INVOICE 
where Payment_time >= '2020-10-10' and Payment_time < 'localtime' 
group by Guest_id 
order by Value desc 
```
```
(b).In past year
Select sum(Payment_amount) as Value,Guest_id 
from INVOICE 
where Payment_time >= '2019-12-10' and Payment_time < 'localtime' 
group by Guest_id 
order by Value desc 
```
```
(c).From the beginning of the records
Select sum(Payment_amount) as Value ,Guest_id 
from INVOICE 
where Payment_time >= '2019-01-01' and Payment_time < 'localtime' 
group by Guest_id 
order by Value desc 
```
### 1.3 Which are the top countries where our customers come from
```
select Country,count(Country) as total_number 
from GUEST 
group by Country 
order by total_number desc 
limit 10 
```
### 1.4 How much did the hotel pay in referral fees for each of the platforms that we have contracted with?
```
(a)Total invoice
select Hotel_id,Channel_id,sum((Booking_fee_percentage*Payment_amount)/100) as Hote 
l_invoice 
from INVOICE 
group by Channel_id,Hotel_id 
order by Hotel_id 
```
```
(b)Invoice in the last month(from the first date to the last date in the last month)
select Hotel_id,Channel_id,sum((Booking_fee_percentage*Payment_amount)/100) as Hotel_invoice 
from INVOICE 
where Payment_time >= '2020-11-01' and Payment_time <='2020-11-30' 
group by Hotel_id,Channel_id 
order by Hotel_id 
```
### 1.5 What is the utilization rate for each hotel
```
select (sum(julianday(Departure_date)- julianday(Desired_arrival_date))/(4*365)) as utilization_rate,hotel_id 
from ROOM_RESERVATION 
where status='Complete'
group by Hotel_id 
```
### 1.6 Calculate the Customer Value in terms of total spent for each customer before the current booking
```
select Guest_id,(sum(Payment_amount)/count(Guest_id))as Expected_value 
from INVOICE 
group by Guest_id 
order by Expected_value desc 
```

### 1.7 Additional SQL Queries
```
(a)Calculate the country with the most value
select sum(Payment_amount)as Total_value,GUEST.Country 
from INVOICE 
left join GUEST 
on INVOICE.Guest_id = GUEST.Guest_id 
group by Country 
order by Total_value desc 
```
```
(b)The most value hotel in Scaco Hotel Group
select sum(Payment_amount)as Total_value,Hotel_id 
from INVOICE
group by Hotel_id 
order by Total_value desc
```
