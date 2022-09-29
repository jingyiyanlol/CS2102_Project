/* list of funcionalities in order from the pdf
Basic:                          |   1. Triggers used
1. add_department               |   -
2. remove_department            |   -
3. add_room                     |   -
4. change_capacity              |   1. UpdatesInsertCapTrigger execute CheckUpdateCapPreConstraint
                                |   2. DeleteAfterUpdateTrigger execute DeleteSessionAfterUpdate
5. add_employee                 |   1. JuniorTrigger execute CheckNonOverlapJunior
                                |   2. SeniorTrigger execute CheckNonOverlapSenior
                                |   3. ManagerTrigger execute CheckNonOverlapManager
                                |   4. BookerTrigger execute CheckIsSeniorManager
                                |   5. DeptTrigger execute CheckDeptIsNotNull
                                |   6. EmailTrigger execute AddEmail
6. remove_employee              |   1. ResignTrigger execute RemoveFutureMeetings
                                |   
Core:                           |
1. search_room                  |   -
2. book_room                    |   1. BookingRoomTrigger execute BookRoomConstraintsCheck
                                |   2. JoiningBookTrigger excute BookJoin
3. unbook_room                  |   1. UnbookTrigger execute CheckUnbookConstraints
4. join_meeting                 |   1. JoinMeetingTrigger execute JoinMeetingConstraintsCheck
5. leave_meeting                |   1. LeaveMeetingTrigger execute LeaveMeetingConstraintsCheck
6. approve_meeting              |   1. ApproveSessionTrigger execute CheckApproveSessionConstraint
                                |
Health:                         |
1. declare_health               |   1. DeclareTempPreTrigger execute TempConstraintsCheck
                                |   
2. contact_tracing              |   -
                                |
Admin:                          |
1. non_compliance               |   -
2. view_booking_report          |   -
3. view_future_meeting          |   -
4. view_manager_report          |   -
*/


DROP FUNCTION IF EXISTS 
-- Core
search_room,
-- Health
contact_tracing,
-- Admin
non_compliance, IsApproved, view_booking_report, view_future_meeting, view_manager_report;

DROP PROCEDURE IF EXISTS
-- Basic
add_department, remove_department, add_room, change_capacity, add_employee, remove_employee,
-- Core
book_room, unbook_room, join_meeting, leave_meeting, approve_meeting,
-- Health
declare_health;

-- Drop Triggers if exists
-- Basic
DROP TRIGGER IF EXISTS JuniorTrigger ON Junior;
DROP TRIGGER IF EXISTS SeniorTrigger ON Senior;
DROP TRIGGER IF EXISTS ManagerTrigger ON Manager;
DROP TRIGGER IF EXISTS UpdatesInsertCapTrigger ON Updates;
DROP TRIGGER IF EXISTS DeleteAfterUpdateTrigger ON Updates;
DROP TRIGGER IF EXISTS ResignTrigger ON Employees;
DROP TRIGGER IF EXISTS BookerTrigger ON Booker;
DROP TRIGGER IF EXISTS DeptTrigger ON Employees;
DROP TRIGGER IF EXISTS EmailTrigger ON Employees;
-- Core
DROP TRIGGER IF EXISTS BookingRoomTrigger ON BookSessions;
DROP TRIGGER IF EXISTS JoiningBookTrigger ON BookSessions;
DROP TRIGGER IF EXISTS UnbookTrigger ON BookSessions;
DROP TRIGGER IF EXISTS leavemeetingtrigger ON Joins;
DROP TRIGGER IF EXISTS JoinMeetingTrigger ON Joins;
DROP TRIGGER IF EXISTS ApproveSessionTrigger ON BookSessions;
-- Health
DROP TRIGGER IF EXISTS DeclareTempPreTrigger ON HealthDeclaration;


-- Trigger Functions
DROP FUNCTION IF EXISTS 
-- Basic
CheckUpdateCapPreConstraint, DeleteSessionAfterUpdate, CheckNonOverlapJunior, 
CheckNonOverlapSenior, CheckNonOverlapManager,CheckIsSeniorManager, 
CheckDeptIsNotNull, AddEmail, RemoveFutureMeetings,
-- Core
BookRoomConstraintsCheck, BookJoin, CheckUnbookConstraints,
JoinMeetingConstraintsCheck, LeaveMeetingConstraintsCheck, CheckApproveSessionConstraint,
-- Health
TempConstraintsCheck;


-- Helper Functions
DROP FUNCTION IF EXISTS
RoomUnavailable, CapOfRoom, MeetingPax, CheckFeverOnThatDay, 
ResignedEmployees, ApprovedMeeting, BookerUnjoin, MostRecentHealthDeclaration,
IsManager, SameDeptAsRoom, daysAftResign;

DROP VIEW IF EXISTS HealthDeclarationWFever;
DROP FUNCTION IF EXISTS ObtainFever;

