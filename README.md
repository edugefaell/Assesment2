# Assesment2
Code for organizing courses and analitics
// PHYS30762 Programming in C++
// Assignment 2

// Program to compute mean, standard deviation and standard
// error of the a set of courses. Data is read from file
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <sstream>
#include <iomanip>
#include <cmath>
#include <algorithm>

class Course 
{
public:
  int code; 
  std::string title;
  std::vector<double> marks;
  double mean;
  double std_deviation;
  double std_error;

  Course(int c, std::string t) : code(c), title(t), mean(0), std_deviation(0), std_error(0) {}

  void calculateStatistics() 
  {
    if(marks.empty()) return; // Guard against division by zero

    double sum = 0;
    for(double mark : marks) 
    {
      sum += mark;
    }
    mean = sum / marks.size();

    double variance_sum = 0;
    for(double mark : marks) 
    {
      variance_sum += std::pow(mark - mean, 2);
    }
    std_deviation = std::sqrt(variance_sum / (marks.size() - 1));
    std_error = std_deviation / std::sqrt(marks.size());
  }
};
//Function to print statistics.
void printOverallStatistics(const std::vector<Course>& courses) 
{
  if(courses.empty()) 
  {
    std::cout << "No courses available to calculate statistics." << std::endl;
    return;
  }

  double total_Sum = 0;
  int total_Marks = 0;

  for(const auto& course : courses) 
  {
    total_Sum += course.mean * course.marks.size();
    total_Marks += course.marks.size();
  }

  double overall_Mean = total_Sum / total_Marks;
  double sum_Of_SquaredDiffs = 0;

  for(const auto& course : courses) 
  {
    for(double mark : course.marks) 
    {
     sum_Of_SquaredDiffs += std::pow(mark - overall_Mean, 2);
    }
  }

  double overall_Std_Deviation = std::sqrt(sum_Of_SquaredDiffs / (total_Marks - 1));
  double overall_Std_Error = overall_Std_Deviation / std::sqrt(total_Marks);

  std::cout << "Overall Mean: " << overall_Mean
            << ", Overall Std Deviation: " << overall_Std_Deviation
            << ", Overall Std Error: " << overall_Std_Error << std::endl;
}
//Function to sort and print selected courses. 
void printCourses(const std::vector<Course>& courses, bool sortByCode) 
{
  std::vector<Course> sortedCourses = courses;

  if(sortByCode) 
  {
    std::sort(sortedCourses.begin(), sortedCourses.end(), [](const Course& a, const Course& b) 
    {
      return a.code < b.code;
    });
  } else 
  {
     std::sort(sortedCourses.begin(), sortedCourses.end(), [](const Course& a, const Course& b)
    {
      return a.title < b.title;
    });
  }

  for(auto it = sortedCourses.begin(); it != sortedCourses.end(); ++it)
  {
    std::stringstream ss;
    ss << "PHYS " << it->code << " " << it->title;
    std::cout << ss.str() << std::endl;
  }
}

int main() 
{
  std::ifstream file("courselist.dat");
  if(!file) 
  {
    std::cerr << "Could not open the file." << std::endl;
    return 1;
  }

  std::vector<Course> courses;
  std::string line;
  int recordCount = 0;
  while(std::getline(file, line)) 
  {
    std::istringstream iss(line);
    double mark;
    int courseCode;
    iss >> mark >> courseCode;
    std::string title;
    std::getline(iss, title);
    auto it = std::find_if(courses.begin(), courses.end(), [courseCode](const Course& c) { return c.code == courseCode; });
    if(it == courses.end()) 
    {
      courses.emplace_back(courseCode, title);
      it = std::prev(courses.end());
    }
    it->marks.push_back(mark);
    ++recordCount;
  }

  std::cout << "Total number of records: " << recordCount << std::endl;

  // User interaction to print courses based on year or all, and sorting preference
  std::cout << "Do you want to print all courses or for a specific year? (all/year): ";
  std::string choice;
  std::cin >> choice;

  std::vector<Course> filteredCourses;
  if(choice == "year")
  {
    std::cout << "Enter the first digit of the course code for the specific year: ";
    char year;
    std::cin>> year;
    std::copy_if(courses.begin(), courses.end(), std::back_inserter(filteredCourses), [year](const Course& course)
    {
      return std::to_string(course.code).front() == year;
    });
    std::cout << "Number of records for the selected year: " << filteredCourses.size() << std::endl;
  } else 
  {
    filteredCourses = courses;
  }

  // Calculate statistics for each course in the filtered selection
  for(Course& course : filteredCourses) 
  {
    course.calculateStatistics();
  }

  // Print overall statistics for the filtered selection
  printOverallStatistics(filteredCourses);

  // Ask user for sorting preference
  std::cout << "Should courses be sorted by title or by course code? (title/code): ";
  std::cin >> choice;
  bool sortByCode = choice == "code";

  // Print the courses as per user sorting preference
  std::cout << "Courses:" << std::endl;
  printCourses(filteredCourses, sortByCode);

  return 0;

}
