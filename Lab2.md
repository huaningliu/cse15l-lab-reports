# A Simple Showcase of playing with Servers and Bugs at CSE 15L
This report aims to have a showcase about the topics of Servers and Bugs, covered in CSE15L at Week2 to Week3. There're mainly two parts, the first one (Part 1) discussed a technical example of implementing web server; the second one (Part 2) provides a case of the "pipeline" for bugs and debugging. An extra place (Part 3) is set to offer some personal comments of the lab experience by the author.

## Part 1
This practice goes based on the *NumberServer.java* and *Server.java* in lab activities, and re-implements the *URLHandler* interface as *Handler* class to achieve a brand new web server called `StringServer`.

### Server Implementation
Below attached the codes for the revised handler. Compared with the original one for numbers, the structure is **eased** to accomodate a simpler and clearer logic, that it handles the following cases of requests
1. `/add-message?s=<string>`: concatenate a new line the requested string to the current running string, and display the running string.
2. `/`: Display the running string. Note the author deviates from the lab instruction to add this case, because usually a user will not directly have a request, they will instead go to kind of "home page" first, then do something. To accomodate this mute habit, the author added this case (initially displaying a white board) to prevent error happening.
3. Other: other requests would trigger a "404 Not Found!" to let the user know they're off the track.

```
class Handler implements URLHandler {
    // The one bit of state on the server: a number that will be manipulated by
    // various requests.
    String s = "";

    public String handleRequest(URI url) {
        if (url.getPath().equals("/")) {
            return s;  // show a whiteboard or current string when input nothing
        } else {
            if (url.getPath().contains("/add-message")) {
                String[] parameters = url.getQuery().split("=");
                if (parameters[0].equals("s")) {
                    s += parameters[1];
                    s += "\n"; // accumulate s
                    return (s);
                }
            }
            return "404 Not Found!";
        }
    }
}
```
Note this block of code is the only revision from the two files in lab activity, so for simplicity purpose the author didn't include other codes. Compared with the original number implementation, the author editted the code in a way replacing the instance variable as integer to be string. The logic is eased so that the `/increment` request is no longer available. Other edits are quite natural, following the syntax of Java.