-----------------------------------------------------------------------------------------------------------------
-- Helper function 1: RoomUnavailable
-- To check whether a room is available on a particular day and time period
-- Returns 1 if room is unavilable on the particular day and time period.
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION RoomUnavailable(qroom_num INTEGER, qfloor_num INTEGER,
    qdate DATE, qstarthour INTEGER, qendhour INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT 1 FROM BookSessions bs1
    -- room is unavailable on that day and time
    WHERE bs1.room_num = qroom_num
    AND bs1.floor_num = qfloor_num
    AND bs1.sdate = qdate
    AND bs1.stime >= qstarthour
    AND bs1.stime < qendhour;
$$ LANGUAGE sql;  


-----------------------------------------------------------------------------------------------------------------
-- Helper function 2: CapOfRoom
-- Returns capacity of a room on a particular date
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION CapOfRoom(qroom_num INTEGER, qfloor_num INTEGER,
    qdate DATE)
RETURNS TABLE(num INTEGER) AS $$
    SELECT u.new_cap
    FROM Updates u
    WHERE u.floor_num = qfloor_num
    AND u.room_num = qroom_num
    AND qdate > u.udate
    AND NOT EXISTS (SELECT 1 
                    FROM Updates u2
                    WHERE u.floor_num = u2.floor_num
                    AND u.room_num = u2.room_num
                    AND qdate > u2.udate
                    AND u2.udate > u.udate);
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 3: MeetingPax
-- Returns current attendees number for a particular meeting
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION MeetingPax(qroom_num INTEGER, qfloor_num INTEGER,
    qdate DATE, qhour INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT COUNT(*)
    FROM Joins j
    WHERE j.floor_num = qfloor_num
    AND j.room_num = qroom_num
    AND j.sdate = qdate
    AND j.stime = qhour;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 4: ResignedEmployees
-- Returns 1 if a employee has resigned
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION ResignedEmployees (IN empl_id INTEGER)
RETURNS Table (num INTEGER) AS $$
    SELECT 1 FROM Employees
    WHERE eid = empl_id AND resigned_date IS NOT NULL;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 5: ApprovedMeeting
-- Returns 1 if the meeting has already been approved
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION ApprovedMeeting(qroom_num INTEGER, qfloor_num INTEGER,
    qdate DATE, qhour INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT 1 FROM BookSessions bs1
    WHERE bs1.room_num = qroom_num
    AND bs1.floor_num = qfloor_num
    AND bs1.sdate = qdate
    AND bs1.stime = qhour
    AND bs1.mid IS NOT NULL;
$$ LANGUAGE sql;  


-----------------------------------------------------------------------------------------------------------------
-- Helper function 6: BookerUnjoin
-- Returns 1 if booker has unjoined his meeting 
----------------------------------------------------------------------------------------------------------------- 
CREATE OR REPLACE FUNCTION BookerUnjoin(qroom_num INTEGER, qfloor_num INTEGER,
    qdate DATE, qtime INTEGER, empl_id INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT 1 FROM
    BookSessions bs
    WHERE bs.room_num = qroom_num
    AND bs.floor_num = qfloor_num
    AND bs.sdate = qdate
    AND bs.stime = qtime
    AND bs.bid = empl_id;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 7: IsManager
-- Returns 1 if a manager
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION IsManager (qmid INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT 1 FROM Manager
    WHERE mid = qmid;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 8: SameDeptAsRoom
-- Returns 1 if manager same department as room
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION SameDeptAsRoom (qroom_num INTEGER, qfloor_num INTEGER, 
    qmid INTEGER)
RETURNS TABLE(num INTEGER) AS $$
    SELECT 1 FROM MeetingRooms mr
    WHERE mr.room_num = qroom_num 
    AND mr.floor_num = qfloor_num
    AND EXISTS (
                SELECT 1 FROM Employees e INNER JOIN Manager m ON e.eid = m.mid
                WHERE m.mid = qmid AND e.did = mr.did
    );
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 9: daysAftResign
-- Given a query date, returns days from resign date to queries date
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION daysAftResign (startdate DATE, enddate DATE, empl_id INTEGER)
RETURNS INTEGER AS $$
DECLARE
    resign_date DATE := NULL;
BEGIN
    -- we assume employee id exist. 
    SELECT resigned_date INTO resign_date FROM Employees WHERE eid = empl_id; 
    
    -- employee has not resigned yet
    IF (resign_date IS NULL) THEN
        RETURN 0;
    
    -- resign date is present

    -- case 1: employee has resigned before start date
    ELSIF (resign_date < startdate) THEN
        RETURN (enddate::DATE - startdate::DATE + 1);
    
    -- case 2: employee has resigned on start date
    ELSIF (resign_date = startdate) THEN
        RETURN (enddate::DATE - startdate::DATE);
    
    -- case 3: employee has resigned after start date but before end date
    ELSIF (resign_date > startdate AND resign_date < enddate) THEN
        RETURN (enddate::DATE - resign_date::DATE);

    -- case 4: employee has resigned on/after end date
    ELSIF (resign_date >= enddate) THEN
        RETURN 0;
    
    END IF;

END
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 10: HealthDeclarationWFever
-- Returns HealthDeclaration table with fever attribute
----------------------------------------------------------------------------------------------------------------- 
CREATE OR REPLACE FUNCTION ObtainFever(IN temp numeric)
RETURNS boolean AS $$
BEGIN
    IF (temp > 37.5) THEN
        RETURN true;
    ELSE 
        RETURN false;
    END IF;
END
$$ LANGUAGE plpgsql;

CREATE VIEW HealthDeclarationWFever AS
    SELECT hd.hdate AS hdate, hd.temp AS temp, ObtainFever(hd.temp) AS fever, hd.eid AS eid
    FROM HealthDeclaration hd;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 11: MostRecentHealthDeclaration
-- Returns 1 if most recent HealthDeclarationWFever indicates fever
-- Returns 0 if nvr declare before or most recent HealthDeclarationWFever does not indicates fever
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION MostRecentHealthDeclaration(IN empl_id INTEGER)
RETURNS TABLE (num INTEGER) AS $$
    SELECT 1
    FROM HealthDeclarationWFever hd
    WHERE hd.eid = empl_id
    AND hd.hdate >= (
        SELECT MAX(hd2.hdate)
        FROM HealthDeclarationWFever hd2
        WHERE hd2.eid = empl_id
    )
    AND hd.fever;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Helper function 12: CheckFeverOnThatDay
-- Returns 1 if having fever on that day OR never declare temperature on that day
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION CheckFeverOnThatDay (IN empl_id INTEGER)
RETURNS TABLE(num INTEGER) AS $$
DECLARE
    current_date DATE := CURRENT_DATE;
BEGIN
    RETURN QUERY SELECT DISTINCT 1 
    FROM HealthDeclarationWFever hd
    WHERE (hd.eid = empl_id AND hd.fever = true AND hd.hdate = current_date)
    OR NOT EXISTS (
                    SELECT 1 FROM HealthDeclarationWFever hd2
                    WHERE hd2.eid = empl_id
                    AND hd2.hdate = current_date
                );
END; 
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 1:  add_department
-- Constraints: None
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE PROCEDURE add_department
    (IN dept_id INTEGER, IN dept_name VARCHAR(50))
AS $$
    INSERT INTO Departments VALUES (dept_id, dept_name);
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 2:  remove_department
-- Constraints: None
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE PROCEDURE remove_department
    (IN dept_id INTEGER)
AS $$
    DELETE FROM Departments 
    WHERE did = dept_id;
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 3:  add_room 
-- Constraints: None
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE PROCEDURE add_room
    (IN room_no INTEGER, IN floor_no INTEGER, 
    IN room_name VARCHAR(50), IN dept_id INTEGER,
    IN room_cap INTEGER, IN update_date DATE)
AS $$
    INSERT INTO MeetingRooms VALUES (room_no, floor_no, room_name, dept_id);
    INSERT INTO Updates VALUES (update_date, room_no, floor_no, NULL, room_cap);
$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 4:  change_capacity
-- Constraints: 
-- C1: Check whether manager same dept as room
-- C2: Remove future meetings that exceed new cap, whether approved or not
-- C3: Manager has not resigned
-- C4: Only one entry per room per day. For other entries, simply update instead of insert
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION CheckUpdateCapPreConstraint()
RETURNS TRIGGER AS $$
BEGIN
    -- check resigned employee
    IF (EXISTS (SELECT * FROM ResignedEmployees(NEW.mid))) THEN
        RAISE EXCEPTION 'Resigned employees cannot change capacity of a room!';
        RETURN NULL;
    -- if first time insertion on Updates table
    ELSIF (NOT EXISTS(SELECT 1 FROM Updates u WHERE u.room_num = NEW.room_num AND u.floor_num = NEW.floor_num )) THEN
        RETURN NEW;
    -- check employee same department as room
    ELSIF ( (EXISTS(SELECT * FROM IsManager(NEW.mid))) AND (NOT EXISTS (SELECT * FROM SameDeptAsRoom(NEW.room_num, NEW.floor_num, NEW.mid))) ) THEN  
        RAISE EXCEPTION 'Only manager belonging to same department as a room can change its capacity!';
        RETURN NULL;
    ELSIF (NEW.mid IS NULL) THEN
        RAISE EXCEPTION 'Only managers can change capacity of the room!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;


-- trigger before update on Updates 
CREATE TRIGGER UpdatesInsertCapTrigger
BEFORE INSERT OR UPDATE ON Updates
FOR EACH ROW
EXECUTE FUNCTION CheckUpdateCapPreConstraint();


-- trigger after update on Updates
-- delete sessions that have capacity larger than new capacity
CREATE OR REPLACE FUNCTION DeleteSessionAfterUpdate()
RETURNS TRIGGER AS $$
BEGIN
    DELETE FROM BookSessions bs
    WHERE bs.sdate > NEW.udate
    AND bs.floor_num = NEW.floor_num
    AND bs.room_num = NEW.room_num
    AND NEW.new_cap < (SELECT * FROM MeetingPax(NEW.room_num, NEW.floor_num, bs.sdate, bs.stime));
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;


-- trigger after update on Updates
CREATE TRIGGER DeleteAfterUpdateTrigger
AFTER INSERT OR UPDATE ON Updates
FOR EACH ROW
EXECUTE FUNCTION DeleteSessionAfterUpdate();


-- change capacity of room where room is the same department as the manager
-- constraint 1: must be manager (trigger)
-- constraint 2: same dept as room (trigger)
-- constraint 3: not resigned (trigger)
-- note: can be having fever. no need to be same dept as booker. 
-- Procedure to update capacity
CREATE OR REPLACE PROCEDURE change_capacity
    (IN floor_no INTEGER, IN room_no INTEGER,
    IN new_room_cap INTEGER, IN update_date DATE,
    IN manager_id INTEGER)
AS $$
BEGIN
    -- if another entry for same day, same room exists
    IF (EXISTS(SELECT 1 FROM Updates u WHERE u.room_num = room_no AND u.floor_num = floor_no AND u.udate = update_date)) THEN
        UPDATE Updates SET mid = manager_id, new_cap = new_room_cap WHERE room_num = room_no AND floor_num = floor_no AND udate = update_date;
    ELSE
        INSERT INTO Updates VALUES (update_date, room_no, floor_no, manager_id, new_room_cap);  
    END IF;
END
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 5:  add_employee
-- Constraints: 
-- C1: generate unique id & email
-- C2: add them to respective subgroups
-----------------------------------------------------------------------------------------------------------------
-- Trigger function to check for overlap for Junior & not in Booker
CREATE OR REPLACE FUNCTION CheckNonOverlapJunior()
RETURNS TRIGGER 
AS $$
BEGIN 
    IF (EXISTS (SELECT 1 FROM Senior WHERE snid = NEW.jid)) THEN
        RAISE EXCEPTION 'Junior ID is present at Senior table!';
        RETURN NULL;
    ELSIF (EXISTS (SELECT 1 FROM Manager WHERE mid = NEW.jid)) THEN
        RAISE EXCEPTION 'Junior ID is present at Manager table!';
        RETURN NULL;
    ELSIF (EXISTS (SELECT 1 FROM Booker WHERE bid = NEW.jid)) THEN
        RAISE EXCEPTION 'Junior ID is present at Booker table!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check whether empl id exists in other subgroups before inserting on Junior
CREATE TRIGGER JuniorTrigger
    BEFORE INSERT ON Junior
    FOR EACH ROW
EXECUTE FUNCTION CheckNonOverlapJunior();

-- Trigger function to check for overlap for Senior
CREATE OR REPLACE FUNCTION CheckNonOverlapSenior()
RETURNS TRIGGER 
AS $$
BEGIN 
    IF (EXISTS (SELECT 1 FROM Junior WHERE jid = NEW.snid)) THEN
        RAISE EXCEPTION 'Senior ID is present at Junior table!';
        RETURN NULL;
    ELSIF (EXISTS (SELECT 1 FROM Manager WHERE mid = NEW.snid)) THEN
        RAISE EXCEPTION 'Senior ID is present at Manager table!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check whether empl id exists in other subgroups before inserting on Senior
CREATE TRIGGER SeniorTrigger
    BEFORE INSERT ON Senior
    FOR EACH ROW
EXECUTE FUNCTION CheckNonOverlapSenior();

-- Trigger function to check for overlap for Manager
CREATE OR REPLACE FUNCTION CheckNonOverlapManager()
RETURNS TRIGGER 
AS $$
BEGIN 
    IF (EXISTS (SELECT 1 FROM Junior WHERE jid = NEW.mid)) THEN
        RAISE EXCEPTION 'Manager ID is present at Junior table!';
        RETURN NULL;
    ELSIF (EXISTS (SELECT 1 FROM Senior WHERE snid = NEW.mid)) THEN
        RAISE EXCEPTION 'Manager ID is present at Senior table!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check whether empl id exists in other subgroups before inserting on Manager
CREATE TRIGGER ManagerTrigger
    BEFORE INSERT ON Manager
    FOR EACH ROW
EXECUTE FUNCTION CheckNonOverlapManager();

-- Trigger function to check that only senior & manager can be included in booker.
CREATE OR REPLACE FUNCTION CheckIsSeniorManager()
RETURNS TRIGGER 
AS $$
BEGIN 
    IF (EXISTS (SELECT 1 FROM Junior WHERE jid = NEW.bid)) THEN
        RAISE EXCEPTION 'ID is present at Junior table & is a Junior employee!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check that only senior & manager can be included in booker.
CREATE TRIGGER BookerTrigger
    BEFORE INSERT ON Booker
    FOR EACH ROW
EXECUTE FUNCTION CheckIsSeniorManager();

-- Trigger function to check department is not NULL when inserted.
CREATE OR REPLACE FUNCTION CheckDeptIsNotNull()
RETURNS TRIGGER 
AS $$
BEGIN 
    IF (NEW.did IS NULL) THEN
        RAISE EXCEPTION 'Department cannot be NULL';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check department is not NULL when inserted.
CREATE TRIGGER DeptTrigger
    BEFORE INSERT ON Employees
    FOR EACH ROW
EXECUTE FUNCTION CheckDeptIsNotNull();

-- Trigger function to check department is not NULL when inserted.
CREATE OR REPLACE FUNCTION AddEmail()
RETURNS TRIGGER 
AS $$
BEGIN 
    -- update email
    UPDATE Employees SET email = (select regexp_replace(NEW.ename, '([^a-zA-Z])', '', 'g')) || '_0' || NEW.eid || '@company2102.com' 
    WHERE eid = NEW.eid;
    RETURN NULL;

END;
$$ LANGUAGE plpgsql;

-- Trigger to check department is not NULL when inserted.
CREATE TRIGGER EmailTrigger
    AFTER INSERT ON Employees
    FOR EACH ROW
EXECUTE FUNCTION AddEmail();


-- Procedure: To add employee
CREATE OR REPLACE PROCEDURE add_employee
    (IN empl_name VARCHAR(50), IN empl_home_no INTEGER, IN empl_mobile_no INTEGER, 
    IN empl_office_no INTEGER, IN empl_kind VARCHAR(50), IN empl_dept_id INTEGER) AS $$
DECLARE
    generated_id INTEGER := 0;
BEGIN

    IF (empl_kind NOT IN ('Junior', 'Senior', 'Manager')) THEN
        RAISE EXCEPTION 'Employee need to be either Junior, Senior or Manager. Pls also ensure you spell them properly';
    END IF;

    INSERT INTO Employees (ename, home_contact, mobile_contact, office_contact, did) VALUES 
        (empl_name, empl_home_no, empl_mobile_no, empl_office_no, empl_dept_id) RETURNING eid INTO generated_id;
    
    
    -- add into respective subgroups
    IF empl_kind = 'Junior' THEN INSERT INTO Junior VALUES (generated_id);
    ELSIF empl_kind = 'Senior' THEN 
        INSERT INTO Booker VALUES (generated_id);
        INSERT INTO Senior VALUES (generated_id);
    ELSIF empl_kind = 'Manager' THEN 
        INSERT INTO Booker VALUES (generated_id);
        INSERT INTO Manager VALUES (generated_id);
    END IF;

    -- RAISE NOTICE 'Employee % has been added.', empl_name;
END;
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Basic 6:  remove_employee
-- Constraints: remove employee from future meetings (regardless pending or approved)
-- Assumption: remove the meetings booked by the employee but keep those approved by employee
-----------------------------------------------------------------------------------------------------------------
-- Trigger function to remove employee from BookSessions (if is booker) and Joins
CREATE OR REPLACE FUNCTION RemoveFutureMeetings()
RETURNS TRIGGER 
AS $$
BEGIN 

    -- update department to be null so that if later department gets removed, no issues.
    UPDATE Employees SET did = NULL WHERE eid = OLD.eid;

    -- remove the meetings booked by the employee but keep those approved by employee
    DELETE FROM BookSessions 
    WHERE bid = NEW.eid AND sdate > NEW.resigned_date;

    -- remove employee from future meetings (regardless pending or approved)
    DELETE FROM Joins
    WHERE eid = NEW.eid AND sdate > NEW.resigned_date;

    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger to remove employee from BookSessions (if is booker) and Joins
CREATE TRIGGER ResignTrigger
    AFTER UPDATE OF resigned_date ON Employees
    FOR EACH ROW
EXECUTE FUNCTION RemoveFutureMeetings();

-- Procedure: To remove employee who has resigned
CREATE OR REPLACE PROCEDURE remove_employee(
    IN emp_id INTEGER, IN res_date DATE
)
AS $$ 
BEGIN
    -- update Employees.resigned_date = res_date where eid = emp_id
    UPDATE Employees 
    SET resigned_date = res_date
    WHERE eid = emp_id;
    
    RAISE NOTICE 'Employee % has resigned on date %', emp_id, res_date;    
END;
$$ LANGUAGE plpgsql;



-----------------------------------------------------------------------------------------------------------------
-- Core 1:  search_room
-- Constraints:
-- C1: search capacity must be lower or equal to capacity of room on search date
-- C2: room is available on the search date and time
-- C3: order the results by asc 
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION search_room
    (IN capacity INTEGER, IN meeting_date DATE, IN shour INT, IN ehour INT)
RETURNS TABLE(floor_no INT, room_no INT, department_id INT, cap INT ) AS $$
BEGIN

    IF (ehour <= shour) THEN
        RAISE EXCEPTION 'Start hour must be lower than end hour';
    ELSIF (capacity < 1) THEN
        RAISE EXCEPTION 'Pls input a valid search capacity.';
    ELSIF (shour < 0 OR ehour > 24) THEN
        RAISE EXCEPTION 'Invalid hours given.';
    END IF;

    RETURN QUERY SELECT mr.floor_num, mr.room_num, mr.did, u1.new_cap
    FROM MeetingRooms mr, Updates u1
    WHERE NOT EXISTS ( -- check for constraint C2
        SELECT * FROM RoomUnavailable(mr.room_num, mr.floor_num, meeting_date, 
        shour, ehour)
        )
    AND u1.room_num = mr.room_num
    AND u1.floor_num = mr.floor_num
    AND u1.new_cap >= capacity
    AND u1.udate = 
    (
        SELECT MAX(u2.udate) -- check against the latest change to capacity before meeting date
        FROM Updates u2
        WHERE u2.room_num = mr.room_num
        AND u2.floor_num = mr.floor_num
        AND u2.udate < meeting_date -- must be at least the day before the meeting
    )
    ORDER BY u1.new_cap ASC;
END
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Core 2:  book_room
-- Constraints:
-- C1: Only senior or manager can book a room (checked by table)
-- C2: Person booking the room is not having fever
-- (Note: for contact tracing, as long as not having fever, can still book)
-- C3: Person booking the room has not resigned
-- Note: If room is unavailable even for partial period, STRICTLY disallow the meeting
-- Note: Booker can be different dept as room
-- Note: Booker can book more than 1 meeting room at the same day and time. We assume the booker rotate between meetings.
-----------------------------------------------------------------------------------------------------------------
-- Trigger function to check for book room constraints
CREATE OR REPLACE FUNCTION BookRoomConstraintsCheck()
RETURNS TRIGGER 
AS $$
DECLARE
    current_date DATE := CURRENT_DATE;
BEGIN
    IF NEW.sdate < current_date THEN
        RAISE EXCEPTION 'Booking cannot be made in the past';
        RETURN NULL;
    -- Check if employee have resigned
    ELSIF EXISTS (SELECT * FROM ResignedEmployees(NEW.bid)) THEN
        RAISE EXCEPTION 'Employee % has resigned, cannot book any meetings.', NEW.bid;
        RETURN NULL;
    -- Check if employee have fever
    ELSIF EXISTS (SELECT * FROM CheckFeverOnThatDay(NEW.bid)) THEN
        RAISE EXCEPTION 'Employee is having a fever or have not declared health, cannot book meeting';
        RETURN NULL;
    ELSE
        -- RAISE NOTICE 'Room has been booked';   
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check for book room constraints
CREATE TRIGGER BookingRoomTrigger
    BEFORE INSERT ON BookSessions
    FOR EACH ROW
EXECUTE FUNCTION BookRoomConstraintsCheck();

-- Procedure: to book a room
CREATE OR REPLACE PROCEDURE book_room
    (IN floor_no INT, IN room_no INT, IN meeting_date DATE, IN shour INT, IN ehour INT, IN employee_id INT)
AS $$
DECLARE
    temp INT := 0;
BEGIN
    temp := shour;
    IF (ehour <= shour) THEN
        RAISE EXCEPTION 'End hour cannot be equal or before start hour of meeting.';
    ELSIF (shour < 0 OR ehour > 24) THEN
        RAISE EXCEPTION 'Invalid hours given.';
    ELSE 
        IF EXISTS (
            SELECT * FROM RoomUnavailable(room_no, floor_no, meeting_date, 
            shour, ehour)
            ) THEN 
            RAISE EXCEPTION 'Room not available for some/whole duration. Pls check for availability first!';
        ELSE
            WHILE temp != ehour LOOP
                INSERT INTO BookSessions VALUES(temp, meeting_date, room_no, floor_no, employee_id);
                temp := temp + 1;
            END LOOP;
        END IF;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Post Trigger function to add booker to meeting
CREATE OR REPLACE FUNCTION BookJoin()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO Joins VALUES(NEW.bid, NEW.stime, NEW.sdate, NEW.room_num, NEW.floor_num);
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Post Trigger to add booker to meeting
CREATE TRIGGER JoiningBookTrigger
AFTER INSERT ON BookSessions
FOR EACH ROW EXECUTE FUNCTION BookJoin();


-----------------------------------------------------------------------------------------------------------------
-- Core 3:  unbook_room
-- Constraints: 
-- C1: Delete even if approved but haven't been conducted yet
-- C2: If meeting has been conducted, cannot unbook (if not screw contact tracing) 
-- C3: Only booker can delete his/her own meetings
-----------------------------------------------------------------------------------------------------------------
-- Trigger function to check whether meeting has been conducted
CREATE OR REPLACE FUNCTION CheckUnbookConstraints()
RETURNS TRIGGER 
AS $$
 DECLARE
    current_date DATE := CURRENT_DATE;
BEGIN
    IF (OLD.sdate < current_date AND OLD.mid IS NOT NULL) THEN
        RAISE EXCEPTION 'Some or all the book session that you want to unbook has been conducted already';
        RETURN NULL;
    END IF; return OLD;
END;
$$ LANGUAGE plpgsql;

-- Trigger to check whether meeting has been conducted
CREATE TRIGGER UnbookTrigger
    BEFORE DELETE ON BookSessions
    FOR EACH ROW
EXECUTE FUNCTION CheckUnbookConstraints();

CREATE OR REPLACE PROCEDURE unbook_room
    (IN floor_no INT, IN room_no INT, IN meeting_date DATE, IN shour INT, IN ehour INT, IN employee_id INT)
AS $$
BEGIN
    IF (ehour <= shour) THEN
        RAISE EXCEPTION 'End hour cannot be equal or before start hour of meeting.';
    ELSIF (shour < 0 OR ehour > 24) THEN
        RAISE EXCEPTION 'Invalid hours given.';
    ELSE
        DELETE FROM BookSessions bs 
        WHERE bs.bid = employee_id
        AND bs.room_num = room_no
        AND bs.floor_num = floor_no
        AND bs.sdate = meeting_date
        AND bs.stime >= shour
        AND bs.stime < ehour;
    END IF;
END;
$$ LANGUAGE plpgsql;

--s1: slot is not present
--s2: slot present, booked by p1  -> yesterday and approved
--s3: slot present, booked NOT by p1


-----------------------------------------------------------------------------------------------------------------
-- Core 4:  join_meeting
-- Constraints: 
-- C1: Employee has not resigned
-- C2: Employee does not have a fever
-- C3: Meeting needs to exist
-- C4: Booking status is still pending
-- C5: Meeting Room Capacity has not been reached
-- Note: We do not check whether employee has other meetings at that time.
-----------------------------------------------------------------------------------------------------------------
-- Function that checks contraints to joining a meeting
CREATE OR REPLACE FUNCTION JoinMeetingConstraintsCheck()
RETURNS TRIGGER 
AS $$
BEGIN
    -- C1: Employee has not resigned
    -- If ResignedEmployees(eid) returns 1, do nothing
    IF EXISTS (SELECT * FROM ResignedEmployees(NEW.eid)) THEN
        RAISE EXCEPTION 'Employee % has resigned, cannot join any meetings.', NEW.eid;
        RETURN NULL;
    
    -- C2: Employee does not have a fever
    -- If MostRecentHealthDeclaration(eid) returns 1, do nothing
    ELSIF EXISTS (SELECT * FROM CheckFeverOnThatDay(NEW.eid)) THEN
        RAISE EXCEPTION 'Employee is having a fever or have not declared health, cannot join meeting';
        RETURN NULL;

    ELSEIF (NEW.sdate < CURRENT_DATE) THEN
        RAISE EXCEPTION 'One cannot join meetings in the past!';
        RETURN NULL;

    -- check whether meeting has been approved
    ELSIF EXISTS (SELECT * FROM ApprovedMeeting(NEW.room_num, NEW.floor_num, NEW.sdate, NEW.stime)) THEN
        RAISE NOTICE 'Meeting has already been approved from %:00 to %:00', NEW.stime, NEW.stime+1;
        RETURN NULL;

    -- check whether still have capacity
    ELSIF ((SELECT * FROM MeetingPax(NEW.room_num, NEW.floor_num, NEW.sdate, NEW.stime)) >= 
        (SELECT * FROM CapOfRoom(NEW.room_num, NEW.floor_num, NEW.sdate))) THEN
        RAISE NOTICE 'Meeting has reached full capacity, cannot join meeting at time %:00.', NEW.stime;
        RETURN NULL;
        
    ELSE
        RAISE NOTICE 'Employee % is added into meeting room #%-% on date % from %:00 to %:00.', 
        NEW.eid, NEW.floor_num, NEW.room_num, NEW.sdate, NEW.stime, NEW.stime+1;   
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Trigger that checks the join constraints using JoinMeetingConstraintsCheck before each INSERT
CREATE TRIGGER JoinMeetingTrigger
    BEFORE INSERT ON Joins
    FOR EACH ROW
EXECUTE FUNCTION JoinMeetingConstraintsCheck();

-- Procedure: For each hour within the start and end time where the employee can join a meeting, we just insert them into joins.
CREATE OR REPLACE PROCEDURE join_meeting(
    IN floor_n INTEGER, IN room_n INTEGER, 
    IN meet_date DATE, IN start_time INTEGER,
    IN end_time INTEGER, IN emp_id INTEGER
)
AS $$ 
DECLARE
    curr_time INTEGER := 0;
BEGIN
    curr_time := start_time;
    -- If end_time <= start_time, raise exception
    IF (end_time <= start_time) THEN
        RAISE EXCEPTION 'End hour cannot be equal or before start hour of meeting.';
    ELSIF (start_time < 0 OR end_time > 24) THEN
        RAISE EXCEPTION 'Invalid hours given.';

    ELSE
        WHILE (curr_time <= end_time - 1) LOOP
            RAISE NOTICE 'start time: %:00', curr_time;
            -- C3: Meeting needs to exist (back up if foreign key constraint fails)
            -- If Meeting does not exists, do nothing
            IF EXISTS ( SELECT 1 FROM BookSessions bs WHERE bs.floor_num = floor_n
                        AND bs.room_num = room_n AND bs.sdate = meet_date AND bs.stime = curr_time
                    ) THEN

                INSERT INTO Joins VALUES (emp_id, curr_time, meet_date, room_n, floor_n);
                    
            ELSE
                RAISE NOTICE 'Booking does not exists at time %:00.', curr_time;
            END IF;

            curr_time := curr_time + 1;

        END LOOP;    
    END IF;    
END;
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Core 5:  leave_meeting
-- Constraints:
-- C1: Employee must be in the meeting
-- C2: Meeting status still pending
-- C3/Assumption: Total participants reach 0 or Booker leaves meeting, booking deleted
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE PROCEDURE leave_meeting(
    IN floor_n INTEGER, IN room_n INTEGER, 
    IN meet_date DATE, IN start_time INTEGER,
    IN end_time INTEGER, IN emp_id INTEGER
)

AS $$
    DECLARE 
        curr_time INTEGER := 0;
        
    BEGIN
        curr_time := start_time;
        -- If end_time <= start_time, raise exception
        IF (end_time <= start_time) THEN
        RAISE EXCEPTION 'End hour cannot be equal or before start hour of meeting.';

        ELSIF (start_time < 0 OR end_time > 24) THEN
            RAISE EXCEPTION 'Invalid hours given.';

        ELSIF  (meet_date < CURRENT_DATE) THEN
            RAISE EXCEPTION 'One cannot unjoin meetings in the past!';
            
        ELSE
            WHILE (curr_time <= end_time - 1) LOOP
                RAISE NOTICE 'start time: %:00', curr_time;
                -- C1: Employee must be in the meeting (Check before DELETE ON Joins)
                -- If employee not even in meeting, do nothing
                IF EXISTS (SELECT 1 FROM JOINS j WHERE floor_n = j.floor_num AND room_n = j.room_num
                AND meet_date = j.sdate AND curr_time = j.stime AND emp_id = j.eid) THEN

                    -- C2: Meeting status still pending (Check before DELETE ON Joins)
                    -- If meeting is already approved, do nothing
                    IF NOT EXISTS (SELECT * FROM ApprovedMeeting(room_n, floor_n, meet_date, curr_time)) THEN
                        -- Then delete the employee from the meeting in Joins, use after trigger to check C3 and C4
                        RAISE NOTICE 'Employee % is removed from meeting on % at meeting room #%-% from %:00 to %:00', 
                        emp_id, meet_date, floor_n, room_n, curr_time, curr_time+1;
                        DELETE FROM Joins WHERE floor_n = floor_num AND room_n = room_num
                        AND meet_date = sdate AND curr_time = stime AND emp_id = eid;
                        
                    ELSE
                        RAISE NOTICE 'Room is already approved from %:00 to %:00, employee cannot leave meeting', 
                        curr_time, curr_time+1;
                    END IF;

                ELSE
                    RAISE NOTICE 'Employee not in meeting from %:00 to %:00 or meeting does not exists', 
                    curr_time, curr_time+1;
                END IF;     

                curr_time := curr_time + 1;
                
            END LOOP;    
        END IF;    
    END;
$$ LANGUAGE plpgsql;


-- Assumption: Booker leaves meeting, booking deleted 
-- Function that removes bookings from BookSessions after checking that the employee leaving is the booker
CREATE OR REPLACE FUNCTION LeaveMeetingConstraintsCheck()
RETURNS TRIGGER AS $$
BEGIN
    -- C3: Booker leaves meeting, booking deleted (AFTER DELETE ON Joins)
    -- OR Total participants reach 0 (will only happen when all members leave and booker last one to leave)
    IF (EXISTS (SELECT * FROM BookerUnjoin(OLD.room_num, OLD.floor_num, OLD.sdate, OLD.stime, OLD.eid))) THEN
        RAISE NOTICE 'Booker has left meeting, booking will be deleted.';
        DELETE FROM BookSessions bs WHERE OLD.floor_num = bs.floor_num AND OLD.room_num = bs.room_num 
        AND OLD.sdate = bs.sdate AND OLD.stime = bs.stime;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger that removes meeting from BookSessions when the booker leaves meeting or total part reach 0
CREATE TRIGGER LeaveMeetingTrigger
AFTER DELETE ON Joins
FOR EACH ROW
EXECUTE FUNCTION LeaveMeetingConstraintsCheck();


-----------------------------------------------------------------------------------------------------------------
-- Core 6:  approve_meeting
-- Constraints:
-- C1: must be manager (trigger)
-- C2: same dept as room (trigger)
-- C3: not resigned (trigger)
-- note: can be having fever. no need to be same dept as booker. 
-----------------------------------------------------------------------------------------------------------------
-- triggering function before update on BookSessions
-- checks whether approver is manager, not resigned and same dept as room
CREATE OR REPLACE FUNCTION CheckApproveSessionConstraint()
RETURNS TRIGGER AS $$
BEGIN
    -- check employee is a manager
    IF (NOT EXISTS(SELECT * FROM IsManager(NEW.mid))) THEN 
        RAISE EXCEPTION 'Employee is not a manager! Employee cannot approve a meeting!';
        RETURN NULL;
    -- check whether approver still currently working
    ELSIF (EXISTS (SELECT * FROM ResignedEmployees(NEW.mid))) THEN
        RAISE EXCEPTION 'Resigned employees cannot approve a meeting!';
        RETURN NULL;
    -- check whether approver same department as room
    ELSIF ((NOT EXISTS (SELECT * FROM SameDeptAsRoom(NEW.room_num, NEW.floor_num, NEW.mid)))) THEN
        RAISE EXCEPTION 'Only manager belonging to the same department as the room can approve the meeting!';
        RETURN NULL;
    ELSEIF (NEW.sdate < CURRENT_DATE) THEN
        RAISE EXCEPTION 'One cannot approve meetings in the past!';
        RETURN NULL;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- trigger before update on BookSessions
CREATE TRIGGER ApproveSessionTrigger
BEFORE UPDATE OF mid ON BookSessions 
FOR EACH ROW
EXECUTE FUNCTION CheckApproveSessionConstraint();


-- procedure to approve meeting where room is the same department as manager
CREATE OR REPLACE PROCEDURE approve_meeting
    (IN floor_no INT, IN room_no INT, 
    IN meeting_date DATE, IN start_hour INT, 
    IN end_hour INT, IN empl_id INT, IN approved boolean)
AS $$
DECLARE
    entry_start_hour INTEGER := start_hour;
BEGIN
    
    IF (start_hour < end_hour AND start_hour >= 0 AND end_hour <= 24)
    THEN
        WHILE entry_start_hour < end_hour LOOP
            IF (approved = true)
            THEN
                IF NOT EXISTS (SELECT * FROM ApprovedMeeting(room_no, floor_no, meeting_date, entry_start_hour)) 
                THEN
                    UPDATE BookSessions bs
                    SET mid = empl_id
                    WHERE bs.floor_num = floor_no
                    AND bs.room_num = room_no
                    AND bs.sdate = meeting_date
                    AND bs.stime = entry_start_hour
                    AND bs.mid IS NULL; 
                END IF;

            ELSE
                IF (meeting_date < CURRENT_DATE) THEN
                    RAISE NOTICE 'Cannot disapprove meeting at %:00 to %:00 because cannot disapprove previous day meetings', entry_start_hour, entry_start_hour + 1;
                ELSIF (EXISTS(SELECT * FROM IsManager(empl_id))) THEN 
                    IF (NOT EXISTS(SELECT * FROM ResignedEmployees(empl_id))) THEN    
                        IF (EXISTS(SELECT * FROM SameDeptAsRoom(room_no, floor_no, empl_id))) THEN 
                            DELETE FROM BookSessions bs
                            WHERE bs.floor_num = floor_no
                            AND bs.room_num = room_no
                            AND bs.sdate = meeting_date
                            AND bs.stime = entry_start_hour
                            AND (bs.mid IS NULL OR bs.mid = empl_id); -- unapprove pending or own approved one
                        
                        ELSE 
                            RAISE NOTICE 'Cannot disapprove meeting at %:00 to %:00 because need to be same department as room', entry_start_hour, entry_start_hour + 1;
                        END IF;
                    ELSE
                        RAISE NOTICE 'Cannot disapprove meeting at %:00 to %:00 because employee cannot be resigned', entry_start_hour, entry_start_hour + 1;
                    END IF;
                ELSE 
                    RAISE NOTICE 'Cannot disapprove meeting at %:00 to %:00 because need to be manager', 
                    entry_start_hour, entry_start_hour + 1;
                END IF;
            END IF;
            entry_start_hour := entry_start_hour + 1;
        END LOOP;
    ELSE 
        RAISE NOTICE 'Invalid hours';
    END IF;
END;
$$ LANGUAGE plpgsql;



-----------------------------------------------------------------------------------------------------------------
-- Health 1:  declare_health
-- Constraints: If resigned, can only declare temperature for before resigned date
-- Pre-trigger to check the constraint
-- Post-trigger to add the fever attribute (to prevent people from cheating the system)
-----------------------------------------------------------------------------------------------------------------
-- Pre-trigger function to ensure resigned employee declare temp before resigned date
CREATE OR REPLACE FUNCTION TempConstraintsCheck()
RETURNS TRIGGER 
AS $$
BEGIN
    IF EXISTS (SELECT * FROM ResignedEmployees(NEW.eid)) THEN
        IF (NEW.hdate > (SELECT resigned_date FROM Employees WHERE eid = NEW.eid)) THEN
            RAISE EXCEPTION 'For resigned employee, you can declare temperature for days before/on resign date!';
            RETURN NULL;
        END IF; RETURN NEW;
    END IF; RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Pre-trigger to ensure resigned employee declare temp before resigned date
CREATE TRIGGER DeclareTempPreTrigger
BEFORE INSERT ON HealthDeclaration
FOR EACH ROW
EXECUTE FUNCTION TempConstraintsCheck();

-- Procedure to declare temperature
CREATE OR REPLACE PROCEDURE declare_health
    (IN empl_id INTEGER, IN declare_date DATE, IN temp numeric) AS $$
BEGIN
    INSERT INTO HealthDeclaration VALUES (declare_date, temp, empl_id);
END;
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Health 1:  contact_tracing
-- Constraints:
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION contact_tracing (IN empl_id INTEGER, IN qdate Date)
RETURNS Table (Employee_ID INTEGER) AS $$
BEGIN
    -- no entry on that day
    IF (NOT EXISTS (SELECT 1 FROM HealthDeclarationWFever WHERE eid = empl_id AND hdate = qdate)) THEN
        RAISE NOTICE 'Employee has no health declaration entry on that day';
    -- no fever
    ELSIF (EXISTS (SELECT 1 FROM HealthDeclarationWFever WHERE eid = empl_id AND hdate = qdate AND fever = false)) THEN
        RAISE NOTICE 'Employee has no fever on that day';
    -- having fever
    ELSIF (EXISTS (SELECT 1 FROM HealthDeclarationWFever WHERE eid = empl_id AND hdate = qdate AND fever = true)) THEN
        -- Employee has fever. Remove him from all future meetings 
        
        -- remove the meetings booked by the employee but keep those approved by employee
        DELETE FROM BookSessions 
            WHERE bid = empl_id AND sdate >= qdate;

        -- remove employee from future meetings (regardless pending or approved)
        DELETE FROM Joins
            WHERE eid = empl_id AND sdate >= qdate;

        -- remove future meetings of close contacts
        -- remove booked meetings
        DELETE FROM BookSessions 
            WHERE bid IN (SELECT * FROM CloseContact(empl_id, qdate)) 
            AND sdate >= qdate AND sdate < qdate + 8;

        -- remove employee from future meetings (regardless pending or approved)
        DELETE FROM Joins
            WHERE eid IN (SELECT * FROM CloseContact(empl_id, qdate)) 
            AND sdate >= qdate AND sdate < qdate + 8;    

        RETURN QUERY SELECT * FROM CloseContact(empl_id, qdate);
    END IF;
END
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION CloseContact (IN empl_id INTEGER, IN qdate Date)
RETURNS Table (Employee_ID INTEGER) AS $$
DECLARE
    having_fever boolean := false;
BEGIN
    RETURN QUERY SELECT DISTINCT j.eid
    FROM BookSessions bs INNER JOIN Joins j
    ON bs.room_num = j.room_num
    AND bs.floor_num = j.floor_num
    AND bs.sdate = j.sdate
    AND bs.stime = j.stime
    WHERE bs.mid IS NOT NULL --approved meetings
    AND j.eid <> empl_id -- not same as queries employee
    AND j.sdate BETWEEN (qdate - 3) AND (qdate - 1) -- current date is future
    -- check whether queried employee has fever
    AND EXISTS (    
                SELECT 1 FROM HealthDeclarationWFever WHERE eid = empl_id AND hdate = qdate AND fever = true
                )
    -- check whether queryed employee has visited same meeting
    AND EXISTS (    
                SELECT 1 FROM Joins j2 
                WHERE j2.room_num = j.room_num
                AND j2.floor_num = j.floor_num
                AND j2.sdate = j.sdate
                AND j2.stime = j.stime
                AND j2.eid = empl_id
    );
END
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Admin 1:  non_compliance
-- Constraints:
-- C1: never declare at least once
-- C2: inclusive of start date and end date
-- C3: output no of days never declare
-- C4: descending order of number of days.
-- C5: for resigned, exclude the days after they resigned
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION non_compliance
    (IN startdate DATE, IN enddate DATE)
RETURNS TABLE(Employee_ID INT, Number_of_days INT)
AS $$
    SELECT e.eid as empl_id, ((enddate::DATE - startdate::DATE + 1) - 
                            daysAftResign(startdate, enddate, e.eid) - (SELECT COUNT(*)
                                                                    FROM HealthDeclarationWFever h
                                                                    WHERE e.eid = h.eid
                                                                    AND h.hdate >= startdate
                                                                    AND h.hdate <= enddate)) as num_days
    FROM Employees e
    WHERE (SELECT COUNT(*) -- how many entries they have
            FROM HealthDeclarationWFever h
            WHERE e.eid = h.eid
            AND h.hdate >= startdate
            AND h.hdate <= enddate) < (enddate::DATE - startdate::DATE + 1 - daysAftResign(startdate, enddate, e.eid))
    ORDER BY num_days DESC

$$ LANGUAGE sql;




-----------------------------------------------------------------------------------------------------------------
-- Admin 2:  view_booking_report 
-- Constraints:
-- C1: For junior employee, return nothing
-- C2: Don't need to care for resigned employees (cuz future meetings get deleted)
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION IsApproved
    (IN manager_id INT)
RETURNS VARCHAR(10) AS $$
    SELECT CASE
        WHEN manager_id IS NULL THEN 'Pending'
        ELSE 'Approved'
    END;
$$ LANGUAGE sql;

CREATE OR REPLACE FUNCTION view_booking_report
    (IN start_date DATE, IN employee_id INTEGER)
RETURNS TABLE(Floor_Number INT, Room_Number INT, date DATE, Start_Hour INT, Is_Approval VARCHAR(10)) AS $$
    SELECT bs.floor_num , bs.room_num, bs.sdate, bs.stime, IsApproved(bs.mid)
    FROM BookSessions bs
    WHERE bs.bid = employee_id
    AND start_date <= bs.sdate
    ORDER BY 
        bs.sdate ASC,
        bs.stime ASC;
    ;

$$ LANGUAGE sql;


-----------------------------------------------------------------------------------------------------------------
-- Admin 3:  view_future_meeting
-- Constraints: 
-- C1: Only meetings that are approved.
-- C2: Don't need to care about resigned employees. Will return nothing or meetings left bef resign date. 
-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION view_future_meeting(
    IN start_date DATE, IN emp_id INTEGER)
RETURNS TABLE(
    Floor_number INT, Room_number INT, Meeting_date DATE, Start_hour INT)
AS $$
BEGIN
    RETURN QUERY SELECT bs.floor_num , bs.room_num, bs.sdate, bs.stime
    FROM BookSessions bs, Joins j 
    WHERE bs.sdate >= start_date AND bs.sdate = j.sdate 
    AND j.eid = emp_id AND bs.stime = j.stime 
    AND bs.floor_num = j.floor_num AND bs.room_num = j.room_num
    AND bs.mid IS NOT NULL
    ORDER BY bs.sdate ASC, bs.stime ASC;
END;
$$ LANGUAGE plpgsql;


-----------------------------------------------------------------------------------------------------------------
-- Admin 4:  view_manager_report
-- Constraints:
-- C1: check whether manager
-- C2: output booked but not approved rooms 
-- C3: booked rooms from date onwards
-- C4: room must be same dept as manager
-- C5: ascending order of date and start hour.
-- For resigned employees, this will return nothing, which we can assume is ok. 

-----------------------------------------------------------------------------------------------------------------
CREATE OR REPLACE FUNCTION view_manager_report
    (IN startdate DATE, IN empl_id INTEGER)
RETURNS TABLE(Floor_Numbner INT, Room_Number INT, 
    Meeting_Date DATE, Start_Hour INT, Employee_ID INT)
AS $$
BEGIN
    IF (EXISTS(SELECT * FROM IsManager(empl_id))) -- c1 constraint
    THEN
        RETURN QUERY SELECT bs.floor_num, bs.room_num, bs.sdate, bs.stime, bs.bid
        FROM BookSessions bs
        WHERE bs.sdate >= startdate -- c3 constraint
        AND bs.mid IS NUll  -- c2 constraint
        -- check for c4 constraint
        AND (EXISTS(SELECT * FROM SameDeptAsRoom(bs.room_num, bs.floor_num, empl_id)))
        ORDER BY bs.sdate ASC, bs.stime ASC; -- c5 constraint
    END IF;
END;
$$ LANGUAGE plpgsql;

-----------------------------------------------------------------------------------------------------------------
-- END OF Proc.sql  