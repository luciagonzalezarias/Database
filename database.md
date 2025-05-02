## Database Code
~~~
CREATE TABLE IF NOT EXISTS Users (
                    UserID INTEGER PRIMARY KEY AUTOINCREMENT,
                    Username TEXT NOT NULL UNIQUE,
                    Password TEXT NOT NULL,
                    Role TEXT NOT NULL CHECK (Role IN ('Admin', 'Lecturer', 'Student'))
                );

                CREATE TABLE IF NOT EXISTS Groups (
                    GroupID INTEGER PRIMARY KEY AUTOINCREMENT,
                    GroupName TEXT NOT NULL UNIQUE
                );

                CREATE TABLE IF NOT EXISTS Courses (
                    CourseID INTEGER PRIMARY KEY AUTOINCREMENT,
                    CourseName TEXT NOT NULL UNIQUE,
                    LecturerID INTEGER,
                    FOREIGN KEY (LecturerID) REFERENCES Users(UserID) ON DELETE SET NULL
                );

                CREATE TABLE IF NOT EXISTS GroupStudents (
                    GroupID INTEGER,
                    StudentID INTEGER,
                    PRIMARY KEY (GroupID, StudentID),
                    FOREIGN KEY (GroupID) REFERENCES Groups(GroupID) ON DELETE CASCADE,
                    FOREIGN KEY (StudentID) REFERENCES Users(UserID) ON DELETE CASCADE
                );

                CREATE TABLE IF NOT EXISTS CourseAssignments (
                    CourseID INTEGER,
                    StudentID INTEGER,
                    PRIMARY KEY (CourseID, StudentID),
                    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID) ON DELETE CASCADE,
                    FOREIGN KEY (StudentID) REFERENCES Users(UserID) ON DELETE CASCADE
                );

                CREATE TABLE IF NOT EXISTS Grades (
                    CourseID INTEGER NOT NULL,
                    StudentID INTEGER NOT NULL,
                    Grade REAL,
                    PRIMARY KEY (CourseID, StudentID),
                    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID) ON DELETE CASCADE,
                    FOREIGN KEY (StudentID) REFERENCES Users(UserID) ON DELETE CASCADE
                );
~~~~

## Inserts
~~~~
INSERT INTO Users (UserID, Username, Password, Role) VALUES
(1, 'AliceAdmin', '123456', 'Admin'),
(2, 'BobLecturer', '123456', 'Lecturer'),
(3, 'CarolLecturer', '123456', 'Lecturer'),
(4, 'DavidLecturer', '123456', 'Lecturer'),
(5, 'EmmaStudent', '123456', 'Student'),
(6, 'FrankStudent', '123456', 'Student'),
(7, 'GraceStudent', '123456', 'Student'),
(8, 'HankStudent', '123456', 'Student'),
(9, 'IvyStudent', '123456', 'Student'),
(10, 'JackStudent', '123456', 'Student'),
(11, 'KateStudent', '123456', 'Student'),
(12, 'EmmaJones', '123456', 'Student');

INSERT INTO Groups (GroupID, GroupName) VALUES
(1, 'Group A'),
(2, 'Group B'),
(3, 'Group C'),
(4, 'Group D'),
(5, 'Group E');

INSERT INTO Courses (CourseID, CourseName, LecturerID) VALUES
(1, 'Math', 2),
(2, 'Physics', 2),
(3, 'Chemistry', 2),
(4, 'History', NULL),
(5, 'Biology', 4),
(6, 'Art', 3),
(7, 'Literature', NULL),
(8, 'Philosophy', 4);      

INSERT INTO GroupStudents (GroupID, StudentID) VALUES
(1, 5), 
(1, 6),
(2, 7), 
(2, 8),
(3, 9),
(5, 10), 
(5, 11);

