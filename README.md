# MockStudentTracker
a mock student tracker system where you can upload a name of a student and add/update their grades. It has also a ranking system so you will know whose students are on top.





import json
import os

DATA_FILE = "students.json"


# =========================
# FILE HANDLING
# =========================

def load_data():
    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE, "r", encoding="utf-8") as file:
                return json.load(file)
        except json.JSONDecodeError:
            print("⚠ Corrupted data file detected.")
            return {}
    return {}


def save_data(data):
    with open(DATA_FILE, "w", encoding="utf-8") as file:
        json.dump(data, file, indent=4)
    print("✔ Data saved successfully.")


# =========================
# HELPER FUNCTIONS
# =========================

def divider():
    print("─" * 50)


def generate_id(data):
    if not data:
        return "S001"

    last_id = max(int(student_id[1:]) for student_id in data.keys())
    return f"S{last_id + 1:03d}"


def get_average(grades):
    if not grades:
        return 0
    return sum(grades.values()) / len(grades)


def get_letter_grade(avg):
    if avg >= 90:
        return "Excellent"
    elif avg >= 80:
        return "Good"
    elif avg >= 70:
        return "Fair"
    elif avg >= 60:
        return "Poor"
    return "Failing"


# =========================
# STUDENT FUNCTIONS
# =========================

def add_student(data):
    divider()
    print("ADD STUDENT")
    divider()

    name = input("Enter student name: ").strip()

    if not name:
        print("✘ Name cannot be empty.")
        return

    for sid, info in data.items():
        if info["name"].lower() == name.lower():
            print(f"✘ Student already exists with ID {sid}")
            return

    student_id = generate_id(data)

    data[student_id] = {
        "name": name,
        "grades": {}
    }

    save_data(data)
    print(f"✔ Student added successfully. ID: {student_id}")


def edit_student_name(data):
    divider()
    print("EDIT STUDENT NAME")
    divider()

    student_id = input("Enter Student ID: ").strip().upper()

    if student_id not in data:
        print("✘ Student not found.")
        return

    new_name = input("Enter new name: ").strip()

    if not new_name:
        print("✘ Name cannot be empty.")
        return

    data[student_id]["name"] = new_name
    save_data(data)

    print("✔ Student name updated.")


def delete_student(data):
    divider()
    print("DELETE STUDENT")
    divider()

    student_id = input("Enter Student ID: ").strip().upper()

    if student_id not in data:
        print("✘ Student not found.")
        return

    confirm = input("Type YES to confirm: ").upper()

    if confirm == "YES":
        del data[student_id]
        save_data(data)
        print("✔ Student deleted.")
    else:
        print("Cancelled.")


# =========================
# GRADE FUNCTIONS
# =========================

def add_grade(data):
    divider()
    print("ADD / UPDATE GRADE")
    divider()

    student_id = input("Enter Student ID: ").strip().upper()

    if student_id not in data:
        print("✘ Student not found.")
        return

    subject = input("Enter Subject: ").strip()

    try:
        grade = float(input("Enter Grade (0-100): "))
    except ValueError:
        print("✘ Invalid grade.")
        return

    if grade < 0 or grade > 100:
        print("✘ Grade must be between 0 and 100.")
        return

    data[student_id]["grades"][subject] = grade

    save_data(data)

    print("✔ Grade saved.")


def delete_grade(data):
    divider()
    print("DELETE GRADE")
    divider()

    student_id = input("Enter Student ID: ").strip().upper()

    if student_id not in data:
        print("✘ Student not found.")
        return

    grades = data[student_id]["grades"]

    if not grades:
        print("No grades recorded.")
        return

    subject = input("Enter Subject to Delete: ").strip()

    if subject not in grades:
        print("✘ Subject not found.")
        return

    del grades[subject]

    save_data(data)

    print("✔ Grade deleted.")


# =========================
# REPORTS
# =========================

def view_student_report(data):
    divider()
    print("STUDENT REPORT")
    divider()

    student_id = input("Enter Student ID: ").strip().upper()

    if student_id not in data:
        print("✘ Student not found.")
        return

    student = data[student_id]

    print(f"\nName : {student['name']}")
    print(f"ID   : {student_id}")

    grades = student["grades"]

    if not grades:
        print("\nNo grades available.")
        return

    divider()

    print(f"{'Subject':20} {'Grade':>10}")

    divider()

    for subject, grade in grades.items():
        print(f"{subject:20} {grade:>10.1f}")

    avg = get_average(grades)

    divider()

    print(f"Average: {avg:.2f}")
    print(f"Letter : {get_letter_grade(avg)}")


