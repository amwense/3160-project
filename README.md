# ITCS 3160 Project
Member: Adona Mwense
# Introduction
In light of events affecting the student mobility on campus, universities are working to see if they should have their own deliveries services to prevent the stream of individuals who have no ties to the university. Thus, ensuring authorized university employees are the only ones delivering food on campus for safety and health reasons. Given certain requirement, I design a fully normalized database system using business rules, entity relationship diagramming, normalization and schema design modeling this system.
# Use Case
A registered client (Student, Faculty or Staff) can make an order and pays for it. A restaurant can receive an order, confirm it an prepare the order. Then the driver (Student) pick us the prepared order and delivers it to the client. And to finish the client rates the driver.

<img src="ITCS3160Pictures/UseCase.jpg" >

# Business Rules

- Persons (campus faculty, staff, students) have accounts in the system.

- Locations are spots on campus where food can be delivered.

- Persons can also be drivers (delivery personnel which have to be approved). 

- All delivery personnel are students.

- There are 8 initial delivery drivers.

- There is a flat fee of $5 for each delivery.

- A person orders food many times.

- An individual delivery is tied to only one person for the order.

- The order is for only one restaurant.  

 
# EERD

<img src="ITCS3160Pictures/ProjectERDiagram.jpg" >

## Data Dictionary

Person
<img src="ITCS3160Pictures/person.png" >

Faculty
<img src="ITCS3160Pictures/faculty.png" >

Staff
<img src="ITCS3160Pictures/staff.png" >

Student
<img src="ITCS3160Pictures/student.png" >

Driver
<img src="ITCS3160Pictures/drivers.png" >

Location
<img src="ITCS3160Pictures/location.png" >

Order
<img src="ITCS3160Pictures/orders.png" >

Restaurant
<img src="ITCS3160Pictures/restaurant.png" >


# MySQL Queries
As a demonstration of how the tables work together here are some simple queries. The first return information about order that were made by students who are registered as drivers in the system. The second query returns how many students from each major made an order. And the last query is simply return the most common locations where faculty orders were delivered to.

Query 1
```sql
select student.name as Client, rest.name as Restaurant_Name, idOrders as Order_ID, price as Total
from orders
inner join student on orders.client_id = student.idStudent
inner join restaurants as rest on orders.restaurant_id = rest.idRestaurants
where client_id IN (select idDrivers from drivers)
order by price desc;
```
Query 2
```sql
select major as Student_Major, count(major) as Total
from student
inner join orders on student.idStudent = orders.client_id
group by major
order by Total desc;
```
Query 3
```sql
select loc.location_name as Location_Name, count(loc.location_name) as Total
from orders
inner join locations as loc on orders.location_id = loc.idLocations
where orders.client_id In (select idFaculty from faculty)
group by loc.location_name
order by Total;
```
# Trigger
Since person has to be either a faculty, a staff or a student. It seems logical to insert have a trigger that insert a faculty, staff or student in the person table before inserting them in their respective table.

Faculty
```sql
CREATE DEFINER=`root`@`localhost` TRIGGER `faculty_BEFORE_INSERT` 
BEFORE INSERT ON `faculty` FOR EACH ROW 
BEGIN
	   insert into person
    values(new.idFaculty, new.name, new.email, new.phone_number);
END
```
Staff
```sql
CREATE DEFINER=`root`@`localhost` TRIGGER `staff_BEFORE_INSERT` 
BEFORE INSERT ON `staff` FOR EACH ROW 
BEGIN
	   insert into person
    values(new.idStaff, new.name, new.email, new.phone_number);
END
```
Student
```sql
CREATE DEFINER=`root`@`localhost` TRIGGER `student_BEFORE_INSERT` 
BEFORE INSERT ON `student` FOR EACH ROW 
BEGIN
	   insert into person
    values(new.idStudent, new.name, new.email, new.phone_number);
END
```
# Stored Procedure
Based on the content of the database some recurring queries might be to try to find who made specific deliveries, what orders a specific client has made or what group of peoples (student, faculty, or staff) makes the most order etc. Upon this, it was decided to include some of the queries as stored procedures.
```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `deliveries_made`(IN i_driver_id INT)
    READS SQL DATA
BEGIN
	select drivers.name as Delivery_Driver, person.name as Client, idOrders
	from orders
	join drivers
	on orders.driver_id = drivers.idDrivers
	join person
	on orders.client_id = person.idPerson
	where driver_id = i_driver_id;
END
```
## Views
While keeping the same idea as for stored procedures and the concept that view are generally used for reporting purposes, it was decided views limiting the number of information about the people in the database to the strict necessary could be helpful in addition to a similar table about the orders made. 
```sql
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `orders_made` AS
    SELECT 
        `person`.`name` AS `Client_Name`,
        `restaurants`.`name` AS `Restaurant`,
        `orders`.`price` AS `Total_Price`
    FROM
        ((`orders`
        JOIN `person` ON ((`orders`.`client_id` = `person`.`idPerson`)))
        JOIN `restaurants` ON ((`orders`.`restaurant_id` = `restaurants`.`idRestaurants`)))
```

```sql
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `root`@`localhost` 
    SQL SECURITY DEFINER
VIEW `person_info` AS
    SELECT 
        `person`.`name` AS `name`, `person`.`email` AS `email`
    FROM
        `person`
```
## Indexes
As the main purpose of indexes is to make query run faster, I did not judge the necessity of creating them for this particular database. Index are more useful in databases that contains a large number of information in diverse table. In large databases, running complex or advanced queries will take longer time compared to a much smaller database. Since this database is small it wouldn’t be much of an improvement to add a table that refers to tables already in the database.

