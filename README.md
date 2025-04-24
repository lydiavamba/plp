-- QUESTION 1
CREATE database Doctor_table;

-- Create Doctor table
CREATE TABLE Doctor (
    doctor_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    specialty VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Create Patient table
CREATE TABLE Patient (
    patient_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    phone VARCHAR(15) UNIQUE,
    email VARCHAR(100) UNIQUE
);

-- Create Appointment table
CREATE TABLE Appointment (
    appointment_id INT AUTO_INCREMENT PRIMARY KEY,
    doctor_id INT NOT NULL,
    patient_id INT NOT NULL,
    appointment_date DATETIME NOT NULL,
    status ENUM('Scheduled', 'Completed', 'Cancelled') DEFAULT 'Scheduled',
    FOREIGN KEY (doctor_id) REFERENCES Doctor(doctor_id),
    FOREIGN KEY (patient_id) REFERENCES Patient(patient_id)
);

-- Create Treatment table
CREATE TABLE Treatment (
    treatment_id INT AUTO_INCREMENT PRIMARY KEY,
    appointment_id INT NOT NULL,
    description TEXT,
    cost DECIMAL(10,2),
    FOREIGN KEY (appointment_id) REFERENCES Appointment(appointment_id)
);

-- Sample Data

-- Doctors
INSERT INTO Doctor (name, specialty, email)
VALUES 
('Dr. John Smith', 'Cardiology', 'john.smith@clinic.com'),
('Dr. Alice Brown', 'Dermatology', 'alice.brown@clinic.com');

-- Patients
INSERT INTO Patient (name, date_of_birth, phone, email)
VALUES 
('Mary Johnson', '1985-03-10', '1234567890', 'mary.j@example.com'),
('James Lee', '1992-07-24', '0987654321', 'james.lee@example.com');

-- Appointments
INSERT INTO Appointment (doctor_id, patient_id, appointment_date, status)
VALUES 
(1, 1, '2025-05-01 10:00:00', 'Scheduled'),
(2, 2, '2025-05-02 14:30:00', 'Completed');

-- Treatments
INSERT INTO Treatment (appointment_id, description, cost)
VALUES 
(2, 'Skin rash treatment and consultation', 120.00);


-- question 2
-- Group Table
CREATE TABLE GroupTable (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- Contact Table
CREATE TABLE Contact (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    group_id INT,
    FOREIGN KEY (group_id) REFERENCES GroupTable(id)
);

# Database connection
def get_db():
    return mysql.connector.connect(
        host="localhost",
        user="your_user",
        password="your_password",
        database="your_database"
    )

# Models
class Contact(BaseModel):
    name: str
    email: str
    phone: str = None
    group_id: int = None

# Routes

@app.post("/contacts/")
def create_contact(contact: Contact):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("INSERT INTO Contact (name, email, phone, group_id) VALUES (%s, %s, %s, %s)",
                   (contact.name, contact.email, contact.phone, contact.group_id))
    db.commit()
    return {"message": "Contact created successfully"}

@app.get("/contacts/")
def read_contacts():
    db = get_db()
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM Contact")
    return cursor.fetchall()

@app.get("/contacts/{contact_id}")
def read_contact(contact_id: int):
    db = get_db()
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM Contact WHERE id = %s", (contact_id,))
    result = cursor.fetchone()
    if not result:
        raise HTTPException(status_code=404, detail="Contact not found")
    return result

@app.put("/contacts/{contact_id}")
def update_contact(contact_id: int, contact: Contact):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("""
        UPDATE Contact SET name=%s, email=%s, phone=%s, group_id=%s WHERE id=%s
    """, (contact.name, contact.email, contact.phone, contact.group_id, contact_id))
    db.commit()
    return {"message": "Contact updated successfully"}

@app.delete("/contacts/{contact_id}")
def delete_contact(contact_id: int):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("DELETE FROM Contact WHERE id = %s", (contact_id,))
    db.commit()# plp
