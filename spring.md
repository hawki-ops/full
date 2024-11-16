# spring

controller

```java

@RestController
@RequestMapping("/fields")
public class FieldController {
    @Autowired
    private FieldService fieldService;

    @PostMapping
    public Field addField(@RequestBody Map<String, Object> request) {
        Long sectionId = ((Number) request.get("sectionId")).longValue();
        String name = (String) request.get("name");
        String type = (String) request.get("type");
        return fieldService.addField(sectionId, name, type);
    }
}

@RestController
@RequestMapping("/sections")
public class SectionController {
    @Autowired
    private SectionService sectionService;

    @PostMapping
    public Section addSection(@RequestBody Map<String, Object> request) {
        String name = (String) request.get("name");
        return sectionService.addSection( name);
    }
}

@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserInfoService userInfoService;

    @Autowired
    private UserService userService;

    @PostMapping
    public ResponseEntity<?> createUser(@RequestBody CreateUserRequestDTO requestDTO) {
        userService.createUser(requestDTO);
        return ResponseEntity.ok().body(Collections.singletonMap("message", "User added successfully!"));
    }

    @GetMapping("/{userId}/info")
    public UserInfoDTO getUserInfo(@PathVariable Long userId) {
        return userInfoService.getUserInfo(userId);
    }

    @Autowired
    private ValueService valueService;

    @PostMapping("/{userId}/values")
    public ResponseEntity<?> addValueToUser(@PathVariable Long userId, @RequestBody AddValueDTO requestDTO) {
        valueService.addValueToUser(userId, requestDTO);
        return ResponseEntity.ok().body(Collections.singletonMap("message", "Value added successfully!"));
    }
}
```

dto
```java
@Setter @Getter
public class AddValueDTO {
    private Long fieldId;
    private String value;

    // Getters and setters
}

@Setter @Getter
public class CreateUserRequestDTO {
    private String username;
    private String email;
    private String password;

    // Getters and setters
}

@Setter
@Getter
public class FieldInfoDTO {
    private String name;
    private String value;

    // Getters and setters
}

@Setter @Getter
public class SectionInfoDTO {
    private String name;
    private List<FieldInfoDTO> fields;

}


@Getter @Setter
public class UserInfoDTO {
    private Long userId;
    private String username;
    private List<SectionInfoDTO> sections;

    // Getters and setters
}
```
model
```java
@Getter @Setter
@Entity
public class Field {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String type;

    @ManyToOne
    @JoinColumn(name = "section_id")
    private Section section;
}

@Getter
@Setter
@Entity
public class JValue {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String jvalue;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private WebUser user;

    @ManyToOne
    @JoinColumn(name = "field_id")
    private Field field;
}


@Getter
@Setter
@Entity
public class OptionValue {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String jvalue;

    @ManyToOne
    @JoinColumn(name = "field_id")
    private Field field;
}

@Getter
@Setter
@Entity
public class Section {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "section", cascade = CascadeType.ALL)
    private List<Field> fields;


    public Section() {}

    public Section(Long sectionId) {
        this.id = sectionId;
    }
}

@Getter
@Setter
@Entity
public class WebUser {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String email;
    private String password;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<JValue> JValues;
}
```
repo
```java
@Repository
public interface FieldRepo extends JpaRepository<Field, Long> {}

@Repository
public interface SectionRepo extends JpaRepository<Section, Long> {
}

@Repository
public interface UserRepo extends JpaRepository<WebUser, Long> {
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}

@Repository
public interface ValueRepo extends JpaRepository<JValue, Long> {

    List<JValue> findAllByUserId(Long userId);

}

```
service
```java
@Service
public class FieldService {
    @Autowired
    private FieldRepo fieldRepo;

    public Field addField(Long sectionId, String name, String type) {
        Field field = new Field();
        field.setName(name);
        field.setType(type);
        field.setSection(new Section(sectionId));
        return fieldRepo.save(field);
    }
}

@Service
public class SectionService {
    @Autowired
    private SectionRepo sectionRepo;

    public Section addSection(String name) {
        Section section = new Section();
        section.setName(name);
        return sectionRepo.save(section);
    }
}

@Service
public class UserInfoService {
    @Autowired
    private UserRepo userRepository;

    @Autowired
    private ValueRepo valueRepository;

    public UserInfoDTO getUserInfo(Long userId) {
        WebUser user = userRepository.findById(userId).orElseThrow(() -> new RuntimeException("User not found"));

        // Fetch values and organize by section
        List<JValue> JValues = valueRepository.findAllByUserId(userId);

        Map<String, SectionInfoDTO> sectionsMap = new HashMap<>();
        for (JValue JValue : JValues) {
            Field field = JValue.getField();
            Section section = field.getSection();

            // Add section if not already in the map
            SectionInfoDTO sectionDTO = sectionsMap.computeIfAbsent(section.getName(), key -> {
                SectionInfoDTO dto = new SectionInfoDTO();
                dto.setName(section.getName());
                dto.setFields(new ArrayList<>());
                return dto;
            });

            // Add field to the section
            FieldInfoDTO fieldDTO = new FieldInfoDTO();
            fieldDTO.setName(field.getName());
            fieldDTO.setValue(JValue.getJvalue());
            sectionDTO.getFields().add(fieldDTO);
        }

        // Prepare final DTO
        UserInfoDTO userInfoDTO = new UserInfoDTO();
        userInfoDTO.setUserId(user.getId());
        userInfoDTO.setUsername(user.getUsername());
        userInfoDTO.setSections(new ArrayList<>(sectionsMap.values()));

        return userInfoDTO;
    }
}

@Service
public class UserService {
    @Autowired
    private UserRepo userRepository;

    public void createUser(CreateUserRequestDTO requestDTO) {
        // Check for duplicate username or email
        if (userRepository.existsByUsername(requestDTO.getUsername())) {
            throw new RuntimeException("Username already exists");
        }
        if (userRepository.existsByEmail(requestDTO.getEmail())) {
            throw new RuntimeException("Email already exists");
        }

        // Create and save new user
        WebUser user = new WebUser();
        user.setUsername(requestDTO.getUsername());
        user.setEmail(requestDTO.getEmail());
        user.setPassword(requestDTO.getPassword()); // Hash password if needed

        userRepository.save(user);
    }
}

@Service
public class ValueService {
    @Autowired
    private UserRepo userRepository;

    @Autowired
    private FieldRepo fieldRepository;

    @Autowired
    private ValueRepo valueRepository;

    public void addValueToUser(Long userId, AddValueDTO requestDTO) {
        // Fetch user
        WebUser user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // Fetch field
        Field field = fieldRepository.findById(requestDTO.getFieldId())
                .orElseThrow(() -> new RuntimeException("Field not found"));

        // Create and save new value
        JValue JValue = new JValue();
        JValue.setUser(user);
        JValue.setField(field);
        JValue.setJvalue(requestDTO.getValue());

        valueRepository.save(JValue);
    }
}



```
