
# JAVA 8 - Cheat Sheet

## Lambda Expression
```java
(int a) -> a * 2; // Calculate the double of a
a -> a * 2; // or simply without type
```
```java
(a, b) -> a + b; // Sum of 2 parameters
```

If the lambda is more than one expression we can use `{ }` and `return`

```java
(x, y) -> {
	int sum = x + y;
	int avg = sum / 2;
	return avg;
}
```

A lambda expression cannot stand alone in Java, it need to be associated to a functional interface.

```java
interface MyMath {
    int getDoubleOf(int a);
}
	
MyMath d = a -> a * 2; // associated to the interface
d.getDoubleOf(4); // is 8
```

---

All examples with "list" use :

```java
List<String> list = [Bohr, Darwin, Galilei, Tesla, Einstein, Newton]
```


## Collections

**sort** `sort(list, comparator)`

```java
list.sort((a, b) -> a.length() - b.length())
list.sort(Comparator.comparing(n -> n.length())); // same
list.sort(Comparator.comparing(String::length)); // same
//> [Bohr, Tesla, Darwin, Newton, Galilei, Einstein]
```

**removeIf**

```java
list.removeIf(w -> w.length() < 6);
//> [Darwin, Galilei, Einstein, Newton]
```

**merge**
`merge(key, value, remappingFunction)`

```java
Map<String, String> names = new HashMap<>();
names.put("Albert", "Ein?");
names.put("Marie", "Curie");
names.put("Max", "Plank");

// Value "Albert" exists
// {Marie=Curie, Max=Plank, Albert=Einstein}
names.merge("Albert", "stein", (old, val) -> old.substring(0, 3) + val);

// Value "Newname" don't exists
// {Marie=Curie, Newname=stein, Max=Plank, Albert=Einstein}
names.merge("Newname", "stein", (old, val) -> old.substring(0, 3) + val);
```


## Method Expressions `Class::staticMethod`

Allows to reference methods (and constructors) without executing them

```java
// Lambda Form:
getPrimes(numbers, a -> StaticMethod.isPrime(a));

// Method Reference:
getPrimes(numbers, StaticMethod::isPrime);
```

| Method Reference | Lambda Form |
| ---------------- | ----------- |
| `StaticMethod::isPrime` | `n -> StaticMethod.isPrime(n)` |
| `String::toUpperCase`   | `(String w) -> w.toUpperCase()` |
| `String::compareTo`     | `(String s, String t) -> s.compareTo(t)` |
| `System.out::println`   | `x -> System.out.println(x)` |
| `Double::new`           | `n -> new Double(n)` |
| `String[]::new`         | `(int n) -> new String[n]` |


## Streams

Similar to collections, but

 - They don't store their own data
 - The data comes from elsewhere (collection, file, db, web, ...)
 - *immutable* (produce new streams)
 - *lazy* (only computes what is necessary !)
 
```java
// Will compute just 3 "filter"
Stream<String> longNames = list
   .filter(n -> n.length() > 8)
   .limit(3);
```

**Create a new stream**

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 5, 7, 11);
Stream<String> stream = Stream.of("Jazz", "Blues", "Rock");
Stream<String> stream = Stream.of(myArray); // or from an array
list.stream(); // or from a list

// Infinit stream [0; inf[
Stream<Integer> integers = Stream.iterate(0, n -> n + 1);
```

**Collecting results**

```java
// Collect into an array (::new is the constructor reference)
String[] myArray = stream.toArray(String[]::new);

// Collect into a List or Set
List<String> myList = stream.collect(Collectors.toList());
Set<String> mySet = stream.collect(Collectors.toSet());

// Collect into a String
String str = list.collect(Collectors.joining(", "));
```

**map** `map(mapper)`<br>
Applying a function to each element

```java
// Apply "toLowerCase" for each element
res = stream.map(w -> w.toLowerCase());
res = stream.map(String::toLowerCase);
//> bohr darwin galilei tesla einstein newton

res = Stream.of(1,2,3,4,5).map(x -> x + 1);
//> 2 3 4 5 6
```

**filter** `filter(predicate)`<br>
Retains elements that match the predicate

```java
// Filter elements that begin with "E"
res = stream.filter(n -> n.substring(0, 1).equals("E"));
//> Einstein

