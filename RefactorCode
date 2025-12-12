#include <iostream>
#include <fstream>
#include <vector>
#include <iomanip>
#include <string>
#include <algorithm>
#include <map>
#include <functional>
#include <memory>
#include <ctime>
#include <sstream>
#include <unordered_map>
using namespace std;

// ==================== SOLID PRINCIPLES ANALYSIS ====================

/*
    SRP (Single Responsibility Principle):
    - Each class has one clear responsibility
    - Methods are focused on single tasks
*/

/*
    OCP (Open/Closed Principle):
    - Classes open for extension but closed for modification
    - New functionality added through inheritance/interfaces
*/

/*
    LSP (Liskov Substitution Principle):
    - Derived classes can substitute their base classes
    - Interfaces properly implemented by concrete classes
*/

/*
    ISP (Interface Segregation Principle):
    - Small, focused interfaces
    - No client forced to depend on unused methods
*/

/*
    DIP (Dependency Inversion Principle):
    - High-level modules depend on abstractions
    - Dependency injection used throughout
*/

// ==================== BASE CLASSES ====================

// SRP: Only stores attendance data
struct AttendanceRecord {
    string date;
    string status; // "Present" or "Absent"
};

// SRP: Only stores student data
struct Student {
    string name;
    int rollNo;
    string studentClass;
    int age;
    string gender;
    float marks[5] = {};
    float percentage = 0;
    char grade = 'F';
    vector<AttendanceRecord> attendanceRecords;
    float gpa = 0.0;
};

// ISP: Small interface for grade strategies
// DIP: High-level modules depend on this abstraction
class IGradeStrategy {
public:
    virtual char calculateGrade(float percentage) const = 0;
    virtual float calculateGPA(float percentage) const = 0;
    virtual ~IGradeStrategy() = default;
};

// ISP: Small interface for exporters
class IExporter {
public:
    virtual void exportData(const vector<Student> &students) const = 0;
    virtual ~IExporter() = default;
};

// ISP: Small interface for report generators
class IReportGenerator {
public:
    virtual void generateReport(const vector<Student> &students) const = 0;
    virtual ~IReportGenerator() = default;
};

// DIP: Abstraction for grade calculation
class IGradeCalculator {
public:
    virtual void calculateGrade(Student &s) const = 0;
    virtual ~IGradeCalculator() = default;
};

// ==================== CONCRETE IMPLEMENTATIONS ====================

// LSP: Properly implements IGradeStrategy
// OCP: New strategies can be added without changing existing code
class DefaultGradeStrategy : public IGradeStrategy {
public:
    char calculateGrade(float percentage) const override {
        if (percentage >= 90) return 'A';
        if (percentage >= 80) return 'B';
        if (percentage >= 70) return 'C';
        if (percentage >= 60) return 'D';
        return 'F';
    }

    float calculateGPA(float percentage) const override {
        if (percentage >= 90) return 4.0;
        if (percentage >= 80) return 3.0;
        if (percentage >= 70) return 2.0;
        if (percentage >= 60) return 1.0;
        return 0.0;
    }
};

// LSP: Properly implements IExporter
// SRP: Only handles CSV export
class CSVExporter : public IExporter {
public:
    void exportData(const vector<Student> &students) const override {
        ofstream file("students.csv");
        file << "Roll,Name,Class,Age,Gender,Percentage,Grade,GPA,Attendance%\n";
        for (const auto &s : students) {
            float attendancePercent = calculateAttendancePercentage(s);
            file << s.rollNo << "," << s.name << "," << s.studentClass << ","
                 << s.age << "," << s.gender << "," << s.percentage << ","
                 << s.grade << "," << fixed << setprecision(2) << s.gpa << ","
                 << attendancePercent << "%\n";
        }
        cout << "Data exported to students.csv\n";
    }

private:
    float calculateAttendancePercentage(const Student &s) const {
        if (s.attendanceRecords.empty()) return 0.0f;
        int presentCount = count_if(s.attendanceRecords.begin(), s.attendanceRecords.end(),
                                   [](const AttendanceRecord &r) { return r.status == "Present"; });
        return (static_cast<float>(presentCount) / s.attendanceRecords.size()) * 100;
    }
};

