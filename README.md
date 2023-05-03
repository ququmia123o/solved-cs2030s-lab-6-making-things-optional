Download Link: https://assignmentchef.com/product/solved-cs2030s-lab-6-making-things-optional
<br>
In this lab, we are going to see how we can use Optional to simplify our tt, allowing us to focus on the core application logic instead of worrying about edge cases. This lab also involves using the HashMap class from Java Collections and more functional interfaces from Java: Consumer, Supplier, and Runnable.

TaskYour task is to read in a roster of students, the modules they take, the assessments they have completed, and the grade for each assessment. Then, given a query consisting of a triplet: a student, a module, and an assessment, retrieve the corresponding grade.

For instance, if the input is:

Steve CS1010 Lab3 ASteve CS1231 Test A+Bruce CS2030 Lab1 C

and the query is Steve CS1231 Test, the program should print A+.

In our scenario, a roster has zero or more students; a student takes zero or more modules, a module has zero or more assessments, and each assessment has exactly one grade. Each of these entities can be uniquely identified by a string.

PreambleMapMap&lt;K,V&gt; is a generic interface from the Java Collection Framework, the implementation of which is useful for storing a collection of items and retrieving an item. It maintains a map (aka dictionary) between keys (of type K) and values (of type V). The two core methods are (i) put, which stores a (key, value) pair into the map, and (ii) get, which returns the value associated with a given key if the key is found or returns null otherwise.

The following examples show how the Map&lt;K,V&gt; interface and its implementation HashMap&lt;K,V&gt; can be used.

jshell&gt; Map&lt;String,Integer&gt; map = new HashMap&lt;&gt;();jshell&gt; map.put(“one”, 1);$.. ==&gt; nulljshell&gt; map.put(“two”, 2);$.. ==&gt; nulljshell&gt; map.put(“three”, 3);$.. ==&gt; nulljshell&gt; map.get(“one”)$.. ==&gt; 1jshell&gt; map.get(“four”)$.. ==&gt; nulljshell&gt; map.entrySet()$.. ==&gt; [one=1, two=2, three=3]jshell&gt; for (Map.Entry e: map.entrySet()) {…&gt;  System.out.println(e.getKey() + “:” + e.getValue());…&gt; }one:1two:2three:3

Level 1Assessment class and Keyable interfaceWe shall start by writing the Assessment class that implements the following Keyable interface.

interface Keyable {String getKey();}

Include a getGrade method that returns the grade of an assessment.

jshell&gt; new Assessment(“Lab1”, “B”)$.. ==&gt; {Lab1: B}jshell&gt; new Assessment(“Lab1”, “B”).getGrade()$.. ==&gt; “B”jshell&gt; new Assessment(“Lab1”, “B”).getKey()$.. ==&gt; “Lab1”jshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test1.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 2Module classWrite the Module class to store (via the put method) the assessments of a module in a map for easy retrieval as part of answering queries. A module can have zero or more assessments, with each assessment having a title as a key — a unique identifier.

jshell&gt; new Module(“CS2040”)$.. ==&gt; CS2040: {}jshell&gt; new Module(“CS2040”).getKey();$.. ==&gt; “CS2040”jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))$.. ==&gt; CS2040: {{Lab1: B}}jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).put(new Assessment(“Lab2″,”A+”))$.. ==&gt; CS2040: {{Lab1: B}, {Lab2: A+}}jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).put(new Assessment(“Lab2″,”A+”)).get(“Lab1”)$.. ==&gt; {Lab1: B}jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).put(new Assessment(“Lab2″,”A+”)).get(“Lab2”)$.. ==&gt; {Lab2: A+}jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).put(new Assessment(“Lab2″,”A+”)).get(“Lab3”)$.. ==&gt; nulljshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test2.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 3Student class and generic class KeyableMapWrite a Student class that stores the modules he/she reads in a map via the put method. A student can read zero or more modules, with each module having a unique module code as its key. You will notice that the implementations of the Student and Module classes are very similar. Hence, by applying the abstraction principle, write a generic class KeyableMap to reduce the duplication.