res = Stream.of(1,2,3,4,5).filter(x -> x < 3);
//> 1 2
```

**reduce**<br>
Reduce the elements to a single value

```java
String reduced = stream
	.reduce("", (acc, el) -> acc + "|" + el);
//> |Bohr|Darwin|Galilei|Tesla|Einstein|Newton
```

**limit** `limit(maxSize)`
The n first elements

```java
res = stream.limit(3);
//> Bohr Darwin Galilei
```

**skip**
Discarding the first n elements

```java
res = strem.skip(2); // skip Bohr and Darwin
//> Galilei Tesla Einstein Newton
```

**distinct**
Remove duplicated elemetns

```java
res = Stream.of(1,0,0,1,0,1).distinct();
//> 1 0
```

**sorted**
Sort elements (must be *Comparable*)

```java
res = stream.sorted();
//> Bohr Darwin Einstein Galilei Newton Tesla 
```

**allMatch**

```java
// Check if there is a "e" in each elements
boolean res = words.allMatch(n -> n.contains("e"));
```

anyMatch: Check if there is a "e" in an element<br>
noneMatch: Check if there is no "e" in elements

**parallel**
Returns an equivalent stream that is parallel

**findAny**
faster than findFirst on parallel streams

### Primitive-Type Streams

Wrappers (like Stream<Integer>) are inefficients. It requires a lot of unboxing and boxing for each element. Better to use `IntStream`, `DoubleStream`, etc.

**Creation**

```java
IntStream stream = IntStream.of(1, 2, 3, 5, 7);
stream = IntStream.of(myArray); // from an array
stream = IntStream.range(5, 80); // range from 5 to 80

Random gen = new Random();
IntStream rand = gen(1, 9); // stream of randoms
```

Use *mapToX* (mapToObj, mapToDouble, etc.) if the function yields Object, double, etc. values.

### Grouping Results

**Collectors.groupingBy**

```java
// Groupe by length
Map<Integer, List<String>> groups = stream
	.collect(Collectors.groupingBy(w -> w.length()));
//> 4=[Bohr], 5=[Tesla], 6=[Darwin, Newton], ...
```

**Collectors.toSet**

```java
// Same as before but with Set
... Collectors.groupingBy(
	w -> w.substring(0, 1), Collectors.toSet()) ...
```

**Collectors.counting**
Count the number of values in a group

**Collectors.summing__**
`summingInt`, `summingLong`, `summingDouble` to sum group values

**Collectors.averaging__**
`averagingInt`, `averagingLong`, ... 

```java
// Average length of each element of a group
Collectors.averagingInt(String::length)
```

*PS*: Don't forget Optional (like `Map<T, Optional<T>>`) with some Collection methods (like `Collectors.maxBy`).


### Parallel Streams

**Creation**

```java
Stream<String> parStream = list.parallelStream();
Stream<String> parStream = Stream.of(myArray).parallel();
```

**unordered**
Can speed up the `limit` or `distinct`

```java
stream.parallelStream().unordered().distinct();
```

*PS*: Work with the streams library. Eg. use `filter(x -> x.length() < 9)` instead of a `forEach` with an `if`.


## Optional
In Java, it is common to use null to denote absence of result.
Problems when no checks: `NullPointerException`.

```java
// Optional<String> contains a string or nothing
Optional<String> res = stream
   .filter(w -> w.length() > 10)
   .findFirst();

// length of the value or "" if nothing
int length = res.orElse("").length();

// run the lambda if there is a value
res.ifPresent(v -> results.add(v));
```

Return an Optional

```java
Optional<Double> squareRoot(double x) {
   if (x >= 0) { return Optional.of(Math.sqrt(x)); }
   else { return Optional.empty(); }
}
```

---

**Note on inferance limitations**

```java
interface Pair<A, B> {
    A first();
    B second();
}
```

A steam of type `Stream<Pair<String, Long>>` :

 - `stream.sorted(Comparator.comparing(Pair::first)) // ok`
 - `stream.sorted(Comparator.comparing(Pair::first).thenComparing(Pair::second)) // dont work`

Java cannot infer type for the `.comparing(Pair::first)` part and fallback to Object, on which `Pair::first` cannot be applied.

The required type for the whole expression cannot be propagated through the method call (`.thenComparing`) and used to infer type of the first part.

Type *must* be given explicitly.

```java
stream.sorted(
    Comparator.<Pair<String, Long>, String>comparing(Pair::first)
    .thenComparing(Pair::second)
) // ok
```