// LSP: Properly implements IReportGenerator
// SRP: Only handles text report generation
class TextReportGenerator : public IReportGenerator {
public:
    void generateReport(const vector<Student> &students) const override {
        string cls;
        cout << "Enter class to view report: ";
        cin >> cls;

        bool found = false;
        for (const auto &s : students) {
            if (s.studentClass == cls) {
                float attendancePercent = calculateAttendancePercentage(s);
                cout << s.rollNo << "\t" << s.name << "\t" << s.grade << "\t"
                     << s.percentage << "%\t" << fixed << setprecision(2) << s.gpa << "\t"
                     << attendancePercent << "%\n";
                found = true;
            }
        }
        if (!found) {
            cout << "No students found in class " << cls << "\n";
        }
    }

private:
    float calculateAttendancePercentage(const Student &s) const {
        if (s.attendanceRecords.empty()) return 0.0f;
        int presentCount = count_if(s.attendanceRecords.begin(), s.attendanceRecords.end(),
                                  [](const AttendanceRecord &r) { return r.status == "Present"; });
        return (static_cast<float>(presentCount) / s.attendanceRecords.size()) * 100;
    }
};

// ==================== CORE MANAGEMENT CLASSES ====================

// DIP: Depends on IGradeStrategy abstraction
// SRP: Only handles grade calculation
class GradeCalculator : public IGradeCalculator {
    shared_ptr<IGradeStrategy> strategy;  // DIP: Using abstraction
public:
    explicit GradeCalculator(shared_ptr<IGradeStrategy> strat) : strategy(move(strat)) {}

    void calculateGrade(Student &s) const override {
        float total = 0;
        for (float mark : s.marks) total += mark;
        s.percentage = total / 5;
        s.grade = strategy->calculateGrade(s.percentage);
        s.gpa = strategy->calculateGPA(s.percentage);
    }
};

// SRP: Only handles file operations
class FileHandler {
public:
    static void saveToFile(const vector<Student> &students, const string &filename = "students.txt") {
        ofstream file(filename);
        for (const auto &s : students) {
            file << s.name << " " << s.rollNo << " " << s.studentClass << " "
                 << s.age << " " << s.gender;
            for (float mark : s.marks) file << " " << mark;
            file << " " << s.attendanceRecords.size();
            for (const auto &att : s.attendanceRecords) {
                file << " " << att.date << " " << att.status;
            }
            file << " " << fixed << setprecision(2) << s.gpa << "\n";
        }
    }

    static vector<Student> loadFromFile(const string &filename = "students.txt") {
        vector<Student> students;
        ifstream file(filename);
        Student s;
        while (file >> s.name >> s.rollNo >> s.studentClass >> s.age >> s.gender >>
               s.marks[0] >> s.marks[1] >> s.marks[2] >> s.marks[3] >> s.marks[4]) {
            int numRecords;
            file >> numRecords;
            s.attendanceRecords.resize(numRecords);
            for (int i = 0; i < numRecords; ++i) {
                file >> s.attendanceRecords[i].date >> s.attendanceRecords[i].status;
            }
            file >> s.gpa;
            students.push_back(s);
        }
        return students;
    }
};

// SRP: Only handles authentication
class AuthManager {
private:
    static const string username;
    static const string password;
public:
    static bool authenticate() {
        string user, pass;
        cout << "Username: ";
        cin >> user;
        cout << "Password: ";
        cin >> pass;
        return user == username && pass == password;
    }
};

const string AuthManager::username = "admin";
const string AuthManager::password = "1234";

// ==================== STUDENT OPERATIONS ====================

// SRP: Handles core student operations
// DIP: Depends on IGradeCalculator abstraction
class StudentOperations {
protected:
    vector<Student> students;
    shared_ptr<IGradeCalculator> gradeCalc;  // DIP: Using abstraction

public:
    // DIP: Dependency injected through constructor
    StudentOperations(shared_ptr<IGradeCalculator> strategy)
        : gradeCalc(move(strategy)) {}