Hint: KeyableMap&lt;V&gt; is a generic class that wraps around a String key (i.e. implements Keyable) and a Map&lt;String,V&gt;. KeyableMap models an entity that contains a map, but is also itself contained in another container (e.g. a student contains a map of modules but could be contained in a roster). The parameter type V is the type of the value of items stored in the map; V must be a subtype of Keyable.

The class KeyableMap is a mutable class — we made this decision since the Map implementation in Java Collection Framework is mutable. KeyableMap provides two core methods:

get(String key) which returns the item with the given key;put(V item) which adds the key-value pair (item.getKey(),item) into the map. The put method returns a KeyableMap. To avoid type mismatch when chaining put method calls together, each sub-class of KeyableMap should override the put method from KeyableMap and change the return type to the corresponding sub-classes. E.g., Student should override put to return a Student through a (guaranteed to be safe) type casting.jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”)$.. ==&gt; {Lab1: B}jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”).getGrade()$.. ==&gt; “B”jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)))$.. ==&gt; Tony: {CS2040: {{Lab1: B}}}jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040”)$.. ==&gt; CS2040: {{Lab1: B}}jshell&gt; Student natasha = new Student(“Natasha”);jshell&gt; natasha.put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)))$.. ==&gt; Natasha: {CS2040: {{Lab1: B}}}jshell&gt; natasha.put(new Module(“CS2030”).put(new Assessment(“PE”, “A+”)).put(new Assessment(“Lab2”, “C”)))$.. ==&gt; Natasha: {CS2030: {{Lab2: C}, {PE: A+}}, CS2040: {{Lab1: B}}}jshell&gt; Student tony = new Student(“Tony”);jshell&gt; tony.put(new Module(“CS1231”).put(new Assessment(“Test”, “A-“)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}}jshell&gt; tony.put(new Module(“CS2100”).put(new Assessment(“Test”, “B”)).put(new Assessment(“Lab1”, “F”)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}, CS2100: {{Lab1: F}, {Test: B}}}jshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test3.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 4Avoiding null with OptionalSo far we have been avoiding invalid retrievals such as

new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab2”)

which would result in a null value. Furthermore, chaining methods likenew Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2030” ).get(“Lab1”)

would result in a NullPointerException being thrown due to calling get(“Lab1”) on a null value. Let’s deal with the generation of null values first, and worry about chaining get methods in the next level.Take a look at the Java documentation of Optional to familiarize yourself with the APIs available. Modify the KeyableMap generic class such that each call to get returns an Optional&lt;V&gt; where V is a subtype of Keyable.

jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”)$.. ==&gt; Optional[{Lab1: B}]jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)))$.. ==&gt; Tony: {CS2040: {{Lab1: B}}}jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040”)$.. ==&gt; Optional[CS2040: {{Lab1: B}}]jshell&gt; Student natasha = new Student(“Natasha”);jshell&gt; natasha.put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)))$.. ==&gt; Natasha: {CS2040: {{Lab1: B}}}jshell&gt; natasha.put(new Module(“CS2030”).put(new Assessment(“PE”, “A+”)).put(new Assessment(“Lab2”, “C”)))$.. ==&gt; Natasha: {CS2030: {{Lab2: C}, {PE: A+}}, CS2040: {{Lab1: B}}}jshell&gt; Student tony = new Student(“Tony”);jshell&gt; tony.put(new Module(“CS1231”).put(new Assessment(“Test”, “A-“)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}}jshell&gt; tony.put(new Module(“CS2100”).put(new Assessment(“Test”, “B”)).put(new Assessment(“Lab1”, “F”)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}, CS2100: {{Lab1: F}, {Test: B}}}jshell&gt; /exit

