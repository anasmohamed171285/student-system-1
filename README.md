/*
    Student Management System
    --------------------------
    A simple C++ console application (OOP) for managing students and their courses.

    Features:
      1. Add Student
      2. Delete Student (by ID or name)
      3. Add Course to Student
      4. Delete Course from Student
      5. Display All Students
      6. Exit

    Compile:
      g++ -std=c++17 -o sms StudentManagementSystem.cpp
    Run:
      ./sms
*/

#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <limits>

using namespace std;

// ------------------------------------------------------------
// Class: Student
// Represents a single student with an ID, name, and list of courses.
// ------------------------------------------------------------
class Student {
private:
    int id;
    string name;
    vector<string> courses;

public:
    Student(int id, const string& name) : id(id), name(name) {}

    int getId() const { return id; }
    string getName() const { return name; }
    const vector<string>& getCourses() const { return courses; }

    void addCourse(const string& course) {
        courses.push_back(course);
    }

    // Returns true if a course was found & removed
    bool removeCourse(const string& course) {
        auto it = find_if(courses.begin(), courses.end(), [&](const string& c) {
            return equalsIgnoreCase(c, course);
        });
        if (it != courses.end()) {
            courses.erase(it);
            return true;
        }
        return false;
    }

    bool hasCourse(const string& course) const {
        return find_if(courses.begin(), courses.end(), [&](const string& c) {
            return equalsIgnoreCase(c, course);
        }) != courses.end();
    }

    static bool equalsIgnoreCase(const string& a, const string& b) {
        if (a.size() != b.size()) return false;
        for (size_t i = 0; i < a.size(); ++i) {
            if (tolower(a[i]) != tolower(b[i])) return false;
        }
        return true;
    }

    void display() const {
        cout << "ID: " << id << " | Name: " << name << " | Courses: ";
        if (courses.empty()) {
            cout << "(none)";
        } else {
            for (size_t i = 0; i < courses.size(); ++i) {
                cout << courses[i];
                if (i != courses.size() - 1) cout << ", ";
            }
        }
        cout << endl;
    }
};

// ------------------------------------------------------------
// Class: StudentManager
// Manages the collection of Student objects: add, delete, search, display.
// ------------------------------------------------------------
class StudentManager {
private:
    vector<Student> students;
    int nextId;

public:
    StudentManager() : nextId(1) {}

    // ---- Add Student ----
    void addStudent() {
        string name;
        cout << "Enter student name: ";
        getline(cin, name);

        if (name.empty()) {
            cout << "Name cannot be empty. Student not added.\n";
            return;
        }

        Student newStudent(nextId, name);
        students.push_back(newStudent);
        cout << "Student added successfully! Assigned ID: " << nextId << endl;
        nextId++;
    }

    // ---- Find student by ID, returns pointer or nullptr ----
    Student* findById(int id) {
        for (auto& s : students) {
            if (s.getId() == id) return &s;
        }
        return nullptr;
    }

    // ---- Find student(s) by name (case-insensitive), returns indices ----
    vector<int> findByName(const string& name) {
        vector<int> matches;
        for (size_t i = 0; i < students.size(); ++i) {
            if (Student::equalsIgnoreCase(students[i].getName(), name)) {
                matches.push_back((int)i);
            }
        }
        return matches;
    }

    // ---- Helper: ask user to pick a student by ID or Name ----
    Student* selectStudent() {
        if (students.empty()) {
            cout << "No students in the system.\n";
            return nullptr;
        }

        cout << "Search student by (1) ID or (2) Name? ";
        int choice;
        if (!(cin >> choice)) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input.\n";
            return nullptr;
        }
        cin.ignore(numeric_limits<streamsize>::max(), '\n');