    virtual void addStudent() {
        Student s;
        cout << "Enter name: ";
        cin.ignore();
        getline(cin, s.name);
        cout << "Enter roll number: ";
        cin >> s.rollNo;
        cout << "Enter class: ";
        cin.ignore();
        getline(cin, s.studentClass);
        cout << "Enter age: ";
        cin >> s.age;
        cout << "Enter gender: ";
        cin.ignore();
        getline(cin, s.gender);

        gradeCalc->calculateGrade(s);
        students.push_back(s);
        cout << "Student added successfully.\n";
    }

    virtual void viewAllStudents() const {
        cout << left << setw(10) << "Roll" << setw(20) << "Name" << setw(10) << "Class"
             << setw(6) << "Age" << setw(10) << "Gender" << setw(10) << "Percentage"
             << setw(8) << "Grade" << setw(8) << "GPA" << "Attendance%\n";

        for (const auto &s : students) {
            float attendancePercent = calculateAttendancePercentage(s);
            cout << setw(10) << s.rollNo << setw(20) << s.name << setw(10) << s.studentClass
                 << setw(6) << s.age << setw(10) << s.gender << setw(10) << fixed
                 << setprecision(2) << s.percentage << setw(8) << s.grade
                 << setw(8) << fixed << setprecision(2) << s.gpa
                 << attendancePercent << "%\n";
        }
    }

    virtual void searchStudent() const {
        int roll;
        cout << "Enter roll number: ";
        cin >> roll;

        auto it = find_if(students.begin(), students.end(),
                         [roll](const Student &s) { return s.rollNo == roll; });

        if (it != students.end()) {
            const auto &s = *it;
            float attendancePercent = calculateAttendancePercentage(s);
            cout << "\nStudent Details:\n"
                 << "Name: " << s.name << "\n"
                 << "Class: " << s.studentClass << "\n"
                 << "Age: " << s.age << "\n"
                 << "Gender: " << s.gender << "\n"
                 << "Percentage: " << s.percentage << "%\n"
                 << "Grade: " << s.grade << "\n"
                 << "GPA: " << fixed << setprecision(2) << s.gpa << "\n"
                 << "Attendance: " << attendancePercent << "%\n"
                 << "Attendance Records (" << s.attendanceRecords.size() << "):\n";
            for (const auto &att : s.attendanceRecords) {
                cout << "  " << att.date << ": " << att.status << "\n";
            }
        } else {
            cout << "Student not found.\n";
        }
    }

    virtual void updateStudent() {
        int roll;
        cout << "Enter roll number to update: ";
        cin >> roll;

        auto it = find_if(students.begin(), students.end(),
                         [roll](const Student &s) { return s.rollNo == roll; });

        if (it != students.end()) {
            Student &s = *it;
            cout << "Enter new name: ";
            cin.ignore();
            getline(cin, s.name);
            cout << "Enter new class: ";
            getline(cin, s.studentClass);
            cout << "Enter new age: ";
            cin >> s.age;
            cout << "Enter new gender: ";
            cin.ignore();
            getline(cin, s.gender);

            gradeCalc->calculateGrade(s);
            cout << "Student updated successfully.\n";
        } else {
            cout << "Student not found.\n";
        }
    }

    virtual void deleteStudent() {
        int roll;
        cout << "Enter roll number to delete: ";
        cin >> roll;

        auto new_end = remove_if(students.begin(), students.end(),
                                [roll](const Student &s) { return s.rollNo == roll; });

        if (new_end != students.end()) {
            students.erase(new_end, students.end());
            cout << "Student deleted successfully.\n";
        } else {
            cout << "Student not found.\n";
        }
    }

    virtual void sortStudents() {
        sort(students.begin(), students.end(),
             [](const Student &a, const Student &b) { return a.rollNo < b.rollNo; });
        cout << "Students sorted by roll number.\n";
    }

    void saveData() const {
        FileHandler::saveToFile(students);
        cout << "Data saved successfully.\n";
    }