### Running the Server
The author proposed a brief instruction regarding the steps of running the program and starting/playing with the server.
1. Check if *StringServer.java* and *Server.java* is ready in current directory. If not, either move the files or direct into the according folder. Also check if the `java` and `javac` commands are ready under current environment. To keep it clean, see more information [here](https://stevela-hn.github.io/cse15l-lab-reports/) in Lab Report one.
2. Run the following commands in order. The first one compiles the program while the second one run the program & start the server.
    ```
    javac Server.java StringServer.java
    ```
    ```
    java StringServer 4000
    ```
    Upon getting done, the terminal should output `Server Started! Visit http://localhost:4000` to indicate a successful running.
3. In a brower of local computer, open http://localhost:4000 to play with the server.

Quick Notes
- `4000` is a **port** that the web server runs on. It's not special, and feel free to pick others. It is an extra part of a URL that’s often used in development.
- `localhost` refers to the current computer.
- You will see an white-board after accessing the link. That's the "home page" the author mentioned in the implementation part.

### Exmaples of Executing the program/server
This section provides some try-outs and according explanations for the server we wrote.

- **Attempt 1**: Here I tested on the "normal functionality" of the web server; that I inputed some strings to see if the page outputs and accumulates as I expected.
    ```
    /add-message?s=Hello
    ```
    The page shows
    ![Image2](L2P1.png)

    ```
    /add-message?s=How are you
    ```
    The page shows
    ![Image2](L2P2.png)
    1. Methods called upon this request: When start the server on command line, we're executing the `start` method in `Server` class. Everytime the author requested like above, the method `handleRequest` in `Handler` class is called.
    2. The argument for `handleRequest`is an `URI` object in `java.net` library, and the according value for this argument is `new URI("http://localhost:3000/add-message?s=Hello")`, extracted by the code `exchange.getRequestURI()` to pass into the method. Note `Hello` is replaced by `How are you` in the second step. Other values also includes a string `s` with value of `Hello` and `Hello \n How are you` each time.
    3. `s` changed to be longer as it is attached with an input string. `new URI()` certainly doesn't change since it's just an argument.

- **Attempt 2**: Here I tested the "Illegal input" case.
    ```
    /add-
    ```
    The page shows
    ![Image3](L2P3.png)
    1. Methods called upon this request: When start the server on command line, we're executing the `start` method in `Server` class. Everytime the author requested like above, the method `handleRequest` in `Handler` class is called.
    2. The argument for `handleRequest`is an `URI` object in `java.net` library, and the according value for this argument is `new URI("http://localhost:3000/add-message?s=Hello")`, extracted by the code `exchange.getRequestURI()` to pass into the method. Note `Hello` is replaced by `How are you` in the second step. Other values also includes a string `s` with value of `Hello` and `Hello \n How are you` each time.
    3. `s` doesn't change at this time, since the string `add-message` is not detected, we will not get into the if-statement to change the value of `s`, i.e. it being instance variable of this class is kept the same after calling this method. `new URI()` certainly doesn't change since it's just an argument.

## Part 2
Here the author picks a case from Lab3 activity (debug session), and presents the "pipeline" of debugging.
### Function Introduction
The function `averageWithoutLowest` aims to get the mean of the numbers in the input array with a lowest number dropped. It returns 0 when there's no element or only one elements. The **original** code attached below.
  ```
  static double averageWithoutLowest(double[] arr) {
    if(arr.length < 2) { return 0.0; }
    double lowest = arr[0];
    for(double num: arr) {
      if(num < lowest) { lowest = num; }
    }
    double sum = 0;
    for(double num: arr) {
      if(num != lowest) { sum += num; }
    }
    return sum / (arr.length - 1);
  }
  ```

### Failure Inducing Input
The author wrote several JUnit tests towards this method, some of which passed, but some failed; we called the inputs of those failed ones failure inducing input. The failure inducing test, within the tester class, are attached below for reference.
  ```
  @Test
  public void testAverageWithoutLowest() {
    double[] input2 = { 3.0, 1.0, 1.0, 1.0, 3.0 };
    assertEquals(2.0, ArrayExamples.averageWithoutLowest(input2), 0.0001);
  }
  ```

### Non-"Failure Inducing Input"
The two tests as inputs below are not failure inducing i.e. the output for these two inputs meets our expectation.
  ```
  @Test
  public void testAverageWithoutLowest() {
    double[] input0 = {};
    assertEquals(0.0, ArrayExamples.averageWithoutLowest(input0), 0.0001);

    double[] input1 = { 3.0, 1.0, 3.0 };
    assertEquals(3.0, ArrayExamples.averageWithoutLowest(input1), 0.0001);
  }
  ```
### Symptom
Run the **three tests above** together, we will have failure below.
![Image4](L2P4.png)

### Bug / De-bug
We found there’s a bug in the summation strategy of the method, that under the case when there are multiple lowest numbers, they drop them all. However, the target of this function is to only drop one of the lowest ones. So I added a boolean pointer `found` to indicate if we have dropped the lowest to avoid repeated counts, and revised the summing logic to make the `sum` available to be added by lowest number after we found the lowest number once. The revised code goes below.
  ```
  static double averageWithoutLowest(double[] arr) {
    if (arr.length < 2) {
      return 0.0;
    }
    double lowest = arr[0];
    for (double num : arr) {
      if (num < lowest) {
        lowest = num;
      }
    }
    double sum = 0;
    boolean found = false;
    for (double num : arr) {
      if ((num != lowest) || (found)) {
        sum += num;
      } else {
        found = true;
      }
    }
    return sum / (arr.length - 1);
  }
  ```

## Part 3
In this section, the author will briefly summarize something new he learned from the lab of week 2 and week 3 at CSE15L.
The author is graduating this quarter, so honestly most of the bulletpoints that covered in the lab activities were not new for him. But he is quite impressed by the **integrative use of Java with server**. Say during an interview, if the author is asked to solve a leetcode problem, then it's not difficult given adequate practice. But if the author is asked to implement a server, this sounds quite scary. But the implementation of `StringServer` in week 3 gives the author a nice "simplified example" of how those "high level terminals" are applied and practiced with programming languages.
