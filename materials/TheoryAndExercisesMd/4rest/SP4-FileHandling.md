<!-- Slide number: 1 -->
# Server ProgrammingFile upload
Juha Hinkula

### Notes:

<!-- Slide number: 2 -->
# Spring Boot – File upload
Add new controller for file uploading

@Controller
public class FileController {
}

Add method into controller that maps ’/’ – endpoint to upload.html thymeleaf template (GET method)

@GetMapping("/")
public String index() {
  return "upload";
}

2
Server Programming
12.2.2019

<!-- Slide number: 3 -->
# Spring Boot – File upload
Add upload.html thymeleaf template to resources/templates folder
enctype defines how data is encoded when sending to server. Use multipart/form-data in the case of file upload.
Form is posted to /upload endpoint
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
  <h1>File upload demo</h1>
  <form method="POST" action="/upload" enctype="multipart/form-data">
  <input type="file" name="file" /><br/>
  <input type="submit" value="Submit" />
</form>
</body>
</html>
3
Server Programming
12.2.2019

<!-- Slide number: 4 -->
# Spring Boot – File upload
Add method into controller for /upload endpoint (POST from the upload.html)
File can be accessed using RequestParam with id ’file’ (Name of the file input field in upload.html is file)
Spring MultipartFile can be used to temporarily store uploaded file.

@PostMapping("/upload")
public String fileUpload(@RequestParam("file") MultipartFile 	file, Model model) {
  // File handling
}

4
Server Programming
12.2.2019

<!-- Slide number: 5 -->
# Spring Boot – File upload
Check if the file is empty (no file has been chosen in upload form or file has no content) in fileUpload method. If it is empty move to uploadstatus thymeleaf template and add message to model attribute.

// In fileUpload method
if (file.isEmpty()) {
  model.addAttribute("msg", "Upload failed");
  return "uploadstatus";
}

5
Server Programming
12.2.2019

<!-- Slide number: 6 -->
# Spring Boot – File upload
After empty file check we will add file handling that save uploaded file to upload folder.
Upload folder can be specified using application.properties file
Add following line to the application.properties file

  upload.path=c:\\temp\\

You can read application.properties values using @Value(${propertyName}) annotation. Add following line at the beginning of your controller. It reads upload.path value from the application.properties file and save it to uploadFolder variable.

  @Value("${upload.path}")
  private String uploadFolder;

6
Server Programming
12.2.2019

<!-- Slide number: 7 -->
// Note! Imports
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
# Spring Boot – File upload
Add following file handling code after the empty file check in your controller. That save uploaded file to the upload folder defined in  application.properties file.

try {
  byte[] bytes = file.getBytes();
  Path path = Paths.get(uploadFolder + file.getOriginalFilename());
  Files.write(path, bytes);
  model.addAttribute("msg", "File " + file.getOriginalFilename() + " 		uploaded");
} catch (IOException e) {
   e.printStackTrace();
}
return "uploadstatus";

7
Server Programming
12.2.2019

<!-- Slide number: 8 -->
# Spring Boot – File upload
Whole fileUpload method
@PostMapping("/upload")
public String fileUpload(@RequestParam("file") MultipartFile file, Model model) {
  if (file.isEmpty()) {
    model.addAttribute("msg", "Upload failed");
    return "uploadstatus";
  }
  try {
   byte[] bytes = file.getBytes();
   Path path = Paths.get(uploadFolder + file.getOriginalFilename());
   Files.write(path, bytes);
   model.addAttribute("msg","File "+file.getOriginalFilename()+" uploaded");
  } catch (IOException e) {
    e.printStackTrace();
  }
return "uploadstatus";
}

8
Server Programming
12.2.2019

<!-- Slide number: 9 -->
# Spring Boot – File upload
Finally we will implement uploadstatus.html thymeleaf template that show the status of upload.

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
  <h3 th:text="${msg}" />
</body>
</html>

9
Server Programming
12.2.2019

<!-- Slide number: 10 -->
# Spring Boot – File upload
We also have to enable multipart file upload. Add following line to the application.properties file

