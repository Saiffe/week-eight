# week-eight
clinic booking system
QUETION ONE'
-- Clinic Booking System Database

-- Drop tables if they already exist (for reruns)
DROP TABLE IF EXISTS Appointment, PatientDoctor, Patient, Doctor, Department;

-- 1. Department Table
CREATE TABLE Department (
    department_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

-- 2. Doctor Table
CREATE TABLE Doctor (
    doctor_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    specialization VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES Department(department_id)
);

-- 3. Patient Table
CREATE TABLE Patient (
    patient_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(15) NOT NULL UNIQUE,
    date_of_birth DATE
);

-- 4. PatientDoctor Table (M:N Relationship)
CREATE TABLE PatientDoctor (
    patient_id INT,
    doctor_id INT,
    PRIMARY KEY (patient_id, doctor_id),
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id)
);

-- 5. Appointment Table
CREATE TABLE Appointment (
    appointment_id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date DATETIME NOT NULL,
    reason TEXT,
    status ENUM('Scheduled', 'Completed', 'Cancelled') DEFAULT 'Scheduled',
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id)
);

-- Insert Sample Data

-- Departments
INSERT INTO Department (name) VALUES 
('Cardiology'),
('Dermatology'),
('Neurology');

-- Doctors
INSERT INTO Doctor (name, email, specialization, department_id) VALUES
('Dr. Alice Smith', 'victorcarlet00@.com', 'Cardiologist', 1),
('Dr. viky Jones', 'brian@.com', 'Dermatologist', 2),
('Dr. Charlie ', 'charlie@.com', 'Neurologist', 3);

-- Patients
INSERT INTO Patient (name, email, phone, date_of_birth) VALUES
('mecha', 'mecha@email.com', '0748791715', '1985-04-12'),
('victor', 'victor@gmail.com', '07654653', '1990-06-25');

-- Patient-Doctor Relationships
INSERT INTO PatientDoctor (patient_id, doctor_id) VALUES
(1, 1),
(1, 2),
(2, 3);

-- Appointments
INSERT INTO Appointment (patient_id, doctor_id, appointment_date, reason, status) VALUES
(1, 1, '2025-05-10 10:00:00', 'Chest pain consultation', 'Scheduled'),
(1, 2, '2025-05-11 14:30:00', 'Skin rash check-up', 'Scheduled'),
(2, 3, '2025-05-12 09:00:00', 'Headache and dizziness', 'Scheduled');

QUETION TWO
Data base schema(my sql)

-- MySQL: task_manager.sql

DROP TABLE IF EXISTS Tasks;
DROP TABLE IF EXISTS Users;

CREATE TABLE Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE Tasks (
    task_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status ENUM('Pending', 'In Progress', 'Completed') DEFAULT 'Pending',
    due_date DATE,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
API setup(node.js+express)
npm init -y
npm install express mysql2 body-parser cors dotenv

*folderstructures

task-manager-api/
├── db.js
├── .env
├── app.js
├── routes/
│   └── tasks.js

*data base credatials

DB_HOST=localhost
DB_USER=root
DB_PASSWORD=yourpassword
DB_NAME=task_manager
PORT=3000

 (MySQL connection)

 const mysql = require('mysql2');
require('dotenv').config();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});

module.exports = pool.promise();

routes/tasks.js (Task CRUD logic)

const express = require('express');
const router = express.Router();
const db = require('../db');

// CREATE task
router.post('/', async (req, res) => {
  const { title, description, due_date, user_id } = req.body;
  try {
    const [result] = await db.execute(
      'INSERT INTO Tasks (title, description, due_date, user_id) VALUES (?, ?, ?, ?)',
      [title, description, due_date, user_id]
    );
    res.json({ message: 'Task created', task_id: result.insertId });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// READ all tasks
router.get('/', async (req, res) => {
  try {
    const [rows] = await db.execute('SELECT * FROM Tasks');
    res.json(rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// READ task by ID
router.get('/:id', async (req, res) => {
  try {
    const [rows] = await db.execute('SELECT * FROM Tasks WHERE task_id = ?', [req.params.id]);
    if (rows.length === 0) return res.status(404).json({ error: 'Task not found' });
    res.json(rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// UPDATE task
router.put('/:id', async (req, res) => {
  const { title, description, status, due_date } = req.body;
  try {
    await db.execute(
      'UPDATE Tasks SET title = ?, description = ?, status = ?, due_date = ? WHERE task_id = ?',
      [title, description, status, due_date, req.params.id]
    );
    res.json({ message: 'Task updated' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// DELETE task
router.delete('/:id', async (req, res) => {
  try {
    await db.execute('DELETE FROM Tasks WHERE task_id = ?', [req.params.id]);
    res.json({ message: 'Task deleted' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

module.exports = router;

app.js (App entry point)

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const taskRoutes = require('./routes/tasks');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(bodyParser.json());

app.use('/api/tasks', taskRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});





