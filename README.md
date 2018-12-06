### spring boot integration test
- `mongodb`
- `mysql` `h2`
- `redis`

#### mongodb
- gradle
```
   compile('org.springframework.boot:spring-boot-starter-data-mongodb')
   
   testCompile('de.flapdoodle.embed:de.flapdoodle.embed.mongo')
```

- entity

```
@Document(collection = "contact")
@Getter
@Builder(toBuilder = true)
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class ContactPerson {

  @JsonFormat(shape = JsonFormat.Shape.STRING)
  private Long id;

  private String contactPerson;

  private String contactPhone;
}
```
- Mongo Respository
```
@RepositoryRestResource(collectionResourceRel = "contact", path = "contact", exported = true)
public interface ContactPersonRepository extends MongoRepository<ContactPerson, Long> {

  public ContactPerson save(ContactPerson contactPerson);

  public List<ContactPerson> findByContactPersonLike(@Param("contactPerson") String contactPerson);

}
```
- controller
```
  @PostMapping("/mongo")
  public ResponseEntity<?> saveContactPerson(@RequestBody ContactPerson contactPerson) {
    return ResponseEntity.ok(contactPersonRepository.save(contactPerson));
  }

  @GetMapping("/mongo/{name}")
  public ResponseEntity<?> getContactPerson(@PathVariable String name) {
    return ResponseEntity.ok(contactPersonRepository.findByContactPersonLike(name));
  }
```

- controller test
```
  @Test
  public void testFindConstactPersonByName() throws Exception {
    ContactPerson contactPerson =
        ContactPerson.builder()
            .id(100L)
            .contactPerson("ceshi1")
            .contactPhone("13500110022")
            .address("北京市朝阳区")
            .remark("ceshi")
            .build();
    mongoTemplate.insert(contactPerson);
    mockMvc
        .perform(get("/v1/api/mmr/mongo/{name}", "ceshi"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$", hasSize(1)))
        .andExpect(jsonPath("$.[0].contactPerson").value("ceshi1"));
  }
```