INSERT INTO CourseAssignments (CourseID, StudentID) VALUES
(1, 5), 
(1, 6),
(2, 5), 
(2, 7),
(3, 8),
(4, 9),
(5, 10),
(6, 10), 
(6, 11);

INSERT INTO Grades (CourseID, StudentID, Grade) VALUES
(1, 5, 9.5),
(1, 6, 6.0),
(2, 5, 4.0),
(2, 7, 7.0),
(3, 8, 10.0),
(5, 10, 5.5),
(6, 10, 3.0),
(6, 11, 6.5),
(1, 5, 8.5);
~~~~

## Querys
~~~~
-- 1. List all users
SELECT * 
FROM Users;

-- 2. List administrators only
SELECT UserID, Username 
FROM Users 
WHERE Role = 'Admin';

-- 3. List Lecturer only
SELECT UserID, Username 
FROM Users 
WHERE Role = 'Lecturer';

-- 4. List Students only
SELECT UserID, Username 
FROM Users 
WHERE Role = 'Student';

-- 5. Count the total number of users
SELECT COUNT(*) AS TotalUsers 
FROM Users;

-- 6. Count users by role
SELECT Role, COUNT(*) AS CountPerRole 
FROM Users 
GROUP BY Role;

-- 7. List all groups
SELECT * 
FROM Groups;

-- 8. List all Courses
SELECT * 
FROM Courses;

-- 9. Courses without a teacher assigned
SELECT CourseID, CourseName 
FROM Courses 
WHERE LecturerID IS NULL;

-- 10. Courses with a teacher assigned
SELECT CourseID, CourseName 
FROM Courses 
WHERE LecturerID IS NOT NULL;

-- 11. Courses together with the name of the teacher
SELECT c.CourseID, c.CourseName, u.Username AS LecturerName
FROM Courses AS c
JOIN Users AS u ON c.LecturerID = u.UserID;

-- 12. Students per group
SELECT g.GroupName, u.Username AS StudentName
FROM Groups AS g
JOIN GroupStudents AS gs ON g.GroupID = gs.GroupID
JOIN Users AS u ON gs.StudentID = u.UserID
ORDER BY g.GroupName;

-- 13. Groups of a specific student (e.g. UserID = 5)
SELECT u.Username, g.GroupName
FROM Users AS u
JOIN GroupStudents AS gs ON u.UserID = gs.StudentID
JOIN Groups AS g ON gs.GroupID  = g.GroupID
WHERE u.UserID = 5;

-- 14. Courses in which a student is enrolled (UserID = 8)
SELECT u.Username, c.CourseName
FROM Users AS u
JOIN CourseAssignments AS ca ON u.UserID = ca.StudentID
JOIN Courses AS c ON ca.CourseID = c.CourseID
WHERE u.UserID = 8;

-- 15. Number of students enrolled per year
SELECT c.CourseName, COUNT(ca.StudentID) AS StudentsEnrolled
FROM Courses AS c
LEFT JOIN CourseAssignments AS ca ON c.CourseID = ca.CourseID
GROUP BY c.CourseID;

-- 16. Number of students per group
SELECT g.GroupName, COUNT(gs.StudentID) AS NumStudents
FROM Groups AS g
LEFT JOIN GroupStudents AS gs ON g.GroupID = gs.GroupID
GROUP BY g.GroupID;

-- 17. Teachers and how many courses they teach
SELECT u.Username AS LecturerName, COUNT(c.CourseID) AS NumCourses
FROM Users AS u
LEFT JOIN Courses AS c ON u.UserID = c.LecturerID
WHERE u.Role = 'Lecturer'
GROUP BY u.UserID;

-- 18. Students and how many courses they are assigned to
SELECT u.Username AS StudentName, COUNT(ca.CourseID) AS NumCourses
FROM Users AS u
LEFT JOIN CourseAssignments AS ca ON u.UserID = ca.StudentID
WHERE u.Role = 'Student'
GROUP BY u.UserID;

