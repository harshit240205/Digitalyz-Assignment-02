# Digitalyz-Assignment-02

# Milestone 2: Course Scheduling

## Overview

This milestone focuses on assigning students to their requested courses while avoiding scheduling conflicts. The final output includes structured schedules for both students and lecturers, along with a report of any unresolved requests.

## Approach

1. **Data Loading:**

   - Loaded the cleaned dataset from `cleaned_data.json`.
   - Extracted relevant data for students, courses, lecturers, and available rooms.

2. **Scheduling Algorithm:**

   - **Student Course Assignment:** Assigned students to their requested courses based on available time blocks.
   - **Conflict Resolution:** Ensured students were not double-booked for the same time slot.
   - **Teacher Schedule Assignment:** Assigned lecturers to courses in available time slots.
   - **Unresolved Requests Handling:** Logged any student requests that could not be fulfilled due to conflicts or missing courses.

## Assumptions

- Each student request is processed sequentially; first-come, first-served.
- A student is assigned to the first available time block for a requested course.
- If no valid slot is available, the request is marked as unresolved.
- Each course is taught by one lecturer, and lecturers are not double-booked.

## Output Files

- `final_schedule.json` → Contains:
  - **Student schedules** with assigned courses and blocks.
  - **Teacher schedules** with assigned teaching times.
  - **Unresolved requests** list.

## Relevant Code

### 1. Data Loading

```python
import json
import pandas as pd

# Load the cleaned dataset
with open("cleaned_data.json", "r") as file:
    data = json.load(file)

students = data["student_requests"]
courses = data["courses"]
lecturers = data["lecturers"]
rooms = data["rooms"]
```

### 2. Student Course Assignment

```python
# Initialize schedules
student_schedule = {}
teacher_schedule = {}
unresolved_requests = []

# Assign students to courses
for student in students:
    student_id = student["student_id"]
    course_code = student["course_code"]
    
    course_data = next((c for c in courses if c["course_code"] == course_code), None)
    if not course_data:
        unresolved_requests.append(student)
        continue
    
    available_blocks = course_data["available_blocks"].split(",") if course_data["available_blocks"] else []
    assigned = False
    
    for block in available_blocks:
        if student_id not in student_schedule:
            student_schedule[student_id] = []
        
        if block not in student_schedule[student_id]:
            student_schedule[student_id].append({"course": course_code, "block": block})
            assigned = True
            break
    
    if not assigned:
        unresolved_requests.append(student)
```

### 3. Teacher Schedule Assignment

```python
# Assign teachers to schedules
for course in courses:
    course_code = course["course_code"]
    lecturer_data = next((l for l in lecturers if l["lecture_code"] == course_code), None)
    
    if not lecturer_data:
        continue
    
    lecturer_id = lecturer_data["lecturer_id"]
    available_blocks = course["available_blocks"].split(",") if course["available_blocks"] else []
    
    for block in available_blocks:
        if lecturer_id not in teacher_schedule:
            teacher_schedule[lecturer_id] = []
        teacher_schedule[lecturer_id].append({"course": course_code, "block": block})
```

### 4. Saving the Output

```python
# Save output schedules
output_data = {
    "student_schedule": student_schedule,
    "teacher_schedule": teacher_schedule,
    "unresolved_requests": unresolved_requests
}

output_file_path = "final_schedule.json"
with open(output_file_path, "w") as file:
    json.dump(output_data, file, indent=4)

print("Scheduling completed. Output saved.")
```


