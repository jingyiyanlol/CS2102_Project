DROP TABLE IF EXISTS
    Departments, Employees, HealthDeclaration, MeetingRooms, Junior, Booker, 
    Senior, Manager, BookSessions, Joins, Updates 
CASCADE;

-- Departments
CREATE TABLE Departments (
    did INTEGER PRIMARY KEY,
    dname VARCHAR(50) NOT NULL
);

-- Employees (also includes 'Works In' relation)
CREATE TABLE Employees (
    eid INTEGER GENERATED ALWAYS AS IDENTITY (MINVALUE 1 START WITH 1) PRIMARY KEY,
    ename VARCHAR(50) NOT NULL,
    email VARCHAR(50) UNIQUE,
    home_contact INTEGER,
    mobile_contact INTEGER,
    office_contact INTEGER,
    resigned_date DATE DEFAULT NULL,
    did INTEGER, -- note: we did not use NOT NULL here becuz when we remove department, for resigned employees there would be issue
    FOREIGN KEY (did) REFERENCES Departments(did)
);

-- Health Declaration (weak entity)
CREATE TABLE HealthDeclaration (
    hdate DATE,
    temp numeric constraint valid_temp check (temp BETWEEN 34.0 AND 43.0),
    eid INTEGER, 
    PRIMARY KEY (eid, hdate),
    FOREIGN KEY (eid) REFERENCES Employees(eid)
        ON UPDATE CASCADE ON DELETE CASCADE
);

-- Meeting Rooms (also includes 'Located In' relation)
CREATE TABLE MeetingRooms (
    room_num INTEGER,
    floor_num INTEGER,
    rname VARCHAR(50) NOT NULL,
    did INTEGER NOT NULL,
    PRIMARY KEY(room_num, floor_num),
    FOREIGN KEY (did) REFERENCES Departments(did)
);

-- ISA Employees to Junior
CREATE TABLE Junior (
    jid INTEGER PRIMARY KEY,
    FOREIGN KEY (jid) REFERENCES Employees(eid)
        ON UPDATE CASCADE ON DELETE CASCADE
);

-- ISA Employees to Booker
CREATE TABLE Booker (
    bid INTEGER PRIMARY KEY,
    FOREIGN KEY (bid) REFERENCES Employees(eid)
        ON UPDATE CASCADE ON DELETE CASCADE
);

-- ISA Booker to Senior
CREATE TABLE Senior (
    snid INTEGER PRIMARY KEY,
    FOREIGN KEY (snid) REFERENCES Booker(bid)
        ON UPDATE CASCADE ON DELETE CASCADE
);

-- ISA Booker to Manager
CREATE TABLE Manager (
    mid INTEGER PRIMARY KEY,
    FOREIGN KEY (mid) REFERENCES Booker(bid)
        ON UPDATE CASCADE ON DELETE CASCADE
);

-- Sessions (weak entity)
-- (also includes 'Books' and 'Approves' relation)
CREATE TABLE BookSessions (
    stime INTEGER constraint valid_hour check (stime BETWEEN -1 AND 24),
    sdate DATE,
    room_num INTEGER,
    floor_num INTEGER,
    bid INTEGER NOT NULL,
    mid INTEGER DEFAULT NULL,
    PRIMARY KEY (stime, sdate, room_num, floor_num),
    FOREIGN KEY (room_num, floor_num) REFERENCES MeetingRooms(room_num, floor_num),
    FOREIGN KEY (bid) REFERENCES Booker(bid),
    FOREIGN KEY (mid) REFERENCES Manager(mid)
);

-- Joins relation (DOES NOT display BookSessions total participation constraint)
CREATE TABLE Joins (
    eid INTEGER,
    stime INTEGER,
    sdate DATE,
    room_num INTEGER,
    floor_num INTEGER,
    PRIMARY KEY (eid, stime, sdate, room_num, floor_num),
    FOREIGN KEY (eid) REFERENCES Employees(eid),
    FOREIGN KEY (stime, sdate, room_num, floor_num) REFERENCES BookSessions(stime, sdate, room_num, floor_num)
        ON DELETE CASCADE
);

-- Updates relation (DOES NOT display MeetingRooms total participation constraint)
CREATE TABLE Updates (
    udate DATE,
    room_num INTEGER,
    floor_num INTEGER,
    mid INTEGER,
    new_cap INTEGER constraint valid_cap check (new_cap > 0),
    PRIMARY KEY (udate, room_num, floor_num),
    FOREIGN KEY (room_num, floor_num) REFERENCES MeetingRooms(room_num, floor_num),
    FOREIGN KEY (mid) REFERENCES Manager(mid)
);




-- NOTES 
-- 1) For Updates table, we will max have 1 entry per meeting room per day. 
--    If multiple queries, simply update.
-- 2) For Joins table, ON DELETE CASCADE means if meeting is removed, all of them attending 
--    would get removed.