spring.servlet.multipart.enabled=true

Default max file size in Spring is 1MB. You can change this value with following settings in the application.properties file

spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

10
Server Programming
12.2.2019

<!-- Slide number: 11 -->
# Spring Boot – File upload
The whole source code can be found from course demos github (Filedemo)

https://github.com/juhahinkula/ServerProgramming.git

11
Server Programming
12.2.2019

<!-- Slide number: 12 -->
# Spring Boot – File upload
How to save uploaded files to database?
Add database & JPA dependencies
Create model class for files. File content is saved to LOB (Large object) field.
@Entity
public class FileModel {
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private long id;
private String fileName, mimeType;

@Lob
private byte[] file;
...constructors, getters, setters

12
Server Programming
12.2.2019

<!-- Slide number: 13 -->
# Spring Boot – File upload
Create repository class for file model class

public interface FileModelRepository extends CrudRepository<FileModel, Long> {
}

13
Server Programming
12.2.2019

<!-- Slide number: 14 -->
# Spring Boot – File upload
In the file controller, create new FileModel entity and save it to a database.

try {
  byte[] bytes = file.getBytes();
  FileModel fileModel = new FileModel(file.getOriginalFilename(), 	file.getContentType(), bytes);
  repository.save(fileModel);

  model.addAttribute("msg", "File " + file.getOriginalFilename() + " 	uploaded");
} catch (IOException e) {
  e.printStackTrace();
}

14
Server Programming
12.2.2019

<!-- Slide number: 15 -->
# Spring Boot – File upload

![](Picture7.jpg)

![](Picture6.jpg)
15
Server Programming
12.2.2019

<!-- Slide number: 16 -->
# Spring Boot – File upload
How to list uploaded files and add download functionality?
Add method to controller for fetching all files.

@GetMapping("/files")
public String fileList(Model model) {
  model.addAttribute("files", repository.findAll());
  return "filelist";
}

16
Server Programming
12.2.2019

<!-- Slide number: 17 -->
# Spring Boot – File upload
Add filelist.html thymeleaf template to show files
...
<body>
<h1>Files</h1>
<table>
<tr>
   <th>Name</th>
   <th>Type</th>
</tr>
<tr th:each = "file : ${files}">
   <td th:text="${file.fileName}"></td>
   <td th:text="${file.mimeType}"></td>
</tr>
</table>
 <a href="/">Upload</a>
</body>
…

17
Server Programming
12.2.2019

<!-- Slide number: 18 -->
# Spring Boot – File upload
Add method to controller for file download. Spring ReponseEntity represents HTTP request or response. File content is added to the body of response. Content-Disposition header indicates that response contains attachment.
@GetMapping("/file/{id}")
public ResponseEntity<byte[]> getFile(@PathVariable Long id) {
  Optional<FileModel> fileOptional = repository.findById(id);

  if(fileOptional.isPresent()) {
   FileModel file = fileOptional.get();
   return ResponseEntity.ok()
	.header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" 	+ file.getFileName() + "\"")
	.body(file.getFile());
}

return ResponseEntity.status(404).body(null);
}

18
Server Programming
12.2.2019

<!-- Slide number: 19 -->
# Spring Boot – File upload
Add download link to filelist
...
<body>
<h1>Files</h1>
<table>
<tr>
   <th>Name</th>
   <th>Type</th>
</tr>
<tr th:each = "file : ${files}">
   <td th:text="${file.fileName}"></td>
   <td th:text="${file.mimeType}"></td>
   <td><a th:href="@{/file/{id}(id=${file.id})}">Download</a></td>
</tr>
</table>
 <a href="/">Upload</a>
</body>
…

19
Server Programming
12.2.2019

<!-- Slide number: 20 -->
# Spring Boot – File upload

![](Picture6.jpg)
20
Server Programming
12.2.2019

<!-- Slide number: 21 -->
# Spring Boot – File upload
The whole source code can be found from course demos github (FiledemoDb)

https://github.com/juhahinkula/ServerProgramming.git

21
Server Programming
12.2.2019