# Future Work
Considering that this project is only a prototype, there would be quite a few ways to amend to it. For instance, ameliorations can be made in order for the database to be operational in the context of food delivery. A table relating restaurant to menus and might be necessary. In addition to that, there could be various delivery fees depending on the distance between the restaurant and the drop off location. Lastly, it would be good to have an user interface to see how a potential users can interact with the database in a working environment depending on their roles.
# MySQL Dump
```sql
-- MySQL dump 10.13  Distrib 8.0.19, for Win64 (x86_64)
--
-- Host: localhost    Database: mydb
-- ------------------------------------------------------
-- Server version	8.0.19

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!50503 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `drivers`
--

DROP TABLE IF EXISTS `drivers`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `drivers` (
  `idDrivers` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `email` varchar(250) DEFAULT NULL,
  `license_num` varchar(45) DEFAULT NULL,
  `rating` int DEFAULT NULL,
  `date_hired` date DEFAULT NULL,
  PRIMARY KEY (`idDrivers`),
  CONSTRAINT `fk_Drivers_Student1` FOREIGN KEY (`idDrivers`) REFERENCES `student` (`idStudent`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `drivers`
--

LOCK TABLES `drivers` WRITE;
/*!40000 ALTER TABLE `drivers` DISABLE KEYS */;
INSERT INTO `drivers` VALUES (781227,'Dominic Dodson','ut.lacus@Proineget.org','256842698532',3,'2020-01-25'),(781237,'Nigel J. Spears','Sed@nisi.com','253654781265',4,'2020-01-05'),(781247,'Conan Z. Perez','massa.Quisque.porttitor@non.com','256932547852',5,'2019-08-02'),(781257,'Jonah C. Mcdowell','cursus.et@ullamcorpervelitin.org','256325485179',5,'2018-10-07'),(781267,'Claire Q. Burt','id.libero@fringillami.ca','741256398546',4,'2020-03-16'),(781277,'Gretchen B. Drake','Nunc.quis@Donecvitaeerat.co.uk','4256325879654',3,'2019-04-13'),(781287,'Vance I. Leblanc','cursus.diam@sitametrisus.co.uk','956321425785',4,'2017-07-15'),(781297,'Tad L. Mckinney','dolor@dictumeu.co.uk','586214257953',5,'2020-04-10');
/*!40000 ALTER TABLE `drivers` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `faculty`
--

DROP TABLE IF EXISTS `faculty`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `faculty` (
  `idFaculty` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `email` varchar(250) DEFAULT NULL,
  `phone_number` varchar(15) DEFAULT NULL,
  `title` varchar(250) DEFAULT NULL,
  `highest_degree` varchar(250) DEFAULT NULL,
  `degreecollege` varchar(250) DEFAULT NULL,
  PRIMARY KEY (`idFaculty`),
  CONSTRAINT `fk_Faculty_Person1` FOREIGN KEY (`idFaculty`) REFERENCES `person` (`idPerson`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `faculty`
--

LOCK TABLES `faculty` WRITE;
/*!40000 ALTER TABLE `faculty` DISABLE KEYS */;
INSERT INTO `faculty` VALUES (761227,'Adara Q. Lynn','vitae@indolorFusce.net','(182) 707-8866','Professor','Master','Auctor Quis Tristique PC'),(761228,'Keely C. Hoffman','aliquet@risusaultricies.co.uk','(922) 839-1471','Professor','Master','Id Risus LLP'),(761229,'August R. Hale','nec.orci.Donec@maurisrhoncusid.ca','(596) 592-9053','Professor','Master','Dapibus Ligula Aliquam Foundation'),(761230,'Avye J. Gray','arcu.vel.quam@eget.edu','(624) 155-7927','Professor','Master','Vulputate Lacus Company'),(761231,'Xena Bond','diam@egestasa.net','(316) 367-2107','Professor','Master','Ac PC'),(761232,'Marah Chapman','Suspendisse@lacusMauris.com','(181) 246-6446','Professor','Master','Inceptos LLC'),(761233,'Lionel Collier','tempor.lorem.eget@lacusCrasinterdum.com','(944) 164-6847','Professor','Master','A Facilisis Ltd'),(761234,'Emily I. Romero','euismod.et.commodo@Vivamus.edu','(164) 615-2350','Professor','Master','Sed Consequat Corp.'),(761235,'Macey Bailey','lectus@vitaerisus.ca','(817) 213-4364','Professor','Master','In Inc.'),(761236,'Tyler F. Price','ipsum@nisiCumsociis.org','(549) 580-5025','Professor','Master','Ultrices Inc.'),(761237,'Linus Burke','mi.felis@tinciduntpede.co.uk','(362) 405-0929','Assistant Professor','Doctorate','Libero Est Congue Company'),(761238,'Alea Crane','sapien.Nunc.pulvinar@commodoipsum.net','(661) 402-6767','Assistant Professor','Doctorate','Sodales PC'),(761239,'Freya O. Newman','tellus.Phasellus@elitpretiumet.ca','(336) 249-6862','Assistant Professor','Doctorate','Quis Accumsan LLP'),(761240,'Jescie V. Mann','at.augue.id@Aliquamultrices.com','(882) 133-6841','Assistant Professor','Doctorate','Amet Lorem Semper Industries'),(761241,'Petra V. Pierce','Nunc.mauris.Morbi@arcuVivamus.edu','(637) 527-4715','Assistant Professor','Doctorate','Etiam LLC'),(761242,'Sybill Mclaughlin','ultrices@Integersemelit.edu','(379) 996-6371','Assistant Professor','Doctorate','Vestibulum Nec Inc.'),(761243,'Olympia Velez','viverra.Maecenas.iaculis@commodoat.net','(495) 728-3343','Assistant Professor','Doctorate','Et Eros PC'),(761244,'Kennedy P. Hatfield','lobortis@ligula.org','(594) 455-8400','Assistant Professor','Doctorate','Rutrum Inc.'),(761245,'Dante D. Watkins','nisl@sitametdiam.co.uk','(965) 473-8041','Assistant Professor','Doctorate','Augue PC'),(761246,'Brenden Washington','mus.Aenean@nasceturridiculus.com','(334) 210-6349','Assistant Professor','Doctorate','Velit Cras Associates'),(761247,'Miranda Richardson','eget.odio@semper.edu','(937) 647-7323','Lecturer','Master','Accumsan Convallis Ante Inc.'),(761248,'Byron F. Lawson','mus@sapienNuncpulvinar.edu','(867) 914-7167','Lecturer','Master','Vitae Odio Industries'),(761249,'Avram Newman','lectus@libero.edu','(802) 457-2383','Lecturer','Master','Nisl Maecenas Malesuada Company'),(761250,'Jacob X. Jackson','sem@dapibus.co.uk','(284) 127-7575','Lecturer','Master','Lorem Ut Aliquam Incorporated'),(761251,'Brenna Carlson','volutpat.Nulla.dignissim@tristique.edu','(265) 558-6879','Lecturer','Master','At Pretium Aliquet Associates'),(761252,'Nasim L. Dalton','vel.pede@ut.edu','(984) 248-8297','Lecturer','Master','In Limited'),(761253,'Cairo Harding','Ut@temporlorem.edu','(230) 962-8128','Lecturer','Master','Accumsan Ltd'),(761254,'Garth N. Burton','iaculis.odio@egestas.ca','(287) 550-3925','Lecturer','Master','Ligula Nullam Enim Industries'),(761255,'Nyssa B. Gregory','vel.nisl.Quisque@quistristiqueac.com','(766) 638-6826','Lecturer','Master','Aliquet Libero Integer Ltd'),(761256,'Jamalia B. Conway','neque.pellentesque@Craseget.co.uk','(715) 161-9706','Lecturer','Master','Vulputate PC'),(761257,'Danielle R. Cobb','Quisque.nonummy.ipsum@DuisgravidaPraesent.org','(339) 997-6624','Professor','Doctorate','Mauris Sit Amet LLP'),(761258,'Tamekah Barber','Donec.fringilla.Donec@semper.net','(616) 959-8865','Professor','Doctorate','Leo Cras Institute'),(761259,'Burton Q. Miller','orci@turpisAliquam.org','(611) 771-5218','Professor','Doctorate','Nascetur Inc.'),(761260,'Lewis S. Wilder','metus@Aliquamrutrum.net','(561) 867-2690','Professor','Doctorate','Eu Limited'),(761261,'Plato L. Lloyd','sit.amet@urna.co.uk','(451) 278-2385','Professor','Doctorate','Mauris Associates'),(761262,'Quynn Beasley','Cras.interdum@nec.co.uk','(440) 203-5748','Professor','Doctorate','Tincidunt Associates'),(761263,'Anika Hodges','molestie.in@lorem.net','(908) 940-6294','Professor','Doctorate','Nibh Phasellus Nulla Industries'),(761264,'Bernard O. Gray','Duis.ac@antedictum.com','(168) 793-6603','Professor','Doctorate','Fusce Fermentum Fermentum Corp.'),(761265,'Melinda U. Weeks','lorem@Aliquamnecenim.co.uk','(120) 848-2171','Professor','Doctorate','Ultrices Sit Amet Ltd'),(761266,'Daniel Cook','dui.Suspendisse.ac@Donecluctusaliquet.org','(558) 169-8561','Professor','Doctorate','Adipiscing Elit Aliquam Industries'),(761267,'Jenette C. Powers','porttitor@commodoipsumSuspendisse.ca','(738) 101-1532','Assistant Professor','Master','Non Lorem Vitae Limited'),(761268,'Drew Whitfield','dolor.elit@sitamet.co.uk','(522) 773-5970','Assistant Professor','Master','Tincidunt Orci Consulting'),(761269,'Imogene Q. Wright','parturient.montes@est.org','(441) 475-6526','Assistant Professor','Master','Eget Magna Suspendisse LLP'),(761270,'Ainsley Y. Curry','sit.amet.consectetuer@laciniaorci.co.uk','(733) 631-7044','Assistant Professor','Master','Tristique Pharetra Consulting'),(761271,'Cassady X. Randolph','ipsum@necluctusfelis.edu','(235) 721-5952','Assistant Professor','Master','Dis Parturient Associates'),(761272,'Rebekah M. Burris','lobortis@etarcu.ca','(456) 636-2455','Assistant Professor','Master','Sapien Cras Dolor Ltd'),(761273,'Brianna Z. Guzman','ac.risus.Morbi@tinciduntneque.co.uk','(398) 393-1295','Assistant Professor','Master','Nec Ante Ltd'),(761274,'Kylynn Calhoun','libero.Proin@arcuVestibulum.net','(236) 602-6142','Assistant Professor','Master','Duis Dignissim Corporation'),(761275,'Adrian A. Aguirre','ut.lacus@Duis.edu','(600) 280-1700','Assistant Professor','Master','Cras Dolor LLC'),(761276,'Nomlanga S. Villarreal','non@Duis.edu','(578) 907-5653','Assistant Professor','Master','Ullamcorper Nisl Corp.'),(761277,'Jaime Weeks','elit.sed.consequat@In.net','(172) 590-5209','Lecturer','Doctorate','Vulputate Eu LLP'),(761278,'Chantale I. Holland','at.egestas@enim.co.uk','(181) 878-8821','Lecturer','Doctorate','Vestibulum Limited'),(761279,'Rosalyn Calderon','Cras.pellentesque@adipiscinglobortisrisus.edu','(562) 593-3128','Lecturer','Doctorate','Sed Dui Fusce Limited'),(761280,'Yoshio Pope','ridiculus.mus@blandit.edu','(967) 576-7558','Lecturer','Doctorate','Vel Vulputate Eu PC'),(761281,'Briar K. Rush','orci.lacus@conguea.ca','(792) 634-5587','Lecturer','Doctorate','Donec Corp.'),(761282,'Selma M. Bernard','aliquet.odio.Etiam@lacusEtiambibendum.ca','(913) 349-5245','Lecturer','Doctorate','Dolor Institute'),(761283,'Nadine Pierce','id.ante@rhoncus.com','(674) 630-7106','Lecturer','Doctorate','Phasellus Dapibus Ltd'),(761284,'Tatiana G. Morton','Aliquam@semperNam.net','(699) 455-4557','Lecturer','Doctorate','Felis Nulla Tempor Limited'),(761285,'Daphne K. Kelley','velit@pharetraQuisqueac.co.uk','(370) 325-2134','Lecturer','Doctorate','Mattis Ornare Lectus Corporation'),(761286,'Inez L. Williamson','eget.laoreet@nec.net','(209) 776-6948','Lecturer','Doctorate','Urna Foundation'),(761287,'Jane Greene','neque.sed@nislarcuiaculis.ca','(204) 756-3437','Professor','Master','Cras Vulputate Consulting'),(761288,'Imelda K. Gamble','placerat.augue.Sed@Sednuncest.net','(943) 145-7500','Professor','Master','Lobortis Corporation'),(761289,'Raya Francis','massa.lobortis@dolorvitae.net','(864) 543-7154','Professor','Master','Malesuada Institute'),(761290,'Cyrus M. Forbes','In.tincidunt.congue@elementumdui.com','(914) 499-5407','Professor','Master','Maecenas Libero LLC'),(761291,'Zorita Harding','Aenean.sed@Duisrisus.ca','(445) 793-9139','Professor','Master','Natoque Penatibus Et Limited'),(761292,'Cody L. Benson','vitae.dolor.Donec@faucibuslectus.com','(573) 609-8739','Professor','Master','Tortor Dictum Corporation'),(761293,'Neville C. Copeland','Quisque.imperdiet@Proin.org','(202) 457-9953','Professor','Master','Ut Consulting'),(761294,'Charde I. Mcgowan','amet.risus.Donec@Fusce.edu','(415) 939-4637','Professor','Master','Quam Curabitur Foundation'),(761295,'Cedric Hunter','ante.Vivamus@Aeneaneuismod.co.uk','(310) 782-6264','Professor','Master','Accumsan Company'),(761296,'Henry R. Sharp','lorem.luctus@SuspendissesagittisNullam.co.uk','(174) 962-4689','Professor','Master','Mauris Sapien Cursus Associates'),(761297,'Penelope J. Hunter','Aenean@ante.com','(213) 452-9506','Assistant Professor','Doctorate','Nunc LLP'),(761298,'Lane K. English','sed@nibh.ca','(880) 593-9657','Assistant Professor','Doctorate','Molestie Arcu Corporation'),(761299,'Rinah O. Hicks','dignissim.Maecenas@tellusjusto.ca','(899) 747-8677','Assistant Professor','Doctorate','Morbi PC'),(761300,'Diana Pickett','senectus.et.netus@ametconsectetueradipiscing.org','(446) 127-8575','Assistant Professor','Doctorate','Vel Faucibus Id Incorporated'),(761301,'Sara Serrano','luctus.et@dolorvitaedolor.org','(165) 963-7982','Assistant Professor','Doctorate','Ac Inc.'),(761302,'Garrison X. Carney','mauris.ipsum.porta@infaucibusorci.edu','(350) 746-4734','Assistant Professor','Doctorate','Faucibus Lectus Foundation'),(761303,'Acton Blanchard','scelerisque.mollis@Namnulla.edu','(774) 885-2326','Assistant Professor','Doctorate','Erat Nonummy Incorporated'),(761304,'Imani Walter','Cum@orcilacus.com','(318) 216-5693','Assistant Professor','Doctorate','Mollis Vitae Ltd'),(761305,'Ashely Downs','euismod.in@nequeMorbi.co.uk','(192) 198-4358','Assistant Professor','Doctorate','Leo In Corp.'),(761306,'Chelsea Mcguire','magna@Donec.ca','(738) 896-0380','Assistant Professor','Doctorate','Lacus Company'),(761307,'Victor Watson','accumsan.interdum.libero@gravida.co.uk','(476) 631-9781','Lecturer','Master','Mauris Ipsum Porta Corporation'),(761308,'Keaton Dillard','sit.amet.luctus@variusNamporttitor.org','(403) 757-0429','Lecturer','Master','Mauris Magna Duis Institute'),(761309,'Kyla Horn','aliquet.molestie@lacus.edu','(354) 886-2900','Lecturer','Master','A Inc.'),(761310,'Wylie T. Riggs','dictum.placerat@arcu.edu','(947) 917-7782','Lecturer','Master','Orci Inc.'),(761311,'Jasper T. Rivas','a.aliquet@justosit.net','(743) 299-3260','Lecturer','Master','Placerat Cras Dictum Incorporated'),(761312,'Amena Jacobs','purus.ac.tellus@nostraper.net','(437) 516-7449','Lecturer','Master','Non Arcu Vivamus Ltd'),(761313,'Gisela Little','nascetur@nuncinterdumfeugiat.com','(256) 944-9066','Lecturer','Master','Magnis Dis Foundation'),(761314,'Arsenio Bernard','luctus.Curabitur.egestas@urnaVivamusmolestie.org','(950) 733-3920','Lecturer','Master','Nunc Institute'),(761315,'Allegra Lloyd','imperdiet.dictum@suscipit.com','(636) 385-1861','Lecturer','Master','Vel Venenatis Vel Ltd'),(761316,'Sybil W. Dalton','consectetuer@consequatauctor.org','(305) 508-4063','Lecturer','Master','Quisque Libero Lacus Inc.'),(761317,'Bernard Bender','sodales@In.co.uk','(580) 719-8007','Professor','Doctorate','Urna Nunc Quis Inc.'),(761318,'Quinn Rivers','nunc.est.mollis@atfringillapurus.edu','(893) 108-3838','Professor','Doctorate','Aliquet Nec Imperdiet Ltd'),(761319,'Holly N. Riley','netus.et.malesuada@massa.ca','(294) 227-7229','Professor','Doctorate','Integer Aliquam Adipiscing Foundation'),(761320,'Kirby T. Wynn','purus@lobortis.org','(340) 501-8612','Professor','Doctorate','Facilisis Ltd'),(761321,'Acton Conrad','Vivamus.nibh@utodiovel.net','(250) 151-2858','Professor','Doctorate','Proin Sed Institute'),(761322,'Marvin Waller','quam.a.felis@loremauctor.edu','(568) 984-1263','Professor','Doctorate','Nunc Id PC'),(761323,'Farrah C. Byers','eu.metus@tellusNunclectus.net','(368) 665-6381','Professor','Doctorate','Nunc Consulting'),(761324,'Charissa A. James','vel.faucibus.id@eutempor.com','(593) 907-8514','Professor','Doctorate','Mauris Erat Industries'),(761325,'Clementine F. Mcneil','primis.in@malesuadaInteger.com','(608) 965-2689','Professor','Doctorate','Ut Cursus LLP'),(761326,'Quail Dalton','velit.Pellentesque.ultricies@Integer.co.uk','(372) 472-2012','Professor','Doctorate','Mauris Sit Consulting');
/*!40000 ALTER TABLE `faculty` ENABLE KEYS */;
UNLOCK TABLES;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8mb4 */ ;
/*!50003 SET character_set_results = utf8mb4 */ ;
/*!50003 SET collation_connection  = utf8mb4_0900_ai_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
/*!50003 CREATE*/ /*!50017 DEFINER=`root`@`localhost`*/ /*!50003 TRIGGER `faculty_BEFORE_INSERT` BEFORE INSERT ON `faculty` FOR EACH ROW BEGIN
	insert into person
    values(new.idFaculty, new.name, new.email, new.phone_number);
END */;;
DELIMITER ;
/*!50003 SET sql_mode              = @saved_sql_mode */ ;
/*!50003 SET character_set_client  = @saved_cs_client */ ;
/*!50003 SET character_set_results = @saved_cs_results */ ;
/*!50003 SET collation_connection  = @saved_col_connection */ ;

--
-- Table structure for table `locations`
--

DROP TABLE IF EXISTS `locations`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `locations` (
  `idLocations` int NOT NULL,
  `location_name` varchar(250) DEFAULT NULL,
  `location_address` varchar(250) DEFAULT NULL,
  `drop_off_point` varchar(250) DEFAULT NULL,
  PRIMARY KEY (`idLocations`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `locations`
--

LOCK TABLES `locations` WRITE;
/*!40000 ALTER TABLE `locations` DISABLE KEYS */;
INSERT INTO `locations` VALUES (987654,'malesuada malesuada.','789 Elit, Street','Cum sociis natoque penatibus et magnis dis parturient'),(987655,'lobortis quis,','152-9870 Tempor, St.','feugiat nec, diam. Duis'),(987656,'dictum cursus.','P.O. Box 255, 176 Justo Av.','ullamcorper. Duis at lacus. Quisque purus sapien, gravida non, sollicitudin'),(987657,'libero mauris,','P.O. Box 725, 3078 Ac St.','vitae, erat. Vivamus'),(987658,'Nunc ac','P.O. Box 560, 5079 Dui. Avenue','diam. Sed diam lorem, auctor quis, tristique'),(987659,'faucibus lectus,','6844 Phasellus Street','dui quis accumsan convallis, ante lectus convallis est, vitae'),(987660,'ante. Vivamus','277-2539 Penatibus Avenue','metus. In nec orci. Donec'),(987661,'quis urna.','8974 Magna Av.','nec orci. Donec nibh. Quisque nonummy'),(987662,'risus. Nulla','Ap #230-1038 Semper, Street','imperdiet ullamcorper. Duis at lacus. Quisque purus sapien,'),(987663,'Suspendisse sagittis.','Ap #657-7104 Curabitur St.','Ut tincidunt vehicula risus. Nulla eget metus'),(987664,'quis diam.','Ap #533-207 A Road','augue. Sed molestie.'),(987665,'eu dui.','P.O. Box 410, 4208 Amet Avenue','et magnis dis parturient montes, nascetur ridiculus mus.'),(987666,'feugiat non,','Ap #872-1143 Feugiat Rd.','sagittis augue, eu tempor erat'),(987667,'Aliquam fringilla','Ap #307-9301 Felis Avenue','Morbi quis urna. Nunc quis'),(987668,'accumsan neque','Ap #574-3275 Nonummy Avenue','parturient montes, nascetur ridiculus mus. Aenean'),(987669,'cubilia Curae;','794-2291 Donec Rd.','eu dui. Cum sociis natoque penatibus et magnis dis'),(987670,'at fringilla','Ap #458-6913 Vivamus Avenue','enim consequat purus. Maecenas libero est,'),(987671,'sollicitudin a,','871-237 Cubilia Ave','diam vel arcu.'),(987672,'Sed nulla','1441 Sit Av.','cubilia Curae; Donec tincidunt.'),(987673,'nascetur ridiculus','8529 Velit St.','ultrices posuere cubilia Curae; Donec tincidunt. Donec vitae'),(987674,'dui nec','4602 Elit. Street','arcu. Vestibulum ante ipsum primis in'),(987675,'dolor dapibus','P.O. Box 104, 7581 Nisl Street','id, ante. Nunc'),(987676,'sollicitudin a,','7586 Ac Rd.','egestas lacinia. Sed congue,'),(987677,'vel lectus.','399-3601 Mattis. Av.','orci lobortis augue scelerisque mollis. Phasellus libero'),(987678,'Vestibulum ut','875-5638 Non Ave','sagittis augue, eu tempor erat neque'),(987679,'libero. Morbi','1998 Semper Avenue','elementum sem, vitae aliquam eros turpis'),(987680,'vitae purus','Ap #160-3147 Feugiat. Road','nunc id enim. Curabitur massa. Vestibulum'),(987681,'metus. Vivamus','9857 Condimentum. Ave','Aliquam rutrum lorem ac risus. Morbi metus. Vivamus euismod'),(987682,'id sapien.','P.O. Box 169, 438 Imperdiet St.','varius orci, in consequat enim diam vel arcu. Curabitur'),(987683,'penatibus et','6322 Fusce St.','orci luctus et ultrices posuere cubilia Curae; Phasellus');
/*!40000 ALTER TABLE `locations` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `orders`
--

DROP TABLE IF EXISTS `orders`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `orders` (
  `idOrders` int NOT NULL,
  `client_id` int NOT NULL,
  `restaurant_id` int NOT NULL,
  `driver_id` int NOT NULL,
  `location_id` int NOT NULL,
  `price` decimal(2,0) DEFAULT NULL,
  `delivery_charge` decimal(2,0) DEFAULT NULL,
  `delivery_time` time DEFAULT NULL,
  PRIMARY KEY (`idOrders`),
  KEY `fk_Orders_Person1_idx` (`client_id`),
  KEY `fk_Orders_Restaurants1_idx` (`restaurant_id`),
  KEY `fk_Orders_Drivers1_idx` (`driver_id`),
  KEY `fk_Orders_Locations1_idx` (`location_id`),
  CONSTRAINT `fk_Orders_Drivers1` FOREIGN KEY (`driver_id`) REFERENCES `drivers` (`idDrivers`),
  CONSTRAINT `fk_Orders_Locations1` FOREIGN KEY (`location_id`) REFERENCES `locations` (`idLocations`),
  CONSTRAINT `fk_Orders_Person1` FOREIGN KEY (`client_id`) REFERENCES `person` (`idPerson`),
  CONSTRAINT `fk_Orders_Restaurants1` FOREIGN KEY (`restaurant_id`) REFERENCES `restaurants` (`idRestaurants`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `orders`
--

LOCK TABLES `orders` WRITE;
/*!40000 ALTER TABLE `orders` DISABLE KEYS */;
INSERT INTO `orders` VALUES (45678,771300,123486,781227,987666,20,5,'11:00:23'),(456852,761227,123509,781267,987676,26,5,'17:00:15'),(456853,761229,123491,781247,987666,20,5,'11:13:00'),(456854,761231,123471,781227,987673,68,5,'06:08:10'),(456855,761233,123488,781237,987671,62,5,'10:02:04'),(456856,761235,123536,781237,987679,89,5,'11:53:04'),(456857,761237,123510,781267,987658,74,5,'23:00:15'),(456858,761239,123549,781257,987667,1,5,'05:15:00'),(456859,761241,123538,781267,987659,65,5,'10:56:02'),(456860,761243,123516,781267,987655,70,5,'19:52:40'),(456861,761245,123518,781297,987682,10,5,'01:25:40'),(456892,761307,123496,781267,987667,33,5,'00:16:00'),(456893,761309,123544,781227,987655,74,5,'12:00:14'),(456894,761311,123531,781237,987675,39,5,'07:09:00'),(456895,761313,123497,781257,987680,26,5,'01:03:05'),(456896,761315,123536,781237,987672,92,5,'09:10:00'),(456897,761317,123530,781237,987680,89,5,'16:18:00'),(456898,761319,123472,781267,987656,53,5,'06:01:16'),(456899,761321,123496,781257,987666,52,5,'21:42:03'),(456900,761323,123456,781277,987668,1,5,'11:41:12'),(456901,761325,123513,781287,987683,14,5,'15:41:50'),(456902,771227,123463,781237,987666,74,5,'15:35:00'),(456903,771229,123507,781237,987672,96,5,'16:33:00'),(456904,771231,123472,781257,987656,92,5,'05:23:00'),(456905,771233,123483,781297,987655,22,5,'20:07:09'),(456906,771235,123507,781247,987680,69,5,'23:07:56'),(456907,771237,123487,781297,987665,71,5,'19:06:58'),(456908,771239,123498,781247,987668,69,5,'12:05:00'),(456909,771241,123470,781297,987674,87,5,'15:51:11'),(456910,771243,123460,781267,987657,26,5,'04:00:46'),(456911,771245,123539,781267,987667,43,5,'03:00:01'),(456922,771267,123487,781257,987664,80,5,'19:42:12'),(456923,771269,123492,781277,987664,23,5,'18:00:54'),(456924,771271,123532,781297,987678,69,5,'04:02:10'),(456925,771273,123492,781237,987671,68,5,'20:06:00'),(456926,771275,123519,781237,987673,19,5,'07:27:14'),(456927,771277,123473,781227,987677,5,5,'07:27:54'),(456928,771279,123540,781287,987679,6,5,'10:04:02'),(456929,771281,123516,781257,987658,84,5,'16:26:10'),(456930,771283,123538,781277,987673,58,5,'00:55:05'),(456931,771285,123475,781277,987672,54,5,'00:02:00'),(456932,781227,123546,781257,987657,33,5,'11:00:31'),(456933,781229,123511,781257,987671,97,5,'17:26:27'),(456934,781231,123483,781237,987666,12,5,'12:00:22'),(456935,781233,123512,781287,987658,91,5,'14:18:00'),(456936,781235,123496,781287,987681,44,5,'09:26:00'),(456937,781237,123517,781267,987671,58,5,'08:41:00'),(456938,781239,123486,781227,987666,20,5,'09:00:20'),(456939,781241,123542,781287,987667,22,5,'00:23:00'),(456940,781243,123525,781287,987654,98,5,'10:00:00'),(456941,781245,123509,781277,987654,58,5,'12:00:18'),(456942,781247,123459,781247,987663,8,5,'08:32:00'),(456943,781249,123522,781287,987666,72,5,'18:25:00'),(456944,781251,123530,781277,987673,24,5,'16:15:00'),(456945,781253,123534,781297,987678,1,5,'06:00:50'),(456946,781255,123526,781247,987666,21,5,'10:11:00'),(456947,781257,123489,781237,987659,47,5,'01:00:12'),(456948,781259,123526,781227,987683,38,5,'03:00:10'),(456949,781261,123499,781267,987680,6,5,'02:00:52'),(456950,781263,123552,781247,987676,67,5,'09:00:11'),(456951,781265,123476,781227,987681,88,5,'13:00:52'),(456952,781267,123521,781227,987673,54,5,'15:00:33'),(456953,781269,123499,781227,987655,11,5,'19:24:00'),(456954,781271,123522,781277,987681,78,5,'07:00:49'),(456955,781273,123494,781247,987657,64,5,'23:00:00'),(456956,781275,123475,781247,987659,74,5,'15:42:18'),(456957,781277,123522,781237,987665,74,5,'15:00:00'),(456958,781279,123501,781267,987680,95,5,'14:50:00'),(456959,781281,123508,781287,987679,33,5,'16:01:00'),(456960,781283,123513,781267,987666,7,5,'08:40:00'),(456961,781285,123469,781237,987682,51,5,'14:22:00'),(456962,781286,123537,781227,987655,86,5,'21:34:00'),(456963,781288,123535,781287,987662,15,5,'20:23:00'),(456964,781290,123498,781247,987683,65,5,'09:00:12'),(456965,781292,123456,781297,987668,67,5,'11:12:00'),(456966,781294,123482,781287,987666,56,5,'23:15:00'),(456967,781296,123526,781297,987673,71,5,'12:56:00'),(456968,781298,123522,781267,987665,18,5,'10:02:00'),(456969,781300,123460,781297,987677,36,5,'21:56:00'),(456970,781303,123538,781287,987661,28,5,'07:00:00'),(456971,781304,123523,781277,987675,88,5,'15:00:00'),(456972,761240,123546,781257,987657,33,5,'20:00:15'),(456973,761232,123511,781257,987671,97,5,'00:25:50'),(456974,761306,123483,781237,987666,12,5,'11:11:00'),(456975,761266,123512,781287,987658,91,5,'00:00:00'),(456976,761255,123496,781287,987681,44,5,'15:15:21'),(456977,771250,123517,781267,987671,58,5,'12:00:45'),(456979,771326,123542,781287,987667,22,5,'13:45:01'),(456980,771253,123525,781287,987654,98,5,'19:12:59'),(456981,771288,123509,781277,987654,58,5,'13:51:53'),(456982,781266,123486,781227,987666,20,5,'11:00:23'),(456983,781326,123542,781287,987667,22,5,'13:45:01'),(456984,781317,123525,781287,987654,98,5,'19:12:59'),(456985,781252,123509,781277,987654,58,5,'13:51:53'),(456986,781266,123486,781227,987666,20,5,'19:00:44'),(456987,781326,123542,781287,987667,22,5,'11:32:20'),(456988,781317,123525,781287,987654,98,5,'06:45:59'),(456989,781252,123509,781277,987654,58,5,'15:56:03'),(456990,781266,123486,781257,987666,20,5,'19:10:42'),(456991,781326,123542,781297,987667,22,5,'10:42:26');
/*!40000 ALTER TABLE `orders` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Temporary view structure for view `orders_made`
--

DROP TABLE IF EXISTS `orders_made`;
/*!50001 DROP VIEW IF EXISTS `orders_made`*/;
SET @saved_cs_client     = @@character_set_client;
/*!50503 SET character_set_client = utf8mb4 */;
/*!50001 CREATE VIEW `orders_made` AS SELECT 
 1 AS `Client_Name`,
 1 AS `Restaurant`,
 1 AS `Total_Price`*/;
SET character_set_client = @saved_cs_client;

--
-- Table structure for table `person`
--

DROP TABLE IF EXISTS `person`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `person` (
  `idPerson` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `email` varchar(250) DEFAULT NULL,
  `phone_number` varchar(15) DEFAULT NULL,
  PRIMARY KEY (`idPerson`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `person`
--

LOCK TABLES `person` WRITE;
/*!40000 ALTER TABLE `person` DISABLE KEYS */;
INSERT INTO `person` VALUES (761227,'Adara Q. Lynn','vitae@indolorFusce.net','(182) 707-8866'),(761228,'Keely C. Hoffman','aliquet@risusaultricies.co.uk','(922) 839-1471'),(761229,'August R. Hale','nec.orci.Donec@maurisrhoncusid.ca','(596) 592-9053'),(761230,'Avye J. Gray','arcu.vel.quam@eget.edu','(624) 155-7927'),(761231,'Xena Bond','diam@egestasa.net','(316) 367-2107'),(761232,'Marah Chapman','Suspendisse@lacusMauris.com','(181) 246-6446'),(761233,'Lionel Collier','tempor.lorem.eget@lacusCrasinterdum.com','(944) 164-6847'),(761234,'Emily I. Romero','euismod.et.commodo@Vivamus.edu','(164) 615-2350'),(761235,'Macey Bailey','lectus@vitaerisus.ca','(817) 213-4364'),(761236,'Tyler F. Price','ipsum@nisiCumsociis.org','(549) 580-5025'),(761237,'Linus Burke','mi.felis@tinciduntpede.co.uk','(362) 405-0929'),(761238,'Alea Crane','sapien.Nunc.pulvinar@commodoipsum.net','(661) 402-6767'),(761239,'Freya O. Newman','tellus.Phasellus@elitpretiumet.ca','(336) 249-6862'),(761240,'Jescie V. Mann','at.augue.id@Aliquamultrices.com','(882) 133-6841'),(761241,'Petra V. Pierce','Nunc.mauris.Morbi@arcuVivamus.edu','(637) 527-4715'),(761242,'Sybill Mclaughlin','ultrices@Integersemelit.edu','(379) 996-6371'),(761243,'Olympia Velez','viverra.Maecenas.iaculis@commodoat.net','(495) 728-3343'),(761244,'Kennedy P. Hatfield','lobortis@ligula.org','(594) 455-8400'),(761245,'Dante D. Watkins','nisl@sitametdiam.co.uk','(965) 473-8041'),(761246,'Brenden Washington','mus.Aenean@nasceturridiculus.com','(334) 210-6349'),(761247,'Miranda Richardson','eget.odio@semper.edu','(937) 647-7323'),(761248,'Byron F. Lawson','mus@sapienNuncpulvinar.edu','(867) 914-7167'),(761249,'Avram Newman','lectus@libero.edu','(802) 457-2383'),(761250,'Jacob X. Jackson','sem@dapibus.co.uk','(284) 127-7575'),(761251,'Brenna Carlson','volutpat.Nulla.dignissim@tristique.edu','(265) 558-6879'),(761252,'Nasim L. Dalton','vel.pede@ut.edu','(984) 248-8297'),(761253,'Cairo Harding','Ut@temporlorem.edu','(230) 962-8128'),(761254,'Garth N. Burton','iaculis.odio@egestas.ca','(287) 550-3925'),(761255,'Nyssa B. Gregory','vel.nisl.Quisque@quistristiqueac.com','(766) 638-6826'),(761256,'Jamalia B. Conway','neque.pellentesque@Craseget.co.uk','(715) 161-9706'),(761257,'Danielle R. Cobb','Quisque.nonummy.ipsum@DuisgravidaPraesent.org','(339) 997-6624'),(761258,'Tamekah Barber','Donec.fringilla.Donec@semper.net','(616) 959-8865'),(761259,'Burton Q. Miller','orci@turpisAliquam.org','(611) 771-5218'),(761260,'Lewis S. Wilder','metus@Aliquamrutrum.net','(561) 867-2690'),(761261,'Plato L. Lloyd','sit.amet@urna.co.uk','(451) 278-2385'),(761262,'Quynn Beasley','Cras.interdum@nec.co.uk','(440) 203-5748'),(761263,'Anika Hodges','molestie.in@lorem.net','(908) 940-6294'),(761264,'Bernard O. Gray','Duis.ac@antedictum.com','(168) 793-6603'),(761265,'Melinda U. Weeks','lorem@Aliquamnecenim.co.uk','(120) 848-2171'),(761266,'Daniel Cook','dui.Suspendisse.ac@Donecluctusaliquet.org','(558) 169-8561'),(761267,'Jenette C. Powers','porttitor@commodoipsumSuspendisse.ca','(738) 101-1532'),(761268,'Drew Whitfield','dolor.elit@sitamet.co.uk','(522) 773-5970'),(761269,'Imogene Q. Wright','parturient.montes@est.org','(441) 475-6526'),(761270,'Ainsley Y. Curry','sit.amet.consectetuer@laciniaorci.co.uk','(733) 631-7044'),(761271,'Cassady X. Randolph','ipsum@necluctusfelis.edu','(235) 721-5952'),(761272,'Rebekah M. Burris','lobortis@etarcu.ca','(456) 636-2455'),(761273,'Brianna Z. Guzman','ac.risus.Morbi@tinciduntneque.co.uk','(398) 393-1295'),(761274,'Kylynn Calhoun','libero.Proin@arcuVestibulum.net','(236) 602-6142'),(761275,'Adrian A. Aguirre','ut.lacus@Duis.edu','(600) 280-1700'),(761276,'Nomlanga S. Villarreal','non@Duis.edu','(578) 907-5653'),(761277,'Jaime Weeks','elit.sed.consequat@In.net','(172) 590-5209'),(761278,'Chantale I. Holland','at.egestas@enim.co.uk','(181) 878-8821'),(761279,'Rosalyn Calderon','Cras.pellentesque@adipiscinglobortisrisus.edu','(562) 593-3128'),(761280,'Yoshio Pope','ridiculus.mus@blandit.edu','(967) 576-7558'),(761281,'Briar K. Rush','orci.lacus@conguea.ca','(792) 634-5587'),(761282,'Selma M. Bernard','aliquet.odio.Etiam@lacusEtiambibendum.ca','(913) 349-5245'),(761283,'Nadine Pierce','id.ante@rhoncus.com','(674) 630-7106'),(761284,'Tatiana G. Morton','Aliquam@semperNam.net','(699) 455-4557'),(761285,'Daphne K. Kelley','velit@pharetraQuisqueac.co.uk','(370) 325-2134'),(761286,'Inez L. Williamson','eget.laoreet@nec.net','(209) 776-6948'),(761287,'Jane Greene','neque.sed@nislarcuiaculis.ca','(204) 756-3437'),(761288,'Imelda K. Gamble','placerat.augue.Sed@Sednuncest.net','(943) 145-7500'),(761289,'Raya Francis','massa.lobortis@dolorvitae.net','(864) 543-7154'),(761290,'Cyrus M. Forbes','In.tincidunt.congue@elementumdui.com','(914) 499-5407'),(761291,'Zorita Harding','Aenean.sed@Duisrisus.ca','(445) 793-9139'),(761292,'Cody L. Benson','vitae.dolor.Donec@faucibuslectus.com','(573) 609-8739'),(761293,'Neville C. Copeland','Quisque.imperdiet@Proin.org','(202) 457-9953'),(761294,'Charde I. Mcgowan','amet.risus.Donec@Fusce.edu','(415) 939-4637'),(761295,'Cedric Hunter','ante.Vivamus@Aeneaneuismod.co.uk','(310) 782-6264'),(761296,'Henry R. Sharp','lorem.luctus@SuspendissesagittisNullam.co.uk','(174) 962-4689'),(761297,'Penelope J. Hunter','Aenean@ante.com','(213) 452-9506'),(761298,'Lane K. English','sed@nibh.ca','(880) 593-9657'),(761299,'Rinah O. Hicks','dignissim.Maecenas@tellusjusto.ca','(899) 747-8677'),(761300,'Diana Pickett','senectus.et.netus@ametconsectetueradipiscing.org','(446) 127-8575'),(761301,'Sara Serrano','luctus.et@dolorvitaedolor.org','(165) 963-7982'),(761302,'Garrison X. Carney','mauris.ipsum.porta@infaucibusorci.edu','(350) 746-4734'),(761303,'Acton Blanchard','scelerisque.mollis@Namnulla.edu','(774) 885-2326'),(761304,'Imani Walter','Cum@orcilacus.com','(318) 216-5693'),(761305,'Ashely Downs','euismod.in@nequeMorbi.co.uk','(192) 198-4358'),(761306,'Chelsea Mcguire','magna@Donec.ca','(738) 896-0380'),(761307,'Victor Watson','accumsan.interdum.libero@gravida.co.uk','(476) 631-9781'),(761308,'Keaton Dillard','sit.amet.luctus@variusNamporttitor.org','(403) 757-0429'),(761309,'Kyla Horn','aliquet.molestie@lacus.edu','(354) 886-2900'),(761310,'Wylie T. Riggs','dictum.placerat@arcu.edu','(947) 917-7782'),(761311,'Jasper T. Rivas','a.aliquet@justosit.net','(743) 299-3260'),(761312,'Amena Jacobs','purus.ac.tellus@nostraper.net','(437) 516-7449'),(761313,'Gisela Little','nascetur@nuncinterdumfeugiat.com','(256) 944-9066'),(761314,'Arsenio Bernard','luctus.Curabitur.egestas@urnaVivamusmolestie.org','(950) 733-3920'),(761315,'Allegra Lloyd','imperdiet.dictum@suscipit.com','(636) 385-1861'),(761316,'Sybil W. Dalton','consectetuer@consequatauctor.org','(305) 508-4063'),(761317,'Bernard Bender','sodales@In.co.uk','(580) 719-8007'),(761318,'Quinn Rivers','nunc.est.mollis@atfringillapurus.edu','(893) 108-3838'),(761319,'Holly N. Riley','netus.et.malesuada@massa.ca','(294) 227-7229'),(761320,'Kirby T. Wynn','purus@lobortis.org','(340) 501-8612'),(761321,'Acton Conrad','Vivamus.nibh@utodiovel.net','(250) 151-2858'),(761322,'Marvin Waller','quam.a.felis@loremauctor.edu','(568) 984-1263'),(761323,'Farrah C. Byers','eu.metus@tellusNunclectus.net','(368) 665-6381'),(761324,'Charissa A. James','vel.faucibus.id@eutempor.com','(593) 907-8514'),(761325,'Clementine F. Mcneil','primis.in@malesuadaInteger.com','(608) 965-2689'),(761326,'Quail Dalton','velit.Pellentesque.ultricies@Integer.co.uk','(372) 472-2012'),(771227,'Olga Mack','consectetuer.adipiscing.elit@et.co.uk','(287) 765-8795'),(771228,'Mufutau N. Mcpherson','molestie.dapibus@vestibulum.org','(106) 523-4816'),(771229,'Rigel T. Burt','fringilla@amet.org','(785) 414-6372'),(771230,'Plato Weeks','varius@neque.net','(539) 561-2308'),(771231,'Vincent Donaldson','Phasellus.at.augue@sedleo.net','(124) 962-7732'),(771232,'Kane M. Flynn','tincidunt@aliquamerosturpis.org','(487) 288-3448'),(771233,'Damian F. Burnett','Etiam@Utnecurna.net','(951) 477-3528'),(771234,'Hilel Newman','lobortis@orcilacus.edu','(978) 474-7906'),(771235,'Unity Y. Powell','Vestibulum.ante.ipsum@Duis.edu','(461) 396-2843'),(771236,'Marvin X. Waters','sed.sapien.Nunc@Ut.com','(571) 376-4507'),(771237,'Ariana R. Welch','sem.magna@ultrices.edu','(567) 179-9526'),(771238,'Aileen Nash','mauris.sagittis.placerat@adipiscingligula.com','(862) 526-9849'),(771239,'Selma Mccarthy','Nunc@consequatlectussit.com','(483) 553-5110'),(771240,'Imogene V. Curtis','risus@elit.ca','(279) 710-0991'),(771241,'Lance Conway','mollis@Phasellusataugue.edu','(275) 979-2151'),(771242,'McKenzie F. Chan','vulputate.ullamcorper.magna@semelit.co.uk','(569) 281-2637'),(771243,'Jaquelyn C. Wade','nec.malesuada.ut@leoCras.co.uk','(209) 831-4640'),(771244,'Deirdre Cortez','Curabitur.consequat.lectus@afelis.org','(366) 592-1781'),(771245,'Francis Joyner','Suspendisse@gravida.ca','(390) 228-4500'),(771246,'Herrod Roth','tempus.non@Cras.ca','(738) 839-4525'),(771247,'Armand Chapman','pede@blanditenim.net','(125) 668-9997'),(771248,'Joy D. Lewis','aliquet.diam.Sed@Mauris.com','(850) 258-3638'),(771249,'Bree Thomas','tellus@pedeultrices.com','(895) 565-1572'),(771250,'Leilani Larson','lacus.pede.sagittis@Curabiturconsequatlectus.co.uk','(162) 902-5915'),(771251,'Abbot N. Fisher','quis.urna@velitdui.co.uk','(360) 789-4750'),(771252,'Karen Sandoval','sed@musProin.net','(748) 351-2619'),(771253,'Amy Donaldson','dolor@laoreet.edu','(200) 758-3288'),(771254,'Haviva Hurst','consectetuer@acnullaIn.co.uk','(163) 994-5759'),(771255,'Otto Watson','faucibus.ut.nulla@quispedePraesent.org','(493) 318-3960'),(771256,'Adam N. Gilliam','diam.Pellentesque@Nunc.com','(117) 772-5815'),(771257,'Barry Key','bibendum@orciadipiscingnon.edu','(826) 481-0594'),(771258,'Daquan E. Martin','ante.iaculis@aliquam.ca','(969) 707-6209'),(771259,'Buffy Price','rutrum.lorem@neque.ca','(909) 909-9444'),(771260,'Solomon Cooke','in@a.ca','(210) 594-6053'),(771261,'Drew S. Mendoza','Aliquam.auctor@mauris.edu','(773) 186-1445'),(771262,'Camden Goff','tellus@ut.edu','(220) 369-2529'),(771263,'Judah G. Mathews','nec.enim.Nunc@justo.ca','(775) 174-5149'),(771264,'Dominic Z. Mason','mi@augueac.net','(565) 524-1006'),(771265,'Amanda N. Matthews','leo.elementum.sem@purusgravida.net','(189) 815-4546'),(771266,'Hilda S. Dillard','ipsum.nunc.id@egestasSedpharetra.ca','(822) 390-8195'),(771267,'Conan Kemp','semper.rutrum@risusatfringilla.net','(971) 417-5225'),(771268,'Karyn Vance','lorem.fringilla@duinec.ca','(443) 466-2818'),(771269,'Octavius Barlow','dapibus.id.blandit@utquam.ca','(729) 268-1243'),(771270,'Thane G. Burns','semper.et.lacinia@magnisdis.org','(150) 906-4430'),(771271,'Hop House','enim@gravidaAliquam.net','(896) 943-5889'),(771272,'Ignacia Marquez','amet.massa.Quisque@Inlorem.org','(108) 358-4189'),(771273,'Rafael L. Kirby','ipsum@nibhQuisquenonummy.com','(659) 117-9761'),(771274,'Shea N. Flowers','ultricies.ligula.Nullam@DuisgravidaPraesent.edu','(379) 970-4695'),(771275,'Gannon Q. Parsons','vitae.dolor.Donec@Aeneangravida.ca','(380) 237-0758'),(771276,'Florence K. Trevino','per@vitaealiquam.com','(631) 840-3133'),(771277,'Mohammad O. David','lectus.justo@pharetrasedhendrerit.com','(947) 893-8296'),(771278,'Thomas E. Stuart','ac.sem.ut@sed.org','(746) 792-7096'),(771279,'Roanna Lang','sociosqu.ad@maurisMorbi.com','(968) 263-3471'),(771280,'Byron D. Wilder','congue.turpis.In@nulla.edu','(967) 935-0384'),(771281,'Camilla Mooney','aliquam@tellusjusto.com','(678) 483-3879'),(771282,'Carter Page','non.sollicitudin@diamvelarcu.edu','(386) 800-2923'),(771283,'Katelyn K. Booker','tellus.eu@scelerisqueloremipsum.com','(645) 864-3147'),(771284,'Uma Riddle','elementum@eu.ca','(965) 501-8412'),(771285,'Bo Burton','odio@arcuVivamus.edu','(225) 218-2675'),(771286,'Ali Durham','auctor.non.feugiat@liberodui.org','(589) 962-1946'),(771287,'Brynne W. Blake','nisi.Aenean.eget@ProinvelitSed.edu','(334) 989-3481'),(771288,'Knox Obrien','Quisque.varius.Nam@faucibus.com','(660) 342-0859'),(771289,'Orson R. Garner','diam.dictum.sapien@leoMorbineque.org','(583) 324-2270'),(771290,'Abdul R. Tyler','lacus.Nulla.tincidunt@apurusDuis.co.uk','(353) 988-7550'),(771291,'Timothy Bolton','dui@ultricesa.net','(300) 899-0129'),(771292,'Gage T. Ferrell','semper.et@tristiquenequevenenatis.org','(479) 941-7226'),(771293,'Casey R. Ellis','pede.nonummy.ut@mienimcondimentum.co.uk','(164) 129-1044'),(771294,'Orli Reese','egestas@facilisiSedneque.com','(366) 206-1269'),(771295,'Idona M. Walls','Aliquam@velitCras.com','(219) 509-0662'),(771296,'Lars Trevino','orci.Donec.nibh@vulputateposuerevulputate.ca','(679) 814-9633'),(771297,'Rigel D. Lancaster','sagittis@sagittisaugue.ca','(684) 428-4346'),(771298,'Riley R. Hyde','Nunc@ligulaAliquamerat.co.uk','(569) 790-4250'),(771299,'Cairo Le','nunc.Quisque.ornare@sociisnatoquepenatibus.net','(796) 397-5104'),(771300,'Jared J. Hammond','egestas.lacinia@gravidasit.ca','(319) 469-1213'),(771301,'Jerry Holloway','Pellentesque.habitant@Quisque.net','(600) 749-1879'),(771302,'Shelley Fitzpatrick','sollicitudin.a@nislelementum.org','(584) 651-8310'),(771303,'Kim Moreno','malesuada@acfeugiat.com','(637) 362-1255'),(771304,'Bradley V. Hartman','Class@nec.net','(674) 742-2069'),(771305,'Yoshio O. Clayton','magna.nec.quam@orciUt.net','(369) 341-9961'),(771306,'Emily N. Figueroa','Donec@cubiliaCurae.co.uk','(174) 523-3583'),(771307,'Kirsten Key','urna.nec.luctus@ullamcorperviverra.com','(848) 251-6329'),(771308,'Iris O. Vazquez','metus@inmagna.edu','(108) 521-0282'),(771309,'Pearl Elliott','nec@orci.com','(230) 988-9134'),(771310,'Gannon Anderson','Cum@Nullamsuscipit.edu','(779) 265-8069'),(771311,'Dennis R. Workman','bibendum.ullamcorper@ametorciUt.org','(734) 498-8519'),(771312,'Jerome Jacobson','nec.tempus@mattisvelitjusto.com','(633) 362-6949'),(771313,'Mariko X. Wilder','Etiam.gravida@ut.edu','(516) 138-3332'),(771314,'Tamara Rodriquez','Lorem.ipsum.dolor@Mauris.org','(478) 735-9788'),(771315,'Camden Mcleod','non.massa@magnaPraesent.ca','(809) 447-6422'),(771316,'Lois Middleton','faucibus@dictum.net','(137) 987-8469'),(771317,'Heidi B. Alexander','Aliquam@egestas.co.uk','(931) 249-4079'),(771318,'India A. Cameron','vestibulum.nec@Aenean.net','(151) 970-9032'),(771319,'Venus Cherry','leo.Morbi.neque@tempusscelerisque.net','(256) 334-9471'),(771320,'Jelani N. Rodriquez','Nunc@feugiatLorem.net','(113) 377-1038'),(771321,'Azalia S. Leonard','Fusce@Morbivehicula.com','(533) 848-6831'),(771322,'Norman Hopkins','Donec.sollicitudin@et.org','(502) 468-1713'),(771323,'Arden I. Conley','et.ipsum@euaccumsansed.edu','(192) 824-0387'),(771324,'Ira Rush','lacinia@malesuadaid.co.uk','(890) 794-4461'),(771325,'Murphy Swanson','leo@estconguea.edu','(739) 544-2407'),(771326,'Xenos Mcneil','fermentum.metus@Maurismolestiepharetra.ca','(333) 826-1777'),(780000,'Adona Mwense','mwense07@gmail.com','(704) 858-6584'),(781227,'Dominic Dodson','ut.lacus@Proineget.org','(614) 660-0550'),(781228,'Rana U. Phelps','mi.felis.adipiscing@amet.co.uk','(868) 549-4244'),(781229,'Candace Marks','nulla@Aeneangravida.com','(175) 501-3072'),(781230,'Brady H. Tran','malesuada@ac.ca','(644) 719-1158'),(781231,'Shannon Q. Parsons','eros.non.enim@elitpretiumet.org','(366) 881-0272'),(781232,'Neil Paul','eu.tellus@sollicitudin.com','(322) 139-8174'),(781233,'Ashton Powers','vitae@necante.ca','(679) 799-3962'),(781234,'Adrienne Mcleod','adipiscing.lobortis.risus@elit.net','(681) 487-8401'),(781235,'Hanae Farmer','eget@et.co.uk','(722) 234-4590'),(781236,'Clio Taylor','eget.odio.Aliquam@etrutrumeu.ca','(701) 826-3149'),(781237,'Nigel J. Spears','Sed@nisi.com','(490) 412-6114'),(781238,'Wynter R. Stevens','Duis.gravida@Vivamus.net','(734) 801-5545'),(781239,'Kevyn X. Burnett','id@nonummy.com','(394) 236-9537'),(781240,'Kylynn Elliott','ullamcorper.Duis@luctus.edu','(926) 612-9572'),(781241,'Jaquelyn Nash','dignissim@facilisisSuspendissecommodo.net','(496) 997-3524'),(781242,'Kyla W. Mcintosh','sem.egestas@tristiquealiquetPhasellus.org','(435) 962-9114'),(781243,'Rama D. Murray','sed.consequat@Aeneanegestas.ca','(791) 559-6258'),(781244,'Shelly V. Baker','parturient@temporaugue.org','(715) 753-0684'),(781245,'August Bates','aliquam.eros.turpis@diamnuncullamcorper.co.uk','(654) 152-8576'),(781246,'Fallon N. Greer','Phasellus.elit@Cumsociis.net','(315) 541-4254'),(781247,'Conan Z. Perez','massa.Quisque.porttitor@non.com','(270) 115-1873'),(781248,'Salvador X. Williamson','nec.cursus.a@litoratorquentper.net','(696) 703-2618'),(781249,'Michelle N. Greene','quam@nonquamPellentesque.edu','(565) 378-0791'),(781250,'Mechelle Galloway','ut@consectetueradipiscing.org','(863) 824-1553'),(781251,'Neil Mcdowell','in@nequeNullam.net','(785) 890-3411'),(781252,'Cherokee Q. Greer','amet.ante.Vivamus@indolor.edu','(837) 602-0353'),(781253,'Roanna U. Parrish','lectus.a.sollicitudin@Fuscedolor.com','(409) 123-5517'),(781254,'Dieter O. Christensen','erat@pharetraNam.edu','(112) 415-9200'),(781255,'Imelda D. Stevens','scelerisque.scelerisque.dui@dignissim.ca','(571) 571-0703'),(781256,'Jenette F. Matthews','cursus.luctus@Ut.net','(405) 942-3612'),(781257,'Jonah C. Mcdowell','cursus.et@ullamcorpervelitin.org','(843) 587-6599'),(781258,'Candice Barron','Etiam.imperdiet@nibhQuisquenonummy.edu','(765) 249-8309'),(781259,'Jolene Luna','mauris.sit@urna.com','(115) 943-7328'),(781260,'Ulric V. England','luctus.aliquet@Nullam.org','(281) 980-4444'),(781261,'Vance R. Watkins','aliquam.eu@massaVestibulum.org','(460) 635-2315'),(781262,'Hu D. Boyd','libero.nec@eget.org','(690) 223-1224'),(781263,'Erich J. Hodge','quam.Curabitur@MaurisnullaInteger.ca','(329) 302-4203'),(781264,'Haley A. Bolton','auctor.nunc.nulla@sitamet.co.uk','(644) 841-3494'),(781265,'Ulysses D. Rios','morbi.tristique@turpis.org','(615) 225-4887'),(781266,'Tatiana Potts','tristique.neque.venenatis@cursusIntegermollis.ca','(377) 262-5289'),(781267,'Claire Q. Burt','id.libero@fringillami.ca','(680) 389-7197'),(781268,'Yuli J. Tate','purus.Nullam.scelerisque@atnisiCum.net','(444) 759-6440'),(781269,'Aurora Y. Orr','Lorem@mollisdui.com','(585) 599-2485'),(781270,'Ahmed O. Cox','eget@utdolor.edu','(119) 704-6994'),(781271,'Rebecca I. Wright','enim.nec@malesuadafames.com','(875) 872-0252'),(781272,'Unity W. Parrish','vel.mauris.Integer@anteipsum.com','(458) 933-5544'),(781273,'Abdul Kaufman','Duis.dignissim@portaelit.net','(602) 493-2935'),(781274,'Macon Curtis','mauris.sapien.cursus@lobortisquama.edu','(373) 684-6002'),(781275,'Ivy Franklin','ultricies.ligula.Nullam@risusQuisquelibero.ca','(512) 101-0330'),(781276,'Hilda F. Morales','Proin.non@urnaNunc.net','(695) 701-6341'),(781277,'Gretchen B. Drake','Nunc.quis@Donecvitaeerat.co.uk','(953) 702-6480'),(781278,'Tara Salazar','non.sollicitudin.a@hendrerit.co.uk','(707) 449-7081'),(781279,'Abbot T. Harrell','vitae.erat@sapien.ca','(373) 320-9506'),(781280,'Ciaran Mcguire','sodales.nisi.magna@eu.edu','(283) 928-6755'),(781281,'Lucy Adkins','Maecenas.ornare.egestas@DonecegestasAliquam.org','(998) 813-8601'),(781282,'May W. Love','laoreet@vehiculaet.edu','(103) 592-9713'),(781283,'Phillip I. Elliott','et@elementum.edu','(988) 992-7939'),(781284,'Rooney Donovan','velit.dui@auguescelerisquemollis.co.uk','(153) 168-8854'),(781285,'John Lindsay','nec@malesuadaaugue.ca','(824) 298-3863'),(781286,'Larissa B. Solis','est.Nunc.laoreet@venenatislacusEtiam.net','(550) 441-1263'),(781287,'Vance I. Leblanc','cursus.diam@sitametrisus.co.uk','(362) 908-0422'),(781288,'Willa Cox','lectus.a.sollicitudin@malesuadaiderat.co.uk','(811) 760-8042'),(781289,'Branden Collier','Nulla.tincidunt@duiCras.ca','(303) 917-6455'),(781290,'Vaughan Acevedo','ridiculus.mus.Proin@faucibuslectusa.ca','(280) 842-1872'),(781291,'Felix Navarro','imperdiet@actellusSuspendisse.edu','(607) 205-0941'),(781292,'Cooper I. Prince','Sed@luctus.edu','(931) 784-2579'),(781293,'Price Wood','non@euismodacfermentum.org','(932) 994-0320'),(781294,'Rahim Rush','ac.arcu@velvenenatisvel.com','(212) 889-4284'),(781295,'Oscar Macias','vulputate.ullamcorper.magna@sodaleselit.org','(639) 699-4508'),(781296,'Kay E. Hammond','amet@Duisvolutpatnunc.edu','(886) 917-3882'),(781297,'Tad L. Mckinney','dolor@dictumeu.co.uk','(680) 369-9733'),(781298,'Abra Hill','aliquet.magna@Fuscefermentumfermentum.co.uk','(980) 279-2743'),(781299,'Paula Lucas','gravida.sit@mauris.ca','(312) 409-3939'),(781300,'Renee Chavez','pretium.et.rutrum@nibhDonecest.edu','(973) 170-2888'),(781301,'Clio Calhoun','mi@arcuAliquam.ca','(904) 186-6159'),(781302,'Oliver Hurley','ipsum.dolor@ametultricies.org','(112) 373-0460'),(781303,'Gloria Lowe','non@fermentum.ca','(408) 321-8905'),(781304,'Benedict V. Cherry','malesuada.vel.venenatis@volutpatornarefacilisis.org','(754) 326-1921'),(781305,'Erica F. Hampton','morbi.tristique@elit.com','(265) 887-9222'),(781306,'Violet E. Clayton','orci.in@disparturient.ca','(749) 306-7231'),(781307,'Sigourney Contreras','ipsum.primis.in@pellentesque.edu','(978) 804-7733'),(781308,'Howard Snider','ac.ipsum.Phasellus@Curabiturmassa.ca','(627) 624-3298'),(781309,'Holmes Barker','porttitor.scelerisque.neque@maurissapien.edu','(185) 196-1299'),(781310,'Edward Vasquez','montes.nascetur@atlibero.edu','(775) 788-0095'),(781311,'Tanner Valencia','dui.Suspendisse.ac@aliquet.ca','(129) 992-2675'),(781312,'Jesse Stewart','Cum@Curabitur.net','(190) 184-8853'),(781313,'Anjolie L. Phelps','est@consectetuer.ca','(436) 108-2034'),(781314,'Adele H. Little','neque.Nullam@vulputatelacus.edu','(687) 643-5062'),(781315,'Imani F. Boyer','ligula.Nullam@a.co.uk','(689) 172-6212'),(781316,'Luke C. Vaughn','elit@natoquepenatibuset.net','(983) 584-2215'),(781317,'Rylee B. Lewis','accumsan@magnaSuspendisse.edu','(649) 283-4760'),(781318,'Isabelle Dillon','dui@auctorMaurisvel.net','(961) 958-9932'),(781319,'Reagan Allen','semper.pretium@rutrum.com','(223) 943-3463'),(781320,'Dorothy Michael','rhoncus.id.mollis@neque.com','(562) 765-3833'),(781321,'Hu T. Vaughn','velit.Cras@Craseutellus.co.uk','(384) 719-8258'),(781322,'Blythe Deleon','mollis.vitae.posuere@nuncest.net','(978) 807-0844'),(781323,'Audrey O. Pennington','interdum.libero@etmagnaPraesent.edu','(348) 783-2061'),(781324,'Zena V. Howard','ligula.Donec@risus.ca','(913) 942-7202'),(781325,'Dale Rivers','luctus@Fusce.org','(838) 423-8298'),(781326,'Paki C. Jacobson','Cras@ornaresagittisfelis.edu','(250) 201-9073');
/*!40000 ALTER TABLE `person` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Temporary view structure for view `person_info`
--

DROP TABLE IF EXISTS `person_info`;
/*!50001 DROP VIEW IF EXISTS `person_info`*/;
SET @saved_cs_client     = @@character_set_client;
/*!50503 SET character_set_client = utf8mb4 */;
/*!50001 CREATE VIEW `person_info` AS SELECT 
 1 AS `name`,
 1 AS `email`*/;
SET character_set_client = @saved_cs_client;

--
-- Table structure for table `restaurants`
--

DROP TABLE IF EXISTS `restaurants`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `restaurants` (
  `idRestaurants` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `address` varchar(250) DEFAULT NULL,
  `phone_number` varchar(15) DEFAULT NULL,
  `website` varchar(250) DEFAULT NULL,
  `schedule` varchar(300) DEFAULT NULL,
  PRIMARY KEY (`idRestaurants`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `restaurants`
--

LOCK TABLES `restaurants` WRITE;
/*!40000 ALTER TABLE `restaurants` DISABLE KEYS */;
INSERT INTO `restaurants` VALUES (123456,'Facilisis Vitae Corporation','602-5919 Montes, St.','(168) 955-4162','leo.','MTW'),(123457,'Pulvinar Arcu PC','Ap #871-7063 Odio. St.','(618) 925-9067','aliquam','M,T,W,R,F'),(123458,'In Inc.','Ap #697-9936 Mauris Rd.','(914) 189-5468','diam','M,T,W,R,F,S'),(123459,'Ac Mi Eleifend Ltd','Ap #221-8555 Sociis Street','(623) 867-1364','arcu.','F,S,S'),(123460,'Magna Et LLC','P.O. Box 374, 8950 Erat Rd.','(967) 785-4245','felis','F,S,S'),(123461,'Orci Incorporated','329 Sodales Street','(227) 912-8777','in,','M,T,W,R,F'),(123462,'Vulputate Risus A Industries','Ap #609-4923 Dignissim Avenue','(490) 615-7947','vel','M,T,W,R,F'),(123463,'Consectetuer LLC','201-1103 Lectus Ave','(379) 310-7877','Aliquam','M,T,W,R,F,S,S'),(123464,'Montes Nascetur Inc.','Ap #640-8377 Ligula. Road','(468) 274-4761','scelerisque','M,T,W,R,F'),(123465,'Tortor Nunc Corporation','Ap #812-3350 Bibendum Ave','(757) 429-7785','fringilla.','M,W,F'),(123466,'Felis Nulla Company','Ap #553-1024 Ridiculus Avenue','(418) 977-6325','Mauris','M,W,F'),(123467,'Taciti LLC','9223 Et Av.','(894) 868-4983','vulputate,','M,W,F'),(123468,'Urna Consulting','338-7709 Ut, Street','(489) 932-3104','ligula.','MTW'),(123469,'Nullam Vitae Diam Corp.','P.O. Box 157, 2230 Donec Rd.','(909) 178-9333','lorem','MTW'),(123470,'Nec Foundation','845-5252 Auctor Ave','(147) 512-6528','sapien.','M,T,W,R,F'),(123471,'Magna Limited','860-755 Pharetra St.','(399) 402-9634','dictum','M,T,W,R,F,S'),(123472,'Amet Faucibus Limited','P.O. Box 999, 9830 Dapibus Rd.','(740) 935-7941','vehicula','MTW'),(123473,'A Institute','961-574 Est Rd.','(751) 987-0877','sagittis','M,T,W,R,F,S'),(123474,'Sed Dictum Proin Associates','530-2998 Mollis Road','(428) 937-6857','parturient','F,S,S'),(123475,'Amet Foundation','746-5331 Convallis, Rd.','(893) 650-2015','ligula.','F,S,S'),(123476,'Fames LLP','P.O. Box 211, 5524 Tincidunt Street','(681) 570-9559','eu','M,W,F'),(123477,'Placerat Orci Lacus Associates','Ap #894-6201 Sed St.','(858) 185-9489','aliquam','M,T,W,R,F'),(123478,'Sit Amet Risus Industries','Ap #119-8261 Lacus. Avenue','(311) 756-3188','adipiscing','M,T,W,R,F,S,S'),(123479,'Est Mauris Eu Foundation','102-8046 Dui, Av.','(243) 608-5694','tristique','F,S,S'),(123480,'Blandit Congue In PC','3286 At St.','(579) 539-4058','eu','M,T,W,R,F,S'),(123481,'Sem Company','8518 Molestie Road','(575) 175-9606','sem','M,T,W,R,F'),(123482,'A Nunc In Industries','472-3630 Et Street','(675) 367-1977','tellus.','M,W,F'),(123483,'Risus Inc.','P.O. Box 843, 9341 Ornare, Ave','(292) 991-7886','varius.','F,S,S'),(123484,'Consequat Nec Consulting','P.O. Box 104, 2128 Scelerisque St.','(810) 421-2203','dolor','M,W,F'),(123485,'In Molestie Tortor Limited','Ap #841-6407 Faucibus Rd.','(866) 974-2147','pede,','M,W,F'),(123486,'Cum Sociis Natoque Institute','P.O. Box 454, 909 Posuere Rd.','(457) 327-3168','dapibus','MTW'),(123487,'Nibh Sit Ltd','721-7800 Nulla Road','(965) 364-6174','malesuada','M,T,W,R,F,S'),(123488,'Tellus Faucibus Institute','Ap #332-3303 Et St.','(477) 723-9603','tincidunt','M,W,F'),(123489,'Sit Amet Inc.','2463 Aenean Road','(103) 615-5009','In','M,T,W,R,F'),(123490,'Sodales At Corporation','425-8020 Rhoncus. Rd.','(168) 819-3024','Pellentesque','M,T,W,R,F,S,S'),(123491,'Quis Consulting','229-5755 Dui, Road','(388) 403-5214','pede.','M,T,W,R,F,S'),(123492,'Mollis Non Cursus Associates','900-1069 Scelerisque Road','(384) 167-5738','lobortis','MTW'),(123493,'Nascetur Ridiculus Ltd','6851 Amet St.','(161) 561-4831','nonummy','F,S,S'),(123494,'Maecenas Libero Institute','Ap #705-9374 Cursus Road','(942) 214-9500','blandit','M,T,W,R,F,S'),(123495,'Ac Mattis Foundation','2678 Nascetur Road','(559) 898-7135','dictum','F,S,S'),(123496,'Nunc Sed Orci Inc.','6087 Duis Avenue','(988) 143-7373','sociis','F,S,S'),(123497,'Venenatis Company','4513 Mus. St.','(679) 501-9820','metus.','F,S,S'),(123498,'In Foundation','P.O. Box 849, 1456 Lacinia Street','(276) 743-7507','varius','M,T,W,R,F'),(123499,'Eleifend Non Dapibus LLP','7804 Erat Ave','(467) 180-4286','ante','MTW'),(123500,'Accumsan Neque Et PC','327-4090 Scelerisque Street','(782) 204-7897','Cum','M,T,W,R,F,S,S'),(123501,'Eget Incorporated','296-9217 Duis Av.','(388) 418-0479','molestie','M,T,W,R,F,S,S'),(123502,'Iaculis Quis Company','Ap #907-5040 In Avenue','(406) 908-9257','natoque','M,T,W,R,F,S'),(123503,'Risus At LLC','P.O. Box 891, 1467 Eu St.','(957) 611-6999','interdum','M,W,F'),(123504,'Eu LLC','P.O. Box 600, 2136 Quisque St.','(967) 875-3937','ornare.','M,T,W,R,F,S'),(123505,'Lectus Nullam Corp.','Ap #443-2570 Integer Street','(723) 821-9011','a,','MTW'),(123506,'Faucibus Morbi Corporation','1986 Vulputate, Ave','(181) 719-8817','dapibus','M,T,W,R,F,S,S'),(123507,'Sit Amet Industries','P.O. Box 397, 8783 Facilisi. Avenue','(517) 212-4418','consectetuer','M,T,W,R,F,S'),(123508,'Ut Consulting','3913 Orci Ave','(808) 287-8429','Ut','F,S,S'),(123509,'Dolor Dapibus Gravida Corp.','1484 Non Ave','(222) 227-9380','magna.','F,S,S'),(123510,'Eu Eleifend Nec Corp.','859-173 Eu Rd.','(946) 926-9357','nisi','F,S,S'),(123511,'Non Inc.','Ap #714-9857 Malesuada Av.','(240) 634-6392','Nulla','M,W,F'),(123512,'Quisque Porttitor Eros Corp.','4573 Vitae, Ave','(453) 722-0129','urna,','M,T,W,R,F,S,S'),(123513,'Sapien Inc.','Ap #653-7290 Ornare St.','(495) 937-6516','nibh.','MTW'),(123514,'Felis Limited','868-8213 Consequat Ave','(373) 313-1548','dictum','M,W,F'),(123515,'Amet Ultricies Sem Industries','901-2872 Lorem. St.','(396) 412-5506','sodales','F,S,S'),(123516,'Ut Sagittis Ltd','P.O. Box 908, 7607 Mi Av.','(564) 267-9321','augue','F,S,S'),(123517,'Et Magnis LLC','P.O. Box 916, 3404 Turpis. Road','(301) 132-0379','nunc','M,T,W,R,F,S'),(123518,'Malesuada PC','597-7722 Ornare, St.','(439) 134-1882','ornare','M,T,W,R,F,S'),(123519,'Fusce Aliquet Inc.','Ap #274-7053 Semper Avenue','(624) 747-5787','orci.','M,T,W,R,F,S,S'),(123520,'Elit Fermentum Associates','Ap #471-6129 Sit Rd.','(159) 415-3170','eu','MTW'),(123521,'Donec Non Justo Inc.','Ap #608-4463 Magnis Rd.','(712) 593-4448','rutrum.','M,T,W,R,F,S'),(123522,'Libero Associates','P.O. Box 140, 1965 Consectetuer Street','(799) 395-9002','dictum','M,T,W,R,F,S,S'),(123523,'Augue Sed Company','Ap #378-5125 Duis Av.','(154) 788-9919','dolor.','F,S,S'),(123524,'Sed Institute','P.O. Box 323, 2759 Habitant Rd.','(135) 590-6487','facilisis','F,S,S'),(123525,'Dui In PC','672-4723 Vel, St.','(625) 873-3165','sem,','F,S,S'),(123526,'Adipiscing Ligula Aenean Institute','P.O. Box 148, 5118 At, Road','(513) 564-9911','nec','M,T,W,R,F'),(123527,'Non Bibendum Sed Associates','748-1769 Placerat. Av.','(490) 366-8710','massa','M,T,W,R,F'),(123528,'Nulla Interdum Corp.','903 Donec Road','(957) 197-1531','Integer','M,W,F'),(123529,'Nonummy Limited','972-4355 Aliquam Rd.','(968) 539-4826','nec,','M,T,W,R,F,S,S'),(123530,'Adipiscing Elit Limited','922 Sem Road','(363) 132-8024','ornare.','M,W,F'),(123531,'Nec Company','606-8711 Sed Av.','(676) 343-2349','sed','MTW'),(123532,'Ut Ipsum Ac Consulting','702-9935 Duis Avenue','(642) 261-7823','metus.','M,T,W,R,F,S'),(123533,'Morbi Quis LLC','P.O. Box 996, 6180 Ligula. Rd.','(712) 655-9764','imperdiet','F,S,S'),(123534,'Dui Nec Inc.','998-8940 Vel Ave','(669) 163-2703','auctor,','M,T,W,R,F,S'),(123535,'Praesent LLP','2491 Non Street','(815) 755-7421','at,','M,T,W,R,F,S,S'),(123536,'Tincidunt Orci Corp.','Ap #603-2183 Enim. Street','(926) 743-1979','ipsum','M,T,W,R,F'),(123537,'Maecenas Malesuada Fringilla Corp.','P.O. Box 877, 1612 Commodo Av.','(393) 593-7846','felis','M,W,F'),(123538,'Accumsan Neque Corp.','7570 Lorem, Road','(143) 674-9708','Proin','M,W,F'),(123539,'Vulputate Nisi Industries','1759 Proin Rd.','(611) 869-5840','neque','MTW'),(123540,'Est PC','Ap #571-2606 Neque. Avenue','(975) 627-8471','ut','M,T,W,R,F,S,S'),(123541,'Aenean Massa Limited','Ap #955-2672 Per Rd.','(192) 599-0274','non','M,T,W,R,F'),(123542,'Magna Cras Inc.','3791 Donec Avenue','(214) 306-9385','nulla.','M,T,W,R,F,S'),(123543,'A Magna Lorem Inc.','3324 Leo, Street','(272) 727-2694','convallis','M,T,W,R,F'),(123544,'Suspendisse Industries','3173 Curabitur Road','(163) 975-3819','rutrum','M,T,W,R,F,S,S'),(123545,'Id Ante Nunc Company','8455 Placerat. Road','(964) 669-1624','accumsan','M,T,W,R,F'),(123546,'Sit Amet PC','Ap #771-9208 Hendrerit Rd.','(194) 689-4502','Vestibulum','F,S,S'),(123547,'Feugiat Consulting','Ap #576-4502 Augue Ave','(198) 663-8720','nunc','M,T,W,R,F,S'),(123548,'Parturient Montes Consulting','816 Ultricies Ave','(367) 621-4560','nisl.','M,T,W,R,F,S,S'),(123549,'Donec Fringilla Institute','Ap #551-6465 Sed, St.','(100) 750-4363','imperdiet','MTW'),(123550,'Adipiscing Ligula Aenean Corporation','206-5329 Et, Rd.','(348) 619-3601','laoreet,','F,S,S'),(123551,'Non Feugiat Corporation','466-1782 Purus. Av.','(971) 840-0197','semper','M,W,F'),(123552,'Pede Cras Vulputate Industries','4683 Diam. Ave','(134) 684-3550','consequat','M,W,F'),(123553,'Lorem Ipsum PC','8051 Nisl. Street','(580) 340-7959','eu','F,S,S'),(123554,'Mollis Incorporated','426-2698 Euismod Road','(229) 867-5586','sit','M,W,F'),(123555,'Ultrices Foundation','6794 Vitae, Street','(769) 907-0886','cubilia','F,S,S');
/*!40000 ALTER TABLE `restaurants` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `staff`
--

DROP TABLE IF EXISTS `staff`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `staff` (
  `idStaff` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `email` varchar(250) DEFAULT NULL,
  `phone_number` varchar(15) DEFAULT NULL,
  `position` varchar(250) DEFAULT NULL,
  `adminYorN` tinyint DEFAULT NULL,
  PRIMARY KEY (`idStaff`),
  CONSTRAINT `fk_Staff_Person1` FOREIGN KEY (`idStaff`) REFERENCES `person` (`idPerson`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `staff`
--

LOCK TABLES `staff` WRITE;
/*!40000 ALTER TABLE `staff` DISABLE KEYS */;
INSERT INTO `staff` VALUES (771227,'Olga Mack','consectetuer.adipiscing.elit@et.co.uk','(287) 765-8795','Advisor',1),(771228,'Mufutau N. Mcpherson','molestie.dapibus@vestibulum.org','(106) 523-4816','Advisor',1),(771229,'Rigel T. Burt','fringilla@amet.org','(785) 414-6372','Advisor',0),(771230,'Plato Weeks','varius@neque.net','(539) 561-2308','Staff Accountant',1),(771231,'Vincent Donaldson','Phasellus.at.augue@sedleo.net','(124) 962-7732','Staff Accountant',1),(771232,'Kane M. Flynn','tincidunt@aliquamerosturpis.org','(487) 288-3448','Staff Accountant',0),(771233,'Damian F. Burnett','Etiam@Utnecurna.net','(951) 477-3528','Director of Development',1),(771234,'Hilel Newman','lobortis@orcilacus.edu','(978) 474-7906','Director of Development',1),(771235,'Unity Y. Powell','Vestibulum.ante.ipsum@Duis.edu','(461) 396-2843','Director of Development',0),(771236,'Marvin X. Waters','sed.sapien.Nunc@Ut.com','(571) 376-4507','Advisor',0),(771237,'Ariana R. Welch','sem.magna@ultrices.edu','(567) 179-9526','Advisor',0),(771238,'Aileen Nash','mauris.sagittis.placerat@adipiscingligula.com','(862) 526-9849','Advisor',0),(771239,'Selma Mccarthy','Nunc@consequatlectussit.com','(483) 553-5110','Staff Accountant',1),(771240,'Imogene V. Curtis','risus@elit.ca','(279) 710-0991','Staff Accountant',1),(771241,'Lance Conway','mollis@Phasellusataugue.edu','(275) 979-2151','Staff Accountant',1),(771242,'McKenzie F. Chan','vulputate.ullamcorper.magna@semelit.co.uk','(569) 281-2637','Director of Development',0),(771243,'Jaquelyn C. Wade','nec.malesuada.ut@leoCras.co.uk','(209) 831-4640','Director of Development',0),(771244,'Deirdre Cortez','Curabitur.consequat.lectus@afelis.org','(366) 592-1781','Director of Development',1),(771245,'Francis Joyner','Suspendisse@gravida.ca','(390) 228-4500','Advisor',0),(771246,'Herrod Roth','tempus.non@Cras.ca','(738) 839-4525','Advisor',1),(771247,'Armand Chapman','pede@blanditenim.net','(125) 668-9997','Advisor',0),(771248,'Joy D. Lewis','aliquet.diam.Sed@Mauris.com','(850) 258-3638','Staff Accountant',0),(771249,'Bree Thomas','tellus@pedeultrices.com','(895) 565-1572','Staff Accountant',0),(771250,'Leilani Larson','lacus.pede.sagittis@Curabiturconsequatlectus.co.uk','(162) 902-5915','Staff Accountant',1),(771251,'Abbot N. Fisher','quis.urna@velitdui.co.uk','(360) 789-4750','Director of Development',1),(771252,'Karen Sandoval','sed@musProin.net','(748) 351-2619','Director of Development',0),(771253,'Amy Donaldson','dolor@laoreet.edu','(200) 758-3288','Director of Development',1),(771254,'Haviva Hurst','consectetuer@acnullaIn.co.uk','(163) 994-5759','Advisor',1),(771255,'Otto Watson','faucibus.ut.nulla@quispedePraesent.org','(493) 318-3960','Advisor',0),(771256,'Adam N. Gilliam','diam.Pellentesque@Nunc.com','(117) 772-5815','Advisor',0),(771257,'Barry Key','bibendum@orciadipiscingnon.edu','(826) 481-0594','Staff Accountant',1),(771258,'Daquan E. Martin','ante.iaculis@aliquam.ca','(969) 707-6209','Staff Accountant',0),(771259,'Buffy Price','rutrum.lorem@neque.ca','(909) 909-9444','Staff Accountant',0),(771260,'Solomon Cooke','in@a.ca','(210) 594-6053','Director of Development',1),(771261,'Drew S. Mendoza','Aliquam.auctor@mauris.edu','(773) 186-1445','Director of Development',0),(771262,'Camden Goff','tellus@ut.edu','(220) 369-2529','Director of Development',0),(771263,'Judah G. Mathews','nec.enim.Nunc@justo.ca','(775) 174-5149','Advisor',1),(771264,'Dominic Z. Mason','mi@augueac.net','(565) 524-1006','Advisor',1),(771265,'Amanda N. Matthews','leo.elementum.sem@purusgravida.net','(189) 815-4546','Advisor',1),(771266,'Hilda S. Dillard','ipsum.nunc.id@egestasSedpharetra.ca','(822) 390-8195','Staff Accountant',1),(771267,'Conan Kemp','semper.rutrum@risusatfringilla.net','(971) 417-5225','Staff Accountant',0),(771268,'Karyn Vance','lorem.fringilla@duinec.ca','(443) 466-2818','Staff Accountant',0),(771269,'Octavius Barlow','dapibus.id.blandit@utquam.ca','(729) 268-1243','Director of Development',0),(771270,'Thane G. Burns','semper.et.lacinia@magnisdis.org','(150) 906-4430','Director of Development',1),(771271,'Hop House','enim@gravidaAliquam.net','(896) 943-5889','Director of Development',0),(771272,'Ignacia Marquez','amet.massa.Quisque@Inlorem.org','(108) 358-4189','Advisor',0),(771273,'Rafael L. Kirby','ipsum@nibhQuisquenonummy.com','(659) 117-9761','Advisor',0),(771274,'Shea N. Flowers','ultricies.ligula.Nullam@DuisgravidaPraesent.edu','(379) 970-4695','Advisor',1),(771275,'Gannon Q. Parsons','vitae.dolor.Donec@Aeneangravida.ca','(380) 237-0758','Staff Accountant',1),(771276,'Florence K. Trevino','per@vitaealiquam.com','(631) 840-3133','Staff Accountant',1),(771277,'Mohammad O. David','lectus.justo@pharetrasedhendrerit.com','(947) 893-8296','Staff Accountant',0),(771278,'Thomas E. Stuart','ac.sem.ut@sed.org','(746) 792-7096','Director of Development',1),(771279,'Roanna Lang','sociosqu.ad@maurisMorbi.com','(968) 263-3471','Director of Development',1),(771280,'Byron D. Wilder','congue.turpis.In@nulla.edu','(967) 935-0384','Director of Development',0),(771281,'Camilla Mooney','aliquam@tellusjusto.com','(678) 483-3879','Advisor',0),(771282,'Carter Page','non.sollicitudin@diamvelarcu.edu','(386) 800-2923','Advisor',1),(771283,'Katelyn K. Booker','tellus.eu@scelerisqueloremipsum.com','(645) 864-3147','Advisor',0),(771284,'Uma Riddle','elementum@eu.ca','(965) 501-8412','Staff Accountant',1),(771285,'Bo Burton','odio@arcuVivamus.edu','(225) 218-2675','Staff Accountant',1),(771286,'Ali Durham','auctor.non.feugiat@liberodui.org','(589) 962-1946','Staff Accountant',0),(771287,'Brynne W. Blake','nisi.Aenean.eget@ProinvelitSed.edu','(334) 989-3481','Director of Development',0),(771288,'Knox Obrien','Quisque.varius.Nam@faucibus.com','(660) 342-0859','Director of Development',1),(771289,'Orson R. Garner','diam.dictum.sapien@leoMorbineque.org','(583) 324-2270','Director of Development',1),(771290,'Abdul R. Tyler','lacus.Nulla.tincidunt@apurusDuis.co.uk','(353) 988-7550','Advisor',0),(771291,'Timothy Bolton','dui@ultricesa.net','(300) 899-0129','Advisor',1),(771292,'Gage T. Ferrell','semper.et@tristiquenequevenenatis.org','(479) 941-7226','Advisor',0),(771293,'Casey R. Ellis','pede.nonummy.ut@mienimcondimentum.co.uk','(164) 129-1044','Staff Accountant',0),(771294,'Orli Reese','egestas@facilisiSedneque.com','(366) 206-1269','Staff Accountant',0),(771295,'Idona M. Walls','Aliquam@velitCras.com','(219) 509-0662','Staff Accountant',0),(771296,'Lars Trevino','orci.Donec.nibh@vulputateposuerevulputate.ca','(679) 814-9633','Director of Development',1),(771297,'Rigel D. Lancaster','sagittis@sagittisaugue.ca','(684) 428-4346','Director of Development',0),(771298,'Riley R. Hyde','Nunc@ligulaAliquamerat.co.uk','(569) 790-4250','Director of Development',0),(771299,'Cairo Le','nunc.Quisque.ornare@sociisnatoquepenatibus.net','(796) 397-5104','Advisor',0),(771300,'Jared J. Hammond','egestas.lacinia@gravidasit.ca','(319) 469-1213','Advisor',1),(771301,'Jerry Holloway','Pellentesque.habitant@Quisque.net','(600) 749-1879','Advisor',1),(771302,'Shelley Fitzpatrick','sollicitudin.a@nislelementum.org','(584) 651-8310','Staff Accountant',1),(771303,'Kim Moreno','malesuada@acfeugiat.com','(637) 362-1255','Staff Accountant',1),(771304,'Bradley V. Hartman','Class@nec.net','(674) 742-2069','Staff Accountant',0),(771305,'Yoshio O. Clayton','magna.nec.quam@orciUt.net','(369) 341-9961','Director of Development',1),(771306,'Emily N. Figueroa','Donec@cubiliaCurae.co.uk','(174) 523-3583','Director of Development',1),(771307,'Kirsten Key','urna.nec.luctus@ullamcorperviverra.com','(848) 251-6329','Director of Development',1),(771308,'Iris O. Vazquez','metus@inmagna.edu','(108) 521-0282','Advisor',0),(771309,'Pearl Elliott','nec@orci.com','(230) 988-9134','Advisor',1),(771310,'Gannon Anderson','Cum@Nullamsuscipit.edu','(779) 265-8069','Advisor',1),(771311,'Dennis R. Workman','bibendum.ullamcorper@ametorciUt.org','(734) 498-8519','Staff Accountant',1),(771312,'Jerome Jacobson','nec.tempus@mattisvelitjusto.com','(633) 362-6949','Staff Accountant',0),(771313,'Mariko X. Wilder','Etiam.gravida@ut.edu','(516) 138-3332','Staff Accountant',0),(771314,'Tamara Rodriquez','Lorem.ipsum.dolor@Mauris.org','(478) 735-9788','Director of Development',0),(771315,'Camden Mcleod','non.massa@magnaPraesent.ca','(809) 447-6422','Director of Development',1),(771316,'Lois Middleton','faucibus@dictum.net','(137) 987-8469','Director of Development',1),(771317,'Heidi B. Alexander','Aliquam@egestas.co.uk','(931) 249-4079','Advisor',0),(771318,'India A. Cameron','vestibulum.nec@Aenean.net','(151) 970-9032','Advisor',1),(771319,'Venus Cherry','leo.Morbi.neque@tempusscelerisque.net','(256) 334-9471','Advisor',0),(771320,'Jelani N. Rodriquez','Nunc@feugiatLorem.net','(113) 377-1038','Staff Accountant',0),(771321,'Azalia S. Leonard','Fusce@Morbivehicula.com','(533) 848-6831','Staff Accountant',1),(771322,'Norman Hopkins','Donec.sollicitudin@et.org','(502) 468-1713','Staff Accountant',0),(771323,'Arden I. Conley','et.ipsum@euaccumsansed.edu','(192) 824-0387','Director of Development',1),(771324,'Ira Rush','lacinia@malesuadaid.co.uk','(890) 794-4461','Director of Development',1),(771325,'Murphy Swanson','leo@estconguea.edu','(739) 544-2407','Director of Development',1),(771326,'Xenos Mcneil','fermentum.metus@Maurismolestiepharetra.ca','(333) 826-1777','Advisor',0);
/*!40000 ALTER TABLE `staff` ENABLE KEYS */;
UNLOCK TABLES;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8mb4 */ ;
/*!50003 SET character_set_results = utf8mb4 */ ;
/*!50003 SET collation_connection  = utf8mb4_0900_ai_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
/*!50003 CREATE*/ /*!50017 DEFINER=`root`@`localhost`*/ /*!50003 TRIGGER `staff_BEFORE_INSERT` BEFORE INSERT ON `staff` FOR EACH ROW BEGIN
	insert into person
    values(new.idStaff, new.name, new.email, new.phone_number);
END */;;
DELIMITER ;
/*!50003 SET sql_mode              = @saved_sql_mode */ ;
/*!50003 SET character_set_client  = @saved_cs_client */ ;
/*!50003 SET character_set_results = @saved_cs_results */ ;
/*!50003 SET collation_connection  = @saved_col_connection */ ;

--
-- Table structure for table `student`
--

DROP TABLE IF EXISTS `student`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `student` (
  `idStudent` int NOT NULL,
  `name` varchar(250) DEFAULT NULL,
  `email` varchar(250) DEFAULT NULL,
  `phone_number` varchar(15) DEFAULT NULL,
  `gradYear` year DEFAULT NULL,
  `major` varchar(250) DEFAULT NULL,
  `type` varchar(15) DEFAULT NULL,
  PRIMARY KEY (`idStudent`),
  CONSTRAINT `fk_Student_Person` FOREIGN KEY (`idStudent`) REFERENCES `person` (`idPerson`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `student`
--

LOCK TABLES `student` WRITE;
/*!40000 ALTER TABLE `student` DISABLE KEYS */;
INSERT INTO `student` VALUES (780000,'Adona Mwense','mwense07@gmail.com','(704) 858-6584',2020,'Mathematics for Business','undergrad'),(781227,'Dominic Dodson','ut.lacus@Proineget.org','(614) 660-0550',2021,'Accounting','Undergrad'),(781228,'Rana U. Phelps','mi.felis.adipiscing@amet.co.uk','(868) 549-4244',2023,' Political Science','Undergrad'),(781229,'Candace Marks','nulla@Aeneangravida.com','(175) 501-3072',2020,'Physics','Undergrad'),(781230,'Brady H. Tran','malesuada@ac.ca','(644) 719-1158',2023,'English','Undergrad'),(781231,'Shannon Q. Parsons','eros.non.enim@elitpretiumet.org','(366) 881-0272',2022,'Finance','Undergrad'),(781232,'Neil Paul','eu.tellus@sollicitudin.com','(322) 139-8174',2022,' Political Science','Undergrad'),(781233,'Ashton Powers','vitae@necante.ca','(679) 799-3962',2021,'Music','Undergrad'),(781234,'Adrienne Mcleod','adipiscing.lobortis.risus@elit.net','(681) 487-8401',2024,'Accounting','Undergrad'),(781235,'Hanae Farmer','eget@et.co.uk','(722) 234-4590',2019,'Accounting','Undergrad'),(781236,'Clio Taylor','eget.odio.Aliquam@etrutrumeu.ca','(701) 826-3149',2020,'Economics','Undergrad'),(781237,'Nigel J. Spears','Sed@nisi.com','(490) 412-6114',2021,'Sociology','Graduate'),(781238,'Wynter R. Stevens','Duis.gravida@Vivamus.net','(734) 801-5545',2023,'Engineering','Graduate'),(781239,'Kevyn X. Burnett','id@nonummy.com','(394) 236-9537',2023,'Physics','Graduate'),(781240,'Kylynn Elliott','ullamcorper.Duis@luctus.edu','(926) 612-9572',2019,'Architecture','Graduate'),(781241,'Jaquelyn Nash','dignissim@facilisisSuspendissecommodo.net','(496) 997-3524',2022,'History','Graduate'),(781242,'Kyla W. Mcintosh','sem.egestas@tristiquealiquetPhasellus.org','(435) 962-9114',2021,'Accounting','Graduate'),(781243,'Rama D. Murray','sed.consequat@Aeneanegestas.ca','(791) 559-6258',2021,'Accounting','Graduate'),(781244,'Shelly V. Baker','parturient@temporaugue.org','(715) 753-0684',2023,' Political Science','Graduate'),(781245,'August Bates','aliquam.eros.turpis@diamnuncullamcorper.co.uk','(654) 152-8576',2021,'Economics','Graduate'),(781246,'Fallon N. Greer','Phasellus.elit@Cumsociis.net','(315) 541-4254',2022,'Mathematics','Graduate'),(781247,'Conan Z. Perez','massa.Quisque.porttitor@non.com','(270) 115-1873',2022,'Mathematics','Undergrad'),(781248,'Salvador X. Williamson','nec.cursus.a@litoratorquentper.net','(696) 703-2618',2022,'Music','Undergrad'),(781249,'Michelle N. Greene','quam@nonquamPellentesque.edu','(565) 378-0791',2022,'Economics','Undergrad'),(781250,'Mechelle Galloway','ut@consectetueradipiscing.org','(863) 824-1553',2023,'Computer Science','Undergrad'),(781251,'Neil Mcdowell','in@nequeNullam.net','(785) 890-3411',2020,'Computer Science','Undergrad'),(781252,'Cherokee Q. Greer','amet.ante.Vivamus@indolor.edu','(837) 602-0353',2023,'Biology','Undergrad'),(781253,'Roanna U. Parrish','lectus.a.sollicitudin@Fuscedolor.com','(409) 123-5517',2019,'Sociology','Undergrad'),(781254,'Dieter O. Christensen','erat@pharetraNam.edu','(112) 415-9200',2022,'Biology','Undergrad'),(781255,'Imelda D. Stevens','scelerisque.scelerisque.dui@dignissim.ca','(571) 571-0703',2022,'Computer Science','Undergrad'),(781256,'Jenette F. Matthews','cursus.luctus@Ut.net','(405) 942-3612',2022,'Sociology','Undergrad'),(781257,'Jonah C. Mcdowell','cursus.et@ullamcorpervelitin.org','(843) 587-6599',2022,'English','Graduate'),(781258,'Candice Barron','Etiam.imperdiet@nibhQuisquenonummy.edu','(765) 249-8309',2021,'Physics','Graduate'),(781259,'Jolene Luna','mauris.sit@urna.com','(115) 943-7328',2021,'English','Graduate'),(781260,'Ulric V. England','luctus.aliquet@Nullam.org','(281) 980-4444',2020,'Mathematics','Graduate'),(781261,'Vance R. Watkins','aliquam.eu@massaVestibulum.org','(460) 635-2315',2023,'History','Graduate'),(781262,'Hu D. Boyd','libero.nec@eget.org','(690) 223-1224',2020,'Economics','Graduate'),(781263,'Erich J. Hodge','quam.Curabitur@MaurisnullaInteger.ca','(329) 302-4203',2021,'History','Graduate'),(781264,'Haley A. Bolton','auctor.nunc.nulla@sitamet.co.uk','(644) 841-3494',2020,'Finance','Graduate'),(781265,'Ulysses D. Rios','morbi.tristique@turpis.org','(615) 225-4887',2021,'Engineering','Graduate'),(781266,'Tatiana Potts','tristique.neque.venenatis@cursusIntegermollis.ca','(377) 262-5289',2019,'Economics','Graduate'),(781267,'Claire Q. Burt','id.libero@fringillami.ca','(680) 389-7197',2021,'Accounting','Undergrad'),(781268,'Yuli J. Tate','purus.Nullam.scelerisque@atnisiCum.net','(444) 759-6440',2022,'English','Undergrad'),(781269,'Aurora Y. Orr','Lorem@mollisdui.com','(585) 599-2485',2023,'Computer Science','Undergrad'),(781270,'Ahmed O. Cox','eget@utdolor.edu','(119) 704-6994',2019,'Music','Undergrad'),(781271,'Rebecca I. Wright','enim.nec@malesuadafames.com','(875) 872-0252',2021,' Political Science','Undergrad'),(781272,'Unity W. Parrish','vel.mauris.Integer@anteipsum.com','(458) 933-5544',2022,'Computer Science','Undergrad'),(781273,'Abdul Kaufman','Duis.dignissim@portaelit.net','(602) 493-2935',2019,'Economics','Undergrad'),(781274,'Macon Curtis','mauris.sapien.cursus@lobortisquama.edu','(373) 684-6002',2023,'Accounting','Undergrad'),(781275,'Ivy Franklin','ultricies.ligula.Nullam@risusQuisquelibero.ca','(512) 101-0330',2022,'Engineering','Undergrad'),(781276,'Hilda F. Morales','Proin.non@urnaNunc.net','(695) 701-6341',2021,'Sociology','Undergrad'),(781277,'Gretchen B. Drake','Nunc.quis@Donecvitaeerat.co.uk','(953) 702-6480',2021,'Architecture','Graduate'),(781278,'Tara Salazar','non.sollicitudin.a@hendrerit.co.uk','(707) 449-7081',2021,'Biology','Graduate'),(781279,'Abbot T. Harrell','vitae.erat@sapien.ca','(373) 320-9506',2020,' Political Science','Graduate'),(781280,'Ciaran Mcguire','sodales.nisi.magna@eu.edu','(283) 928-6755',2019,'Architecture','Graduate'),(781281,'Lucy Adkins','Maecenas.ornare.egestas@DonecegestasAliquam.org','(998) 813-8601',2021,'Engineering','Graduate'),(781282,'May W. Love','laoreet@vehiculaet.edu','(103) 592-9713',2020,'English','Graduate'),(781283,'Phillip I. Elliott','et@elementum.edu','(988) 992-7939',2019,'Engineering','Graduate'),(781284,'Rooney Donovan','velit.dui@auguescelerisquemollis.co.uk','(153) 168-8854',2020,' Nursing','Graduate'),(781285,'John Lindsay','nec@malesuadaaugue.ca','(824) 298-3863',2022,'Music','Graduate'),(781286,'Larissa B. Solis','est.Nunc.laoreet@venenatislacusEtiam.net','(550) 441-1263',2022,' Political Science','Graduate'),(781287,'Vance I. Leblanc','cursus.diam@sitametrisus.co.uk','(362) 908-0422',2020,'Biology','Undergrad'),(781288,'Willa Cox','lectus.a.sollicitudin@malesuadaiderat.co.uk','(811) 760-8042',2020,'Biology','Undergrad'),(781289,'Branden Collier','Nulla.tincidunt@duiCras.ca','(303) 917-6455',2022,'Architecture','Undergrad'),(781290,'Vaughan Acevedo','ridiculus.mus.Proin@faucibuslectusa.ca','(280) 842-1872',2021,'Mathematics','Undergrad'),(781291,'Felix Navarro','imperdiet@actellusSuspendisse.edu','(607) 205-0941',2020,'Engineering','Undergrad'),(781292,'Cooper I. Prince','Sed@luctus.edu','(931) 784-2579',2022,'Architecture','Undergrad'),(781293,'Price Wood','non@euismodacfermentum.org','(932) 994-0320',2019,'English','Undergrad'),(781294,'Rahim Rush','ac.arcu@velvenenatisvel.com','(212) 889-4284',2023,'Engineering','Undergrad'),(781295,'Oscar Macias','vulputate.ullamcorper.magna@sodaleselit.org','(639) 699-4508',2022,'Engineering','Undergrad'),(781296,'Kay E. Hammond','amet@Duisvolutpatnunc.edu','(886) 917-3882',2023,'Accounting','Undergrad'),(781297,'Tad L. Mckinney','dolor@dictumeu.co.uk','(680) 369-9733',2023,'Computer Science','Graduate'),(781298,'Abra Hill','aliquet.magna@Fuscefermentumfermentum.co.uk','(980) 279-2743',2022,'Finance','Graduate'),(781299,'Paula Lucas','gravida.sit@mauris.ca','(312) 409-3939',2022,'Architecture','Graduate'),(781300,'Renee Chavez','pretium.et.rutrum@nibhDonecest.edu','(973) 170-2888',2020,'Physics','Graduate'),(781301,'Clio Calhoun','mi@arcuAliquam.ca','(904) 186-6159',2019,'Accounting','Graduate'),(781302,'Oliver Hurley','ipsum.dolor@ametultricies.org','(112) 373-0460',2020,'History','Graduate'),(781303,'Gloria Lowe','non@fermentum.ca','(408) 321-8905',2023,' Nursing','Graduate'),(781304,'Benedict V. Cherry','malesuada.vel.venenatis@volutpatornarefacilisis.org','(754) 326-1921',2024,'Engineering','Graduate'),(781305,'Erica F. Hampton','morbi.tristique@elit.com','(265) 887-9222',2022,'Architecture','Graduate'),(781306,'Violet E. Clayton','orci.in@disparturient.ca','(749) 306-7231',2019,' Nursing','Graduate'),(781307,'Sigourney Contreras','ipsum.primis.in@pellentesque.edu','(978) 804-7733',2023,'Sociology','Undergrad'),(781308,'Howard Snider','ac.ipsum.Phasellus@Curabiturmassa.ca','(627) 624-3298',2022,'Architecture','Undergrad'),(781309,'Holmes Barker','porttitor.scelerisque.neque@maurissapien.edu','(185) 196-1299',2020,'Finance','Undergrad'),(781310,'Edward Vasquez','montes.nascetur@atlibero.edu','(775) 788-0095',2023,'Mathematics','Undergrad'),(781311,'Tanner Valencia','dui.Suspendisse.ac@aliquet.ca','(129) 992-2675',2019,'Music','Undergrad'),(781312,'Jesse Stewart','Cum@Curabitur.net','(190) 184-8853',2021,'Sociology','Undergrad'),(781313,'Anjolie L. Phelps','est@consectetuer.ca','(436) 108-2034',2021,' Nursing','Undergrad'),(781314,'Adele H. Little','neque.Nullam@vulputatelacus.edu','(687) 643-5062',2020,'Physics','Undergrad'),(781315,'Imani F. Boyer','ligula.Nullam@a.co.uk','(689) 172-6212',2022,'Computer Science','Undergrad'),(781316,'Luke C. Vaughn','elit@natoquepenatibuset.net','(983) 584-2215',2020,'Architecture','Undergrad'),(781317,'Rylee B. Lewis','accumsan@magnaSuspendisse.edu','(649) 283-4760',2021,'Mathematics','Graduate'),(781318,'Isabelle Dillon','dui@auctorMaurisvel.net','(961) 958-9932',2023,'Engineering','Graduate'),(781319,'Reagan Allen','semper.pretium@rutrum.com','(223) 943-3463',2020,'History','Graduate'),(781320,'Dorothy Michael','rhoncus.id.mollis@neque.com','(562) 765-3833',2021,'History','Graduate'),(781321,'Hu T. Vaughn','velit.Cras@Craseutellus.co.uk','(384) 719-8258',2022,' Political Science','Graduate'),(781322,'Blythe Deleon','mollis.vitae.posuere@nuncest.net','(978) 807-0844',2022,'English','Graduate'),(781323,'Audrey O. Pennington','interdum.libero@etmagnaPraesent.edu','(348) 783-2061',2020,' Political Science','Graduate'),(781324,'Zena V. Howard','ligula.Donec@risus.ca','(913) 942-7202',2020,'Finance','Graduate'),(781325,'Dale Rivers','luctus@Fusce.org','(838) 423-8298',2022,'Biology','Graduate'),(781326,'Paki C. Jacobson','Cras@ornaresagittisfelis.edu','(250) 201-9073',2024,'Biology','Graduate');
/*!40000 ALTER TABLE `student` ENABLE KEYS */;
UNLOCK TABLES;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8mb4 */ ;
/*!50003 SET character_set_results = utf8mb4 */ ;
/*!50003 SET collation_connection  = utf8mb4_0900_ai_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
/*!50003 CREATE*/ /*!50017 DEFINER=`root`@`localhost`*/ /*!50003 TRIGGER `student_BEFORE_INSERT` BEFORE INSERT ON `student` FOR EACH ROW BEGIN
	insert into person
    values(new.idStudent, new.name, new.email, new.phone_number);
END */;;
DELIMITER ;
/*!50003 SET sql_mode              = @saved_sql_mode */ ;
/*!50003 SET character_set_client  = @saved_cs_client */ ;
/*!50003 SET character_set_results = @saved_cs_results */ ;
/*!50003 SET collation_connection  = @saved_col_connection */ ;

--
-- Dumping events for database 'mydb'
--

--
-- Dumping routines for database 'mydb'
--
/*!50003 DROP PROCEDURE IF EXISTS `deliveries_made` */;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8mb4 */ ;
/*!50003 SET character_set_results = utf8mb4 */ ;
/*!50003 SET collation_connection  = utf8mb4_0900_ai_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `deliveries_made`(IN i_driver_id INT)
    READS SQL DATA
BEGIN
	select drivers.name as Delivery_Driver, person.name as Client, idOrders
	from orders
	join drivers
	on orders.driver_id = drivers.idDrivers
	join person
	on orders.client_id = person.idPerson
	where driver_id = i_driver_id;
END ;;
DELIMITER ;
/*!50003 SET sql_mode              = @saved_sql_mode */ ;
/*!50003 SET character_set_client  = @saved_cs_client */ ;
/*!50003 SET character_set_results = @saved_cs_results */ ;
/*!50003 SET collation_connection  = @saved_col_connection */ ;
/*!50003 DROP PROCEDURE IF EXISTS `orders_by_restaurants` */;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8mb4 */ ;
/*!50003 SET character_set_results = utf8mb4 */ ;
/*!50003 SET collation_connection  = utf8mb4_0900_ai_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `orders_by_restaurants`()
BEGIN
	select restau.name, count(distinct restaurant_id) as Number_of_Orders
	from orders
	join restaurants as restau
	on orders.restaurant_id = restau.idRestaurants
	group by restau.name
	order by count(distinct restaurant_id);
    
END ;;
DELIMITER ;
/*!50003 SET sql_mode              = @saved_sql_mode */ ;
/*!50003 SET character_set_client  = @saved_cs_client */ ;
/*!50003 SET character_set_results = @saved_cs_results */ ;
/*!50003 SET collation_connection  = @saved_col_connection */ ;

--
-- Final view structure for view `orders_made`
--

/*!50001 DROP VIEW IF EXISTS `orders_made`*/;
/*!50001 SET @saved_cs_client          = @@character_set_client */;
/*!50001 SET @saved_cs_results         = @@character_set_results */;
/*!50001 SET @saved_col_connection     = @@collation_connection */;
/*!50001 SET character_set_client      = utf8mb4 */;
/*!50001 SET character_set_results     = utf8mb4 */;
/*!50001 SET collation_connection      = utf8mb4_0900_ai_ci */;
/*!50001 CREATE ALGORITHM=UNDEFINED */
/*!50013 DEFINER=`root`@`localhost` SQL SECURITY DEFINER */
/*!50001 VIEW `orders_made` AS select `person`.`name` AS `Client_Name`,`restaurants`.`name` AS `Restaurant`,`orders`.`price` AS `Total_Price` from ((`orders` join `person` on((`orders`.`client_id` = `person`.`idPerson`))) join `restaurants` on((`orders`.`restaurant_id` = `restaurants`.`idRestaurants`))) */;
/*!50001 SET character_set_client      = @saved_cs_client */;
/*!50001 SET character_set_results     = @saved_cs_results */;
/*!50001 SET collation_connection      = @saved_col_connection */;

--
-- Final view structure for view `person_info`
--

/*!50001 DROP VIEW IF EXISTS `person_info`*/;
/*!50001 SET @saved_cs_client          = @@character_set_client */;
/*!50001 SET @saved_cs_results         = @@character_set_results */;
/*!50001 SET @saved_col_connection     = @@collation_connection */;
/*!50001 SET character_set_client      = utf8mb4 */;
/*!50001 SET character_set_results     = utf8mb4 */;
/*!50001 SET collation_connection      = utf8mb4_0900_ai_ci */;
/*!50001 CREATE ALGORITHM=UNDEFINED */
/*!50013 DEFINER=`root`@`localhost` SQL SECURITY DEFINER */
/*!50001 VIEW `person_info` AS select `person`.`name` AS `name`,`person`.`email` AS `email` from `person` */;
/*!50001 SET character_set_client      = @saved_cs_client */;
/*!50001 SET character_set_results     = @saved_cs_results */;
/*!50001 SET collation_connection      = @saved_col_connection */;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2020-05-04 20:02:48

```
