# 資工三 110590029 陳思群
## 1.
### ( a )
```
UPDATE COURSE
SET CreditHour = 2
WHERE CourseName = 'Object-Oriented Programming' AND Department = 'EECS';
```
### ( b )
```
DELETE FROM STUDENT
WHERE Name = 'David' AND StudentNumber = '005';
```
### ( c )
```
SELECT DISTINCT C.CourseName
FROM COURSE C
JOIN SECTION S ON C.CourseNumber = S.CourseNumber
WHERE S.Instructor = 'John' AND S.Year IN (2022, 2023);
```
### ( d )
```
SELECT S.CourseNumber, S.Semester, S.Year, COUNT(G.StudentNumber) AS NumberOfStudents
FROM SECTION S
JOIN GRADE_REPORT G ON S.SectionNumber = G.SectionNumber
WHERE S.Instructor = 'Eric'
GROUP BY S.SectionNumber, S.CourseNumber, S.Semester, S.Year;
```
### ( e )
```
SELECT P.PrerequisiteCourseNumber, C.CourseName
FROM PREREQUISITE P
JOIN COURSE C ON P.PrerequisiteCourseNumber = C.CourseNumber
WHERE C.CourseName = 'Database Systems' AND C.Department = 'EECS';
```
### ( f )
```
SELECT S.Name, C.CourseNumber, C.CourseName, C.CreditHour, SECT.Semester, SECT.Year, G.Grade
FROM STUDENT S
JOIN GRADE_REPORT G ON S.StudentNumber = G.StudentNumber
JOIN SECTION SECT ON G.SectionNumber = SECT.SectionNumber
JOIN COURSE C ON SECT.CourseNumber = C.CourseNumber
WHERE S.Class = 3 AND S.Major = 'EECS';
```
### ( g )
```
SELECT DISTINCT S.Name
FROM STUDENT S
WHERE NOT EXISTS (
    SELECT *
    FROM GRADE_REPORT G
    WHERE S.StudentNumber = G.StudentNumber AND G.Grade < 80
);
```
### ( h )
```
SELECT S.Name, S.Major
FROM STUDENT S
WHERE NOT EXISTS (
    SELECT *
    FROM GRADE_REPORT G
    WHERE S.StudentNumber = G.StudentNumber AND G.Grade < 60
);
```
### ( i )
```
SELECT DISTINCT S.StudentNumber, S.Name, S.Major
FROM STUDENT S
JOIN GRADE_REPORT G ON S.StudentNumber = G.StudentNumber
WHERE G.Grade < 60
ORDER BY S.StudentNumber;
```
### ( j )
```
SELECT S.Name, AVG(G.Grade) AS AverageGrade
FROM STUDENT S
JOIN GRADE_REPORT G ON S.StudentNumber = G.StudentNumber
JOIN SECTION SECT ON G.SectionNumber = SECT.SectionNumber
WHERE SECT.Year = 2023
GROUP BY S.StudentNumber, S.Name
HAVING AVG(G.Grade) > 80.0;
```
### ( k )
```
SELECT S.Major, COUNT(*) AS NumberOfStudents
FROM STUDENT S
JOIN GRADE_REPORT G ON S.StudentNumber = G.StudentNumber
GROUP BY S.Major
HAVING AVG(G.Grade) < 60.0;
```
### ( l )
```
CREATE VIEW StudentCourseView AS
SELECT 
    S.StudentNumber,
    S.Name AS StudentName,
    C.CourseName,
    SECT.Semester,
    SECT.Year,
    G.Grade
FROM STUDENT S
JOIN GRADE_REPORT G ON S.StudentNumber = G.StudentNumber
JOIN SECTION SECT ON G.SectionNumber = SECT.SectionNumber
JOIN COURSE C ON SECT.CourseNumber = C.CourseNumber;
```
```
SELECT * FROM StudentCourseView
```
## 2.
### ( a )
```
CREATE TABLE STUDENT (
    StudentNumber VARCHAR(10) PRIMARY KEY,
    Name VARCHAR(50),
    Class INT,
    Major VARCHAR(50)
);

CREATE TABLE COURSE (
    CourseNumber VARCHAR(10) PRIMARY KEY,
    CourseName VARCHAR(100),
    CreditHour INT,
    Department VARCHAR(50)
);

CREATE TABLE SECTION (
    SectionNumber VARCHAR(10) PRIMARY KEY,
    CourseNumber VARCHAR(10),
    Semester VARCHAR(20),
    Year INT,
    Instructor VARCHAR(50),
    FOREIGN KEY (CourseNumber) REFERENCES COURSE(CourseNumber)
);

CREATE TABLE GRADE_REPORT (
    StudentNumber VARCHAR(10),
    SectionNumber VARCHAR(10),
    Grade INT,
    PRIMARY KEY (StudentNumber, SectionNumber),
    FOREIGN KEY (StudentNumber) REFERENCES STUDENT(StudentNumber),
    FOREIGN KEY (SectionNumber) REFERENCES SECTION(SectionNumber)
);

CREATE TABLE PREREQUISITE (
    CourseNumber VARCHAR(10),
    PrerequisiteCourseNumber VARCHAR(10),
    PRIMARY KEY (CourseNumber, PrerequisiteCourseNumber),
    FOREIGN KEY (CourseNumber) REFERENCES COURSE(CourseNumber),
    FOREIGN KEY (PrerequisiteCourseNumber) REFERENCES COURSE(CourseNumber)
);
```
### ( b )
```
INSERT INTO STUDENT VALUES
    ('001', 'Alice', 2, 'Computer Science'),
    ('002', 'Bob', 3, 'Electrical Engineering'),
    ('003', 'Charlie', 1, 'Mechanical Engineering');

INSERT INTO COURSE VALUES
    ('CS101', 'Introduction to Programming', 3, 'Computer Science'),
    ('EE201', 'Circuit Analysis', 4, 'Electrical Engineering'),
    ('ME301', 'Thermodynamics', 3, 'Mechanical Engineering');

INSERT INTO SECTION VALUES
    ('S101', 'CS101', 'Fall', 2022, 'John Doe'),
    ('S102', 'EE201', 'Spring', 2023, 'Jane Smith'),
    ('S103', 'ME301', 'Fall', 2022, 'John Doe');

INSERT INTO GRADE_REPORT VALUES
    ('001', 'S101', 85),
    ('002', 'S102', 72),
    ('003', 'S103', 60);

INSERT INTO PREREQUISITE VALUES
    ('CS101', 'ME301'),
    ('EE201', 'CS101'),
    ('ME301', 'EE201');
```
### ( c ) SQL queries from problem 1
### ( d )
```
DELIMITER //

CREATE PROCEDURE CalculateAverageGradeLetter(IN studentNum INT)
BEGIN
    DECLARE avgGrade FLOAT;
    DECLARE gradeLetter VARCHAR(10);

    -- Calculate average grade for the student
    SELECT AVG(Grade) INTO avgGrade
    FROM GRADE_REPORT
    WHERE StudentNumber = studentNum;

    -- Determine the grade letter
    IF avgGrade >= 60 THEN
        SET gradeLetter = 'PASS';
    ELSE
        SET gradeLetter = 'FAIL';
    END IF;

    -- Print the result
    SELECT CONCAT('Student ', studentNum, ' has an average grade of ', gradeLetter) AS Result;
    
END //

DELIMITER ;
```
```
-- Call the stored procedure with a specific student number (e.g., 101)
CALL CalculateAverageGradeLetter(101);
```