---

## Exercices et exemples:

//                        Exercice 1 - Unité 1							              //
// Supprime
```java
public class Exercise
{
   public static void main(String[] args) throws IOException
   {
      List<String> words = Files.readAllLines(Paths.get("pays.txt"));
      words.removeIf(obj -> obj.length() < 20);
      System.out.println(words);
   }
} 
```

/*************************************************************************/
//																		                                    //
//                        Exercice 2 - Unité 1							              //
//																		                                    //
/*************************************************************************/	
List<String> words = Files.readAllLines(Paths.get("pays.txt"));
words.removeIf(w -> 
 { 
    int taille = w.length();
    String first = w.substring(0,1);
    String last = w.substring(taille-1, taille);
    return !first.equalsIgnoreCase(last);
 });

/*************************************************************************/
//																		                                    //
//                        Exercice 3 - Unité 1							              //
//																		                                    //
/*************************************************************************/	
public class Index
{
   public static void main(String[] args) throws IOException
   {
      Scanner in = new Scanner(System.in);
      Map<String, String> index = new TreeMap<>();
      int line = 0;
      while (in.hasNextLine())
      {
         line++;
         for (String word : in.nextLine().split("[^'\\pL]+"))
         {
     		//we merge word into a map with the line in first and then
     		//we take the old value and we add to it the new val value
     		//like this we can keep a trace of old value and add new ones
            index.merge(word, Integer.toString(line), (old, val)-> old + ", " + val);
         }
      }
      System.out.println(index);
   }
}

/*************************************************************************/
//																		                                    //
//                        Exercice 1 - Unité 2							              //
//																		                                    //
/*************************************************************************/								
class Words
{	//static method to access it from the class name
   public static int vowels(String w)
   {
      return w.length() - w.toLowerCase().replaceAll("[aâàäæeéêèëiîïoôœöuûùüyÿ]", "").length();
   }
}

public class Exercise
{
   public static void main(String[] args) throws IOException
   {
      List<String> words = Files.readAllLines(Paths.get("pays.txt"));
      Collections.sort(words, Comparator.comparing(Words::vowels).thenComparing(String::compareTo));
      System.out.println(words);
   }
}

/*************************************************************************/
//																		                                   //
//                        Exercice 2 - Unité 2							             //
//																		                                   //
/*************************************************************************/	
class Collections 
{
   public static <T> T[] toArray(Collection<T> coll, 
      Function<Integer, T[]> constructor)
   {
      T[] result = constructor.apply(coll.size());
      Iterator<T> iter = coll.iterator();
      for (int i = 0; i < result.length; i++)
         result[i] = iter.next();
      return result;
   }
}

public class Exercise
{
   public static void main(String[] args) throws IOException
   {
      List<String> words = Files.readAllLines(Paths.get("pays.txt"));

      // TODO: Turn words into an array of type String[].

      String[] wordArray = Collections.toArray(words, String[]::new); //create new array of strings
      Arrays.sort(wordArray);
      for (int i = 0; i < 10; i++)
         System.out.println(wordArray[i].toUpperCase());

      // Note: If wordArray was an Object[], you couldn't call 
      // wordArray[i].toUpperCase().
   }
}

/*************************************************************************/
//																		                                   //
//                        Exercice 3 - Unité 2							             //
//																		                                   //
/*************************************************************************/	
// TODO: Make this method receive and return an array.
// Accept a constructor expression for constructing the 
// returned array.

public class Util
{
	//First parameter of Function<What apply will take, What apply will return>
	//La Function prend un int en paramètre et retournera un nouveau tableau de T[]
	//Here we want apply to take the size of the String[] and to return a new T [] array
   public static <T> T[] filter(T[] values, Predicate<T> p, Function<Integer, T[]> f)
   {
      List<T> result = new ArrayList<>();
      for (T value : values)
      {
      	//predicate test value and add it if it's true
         if (p.test(value)) { result.add(value); }
      }
      
     //on retourne un array de string et on lui passe la taille
     return result.toArray(f.apply(result.size()));
   }
}

//Call of the method
String[] wordsWithA = Util.filter(words, w -> w.contains("a"), String[]::new);

//Way to display an array
System.out.println(Arrays.toString(wordsWithA));

