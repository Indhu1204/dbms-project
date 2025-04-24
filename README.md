body {
    font-family: Arial, sans-serif;
    margin: 0;
    background-color: #f3f3f3;
    color: #333;
  }
  
  header {
    background-color: #2e8b57;
    color: white;
    padding: 15px 20px;
  }
  
  header h1 {
    margin: 0;
    font-size: 24px;
  }
  
  nav ul {
    list-style: none;
    padding: 0;
    display: flex;
    gap: 15px;
    margin-top: 10px;
  }
  
  nav a {
    color: white;
    text-decoration: none;
  }
  
  nav a:hover {
    text-decoration: underline;
  }
  
  section {
    padding: 40px 20px;
  }
  
  .hero {
    background-color: white;
    text-align: center;
    padding: 60px 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
  
  h2, h3 {
    margin-bottom: 10px;
  }
  
  .form-container {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
  }
  
  input, textarea, button {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  
  button {
    background-color: #2e8b57;
    color: white;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #276c48;
  }
  
  .subMenu {
    display: flex;
    box-sizing: border-box;
    padding: 1%;
    flex-wrap: wrap;
  }
  
  .subMenu > div {
    display: flex;
    flex-wrap: wrap;
    box-sizing: border-box;
    padding: 1%;
    margin: 1%;
    width: 31%;
    height: 50%;
    justify-content: center;
    text-align: left;
  }
  
  .subMenu img {
    width: 200px;
    height: 200px;
  }
  
  .subMenu button {
    margin-top: 10px;
    padding: 8px 16px;
    background-color: #007BFF;
    color: white;
    border: none;
    border-radius: 5px;
  }
  
  strong {
    color: #333;
  }
  
  .contact-section {
    background-color: #2e8b57;
    color: white;
    text-align: center;
  }
  from flask import Flask, render_template, request, redirect, url_for
import mysql.connector
import os

app = Flask(__name__)

# Ensure the upload folder exists
UPLOAD_FOLDER = 'static/uploads'
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

# MySQL DB connection
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="2424",
    database="DB"
)
cursor = conn.cursor(dictionary=True)
drop database DB;
CREATE DATABASE DB;
USE DB;

CREATE TABLE Animal (
    AnimalId INT PRIMARY KEY,
    Name VARCHAR(255),
    Species VARCHAR(255),
    Breed VARCHAR(255),
    Age INT,
    HealthStatus VARCHAR(255),
    VaccinationStatus VARCHAR(255),
    Description VARCHAR(255),
    ImagePath VARCHAR(255) 
);

CREATE TABLE Adopter (
    AdopterId INT PRIMARY KEY,
    FirstName VARCHAR(255),
    LastName VARCHAR(255),
    PhoneNumber VARCHAR(20),
    Email VARCHAR(255),
    Address VARCHAR(255),
    AdoptionDate DATE,
    PreferredAnimal VARCHAR(255)
);

CREATE TABLE Staff (
    StaffId INT PRIMARY KEY,
    FirstName VARCHAR(255),
    LastName VARCHAR(255),
    Role VARCHAR(255),
    PhoneNumber VARCHAR(20),
    Salary DECIMAL(10,2),
    HireDate DATE
);

CREATE TABLE Adoption (
    AdoptionId INT PRIMARY KEY,
    AnimalId INT,
    AdopterId INT,
    StaffId INT,
    AdoptionDate DATE,
    AdoptionFees DECIMAL(10,2),
    FOREIGN KEY (AnimalId) REFERENCES Animal(AnimalId),
    FOREIGN KEY (AdopterId) REFERENCES Adopter(AdopterId),
    FOREIGN KEY (StaffId) REFERENCES Staff(StaffId)
);

CREATE TABLE Veterinarian (
    VetId INT PRIMARY KEY,
    FirstName VARCHAR(255),
    LastName VARCHAR(255),
    Email VARCHAR(255),
    Specialization VARCHAR(255),
    ClinicName VARCHAR(255),
    ClinicAddress VARCHAR(255)
);

CREATE TABLE MedicalRecord (
    RecordId INT PRIMARY KEY,
    AnimalId INT,
    VetId INT,
    VisitDate DATE,
    Treatment VARCHAR(255),
    Medication VARCHAR(20),
    NextVisitDate DATE,
    FOREIGN KEY (AnimalId) REFERENCES Animal(AnimalId),
    FOREIGN KEY (VetId) REFERENCES Veterinarian(VetId)
);

CREATE TABLE Donation (
    DonationId INT PRIMARY KEY,
    DonorName VARCHAR(255),
    Email VARCHAR(255),
    PhoneNumber VARCHAR(20),
    DonationAmount DECIMAL(10,2),
    DonationDate DATE,
    PaymentMethod VARCHAR(255)
);

@app.route('/')
def index():
    return render_template("index.html")

@app.route('/animals')
def show_animals():
    cursor.execute("SELECT * FROM Animal")
    animals = cursor.fetchall()
    return render_template("animals.html", animals=animals)

@app.route('/adopters')
def show_adopters():
    cursor.execute("SELECT * FROM Adopter")
    adopters = cursor.fetchall()
    return render_template("adopters.html", adopters=adopters)

@app.route('/add-animal', methods=['GET', 'POST'])
def add_animal():
    if request.method == 'POST':
        image = request.files['image']
        image_filename = image.filename
        image_path = os.path.join(UPLOAD_FOLDER, image_filename)
        image.save(image_path)

        data = (
            request.form['AnimalId'],
            request.form['Name'],
            request.form['Species'],
            request.form['Breed'],
            request.form['Age'],
            request.form['HealthStatus'],
            request.form['VaccinationStatus'],
            request.form['Description'],
            image_filename
        )

        cursor.execute("""
            INSERT INTO Animal (AnimalId, Name, Species, Breed, Age, HealthStatus, VaccinationStatus, Description, ImagePath)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
        """, data)
        conn.commit()
        return redirect(url_for('show_animals'))
    return render_template('add_animal.html')

@app.route('/add_adopter', methods=['GET', 'POST'])
def add_adopter():
    if request.method == 'POST':
        data = (
            request.form['AdopterId'],
            request.form['FirstName'],
            request.form['LastName'],
            request.form['PhoneNumber'],
            request.form['Email'],
            request.form['Address'],
            request.form['AdoptionDate'],
            request.form['PreferredAnimal']
        )
        cursor.execute("""
            INSERT INTO Adopter (AdopterId, FirstName, LastName, PhoneNumber, Email, Address, AdoptionDate, PreferredAnimal)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
        """, data)
        conn.commit()
        return redirect(url_for('show_adopters'))
    return render_template('add_adopter.html')

if __name__ == "__main__":
    app.run(debug=True)

  
  footer {
    background-color: #333;
    color: white;
    text-align: center;
    padding: 10px;
  }
  
