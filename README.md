1.  Replace src/App.jsx
import { useEffect, useState } from "react";
import axios from "axios";

function App() {
  const [students, setStudents] = useState([]);

  const [formData, setFormData] = useState({
    regNo: "",
    name: "",
    department: "",
    email: "",
    cgpa: ""
  });

  const [editId, setEditId] = useState(null);

  const API = "http://localhost:5000/students";

  const fetchStudents = async () => {
    try {
      const response = await axios.get(API);
      setStudents(response.data);
    } catch (error) {
      console.log(error);
    }
  };

  useEffect(() => {
    fetchStudents();
  }, []);

  const handleChange = (event) => {
    setFormData({
      ...formData,
      [event.target.name]: event.target.value
    });
  };

  const handleSubmit = async (event) => {
    event.preventDefault();

    try {
      if (editId) {
        await axios.put(`${API}/${editId}`, formData);
        setEditId(null);
      } else {
        await axios.post(API, formData);
      }

      setFormData({
        regNo: "",
        name: "",
        department: "",
        email: "",
        cgpa: ""
      });

      fetchStudents();
    } catch (error) {
      console.log(error);
    }
  };

  const handleEdit = (student) => {
    setFormData({
      regNo: student.regNo,
      name: student.name,
      department: student.department,
      email: student.email,
      cgpa: student.cgpa
    });

    setEditId(student._id);
  };

  const handleDelete = async (id) => {
    try {
      await axios.delete(`${API}/${id}`);
      fetchStudents();
    } catch (error) {
      console.log(error);
    }
  };

  return (
    <div>
      <h1>Student Management System</h1>

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="regNo"
          placeholder="Register Number"
          value={formData.regNo}
          onChange={handleChange}
          required
        />

        <input
          type="text"
          name="name"
          placeholder="Student Name"
          value={formData.name}
          onChange={handleChange}
          required
        />

        <input
          type="text"
          name="department"
          placeholder="Department"
          value={formData.department}
          onChange={handleChange}
          required
        />

        <input
          type="email"
          name="email"
          placeholder="Email"
          value={formData.email}
          onChange={handleChange}
        />

        <input
          type="number"
          step="0.01"
          name="cgpa"
          placeholder="CGPA"
          value={formData.cgpa}
          onChange={handleChange}
        />

        <button type="submit">
          {editId ? "Update Student" : "Add Student"}
        </button>
      </form>

      <hr />

      <h2>Student List</h2>

      <table border="1">
        <thead>
          <tr>
            <th>Register No</th>
            <th>Name</th>
            <th>Department</th>
            <th>Email</th>
            <th>CGPA</th>
            <th>Actions</th>
          </tr>
        </thead>

        <tbody>
          {students.map((student) => (
            <tr key={student._id}>
              <td>{student.regNo}</td>
              <td>{student.name}</td>
              <td>{student.department}</td>
              <td>{student.email}</td>
              <td>{student.cgpa}</td>

              <td>
                <button onClick={() => handleEdit(student)}>
                  Edit
                </button>

                <button onClick={() => handleDelete(student._id)}>
                  Delete
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default App;





2.  Create server.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();

app.use(cors());
app.use(express.json());

mongoose
  .connect("mongodb://127.0.0.1:27017/studentdb")
  .then(() => console.log("MongoDB Connected"))
  .catch((err) => console.log(err));

const studentSchema = new mongoose.Schema({
  regNo: {
    type: String,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  department: {
    type: String,
    required: true
  },
  email: String,
  cgpa: Number
});

const Student = mongoose.model("Student", studentSchema);

app.get("/", (req, res) => {
  res.send("Backend API is running");
});

// CREATE
app.post("/students", async (req, res) => {
  try {
    const student = new Student(req.body);
    const savedStudent = await student.save();

    res.status(201).json(savedStudent);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// READ ALL
app.get("/students", async (req, res) => {
  try {
    const students = await Student.find();
    res.json(students);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// READ ONE
app.get("/students/:id", async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);

    if (!student) {
      return res.status(404).json({
        message: "Student not found"
      });
    }

    res.json(student);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// UPDATE
app.put("/students/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );

    if (!student) {
      return res.status(404).json({
        message: "Student not found"
      });
    }

    res.json(student);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

// DELETE
app.delete("/students/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(
      req.params.id
    );

    if (!student) {
      return res.status(404).json({
        message: "Student not found"
      });
    }

    res.json({
      message: "Student deleted successfully"
    });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

app.listen(5000, () => {
  console.log("Server running on port 5000");
});
