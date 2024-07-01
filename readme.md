#Backend express
# First, ensure you have the necessary packages installed:

```bash
npm install express mongoose body-parser
```
Now, create a file called server.js and add the following code:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

// Middleware
app.use(bodyParser.json());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/school', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

const db = mongoose.connection;

db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', () => {
    console.log('Connected to MongoDB');
});

// Define schema and model
const studentSchema = new mongoose.Schema({
    name: String,
    roll_No: Number
});

const Student = mongoose.model('Student', studentSchema);

// CRUD Operations

// Create
app.post('/students', async (req, res) => {
    const student = new Student(req.body);
    try {
        const savedStudent = await student.save();
        res.status(201).send(savedStudent);
    } catch (err) {
        res.status(400).send(err);
    }
});

// Read all
app.get('/students', async (req, res) => {
    try {
        const students = await Student.find();
        res.status(200).send(students);
    } catch (err) {
        res.status(400).send(err);
    }
});

// Read one by roll_No
app.get('/students/:roll_No', async (req, res) => {
    try {
        const student = await Student.findOne({ roll_No: req.params.roll_No });
        if (!student) return res.status(404).send('Student not found');
        res.status(200).send(student);
    } catch (err) {
        res.status(400).send(err);
    }
});

// Update
app.put('/students/:roll_No', async (req, res) => {
    try {
        const student = await Student.findOneAndUpdate({ roll_No: req.params.roll_No }, req.body, { new: true });
        if (!student) return res.status(404).send('Student not found');
        res.status(200).send(student);
    } catch (err) {
        res.status(400).send(err);
    }
});

// Delete
app.delete('/students/:roll_No', async (req, res) => {
    try {
        const student = await Student.findOneAndDelete({ roll_No: req.params.roll_No });
        if (!student) return res.status(404).send('Student not found');
        res.status(200).send('Student deleted');
    } catch (err) {
        res.status(400).send(err);
    }
});

// Start the server
app.listen(port, () => {
    console.log(`Server is running on port ${port}`);
});
```
### Explanation:
### 1. Setup: The script sets up an Express application and connects to a MongoDB database named school.

### 2. Schema and Model: A Mongoose schema and model for Student is defined, with name and roll_No fields.

### 3. CRUD Operations:

- Create: Adds a new student.
- Read All: Retrieves all students.
- Read One: Retrieves a student by roll_No.
- Update: Updates a student's information by roll_No.
- Delete: Deletes a student by roll_No.
- Server: The server listens on port 3000.

To test the application, you can use a tool like Postman to send HTTP requests to the endpoints.

# frontend using react 
Sure! Below is a simple example of how to create a React front-end to interact with the Express API for managing students.

First, ensure you have the necessary packages installed in your React project:
```bash
npx create-react-app student-management

cd student-management
npm install axios
```
In the src directory of your React project, create a new file StudentService.js to handle API requests:
```javascript
import axios from 'axios';

const API_URL = 'http://localhost:3000/students';

export const getStudents = async () => {
    return await axios.get(API_URL);
};

export const getStudentByRollNo = async (roll_No) => {
    return await axios.get(`${API_URL}/${roll_No}`);
};

export const createStudent = async (student) => {
    return await axios.post(API_URL, student);
};

export const updateStudent = async (roll_No, student) => {
    return await axios.put(`${API_URL}/${roll_No}`, student);
};

export const deleteStudent = async (roll_No) => {
    return await axios.delete(`${API_URL}/${roll_No}`);
};
```
In the src directory, update App.js to include basic CRUD operations:
```javascript
import React, { useState, useEffect } from 'react';
import { getStudents, createStudent, updateStudent, deleteStudent } from './StudentService';

const App = () => {
    const [students, setStudents] = useState([]);
    const [name, setName] = useState('');
    const [rollNo, setRollNo] = useState('');
    const [editMode, setEditMode] = useState(false);
    const [currentRollNo, setCurrentRollNo] = useState(null);

    useEffect(() => {
        fetchStudents();
    }, []);

    const fetchStudents = async () => {
        const response = await getStudents();
        setStudents(response.data);
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        if (editMode) {
            await updateStudent(currentRollNo, { name, roll_No: rollNo });
            setEditMode(false);
            setCurrentRollNo(null);
        } else {
            await createStudent({ name, roll_No: rollNo });
        }
        setName('');
        setRollNo('');
        fetchStudents();
    };

    const handleEdit = (student) => {
        setEditMode(true);
        setName(student.name);
        setRollNo(student.roll_No);
        setCurrentRollNo(student.roll_No);
    };

    const handleDelete = async (roll_No) => {
        await deleteStudent(roll_No);
        fetchStudents();
    };

    return (
        <div className="App">
            <h1>Student Management</h1>
            <form onSubmit={handleSubmit}>
                <input 
                    type="text" 
                    placeholder="Name" 
                    value={name} 
                    onChange={(e) => setName(e.target.value)} 
                    required 
                />
                <input 
                    type="number" 
                    placeholder="Roll No" 
                    value={rollNo} 
                    onChange={(e) => setRollNo(e.target.value)} 
                    required 
                />
                <button type="submit">{editMode ? 'Update' : 'Create'}</button>
            </form>
            <h2>Students List</h2>
            <ul>
                {students.map(student => (
                    <li key={student.roll_No}>
                        {student.name} (Roll No: {student.roll_No})
                        <button onClick={() => handleEdit(student)}>Edit</button>
                        <button onClick={() => handleDelete(student.roll_No)}>Delete</button>
                    </li>
                ))}
            </ul>
        </div>
    );
};

export default App;
```
# Explanation:
### 1. StudentService.js: This file contains functions to interact with the backend API using Axios.

- `getStudents`: Fetches all students.
- `getStudentByRollNo`: Fetches a student by roll number.
- `createStudent`: Adds a new student.
- `updateStudent`: Updates a student's information.
- `deleteStudent`: Deletes a student.
- `App.js`: The main React component.

### 2. State Management: Manages state for students, form inputs, and edit mode.
- useEffect: Fetches students when the component mounts.
- fetchStudents: Fetches and sets the students state.
- handleSubmit: Handles form submission for creating or updating a student.
- handleEdit: Prepares the form for editing a student's information.
- handleDelete: Deletes a student and refreshes the students list.
This setup provides a basic UI to create, read, update, and delete student records, connecting the React front-end with the Express back-end API.