    void loadData() {
        students = FileHandler::loadFromFile();
        for (auto &s : students) {
            gradeCalc->calculateGrade(s);
        }
    }

protected:
    float calculateAttendancePercentage(const Student &s) const {
        if (s.attendanceRecords.empty()) return 0.0f;
        int presentCount = count_if(s.attendanceRecords.begin(), s.attendanceRecords.end(),
                                  [](const AttendanceRecord &r) { return r.status == "Present"; });
        return (static_cast<float>(presentCount) / s.attendanceRecords.size()) * 100;
    }
};

// ==================== EXTENDED FUNCTIONALITY ====================

// OCP: Extends StudentOperations without modifying it
// DIP: Depends on abstractions (IExporter, IReportGenerator)
class ExtendedStudentOperations : public StudentOperations {
    shared_ptr<IExporter> exporter;          // DIP: Using abstraction
    shared_ptr<IReportGenerator> reportGenerator;  // DIP: Using abstraction

public:
    // DIP: Dependencies injected through constructor
    ExtendedStudentOperations(shared_ptr<IGradeCalculator> gradeStrategy,
                              shared_ptr<IExporter> exp,
                              shared_ptr<IReportGenerator> repGen)
        : StudentOperations(move(gradeStrategy)),
          exporter(move(exp)),
          reportGenerator(move(repGen)) {}

    void markAttendance() {
        string date;
        cout << "Enter date (YYYY-MM-DD): ";
        cin >> date;

        for (auto &s : students) {
            cout << "Mark attendance for " << s.name << " (P/A): ";
            char a;
            cin >> a;
            s.attendanceRecords.push_back({date, (toupper(a) == 'P') ? "Present" : "Absent"});
        }
        cout << "Attendance marked for " << date << "\n";
    }

    void viewAttendanceByDate() const {
        string date;
        cout << "Enter date to view attendance (YYYY-MM-DD): ";
        cin >> date;

        cout << "Attendance for " << date << ":\n";
        cout << left << setw(10) << "Roll" << setw(20) << "Name" << setw(10) << "Status\n";

        bool found = false;
        for (const auto &s : students) {
            auto it = find_if(s.attendanceRecords.begin(), s.attendanceRecords.end(),
                             [&date](const AttendanceRecord &r) { return r.date == date; });
            if (it != s.attendanceRecords.end()) {
                cout << setw(10) << s.rollNo << setw(20) << s.name
                     << setw(10) << it->status << "\n";
                found = true;
            }
        }
        if (!found) {
            cout << "No attendance records found for " << date << "\n";
        }
    }

    void viewMonthlyAttendance() const {
        string monthYear;
        cout << "Enter month and year (MM-YYYY): ";
        cin >> monthYear;

        cout << "Monthly Attendance Report for " << monthYear << ":\n";
        cout << left << setw(10) << "Roll" << setw(20) << "Name" << setw(10) << "Present"
             << setw(10) << "Absent" << setw(15) << "Attendance%\n";

        for (const auto &s : students) {
            int present = 0, total = 0;
            for (const auto &att : s.attendanceRecords) {
                if (att.date.substr(5, 2) + "-" + att.date.substr(0, 4) == monthYear) {
                    total++;
                    if (att.status == "Present") present++;
                }
            }
            if (total > 0) {
                float percent = (static_cast<float>(present) / total) * 100;
                cout << setw(10) << s.rollNo << setw(20) << s.name
                     << setw(10) << present << setw(10) << (total - present)
                     << fixed << setprecision(2) << percent << "%\n";
            }
        }
    }

    void enterMarks() {
        int roll;
        cout << "Enter roll number: ";
        cin >> roll;

        auto it = find_if(students.begin(), students.end(),
                         [roll](const Student &s) { return s.rollNo == roll; });

        if (it != students.end()) {
            Student &s = *it;
            cout << "Enter marks for 5 subjects (space separated): ";
            for (int i = 0; i < 5; ++i) {
                cin >> s.marks[i];
            }
            gradeCalc->calculateGrade(s);
            cout << "Marks updated. New grade: " << s.grade << "\n";
        } else {
            cout << "Student not found.\n";
        }
    }