/*************************************************************************/
//																		                                   //
//                        Exercice 1 - Unité 3							             //
//																		                                   //
/*************************************************************************/	
public class Words
{
   public static long distinctVowels(String str)
   {
        return Stream.of(str.split("")) //on découpe chaque lettres
        		//on prend un lettre et on lui enlève ces accents 
        		//normalize retourne un tableau pour à [a,`]
                .map(c -> Normalizer.normalize(c,Normalizer.Form.NFD)
                                    .substring(0,1))
                //on filtre notre lettre pour savoir si elle fait partie des
            	//voyelles
                .filter(s -> "aeiou".contains(s))
                .distinct()	//on prend chaque valeur 1 fois si identique
                .count();	//on compte
   }
}


/*************************************************************************/
//																		                                   //
//                        Exercice 2 - Unité 3							             //
//																		                                   //
/*************************************************************************/	
public class Streams
{
   public static void main(String[] args) throws IOException
   {
		Scanner in = new Scanner(System.in);
		String filename = in.next();
		try (Stream<String> lineStream =  Files.lines(Paths.get(filename))){
			long count = lineStream
				.filter(s -> Words.distinctVowels(s) == 5)
				.count();
			System.out.println(count + " words with 5 distinct vowels");         
 		}   

		try (Stream<String> lineStream = Files.lines(Paths.get(filename))){
	  		//On souhaite récupérer une liste de string depuis le Stream lineStream
	  		//on filtre les mots ayant 5 voyelles distinctes
	        List<String> result = lineStream.filter(s -> Words.distinctVowels(s) == 5)
							        //on trie grace a sorted(Comparator.comparing( Fonction de tri )) Ici la longueur
							        .sorted(Comparator.comparing(String::length))
							        //on limite au 20 premier résultat
							        .limit(20)
							        //on collect (Collectors) En toList toArray .. etc
							        .collect(Collectors.toList());
	         System.out.println(result);
	      }   
   }
}

/*************************************************************************/
//																		                                   //
//                        Exercice 3 - Unité 3							             //
//																		                                   //
/*************************************************************************/	
public class Streams
{
   List<Pair<String, Long>> wordsWithManyVowels(Stream<String> words, int n)
   {
      return words
         .map(w -> Pair.of(w, 
            (Words.vowels(w) - (w.length() - Words.vowels(w)))))
         .sorted(
            Comparator.comparingLong((Pair<String,Long> p) -> -p.second())
                     .thenComparing((Pair<String,Long> p) -> p.first()))
         .limit(n)
         .collect(Collectors.toList());
   }
}


/*************************************************************************/
//																		                                    //
//                        		Collections								                  //
//																		                                    //
/*************************************************************************/	

//Exemples avec des collections



/*************************************************************************/
//                       Exemples de Labdas								               //
/*************************************************************************/	

//list = (0, 44 , 33) retourne e0,e44,o33
public String getString(List<Integer> list) {
	return list.stream()
                .map(n -> n % 2 == 0 ? "e" + n : "o" + n)
                //joining retourne une string séparé par ici une virgule
                .collect(Collectors.joining(",")); 
}

/*************************************************************************/
//                       Longest word 									                 //
/*************************************************************************/	
//opening est une liste de string
//Lambda
Collections.max(opening, (s,t) -> s.length() - t.length());
//Compartor
Collections.max(opening,Comparator.comparing(String::length));
//Compartor Stream
opening.stream()
       .sorted(Comparator.comparing(String::length).reversed())
       .findFirst().get();	//get retourne un string

/*************************************************************************/
//                       Filter using predicate	greater < 5				       //
/*************************************************************************/	
opening.stream()
	   .filter(Predicates.greater(String::length,5)) //we pass string.length to func
	   .distinct()
	   .toArray(String[]::new);

//L'interface Predicates
interface Predicates{ //func will take a T and an Integer
					  //here T is a String and Integer = to the String.length()
	public static <T> Predicate<T> greater(Function<T,Integer> func, int n){
		//so func.apply(p) will return the length of p (a string)
		//A predicate is a lamba that return true or false
		return p -> func.apply(p) > n; 
	
	}
}

System.out.println(Arrays.toString(array));

//----Same without predicates interface
array = opening.stream().filter(s -> s.length() > 5).distinct().toArray(String[]::new);