-- 19. Students and how many groups they have
SELECT u.Username AS StudentName, COUNT(gs.GroupID) AS NumGroups
FROM Users AS u
LEFT JOIN GroupStudents AS gs ON u.UserID = gs.StudentID
WHERE u.Role = 'Student'
GROUP BY u.UserID;

-- 20. Also show whether or not the course has a teacher assigned to it.
SELECT 
c.CourseName,
COALESCE(u.Username, 'Unassigned') AS LecturerName,
CASE 
WHEN u.UserID IS NULL THEN 'No'
ELSE 'Yes'
END AS HasLecturer
FROM Courses AS c
LEFT JOIN Users AS u ON c.LecturerID = u.UserID;


-- 21. Average grade per year
SELECT c.CourseName, AVG(g.Grade) AS AvgGrade
FROM Grades  AS g
JOIN Courses AS c ON g.CourseID = c.CourseID
GROUP BY g.CourseID;

-- 22. Average grade per student
SELECT u.Username AS StudentName, AVG(g.Grade) AS AvgGrade
FROM Grades AS g
JOIN Users  AS u ON g.StudentID = u.UserID
GROUP BY g.StudentID;

-- 23. Maximum and minimum grade per course
SELECT c.CourseName,
MAX(g.Grade) AS MaxGrade,
MIN(g.Grade) AS MinGrade
FROM Grades  AS g
JOIN Courses AS c ON g.CourseID = c.CourseID
GROUP BY g.CourseID;

-- 24. Pass rate per course (grade ≥ 5)
SELECT c.CourseName,
SUM(CASE WHEN g.Grade >= 5 THEN 1 ELSE 0 END) * 1.0 / COUNT(*) AS PassRate
FROM Grades  AS g
JOIN Courses AS c ON g.CourseID = c.CourseID
GROUP BY g.CourseID;

-- 25. Courses with at least one registered mark
SELECT c.CourseName, COUNT(*) AS NumGrades
FROM Grades  AS g
JOIN Courses AS c ON g.CourseID = c.CourseID
GROUP BY g.CourseID
HAVING COUNT(*) > 0;

-- 26. Students with no marks recorded
SELECT u.Username AS StudentName
FROM Users AS u
WHERE u.Role = 'Student'
AND NOT EXISTS (
SELECT 1 FROM Grades g WHERE g.StudentID = u.UserID);

-- 27. Courses with no students enrolled
SELECT CourseName
FROM Courses
WHERE CourseID NOT IN (
SELECT CourseID FROM CourseAssignments
);

-- 28. Show all students with their courses and grades (if they have them).
SELECT u.Username, c.CourseName, g.Grade
FROM Users u
JOIN CourseAssignments ca ON u.UserID = ca.StudentID
JOIN Courses c ON ca.CourseID = c.CourseID
LEFT JOIN Grades g ON g.StudentID = u.UserID AND g.CourseID = c.CourseID
WHERE u.Role = 'Student';

-- 29. Courses with more than one student enrolled
SELECT c.CourseName, COUNT(ca.StudentID) AS TotalStudents
FROM Courses c
JOIN CourseAssignments ca ON c.CourseID = ca.CourseID
GROUP BY c.CourseID
HAVING COUNT(ca.StudentID) > 1;

-- 30. Show courses taught by each teacher with names
SELECT 
u.Username AS Lecturer, 
GROUP_CONCAT(c.CourseName, ', ') AS CoursesTaught
FROM Users u
LEFT JOIN Courses c ON u.UserID = c.LecturerID
WHERE u.Role = 'Lecturer'
GROUP BY u.UserID;

-- 31. Teachers teaching more than 2 courses
SELECT u.Username
FROM Users u
JOIN Courses c ON u.UserID = c.LecturerID
WHERE u.Role = 'Lecturer'
GROUP BY u.UserID
HAVING COUNT(*) > 2;

