import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

@SpringBootApplication
@RestController
@RequestMapping("/api")
public class StudentDetailsApplication {

    private List<Student> studentDetails;

    public static void main(String[] args) {
        SpringApplication.run(StudentDetailsApplication.class, args);
    }

    @GetMapping("/students")
    public ResponseEntity<List<Student>> getStudents(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int pageSize
    ) {
        loadData();
        int totalStudents = studentDetails.size();
        int totalPages = (totalStudents + pageSize - 1) / pageSize;

        int startIndex = (page - 1) * pageSize;
        int endIndex = Math.min(startIndex + pageSize, totalStudents);

        List<Student> students = studentDetails.subList(startIndex, endIndex);

        // Create a custom response with pagination details
        PaginatedResponse<List<Student>> response = new PaginatedResponse<>(page, pageSize, totalPages, totalStudents, students);
        return ResponseEntity.ok(response);
    }

    @PostMapping("/students/filter")
    public ResponseEntity<List<Student>> filterStudents(@RequestBody FilterCriteria filterCriteria) {
        loadData();

        List<Student> filteredStudents = new ArrayList<>();
        for (Student student : studentDetails) {
            // Apply filtering logic based on filter criteria
            if (filterCriteria.getId() != null && !student.getId().equals(filterCriteria.getId())) {
                continue;
            }
            if (filterCriteria.getName() != null && !student.getName().equalsIgnoreCase(filterCriteria.getName())) {
                continue;
            }
            if (filterCriteria.getTotalMarks() != null && student.getTotalMarks() != filterCriteria.getTotalMarks()) {
                continue;
            }

            filteredStudents.add(student);
        }

        return ResponseEntity.ok(filteredStudents);
    }

    private void loadData() {
        if (studentDetails == null) {
            studentDetails = new ArrayList<>();
            try (BufferedReader reader = new BufferedReader(new FileReader("student_details.csv"))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    String[] data = line.split(",");
                    int id = Integer.parseInt(data[0]);
                    String name = data[1];
                    int totalMarks = Integer.parseInt(data[2]);
                    Student student = new Student(id, name, totalMarks);
                    studentDetails.add(student);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