    void calculateGPA() const {
        cout << left << setw(20) << "Name" << setw(10) << "GPA (5.0 scale)\n";
        for (const auto &s : students) {
            cout << setw(20) << s.name << setw(10) << fixed << setprecision(2)
                 << (s.gpa * 5.0 / 4.0) << "\n";
        }
    }

    // DIP: Delegates to reportGenerator abstraction
    void generateClassReport() const {
        reportGenerator->generateReport(students);
    }

    // DIP: Delegates to exporter abstraction
    void exportData() const {
        exporter->exportData(students);
    }

    void backupData() const {
        time_t now = time(nullptr);
        char buffer[80];
        strftime(buffer, sizeof(buffer), "backup_%Y%m%d_%H%M%S.txt", localtime(&now));
        string filename = buffer;
        FileHandler::saveToFile(students, filename);
        cout << "Backup created successfully: " << filename << "\n";
    }

    void showStatistics() const {
        string cls;
        cout << "Enter class for statistics: ";
        cin >> cls;

        vector<Student> classStudents;
        for (const auto &s : students) {
            if (s.studentClass == cls)
                classStudents.push_back(s);
        }

        if (classStudents.empty()) {
            cout << "No students found in class " << cls << "\n";
            return;
        }

        float totalPercentage = 0;
        float totalGPA = 0;
        float totalAttendance = 0;
        map<char, int> gradeCount;

        for (const auto &s : classStudents) {
            totalPercentage += s.percentage;
            totalGPA += s.gpa;
            totalAttendance += calculateAttendancePercentage(s);
            gradeCount[s.grade]++;
        }

        int count = classStudents.size();
        cout << "\nClass " << cls << " Statistics:\n";
        cout << "Total Students: " << count << "\n";
        cout << "Average Percentage: " << fixed << setprecision(2)
             << (totalPercentage / count) << "%\n";
        cout << "Average GPA (4.0 scale): " << fixed << setprecision(2)
             << (totalGPA / count) << "\n";
        cout << "Average GPA (5.0 scale): " << fixed << setprecision(2)
             << (totalGPA / count * 5.0 / 4.0) << "\n";
        cout << "Average Attendance: " << fixed << setprecision(2)
             << (totalAttendance / count) << "%\n";
        cout << "Grade Distribution:\n";
        for (const auto &pair : gradeCount) {
            cout << "Grade " << pair.first << ": " << pair.second << " students\n";
        }
    }

    void importFromCSV() {
        string filename;
        cout << "Enter CSV filename to import: ";
        cin.ignore();
        getline(cin, filename);

        ifstream file(filename);
        if (!file.is_open()) {
            cout << "Failed to open file: " << filename << "\n";
            return;
        }

        string line;
        getline(file, line); // Skip header
        while (getline(file, line)) {
            stringstream ss(line);
            Student s;
            string temp;

            getline(ss, temp, ','); // Roll
            s.rollNo = stoi(temp);
            getline(ss, s.name, ',');
            getline(ss, s.studentClass, ',');
            getline(ss, temp, ','); // Age
            s.age = stoi(temp);
            getline(ss, s.gender, ',');
            getline(ss, temp, ','); // Percentage
            s.percentage = stof(temp);
            getline(ss, temp, ','); // Grade
            s.grade = temp[0];
            getline(ss, temp, ','); // GPA
            s.gpa = stof(temp);

            students.push_back(s);
            gradeCalc->calculateGrade(s);
        }
        cout << "Data imported successfully from " << filename << "\n";
    }