/*************************************************************************/
//               Triangular method	- Iterate with interface 			       //
/*************************************************************************/	
 List<Pair<Integer, Integer>> triangles = Streams.triangular(7) //limite a 7
 												 .collect(Collectors.toList());

 //we need a method that construct the triangular stream
 interface Strams{

 	public static Streams<Pair<Integer, Integer>> triangular(int max){
 		return Stream.iterate(Pair.of(1,1), //Iterate prend une valeur de départ
 							//en second parametre prend les itérations à faire
 							 (Pair<Integer, Integer> p) -> Pair.of(p.first() + 1, 
 							 									  (p.second() -1 ) + p.fist()))
 					 .limit(max);
 	}
 }


/*************************************************************************/
//               Trier les valeurs pair par nom 						              //
/*************************************************************************/	
System.out.println(list.stream()
                    .filter((Pair<String, Integer> p) -> p.second() % 2 == 0)
                    .sorted(Comparator.comparing(Pair::second))
                    .map(Pair::first)
                    .collect(Collectors.toList())
);

System.out.println("-- 5. Sorted even values names");
System.out.println(list.stream()
        .filter((Pair<String, Integer> p) -> p.second() % 2 == 0)
        .sorted(Comparator.comparing((Pair<String, Integer> p) -> p.second()))
        .map((Pair<String,Integer> p) -> p.first())
        .collect(Collectors.toList()));

/*************************************************************************/
//               Trier les valeurs grâce à une interface				          //
/*************************************************************************/	

public class Filterable{

    public static <T> List<T> filter(List<T> mylist, Predicate<T> predicate){

        List<T> list = new ArrayList<>();
        for(T val: mylist){
            if(predicate.test(val)){ list.add(val);}
        }
        return list;

    }
}

List<Dragon> oldest = Filterable.filter(dragons, d -> d.color() == Dragon.Color.Red 
														&& d.age() >= 4000);

/*************************************************************************/
//                       Summary et Moyennes							               //
/*************************************************************************/

list.stream()
  .mapToInt(i -> i) //lambda basique
  .average()
  .getAsDouble();

collectionKids.stream()
                .mapToInt(t -> t.age) //toujours fournir une lambda 
                .summaryStatistics() //.getCount() .getMax() .getSum()
                .getAverage();





/*************************************************************************/
//                       List vs Maps									                    //
/*************************************************************************/
List.
	forEach(Iterables)
	//ne marche pas sur Array.asList
	//faire List<String> str2 = new ArrayList<String>(str1);
	//removeIf mettre l'inverse de ce que l'on souhaite
	removeIf(Collections) 
	replaceAll()
	sort()

Map.
	forEach()
	computeIfAbsent()
	merge()
	replaceAll()


/*************************************************************************/
//                       Accumulator									                   //
/*************************************************************************/
//Accumulator sum, lambda
Accumulator.accumulate(list, 0, (r, n) -> r + n.second()));

//on redefinit un interface avec la méthode static accumulate
//qui va prendre une liste de Pair<T, Integer> 
//un int de départ
//une Function qui prendre des Pair<T, Integer> en entree et retournera un Integer en sortie
interface Accumulator{
    public static <T> int accumulate(List<T> list, int init, FunctionAcc<T, Integer> f ){
        int sum = init;
        for (int i = 0; i < list.size(); i++){
            sum = f.apply(sum, list.get(i));
        }
        return sum;
    }
}

interface FunctionAcc<T,R>{
    //doit être redéfinie
    R apply(int init, T arg);
}

//il faut redéfinir la fonction apply de notre FunctionAcc anonyme
int sum = Accumulator.accumulate(list, 0, new FunctionAccumulateur<Pair<String, Integer>, Integer>() {
    @Override
    public Integer apply(int init, Pair<String, Integer> arg) {
        return init + arg.second();
    }
});

//ou ecrire notre Function en dehors et la passe a l'accumulator
FunctionAccumulateur<Pair<String, Integer>, Integer> f = new FunctionAccumulateur<Pair<String, Integer>, Integer>() {
    @Override
    public Integer apply(int init, Pair<String, Integer> pair) {
        return init + pair.second();
    }
};


System.out.println(
        Accumulator.accumulate(list,0 , f)
);

This cheat sheet was based on the lecture of Cay Horstmann
http://horstmann.com/heig-vd/spring2015/poo/
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYxMjMxNjUxNSw0NDEzOTg2NV19
-->