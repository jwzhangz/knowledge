
## Testing enterprise applications
![](http://jmockit.org/tutorial/TestedAPI.png)  
An enterprise application targets a particular business domain, usually having a GUI for multiple concurrent users and an application database for many entity types; also, it often integrates with other applications inside or outside the organization. In Java, the Java EE APIs and/or the Spring framework are typically used when building such applications.

In this chapter we describe an approach to test Java enterprise applications by writing out-of-container integration tests, where each test exercises a single step in a well-defined business scenario (also known as a "use case" or "usage" scenario). With a typical layered architecture, such a test calls a public method from a component in the highest layer (normally, the application layer), which then calls down to lower layers.

### An example
For demonstration, we will use a Java EE version of the Spring Pet Clinic sample application. The full code is available in the project repository. The application codebase is organized in four layers: UI or presentation layer, application layer, domain layer, and infrastructure layer.

The application's domain model (following the approach and terminology of Domain Driven Design) has six domain entities: Vet (a veterinarian), Specialty (a vet's specialty), Pet, PetType, Owner (a pet's owner), and Visit (a visit from a pet and its owner to the clinic). Besides entities, the domain model (and layer) of the application also includes domain service classes. In this simple domain, we have only one such class for each entity type (VetMaintenance, PetMaintenance, and so on).

In DDD, entities are added into and reconstituted or removed from persistent storage through "repository" components. Given that we use a sophisticated ORM API (JPA), there is only one such repository, which is not domain or application specific and therefore goes into the infrastructure layer: the Database class. The application uses a relational database, specifically an in-memory HSqlDb database in the sample application, so that it can be self-contained.

The application layer contains application service classes, which translate user input from the UI to calls into the lower layers, and make output data available for display in the UI. This is the layer at which database transactions are demarcated.
#### Using Java EE
With Java EE 7, we use JPA for the domain @Entity types, EJBs (stateless session beans) or simply @Transactional classes for domain services, and JSF @ViewScoped beans for the application services. Code for the UI layer is not included in the sample, as it wouldn't be exercised by the integration test suite anyway. (In Java EE, this layer would be comprised of JSF facelets in the form of ".xhtml" files.)

For our first integration test class, let's consider the Vet screen, which simply displays a list of all vets with their specialties.

```java
public final class VetScreenTest
{
   @TestUtil VetData vetData;
   @SUT VetScreen vetScreen;

   @Test
   public void findVets()
   {
      // Inserts input data (instances of Vet and Specialty) into the database.
      Vet vet2 = vetData.create("Helen Leary", "radiology");
      Vet vet0 = vetData.create("James Carter");
      Vet vet1 = vetData.create("Linda Douglas", "surgery", "dentistry");
      List<Vet> vetsInOrderOfLastName = asList(vet0, vet1, vet2);

      // Exercises the code under test (VetScreen, VetMaintenance, Vet, Specialty).
      vetScreen.showVetList();
      List<Vet> vets = vetScreen.getVets();

      // Verifies the output is as expected.
      vets.retainAll(vetsInOrderOfLastName);
      assertEquals(vetsInOrderOfLastName, vets); // checks the contents and ordering of the list

      Vet vetWithSpecialties = vets.get(1); // this will be "vet1"...
      assertEquals(2, vetWithSpecialties.getNrOfSpecialties()); // ...which we know has two specialties

      vetData.refresh(vetWithSpecialties); // ensures the Vet contains data actually in the db
      List<Specialty> specialtiesInOrderOfName = vetWithSpecialties.getSpecialties();
      assertEquals("dentistry", specialtiesInOrderOfName.get(0).getName()); // checks that specialties...
      assertEquals("surgery", specialtiesInOrderOfName.get(1).getName()); // ...are in the correct order
   }
}
```
The first thing to notice in the test above is that it starts at the topmost layer of the application, which is coded in Java and therefore runs entirely in the JVM: the application layer. So, the test is not concerned with any HTTP requests and responses, or how application URLs map to components in the application layer; such details vary with the technology used for UI implementation (JSF, JSP, Struts, GWT, Spring MVC, etc.), and are considered out of the scope of the integration tests.

The second thing to notice is the test code is very clean and focused on what it's trying to test. It's written entirely in terms of the application and its business domain, without low-level concerns such as deployment, database configuration, or transactions.

Finally, the third thing to notice is that no JMockit API at all appears in the test class. All we have is the use of the @TestUtil and @SUT annotations. These are user-defined annotations, with arbitrary names chosen according to the preferences of the team. In our sample code, they are defined as follows.

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Tested(availableDuringSetup = true, fullyInitialized = true)
public @interface TestUtil {} // a test utility object

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Tested(fullyInitialized = true)
public @interface SUT {} // the System Under Test
```