As you can see, the only difference is that each value returned from an invocation of the get method is wrapped in an Optional.

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test4.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 5“Chaining” OptionalsChaining Optionals together as such

new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040” ).get(“Lab1”)

will now result in a compilation error instead. This is because the Optional class does not have a get(String) method defined (although it does define a get() method which, other than for debugging purposes, should typically be avoided).Rather than chaining in the usual way, we do it through a map or flatMap. Let’s start with map.

jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”)$.. ==&gt; Optional[{Lab1: B}]

jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”).getGrade()|  Error:|  cannot find symbol|    symbol:   method getGrade()|  new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”).getGrade()|  ^————————————————————————^

jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”).map(x -&gt; x.getGrade())$.. ==&gt; Optional[B]

jshell&gt; new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)).get(“Lab1”).map(Assessment::getGrade)$.. ==&gt; Optional[B]

As expected, invoking getGrade() on an Optional results in a compilation error. However, we can perform a similar chaining effect by passing in the functionality of getGrade (either in the form of a lambda or method reference) to Optional’s map method. Notice the return value is actually wrapped in another Optional. When using map, you can think of the operation as “taking the value out of the Optional box, transforming it via the function passed to map, and wrap the transformed value back in another Optional”.

Now this is where things start to get interesting! Look at the following:

jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040” ).map(x -&gt; x.get(“Lab1”))$.. ==&gt; Optional[Optional[{Lab1: B}]]

Observe that the return value is an Optional wrapped around another Optional that wraps around the desired value! Why is this so? The difference lies in the return type of Assessment::getGrade (read getGrade method of the Assessment class) and Module::get. The former returns a String, while the latter returns an Optional.In x -&gt; x.getGrade() (or Assessment::getGrade), the transformed value is simply the grade, and this is wrapped in an Optional. However, passing x -&gt; x.get(“Lab1”) in the above code snippet results in a transformed value of Optional. And this transformed value is wrapped around another Optional via the map operation!

As such, we use the flatMap method instead. You may think of flatMap as flattening the Optionals into a single one.

jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040” ).flatMap(x -&gt; x.get(“Lab1”))$.. ==&gt; Optional[{Lab1: B}]

jshell&gt; new Student(“Tony”).put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”))).get(“CS2040” ).flatMap(x -&gt; x.get(“Lab1”)).map(Assessment::getGrade)$.. ==&gt; Optional[B]

Now you are ready to create a roster. Define a Roster class that stores the students in a map via the put method. A roster can have zero or more students, with each student having a unique id as its key. Once again, notice the similarities between Roster, Student and Module.

Define a method called getGrade in Roster to answer the query from the user. The method takes in three String parameters, corresponds to the student id, the module code, and the assessment title, and returns the corresponding grade.

In cases where there are no such student, or the student does not read the given module, or the module does not have the corresponding assessment, then output No such record followed by details of the query. Here, you might find Optional::orElse useful.

