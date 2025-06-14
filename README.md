# Student-management-system
students = []

def add_student():
    """Collects student information from the user and adds it to the students list."""
    name = input("Enter student name: ")
    student_id = input("Enter student ID: ")
    
    # Collect and validate grades
    while True:
        try:
            math_grade = int(input("Enter Math grade: "))
            science_grade = int(input("Enter Science grade: "))
            english_grade = int(input("Enter English grade: "))
            break
        except ValueError:
            print("Invalid input! Grades must be integers. Please try again.")
    
    # Create student dictionary
    student = {
        'name': name,
        'student_id': student_id,
        'math': math_grade,
        'science': science_grade,
        'english': english_grade
    }
    
    students.append(student)
    print(f"Student {name} added successfully!\n")

def view_students():
    """Displays all student records in a formatted table."""
    if not students:
        print("No students to display.")
        return
    
    # Print table header
    print("\n{:<20} {:<15} {:<10} {:<10} {:<10}".format(
        "Name", "Student ID", "Math", "Science", "English"))
    print("-" * 65)
    
    # Print each student's details
    for student in students:
        print("{:<20} {:<15} {:<10} {:<10} {:<10}".format(
            student['name'],
            student['student_id'],
            student['math'],
            student['science'],
            student['english']
        ))
    print()

def get_average_grade():
    """Calculates and returns the average grade of all students across all subjects."""
    if not students:
        return 0.0
    
    total_grades = 0
    total_subjects = 0
    
    for student in students:
        total_grades += student['math'] + student['science'] + student['english']
        total_subjects += 3
    
    return round(total_grades / total_subjects, 2)
     student_data

def main():
    while True:
        print("\nStudent Management System")
        print("1. Add Student")
        print("2. View Students")
        print("3. View Average Grade")
        print("4. Exit")
        
        choice = input("Enter your choice (1-4): ")
        
        if choice == '1':
            add_student()
        elif choice == '2':
            view_students()
        elif choice == '3':
            average = get_average_grade()
            print(f"\nAverage grade of all students: {average}\n")
        elif choice == '4':
            print("Exiting program. Goodbye!")
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 4.")

if __name__ == "__main__":
    main()