def list_students(data):
    divider()
    print("ALL STUDENTS")
    divider()

    if not data:
        print("No students registered.")
        return

    print(f"{'ID':<8}{'NAME':<25}{'AVG':<10}{'GRADE'}")

    divider()

    for sid, info in sorted(data.items()):
        avg = get_average(info["grades"])

        if info["grades"]:
            grade_letter = get_letter_grade(avg)
            avg_display = f"{avg:.2f}"
        else:
            avg_display = "-"
            grade_letter = "N/A"

        print(
            f"{sid:<8}"
            f"{info['name']:<25}"
            f"{avg_display:<10}"
            f"{grade_letter}"
        )


def search_student(data):
    divider()
    print("SEARCH STUDENT")
    divider()

    keyword = input("Enter student name: ").lower()

    found = False

    for sid, info in data.items():
        if keyword in info["name"].lower():
            print(f"{sid} - {info['name']}")
            found = True

    if not found:
        print("No matching student found.")


def top_student(data):
    divider()
    print("TOP PERFORMING STUDENT")
    divider()

    ranked = []

    for sid, info in data.items():
        if info["grades"]:
            ranked.append(
                (
                    get_average(info["grades"]),
                    sid,
                    info["name"]
                )
            )

    if not ranked:
        print("No grades available.")
        return

    ranked.sort(reverse=True)

    avg, sid, name = ranked[0]

    print(f"Name    : {name}")
    print(f"ID      : {sid}")
    print(f"Average : {avg:.2f}")
    print(f"Grade   : {get_letter_grade(avg)}")


def ranking_system(data):
    divider()
    print("CLASS RANKING")
    divider()

    ranked = []

    for sid, info in data.items():
        if info["grades"]:
            avg = get_average(info["grades"])
            ranked.append((avg, sid, info["name"]))

    if not ranked:
        print("No grades available.")
        return

    ranked.sort(reverse=True)

    print(f"{'Rank':<6}{'ID':<8}{'Name':<25}{'Average'}")
    divider()

    for rank, (avg, sid, name) in enumerate(ranked, start=1):
        print(f"{rank:<6}{sid:<8}{name:<25}{avg:.2f}")


def class_statistics(data):
    divider()
    print("CLASS STATISTICS")
    divider()

    averages = []

    for info in data.values():
        if info["grades"]:
            averages.append(
                get_average(info["grades"])
            )

    if not averages:
        print("No grades available.")
        return

    highest = max(averages)
    lowest = min(averages)
    overall = sum(averages) / len(averages)

    print(f"Highest Average : {highest:.2f}")
    print(f"Lowest Average  : {lowest:.2f}")
    print(f"Class Average   : {overall:.2f}")


def honor_roll(data):
    divider()
    print("HONOR ROLL")
    divider()

    found = False

    for sid, info in data.items():

        if info["grades"]:

            avg = get_average(info["grades"])

            if avg >= 90:
                print(
                    f"{sid:<8}"
                    f"{info['name']:<25}"
                    f"{avg:.2f}"
                )
                found = True

    if not found:
        print("No honor students found.")


# =========================
# MENU
# =========================

def menu():
    print("\n" + "═" * 50)
    print("     STUDENT GRADE TRACKER v2.0")
    print("═" * 50)

    print("[1] Add Student")
    print("[2] Add / Update Grade")
    print("[3] View Student Report")
    print("[4] List All Students")
    print("[5] Delete Student")
    print("[6] Search Student")
    print("[7] Edit Student Name")
    print("[8] Delete Grade")
    print("[9] Top Performing Student")
    print("[10] Class Ranking")
    print("[11] Class Statistics")
    print("[12] Honor Roll")
    print("[0] Exit")

    print("═" * 50)


# =========================
# MAIN PROGRAM
# =========================

def main():
    data = load_data()

    while True:
        menu()

        choice = input("Choose option: ").strip()

        if choice == "1":
            add_student(data)

        elif choice == "2":
            add_grade(data)

        elif choice == "3":
            view_student_report(data)

        elif choice == "4":
            list_students(data)

        elif choice == "5":
            delete_student(data)

        elif choice == "6":
            search_student(data)

        elif choice == "7":
            edit_student_name(data)

        elif choice == "8":
            delete_grade(data)

        elif choice == "9":
            top_student(data)

        elif choice == "10":
            ranking_system(data)

        elif choice == "11":
            class_statistics(data)

        elif choice == "12":
            honor_roll(data)

        elif choice == "0":
            print("\nGoodbye! 👋")
            break

        else:
            print("✘ Invalid choice.")


if __name__ == "__main__":
    main()