jshell&gt; Student natasha = new Student(“Natasha”);jshell&gt; natasha.put(new Module(“CS2040”).put(new Assessment(“Lab1”, “B”)))$.. ==&gt; Natasha: {CS2040: {{Lab1: B}}}jshell&gt; natasha.put(new Module(“CS2030”).put(new Assessment(“PE”, “A+”)).put(new Assessment(“Lab2”, “C”)))$.. ==&gt; Natasha: {CS2030: {{Lab2: C}, {PE: A+}}, CS2040: {{Lab1: B}}}jshell&gt; Student tony = new Student(“Tony”);jshell&gt; tony.put(new Module(“CS1231”).put(new Assessment(“Test”, “A-“)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}}jshell&gt; tony.put(new Module(“CS2100”).put(new Assessment(“Test”, “B”)).put(new Assessment(“Lab1”, “F”)))$.. ==&gt; Tony: {CS1231: {{Test: A-}}, CS2100: {{Lab1: F}, {Test: B}}}jshell&gt; Roster roster = new Roster(“AY1920”).put(natasha).put(tony)jshell&gt; rosterroster ==&gt; AY1920: {Natasha: {CS2030: {{Lab2: C}, {PE: A+}}, CS2040: {{Lab1: B}}}, Tony: {CS1231: {{Test: A-}}, CS2100: {{Lab1: F}, {Test: B}}}}jshell&gt; roster.get(“Tony”).flatMap(x -&gt; x.get(“CS1231”)).flatMap(x -&gt; x.get(“Test”)).map(Assessment::getGrade)$.. ==&gt; Optional[A-]jshell&gt; roster.get(“Natasha”).flatMap(x -&gt; x.get(“CS2040”)).flatMap(x -&gt; x.get(“Lab1”)).map(Assessment::getGrade)$.. ==&gt; Optional[B]jshell&gt; roster.get(“Tony”).flatMap(x -&gt; x.get(“CS1231”)).flatMap(x -&gt; x.get(“Exam”)).map(Assessment::getGrade)$.. ==&gt; Optional.emptyjshell&gt; roster.get(“Steve”).flatMap(x -&gt; x.get(“CS1010”)).flatMap(x -&gt; x.get(“Lab1”)).map(Assessment::getGrade)$.. ==&gt; Optional.emptyjshell&gt; roster.getGrade(“Tony”,”CS1231″,”Test”)$.. ==&gt; “A-”jshell&gt; roster.getGrade(“Natasha”,”CS2040″,”Lab1″)$.. ==&gt; “B”jshell&gt; roster.getGrade(“Tony”,”CS1231″,”Exam”);$.. ==&gt; “No such record: Tony CS1231 Exam”jshell&gt; roster.getGrade(“Steve”,”CS1010″,”Lab1″);$.. ==&gt; “No such record: Steve CS1010 Lab1”jshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test5.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 6The Main classNow use the classes that you have built and write a Main class to solve the following:

Read the following from the standard input:

The first token read is an integer N, indicating the number of records to be read.The subsequent inputs consist of N records, each record consists of four words, separated by one or more spaces. The four words correspond to the student id, the module code, the assessment title, and the grade, respectively.The subsequent inputs consist of zero or more queries. Each query consists of three words, separated by one or more spaces. The three words correspond to the student id, the module code, and the assessment title.For each query, if a match in the input is found, print the corresponding grade to the standard output. Otherwise, print “No Such Record:” followed by the three words given in the query, separated by exactly one space.

See sample input and output below. Inputs are underlined.

$ java Main12Jack   CS2040 Lab4    BJack   CS2040 Lab6    CJane   CS1010 Lab1    AJane   CS2030 Lab1    A+Janice CS2040 Lab1    A+Janice CS2040 Lab4    A+Jim    CS1010 Lab9    A+Jim    CS2010 Lab1    CJim    CS2010 Lab2    BJim    CS2010 Lab8    A+Joel   CS2030 Lab3    CJoel   CS2030 Midterm AJack   CS2040 Lab4Jack   CS2040 Lab6Janice CS2040 Lab1Janice CS2040 Lab4Joel   CS2030 MidtermJason  CS1010 Lab1Jack   CS2040 Lab5Joel   CS2040 Lab3BCA+A+ANo such record: Jason CS1010 Lab1No such record: Jack CS2040 Lab5No such record: Joel CS2040 Lab3

Note:

You might find Optional::ifPresentOrElse useful.On CodeCrunch, we will check for any use of null in your code. Any occurence of the string null would fail the CodeCrunch test. Thus, avoid variable or method names that contain the substring null.Further, we will disallow the use of methods Optional::get and Optional::isPresent, as the former could cause NullPointerException, while the latter is essentially the same as checking for null.Compile and check your styling by issuing the following

$ javac -Xlint:rawtypes *.java$ checkstyle *.java