        if (choice == 1) {
            cout << "Enter student ID: ";
            int id;
            if (!(cin >> id)) {
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
                cout << "Invalid ID.\n";
                return nullptr;
            }
            cin.ignore(numeric_limits<streamsize>::max(), '\n');

            Student* s = findById(id);
            if (!s) cout << "No student found with ID " << id << ".\n";
            return s;
        } else if (choice == 2) {
            cout << "Enter student name: ";
            string name;
            getline(cin, name);

            vector<int> matches = findByName(name);
            if (matches.empty()) {
                cout << "No student found with name \"" << name << "\".\n";
                return nullptr;
            } else if (matches.size() == 1) {
                return &students[matches[0]];
            } else {
                // Multiple students share the same name; let user pick by ID
                cout << "Multiple students found with that name:\n";
                for (int idx : matches) {
                    students[idx].display();
                }
                cout << "Please enter the exact ID of the student you mean: ";
                int id;
                if (!(cin >> id)) {
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                    cout << "Invalid ID.\n";
                    return nullptr;
                }
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
                Student* s = findById(id);
                if (!s) cout << "No student found with that ID.\n";
                return s;
            }
        } else {
            cout << "Invalid choice.\n";
            return nullptr;
        }
    }

    // ---- Delete Student ----
    void deleteStudent() {
        if (students.empty()) {
            cout << "No students in the system.\n";
            return;
        }

        cout << "Search student to delete by (1) ID or (2) Name? ";
        int choice;
        if (!(cin >> choice)) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input.\n";
            return;
        }
        cin.ignore(numeric_limits<streamsize>::max(), '\n');

        if (choice == 1) {
            cout << "Enter student ID: ";
            int id;
            if (!(cin >> id)) {
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
                cout << "Invalid ID.\n";
                return;
            }
            cin.ignore(numeric_limits<streamsize>::max(), '\n');

            auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
                return s.getId() == id;
            });
            if (it == students.end()) {
                cout << "No student found with ID " << id << ".\n";
                return;
            }
            cout << "Deleted student: " << it->getName() << " (ID: " << it->getId() << ")\n";
            students.erase(it);
        } else if (choice == 2) {
            cout << "Enter student name: ";
            string name;
            getline(cin, name);

            vector<int> matches = findByName(name);
            if (matches.empty()) {
                cout << "No student found with name \"" << name << "\".\n";
                return;
            } else if (matches.size() == 1) {
                cout << "Deleted student: " << students[matches[0]].getName()
                     << " (ID: " << students[matches[0]].getId() << ")\n";
                students.erase(students.begin() + matches[0]);
            } else {
                cout << "Multiple students found with that name:\n";
                for (int idx : matches) {
                    students[idx].display();
                }
                cout << "Please enter the exact ID of the student to delete: ";
                int id;
                if (!(cin >> id)) {
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                    cout << "Invalid ID.\n";
                    return;
                }
                cin.ignore(numeric_limits<streamsize>::max(), '\n');

                auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
                    return s.getId() == id;
                });
                if (it == students.end()) {
                    cout << "No student found with that ID.\n";
                    return;
                }
                cout << "Deleted student: " << it->getName() << " (ID: " << it->getId() << ")\n";
                students.erase(it);
            }
        } else {
            cout << "Invalid choice.\n";
        }
    }

    // ---- Add Course to Student ----
    void addCourseToStudent() {
        Student* s = selectStudent();
        if (!s) return;

        cout << "Enter course name to add: ";
        string course;
        getline(cin, course);

        if (course.empty()) {
            cout << "Course name cannot be empty.\n";
            return;
        }

        if (s->hasCourse(course)) {
            cout << "Student already enrolled in \"" << course << "\".\n";
            return;
        }

        s->addCourse(course);
        cout << "Course \"" << course << "\" added to " << s->getName() << ".\n";
    }

    // ---- Delete Course from Student ----
    void deleteCourseFromStudent() {
        Student* s = selectStudent();
        if (!s) return;

        if (s->getCourses().empty()) {
            cout << s->getName() << " has no courses enrolled.\n";
            return;
        }

        cout << s->getName() << "'s current courses: ";
        const auto& courses = s->getCourses();
        for (size_t i = 0; i < courses.size(); ++i) {
            cout << courses[i];
            if (i != courses.size() - 1) cout << ", ";
        }
        cout << endl;

        cout << "Enter course name to remove: ";
        string course;
        getline(cin, course);

        if (s->removeCourse(course)) {
            cout << "Course \"" << course << "\" removed from " << s->getName() << ".\n";
        } else {
            cout << "Course \"" << course << "\" not found for " << s->getName() << ".\n";
        }
    }

    // ---- Display All Students ----
    void displayAllStudents() const {
        if (students.empty()) {
            cout << "No students in the system.\n";
            return;
        }

        cout << "\n==================== Student List ====================\n";
        for (const auto& s : students) {
            s.display();
        }
        cout << "=======================================================\n";
    }
};

// ------------------------------------------------------------
// Helper: Display the main menu
// ------------------------------------------------------------
void displayMenu() {
    cout << "\n========== Student Management System ==========\n";
    cout << "1. Add Student\n";
    cout << "2. Delete Student\n";
    cout << "3. Add Course to Student\n";
    cout << "4. Delete Course from Student\n";
    cout << "5. Display All Students\n";
    cout << "6. Exit\n";
    cout << "=================================================\n";
    cout << "Enter your choice: ";
}

// ------------------------------------------------------------
// Main Program
// ------------------------------------------------------------
int main() {
    StudentManager manager;
    int choice;
    bool running = true;

    cout << "Welcome to the Student Management System!\n";

    while (running) {
        displayMenu();

        if (!(cin >> choice)) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input. Please enter a number between 1 and 6.\n";
            continue;
        }
        cin.ignore(numeric_limits<streamsize>::max(), '\n'); // clear leftover newline

        switch (choice) {
            case 1:
                manager.addStudent();
                break;
            case 2:
                manager.deleteStudent();
                break;
            case 3:
                manager.addCourseToStudent();
                break;
            case 4:
                manager.deleteCourseFromStudent();
                break;
            case 5:
                manager.displayAllStudents();
                break;
            case 6:
                cout << "Exiting program. Goodbye!\n";
                running = false;
                break;
            default:
                cout << "Invalid choice. Please select an option between 1 and 6.\n";
        }
    }

    return 0;
}