-- 32. Groups with more than one student
SELECT g.GroupName, COUNT(gs.StudentID) AS TotalStudents
FROM Groups g
JOIN GroupStudents gs ON g.GroupID = gs.GroupID
GROUP BY g.GroupID
HAVING COUNT(*) > 1;

-- 33. Students with the highest mark recorded (10)
SELECT DISTINCT u.Username
FROM Users u
JOIN Grades g ON u.UserID = g.StudentID
WHERE g.Grade = 10.0;

-- 34. Students and how many courses they have passed (grade ≥ 5)
SELECT u.Username, COUNT(*) AS PassedCourses
FROM Users u
JOIN Grades g ON u.UserID = g.StudentID
WHERE g.Grade >= 5
GROUP BY u.UserID;

-- 35. Students with all courses passed
SELECT u.Username
FROM Users u
JOIN Grades g ON u.UserID = g.StudentID
GROUP BY u.UserID
HAVING MIN(g.Grade) >= 5;

-- 36. Check if a user exists with a certain username (basic login)
SELECT UserID, Role 
FROM Users 
WHERE Username = 'EmmaJones';

-- 37. Get students not assigned to a particular group (e.g. GroupID = 1)
SELECT UserID, Username 
FROM Users 
WHERE Role = 'Student'
AND UserID NOT IN (
SELECT StudentID FROM GroupStudents WHERE GroupID = 1
);

-- 38. See if there are duplicate grades per student and course (completeness)
SELECT CourseID, StudentID, COUNT(*) 
FROM Grades 
GROUP BY CourseID, StudentID
HAVING COUNT(*) > 1;

-- 39. View students who have been assigned a course but have not yet received a grade
SELECT u.Username, c.CourseName
FROM CourseAssignments ca
JOIN Users u ON ca.StudentID = u.UserID
JOIN Courses c ON ca.CourseID = c.CourseID
LEFT JOIN Grades g 
ON g.CourseID = ca.CourseID AND g.StudentID = ca.StudentID
WHERE g.Grade IS NULL;

-- 40. Show the total number of grades recorded and the overall average.
SELECT COUNT(*) AS TotalGrades, ROUND(AVG(Grade), 2) AS GlobalAverage
FROM Grades;

-- 41. Show students sorted by total number of courses assigned (descending)
SELECT u.Username, COUNT(ca.CourseID) AS TotalCourses
FROM Users u
JOIN CourseAssignments ca ON u.UserID = ca.StudentID
WHERE u.Role = 'Student'
GROUP BY u.UserID
ORDER BY TotalCourses DESC;

-- 42. Show teachers with no active courses (no students enrolled in them)
SELECT DISTINCT u.Username AS Lecturer
FROM Users u
JOIN Courses c ON u.UserID = c.LecturerID
LEFT JOIN CourseAssignments ca ON c.CourseID = ca.CourseID
WHERE u.Role = 'Lecturer'
GROUP BY u.UserID, c.CourseID
HAVING COUNT(ca.StudentID) = 0;

-- 43. See which teachers are teaching courses with students with grades lower than 5
SELECT DISTINCT u.Username AS Lecturer
FROM Users u
JOIN Courses c ON u.UserID = c.LecturerID
JOIN Grades g ON c.CourseID = g.CourseID
WHERE g.Grade < 5;

-- 44. Pass rate per course (grade < 5)
SELECT 
c.CourseName,
SUM(CASE WHEN g.Grade < 5 THEN 1 ELSE 0 END) * 1.0 / COUNT(*) AS FailRate
FROM Grades AS g
JOIN Courses AS c ON g.CourseID = c.CourseID
GROUP BY g.CourseID;

-- 45. Show courses where all students have passed
SELECT c.CourseName
FROM Courses c
JOIN Grades g ON c.CourseID = g.CourseID
GROUP BY c.CourseID
HAVING MIN(g.Grade) >= 5;
~~~~
