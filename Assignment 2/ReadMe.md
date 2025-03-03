# ERD Diagram

https://docs.google.com/viewer?url=https://drive.google.com/file/d/1XhY-WKCME18gi8w8M_YGuWQLdPMgxqYA/view?usp=sharing

## Python

from fastapi import FastAPI
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

class User(BaseModel):
    id: int
    name: str
    age: int
    email: str

# In-memory storage for demo purposes
users_db = []

@app.get("/")
def read_root():
    return {"message": "Welcome to the FastAPI application!"}

@app.get("/users", response_model=List[User])
def get_users():
    return users_db

@app.get("/users/{user_id}", response_model=Optional[User])
def get_user(user_id: int):
    for user in users_db:
        if user.id == user_id:
            return user
    return None

@app.post("/users", response_model=User)
def create_user(user: User):
    users_db.append(user)
    return user

@app.put("/users/{user_id}", response_model=Optional[User])
def update_user(user_id: int, user: User):
    for i, u in enumerate(users_db):
        if u.id == user_id:
            users_db[i] = user
            return user
    return None

@app.delete("/users/{user_id}", response_model=Optional[User])
def delete_user(user_id: int):
    for i, u in enumerate(users_db):
        if u.id == user_id:
            return users_db.pop(i)
    return None

## Pydantic Model

from pydantic import BaseModel
from typing import List, Optional

class Report(BaseModel):
    report_id: int
    user_id: int
    findings: str
    recommendations: str

class HealthData(BaseModel):
    health_data_id: int
    user_id: int
    

class Consultation(BaseModel):
    consultation_id: int
    date: str  # or datetime
    time: str  # or datetime
    user_id: int
    doctor_id: int
    

class Doctor(BaseModel):
    doctor_id: int
    name: str
    specialization: str
   

class User(BaseModel):
    user_id: int
    name: str
    age: int
   

# Example of creating instances
doctor = Doctor(
    doctor_id=1,
    name="Dr. Smith",
    specialization="Primary Care",
)

user = User(
    user_id=1,
    name="Joe",
    age=80,
    appointments=[],
    health_data=[],
    reports=[]
)

consultation = Consultation(
    consultation_id=1,
    date="2023-05-01",
    time="10:00",
    user_id=1,
    doctor_id=1,
    status="Scheduled"
)

health_data = HealthData(
    health_data_id=1,
    user_id=1,
)

report = Report(
    report_id=1,
    user_id=1,
    findings="Blood sugar levels are stable.",
    recommendations="Continue current medication."
)

# Adding instances to user
user.appointments.append(consultation)
user.health_data.append(health_data)
user.reports.append(report)

## CRUD EXAMPLE

from fastapi import FastAPI, HTTPException
from typing import List
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    user_id: int
    name: str
    age: int
    
users_db = []

@app.post("/users", response_model=User)
def create_user(user: User):
    users_db.append(user)
    return user

@app.get("/users", response_model=List[User])
def get_users():
    return users_db

@app.get("/users/{user_id}", response_model=User)
def get_user(user_id: int):
    user = next((u for u in users_db if u.user_id == user_id), None)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.put("/users/{user_id}", response_model=User)
def update_user(user_id: int, updated_user: User):
    for i, user in enumerate(users_db):
        if user.user_id == user_id:
            users_db[i] = updated_user
            return updated_user
    raise HTTPException(status_code=404, detail="User not found")

@app.delete("/users/{user_id}", response_model=User)
def delete_user(user_id: int):
    for i, user in enumerate(users_db):
        if user.user_id == user_id:
            return users_db.pop(i)
    raise HTTPException(status_code=404, detail="User not found")