    void findTopper() const {
        string cls;
        cout << "Enter class to find topper: ";
        cin >> cls;

        float maxPercentage = -1;
        const Student *topper = nullptr;

        for (const auto &s : students) {
            if (s.studentClass == cls && s.percentage > maxPercentage) {
                maxPercentage = s.percentage;
                topper = &s;
            }
        }

        if (topper) {
            float attendancePercent = calculateAttendancePercentage(*topper);
            cout << "Topper of class " << cls << ":\n";
            cout << "Name: " << topper->name << "\n";
            cout << "Roll No: " << topper->rollNo << "\n";
            cout << "Percentage: " << fixed << setprecision(2) << topper->percentage << "%\n";
            cout << "Grade: " << topper->grade << "\n";
            cout << "GPA (4.0 scale): " << fixed << setprecision(2) << topper->gpa << "\n";
            cout << "GPA (5.0 scale): " << fixed << setprecision(2) << (topper->gpa * 5.0 / 4.0) << "\n";
            cout << "Attendance: " << attendancePercent << "%\n";
        } else {
            cout << "No students found in class " << cls << "\n";
        }
    }
};

// ==================== MENU SYSTEM ====================

// SRP: Only handles menu presentation and flow
// DIP: Depends on ExtendedStudentOperations abstraction
class MenuSystem {
    unique_ptr<ExtendedStudentOperations> ops;  // DIP: Using abstraction
    map<int, function<void()>> menuActions;

    void initializeMenu() {
        menuActions[1] = [this]() { ops->addStudent(); };
        menuActions[2] = [this]() { ops->viewAllStudents(); };
        menuActions[3] = [this]() { ops->searchStudent(); };
        menuActions[4] = [this]() { ops->updateStudent(); };
        menuActions[5] = [this]() { ops->deleteStudent(); };
        menuActions[6] = [this]() { ops->enterMarks(); };
        menuActions[7] = [this]() { ops->calculateGPA(); };
        menuActions[8] = [this]() { ops->markAttendance(); };
        menuActions[9] = [this]() { ops->generateClassReport(); };
        menuActions[10] = [this]() { ops->exportData(); };
        menuActions[11] = [this]() { ops->sortStudents(); };
        menuActions[12] = [this]() { ops->backupData(); };
        menuActions[13] = [this]() { ops->showStatistics(); };
        menuActions[14] = [this]() { ops->importFromCSV(); };
        menuActions[15] = [this]() { ops->findTopper(); };
        menuActions[16] = [this]() { ops->viewAttendanceByDate(); };
        menuActions[17] = [this]() { ops->viewMonthlyAttendance(); };
    }

public:
    MenuSystem() {
        // DIP: Creating concrete implementations and injecting them
        auto gradeStrategy = make_shared<DefaultGradeStrategy>();
        auto exporter = make_shared<CSVExporter>();
        auto reportGen = make_shared<TextReportGenerator>();
        auto gradeCalc = make_shared<GradeCalculator>(gradeStrategy);

        // DIP: Creating ExtendedStudentOperations with dependencies
        ops = make_unique<ExtendedStudentOperations>(gradeCalc, exporter, reportGen);
        ops->loadData();
        initializeMenu();
    }

    void run() {
        if (!AuthManager::authenticate()) {
            cout << "Authentication failed. Exiting...\n";
            return;
        }

        int choice;
        do {
            cout << "\n==== Student Management System ====\n";
            cout << "1. Add Student\n2. View All Students\n3. Search Student\n"
                 << "4. Update Student\n5. Delete Student\n6. Enter Marks\n"
                 << "7. Calculate GPA\n8. Mark Attendance\n9. Class Report\n"
                 << "10. Export Data\n11. Sort Students\n12. Backup Data\n"
                 << "13. Show Statistics\n14. Import from CSV\n15. Find Topper\n"
                 << "16. View Attendance by Date\n17. View Monthly Attendance\n"
                 << "18. Save & Exit\n"
                 << "Enter choice: ";

            cin >> choice;

            if (choice == 18) {
                ops->saveData();
                cout << "Exiting system...\n";
                break;
            }

            if (menuActions.count(choice)) {
                menuActions[choice]();
            } else {
                cout << "Invalid choice. Try again.\n";
            }

        } while (true);
    }
};

// ==================== MAIN FUNCTION ====================

int main() {
    // DIP: High-level module depends on abstraction
    MenuSystem system;
    system.run();
    return 0;
}
