-- database design and SQL example 

-- Designed and implemented a relational database for an animal shelter, optimizing data organization and facilitating analysis of animal adoption trends. Created multiple tables, including Description, Biology, and Dates, with specific attributes for tracking each animal’s characteristics and history. Imported raw data from a CSV file using SQL LOAD DATA INFILE, then structured it into normalized tables with appropriate primary and foreign keys to establish clear relationships. Developed SQL queries to generate meaningful insights, such as ranking animals by adoption percentage, identifying animals by weight, and filtering data by age and weight criteria. This project showcased skills in database design, data import, normalization, and advanced SQL querying for data analysis.

-- Drop and create the database
DROP DATABASE IF EXISTS Animal_Shelter;
CREATE DATABASE Animal_Shelter CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE Animal_Shelter;

-- Create the main table for storing raw data if you plan to load all columns first
CREATE TABLE Animal_Shelter_Data (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Animal VARCHAR(50),
    Status VARCHAR(20),
    Weight DECIMAL(5,2),
    Age INT,
    Size VARCHAR(10),
    Gender VARCHAR(10),
    Arrival_Date DATE,
    Adoption_Date DATE
);

-- Load the data from CSV into Animal_Shelter_Data table
LOAD DATA INFILE 'C:\\Users\\Peter Correa\\Desktop\\Animal_Shelter (1).csv'  --change to local file for user
INTO TABLE Animal_Shelter_Data
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n';

-- Create the Description table
CREATE TABLE Description (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Animal VARCHAR(50),
    Status VARCHAR(20)
);

-- Populate Description from the main data table
INSERT INTO Description (ID, Animal, Status)
SELECT ID, Animal, Status FROM Animal_Shelter_Data;

-- Create the Biology table
CREATE TABLE Biology (
    Animal_ID INT PRIMARY KEY,
    Weight DECIMAL(5,2),
    Age INT,
    Size VARCHAR(10),
    Gender VARCHAR(10),
    FOREIGN KEY (Animal_ID) REFERENCES Description(ID)
);

-- Populate Biology from the main data table
INSERT INTO Biology (Animal_ID, Weight, Age, Size, Gender)
SELECT ID, Weight, Age, Size, Gender FROM Animal_Shelter_Data;

-- Create the Dates table
CREATE TABLE Dates (
    Animal_ID INT PRIMARY KEY,
    Arrival_Date DATE,
    Adoption_Date DATE,
    FOREIGN KEY (Animal_ID) REFERENCES Description(ID)
);

-- Populate Dates from the main data table
INSERT INTO Dates (Animal_ID, Arrival_Date, Adoption_Date)
SELECT ID, Arrival_Date, Adoption_Date FROM Animal_Shelter_Data;

-- Query 1: Rank the top 5 animals with the highest adoption percentage per species
SELECT Animal, COUNT(*) AS Total_Animals,
    SUM(CASE WHEN Status = 'Adopted' THEN 1 ELSE 0 END) AS Total_Adopted,
    (SUM(CASE WHEN Status = 'Adopted' THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS Adoption_Percentage
FROM Description
GROUP BY Animal
ORDER BY Adoption_Percentage DESC
LIMIT 5;

-- Query 2: Display the 7 animals' ID and status with the lowest weight
SELECT d.ID, d.Status
FROM Description d
JOIN Biology b ON d.ID = b.Animal_ID
ORDER BY b.Weight ASC
LIMIT 7;

-- Query 3: Display the age, animal, and adoption status for animals with an age up to 2 years
SELECT b.Age, d.Animal, d.Status
FROM Description d
JOIN Biology b ON d.ID = b.Animal_ID
WHERE b.Age <= 2;

-- Query 4: Display the top 5 animals with age less than 2 and the highest weight
SELECT d.Animal, b.Age, b.Weight
FROM Description d
JOIN Biology b ON d.ID = b.Animal_ID
WHERE b.Age < 2
ORDER BY b.Weight DESC
LIMIT 5;
