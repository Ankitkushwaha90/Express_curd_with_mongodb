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
