# Project Work Time - Collections

## Learning Goals

- Spend some time working on your project.

## Project Work Time

You can now work on Task 2 in the Module Project. Task 2 has been copied here
for your convenience.

## Task 2 - Evolve the Array to a List

Currently, the `ConcertRepository` class has an array of concerts. But now that
we have learned about more data structures, let's replace the array with a
`List`. Since a `List` is dynamically sized, as in we can continue to add
elements to it, we won't be restricted by how many concerts we could have!

1. Modify the `ConcertRepository` class.
    1. Replace the array of `Concert` objects with a `List` of `Concert` objects.
    2. Make the appropriate adjustments.
    3. Remove the instance variable `currentSize`.
    4. Remove the method `getCurrentSize()`.
    5. Create a method called `getAllConcerts()` that takes no parameters and
       returns the `List` of `Concert` objects.
2. Modify the `ConcertService` appropriately given the changes made to the
   `ConcertRepository` class.
3. Modify the following unit tests below and ensure the tests all pass.

Update the `ConcertRepositoryTest`:

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class ConcertRepositoryTest {

    private ConcertRepository repository;

    @BeforeEach
    void setUp() {
        repository = new ConcertRepository();
    }

    @Test
    void constructor() {

        // current size is 0 since no concerts have been added
        assertEquals(0, repository.getAllConcerts().size());
    }

    @Test
    void addGet1() {
        assertEquals(0, repository.getAllConcerts().size());

        // add a concert
        assertTrue(repository.add(new Concert("Artist0", 1000)));
        assertEquals(1, repository.getAllConcerts().size());

        // retrieve the concert using index 0
        Concert c = repository.get(0);
        assertEquals("Artist0", c.getPerformer());
        assertEquals(1000, c.getAvailable());

    }

    @Test
    void addGet5() {
        assertEquals(0, repository.getAllConcerts().size());

        // add 5 concerts
        for (int i = 0; i < 5; i++) {
            // add the concert
            assertTrue(repository.add(new Concert("Artist" + i, i * 1000)));

            // retrieve the concert using index i
            assertNotNull(repository.get(i));

            // confirm the current size
            assertEquals(i + 1, repository.getAllConcerts().size());
        }

        // confirm the size did not increase
        assertEquals(5, repository.getAllConcerts().size());
    }

    @Test
    void getConcertState() {
        // add 3 concerts
        assertTrue(repository.add(new Concert("The Weekend", 1000)));
        assertTrue(repository.add(new Concert("Taylor Swift", 500)));
        assertTrue(repository.add(new Concert("Harry Styles", 20000)));
        assertEquals(3, repository.getAllConcerts().size());

        // confirm each concert was inserted in the correct array position
        assertEquals("The Weekend", repository.get(0).getPerformer());
        assertEquals("Taylor Swift", repository.get(1).getPerformer());
        assertEquals("Harry Styles", repository.get(2).getPerformer());
    }

    @Test
    public void getOutOfBounds() {
        assertTrue(repository.add(new Concert("artist1", 1000)));
        assertTrue(repository.add(new Concert("artist2", 1000)));
        assertTrue(repository.add(new Concert("artist3", 1000)));

        // test that out of bounds index returns null
        assertNull(repository.get(-1));
        assertNull(repository.get(3));
    }


    @Test
    void findByPerformer() {
        repository.add(new Concert("Taylor Swift", 1000));
        repository.add(new Concert("The Weekend", 500));

        Concert c1 = repository.findByPerformer("Taylor Swift");
        assertEquals("Taylor Swift", c1.getPerformer());
        assertEquals(1000, c1.getAvailable());
        assertEquals(0, c1.getWaitlist());

        Concert c2 = repository.findByPerformer("The Weekend");
        assertEquals("The Weekend", c2.getPerformer());
        assertEquals(500, c2.getAvailable());
        assertEquals(0, c2.getWaitlist());

        //unknown performer
        Concert c3 = repository.findByPerformer("Unknown Singer");
        assertNull(c3);

    }

    @Test
    void caseInsensitiveFind() {
        repository.add(new Concert("Taylor Swift", 1000));
        repository.add(new Concert("The Weekend", 500));

        Concert c1 = repository.findByPerformer("TAYLOR swift");
        assertEquals("Taylor Swift", c1.getPerformer());
        assertEquals(1000, c1.getAvailable());
        assertEquals(0, c1.getWaitlist());

    }
}
```

Update the `ConcertServiceTest`:

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.io.ByteArrayOutputStream;
import java.io.PrintStream;

import static org.junit.jupiter.api.Assertions.*;

class ConcertServiceTest {

    private final PrintStream standardOut = System.out;
    private final ByteArrayOutputStream outputStreamCaptor = new ByteArrayOutputStream();
    private ConcertService concertService;

    @BeforeEach
    public void setUp() {
        System.setOut(new PrintStream(outputStreamCaptor));
        concertService = new ConcertService();
    }

    @AfterEach
    public void tearDown() {
        System.setOut(standardOut);
    }

    @Test
    void duplicateAdd() {
        concertService.addConcert("Taylor Swift" , 100);
        concertService.addConcert("Taylor Swift" , 200);
        assertEquals("Added concert\n" +
                        "Concert with Taylor Swift already exists. Unable to add concert",
                outputStreamCaptor.toString().trim());
    }

    @Test
    void displayEmpty() {
        concertService.displayConcerts();
        assertEquals("", outputStreamCaptor.toString().trim());
    }

    @Test
    void displayNonEmpty() {
        concertService.addConcert("Taylor Swift" , 100);
        concertService.addConcert("The Weekend", 5000);
        concertService.displayConcerts();
        assertEquals("Added concert\n" +
                     "Added concert\n" +
                     "Concert{performer='Taylor Swift', available=100, waitlist=0}\n" +
                     "Concert{performer='The Weekend', available=5000, waitlist=0}",
                     outputStreamCaptor.toString().trim());
    }



    @Test
    void purchaseTicket() {
        concertService.addConcert("Taylor Swift" , 3);
        concertService.purchaseTicket("Taylor Swift");
        concertService.purchaseTicket("Taylor Swift");
        concertService.purchaseTicket("Taylor Swift");
        // sold out, ticket unavailable
        concertService.purchaseTicket("Taylor Swift");
        assertEquals("Added concert\n" +
                     "Ticket purchased\n" +
                     "Ticket purchased\n" +
                     "Ticket purchased\n" +
                     "Ticket unavailable",
                outputStreamCaptor.toString().trim());
    }

    @Test
    void purchaseTicketUnknownArtist() {
        concertService.addConcert("Taylor Swift" , 1000);
        concertService.purchaseTicket("Unknown Singer");
        assertEquals("Added concert\n" +
                      "No concert for Unknown Singer",
                outputStreamCaptor.toString().trim());
    }

    @Test
    void addToWaitlist() {
        concertService.addConcert("Taylor Swift" , 100);
        concertService.addConcert("The Weekend", 5000);
        concertService.addToWaitlist("Taylor Swift");
        concertService.addToWaitlist("Taylor Swift");
        concertService.addToWaitlist("The Weekend");
        // no concert
        concertService.addToWaitlist("Unknown Singer");

        assertEquals("Added concert\n" +
                     "Added concert\n" +
                     "Added to waitlist\n" +
                     "Added to waitlist\n" +
                     "Added to waitlist\n" +
                     "No concert for Unknown Singer",
                    outputStreamCaptor.toString().trim());
    }
}
```

This task can be completed now.

## Resources

- [Java 17 List Interface](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html)
- [Java 17 ArrayList Class](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html)
