*Note*: To enable Markdown view, press `alt + m` in _Sublime_ and `^ Shift M` in _Atom_.

# Java Unit Test
Expecting an exception in _JUnit5_:
```Java
    assertThrows(InvalidLocationException.class, () -> {
        methodThatThrowsException();    
    });
```


- Check hamcrest for Matchers (when writing java tests).

- To catch exceptions and the message in *JUnit5*:
```Java
    @Test
    void overstockingNotAllowed() throws Exception {
        IllegalArgumentException theException = assertThrows(IllegalArgumentException.class, () -> {
            bin.restock("Fritos", 2600, 100, 50);
        });

        assertEquals("There are only 10 spots left", theException.getMessage());
    }
```

and in *JUnit4*:
```Java
    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void overstockingNotAllowed() throws Exception {
        thrown.expect(IllegalArgumentException.class);
        thrown.expectMessage("There are only 10 spots left");

        bin.restock("Fritos", 2600, 100, 50);
    }
```


* in Gradle, we can define dependencies like this:
  `compile group: 'org.apache.commons', name: 'commons-csv', version: '1.2'`
  or in short form like this:
  `compile 'org.apache.commons:commons-csv:1.2'` (which is one single string, separated with colons)

* you can run `./gradlew` to install gradle (?) and `./gradlew dependencies` to install dependencies
* in a version number: *V2.5.1* —> major.minor.patch



# HTTP Basics
* _telnet_ is a user command and underlying TCP/IP protocol for accessing remote computers.

* HTTP request consists of:
** _request line_: `GET|POST|...` `[URI]` `HTTP/<version>` —––> `GET /xml HTTP/1.1`
** _headers_: `[header name]: [header value]` —————> `Host: httpbin.org`
                `(each on its own line)`             `User-agent: telnet`
                                                     `Accept-Language: en-US`
** Blank line

** Request body (optional)

#### URL Vs. URI
* *URL* stands for Unique Resource Locator
* *URI* stands for Unique Resource Identifier
* A *URL* is essentially a *URI* that also contains information about how to locate the resource
* In `GET /xml HTTP/1.1`, `/xml` is the _URI_ and the _URL_ would be `http://httpbin.org/xml`

#### HTTP Form
```html
<form method="post" action="/register">
	<input type="text" name="username" placeholder="Username">
	<input type="password" name="password" placeholder="Password">
	<textarea name="notes" placeholder="blah"></textarea>
	<button type="submit">Register</button>
</form>
```


# Java Lambdas
* _SAM_ is Single Abstract Method
* _Lambda_: unnamed anonymous functions
* Here is a piece of code in the old way:

```Java
List<Book> books = Books.all();
Collections.sort(books, new Comparator<Book>() {
	@Override
	public int compare(Book b1, Book b2) {
		return b1.getTitle().compareTo(b2.getTitle());
	}
});

for (Book book : books) {
	System.out.println(book);
}
```

Here is the same logic using lambdas:

```Java
List<Book> books = Books.all();
Collections.sort(books, (b1, b2) -> b1.getTitle().compareTo(b2.getTitle()));

books.forEach(book -> System.out.println(book));
```
and even more concise using method references:

```java
List<Book> books = Books.all();
Collections.sort(books, Comparator.comparing(Book::getTitle));

books.forEach(System.out::println);
```

# Git
* git clone and init are used to setup new repositories
* git add, status and commit are used to commit new versions of file
* git log to view list of old commits
* git mv and rm to move and remove files tracked by git
* git push and pull to synchronize with remote repos

* to specify options in git commands: single dash followed by a letter or double dash followed by word: `git commit -am "message"` or `git --help`
* three possible statuses of files are:
	- _modified_
	- _staged_
	- _commited_
* `git diff` will show the un-staged changes to your repo (locally)
* `git diff --staged` will show difference between staged changes and last commit
* To delete a previously committed file: `git rm <file-name>` then you commit it
* To move a file (or rename it) do `git mv file.html file.txt`
* To unstage a file (moving it from _staged_ to _modified_) do `git reset HEAD <file-name>`
* To undo changes in a _modified_ file, which is basically moving it from _modified_ stage to previous _commited_ stage, do `git checkout -- <file-name>`

### Reverting changes
* When you do `git log` you'll see commit history. Those long texts are *Commit SHA* or _Simple Hashing Algorithm_ which uniquely identify commits. You can use only the first 5 characters in SHA to revert them like: `git revert a090b`
* If you wanna revert latest commit, you can also do `git revert HEAD`

### Repos
* _Local Repo_ is our local copy of project
* _Remote Repos_ are remote versions, like a copy on another machine
* It is common to have a _Central Repo_ like the one on github that everyone push and pull from it.
* `git clone <url>` to clone a repo
* `git remote` will show you remotes
* to add a remote repo do `git remote add <remote-name> <url>`

# Spark
* It's a web framework and shouldn't be confused with cluster-computing framework called _Apache Spark_
* To use HTML templates, we use _Handlebars/mustache_ plugin (we add dependency on gradle and also there is an intelliJ plugin for it)
* under _main > resources_ we created _templates > index.hbs_
* inside index.hbs, we type `html:5` and then hit tab, it will generate a quick html template.
