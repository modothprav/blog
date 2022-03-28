---
title: Page Object Model with Selenium
date: 2022-03-28
tags: ["programming", "selenium", "java"]
---

The Page Object Model is a *design pattern* that is encouraged to be used during automation testing with [Selenium](https://www.selenium.dev/documentation/). This design pattern creates an object repository of the web elements present on a web page.  A Page object is represented as an Object-oriented class, which can contain methods that allow tests and other automation workflows to interact with the user interface of the web page. 

![Cover](https://images.unsplash.com/photo-1522542550221-31fd19575a2d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1470&q=80)

## Advantages

One main benefit of using this design pattern is the ease of maintenance and flexibility for future changes. If the user interface of the web page you are automating/testing were to change, you’d know exactly where to look in your codebase as each page object will have its own file. From there it’s usually as simple as changing the way an element is located or modifying a particular method within the class. 

This separation between page-specific code and test code means that again if the UI were to change, the test code wouldn’t need to be modified, as all modifications in response to that UI change can be made in the page object class file.

Another benefit is the code reusability since each page object is independent of each other, the same methods can be reused in different tests and automation workflows. This reduces unnecessary lines of code and gives more emphasis to behaviour compared to locating and scraping web elements. 

Finally, this design pattern greatly improves readability for a developer, since each webpage will have its own page object file, browsing the code base will be fairly simple if organised accordingly. The developer will know exactly what actions are going to be performed on each webpage as outlined in the page object files. Again due to each page being independent of each other, the developer can confidently make any modifications to a single page object class file, without affecting other class files. 

## Example Test *without* POM pattern

Now here’s an example of a test function that test’s the login process within the Yahoo login page. This test method doesn’t utilise the Page Object class we have defined in a later example. The function enters the username and clicks next to then be prompted to enter the password.

```java
  /**
  * Tests login feature
  */
  public class Login {
    @Test
    public void testLogin() {
      // Enter Username and click next
      driver.findElement(By.id("login-username")).sendKeys("Pravin");
      driver.findElement(By.id("sign-in")).click();

      // Enter password and click next
      driver.findElement(By.id("login-passwd")).sendKeys("password");
      driver.findElement(By.id("login-signin")).click();

      // Verify and check if homepage is authenticated for current user
      By elementPath = By.xpath("//div[text()='Pravin']"
      assertTrue(driver.findElement(elementPath)).isDisplayed());
    }
  }
```

> Now imagine if multiple test cases followed this same structure above, it would be a real hassle to modify all of these test cases if the user interface of the web page were to change.
> 

## What is Page factory?

Page factory is another class provided by Selenium, to enable support for the Page Object Model design pattern. The Page factory class helps with *instantiating* the web elements or fields in a Page Object class file. 

This class provides the developer with the `@FindBy` annotation and the static `initElements()` method, to instantiate the Web element of a page upon object construction. This can be seen more clearly with an example file below, which is a page object class of the Yahoo login Web page. 

The `initElements()` method will locate all the web elements defined as fields in the class file, using the locators specified in the `@FindBy` annotation. 

> **Note:** For a more thorough explanation of Page factory check out these docs on [GitHub](https://github.com/SeleniumHQ/selenium/wiki/PageFactory)
> 

## Example Class file with POM pattern in use - Yahoo Login page

```java
  public class YahooLogin {
    protected WebDriver driver;

    // Object Repository
    @FindBy(id = "login-username")
    private WebElement usernameField;

    @FindBy(id = "login-passwd")
    private WebElement passwordField;

    @FindBy(id = "login-signin")
    private WebElement nextButton;

    @FindBy(id = "persistent")
    private WebElement sessionCheckBox;

    /**
     * Constructor
     */
    public YahooLogin(WebDriver driver) {
      this.driver = driver
      PageFactory.initElements(driver, this);
    }

    /**
     * Toggles the 'Stay Signed in' checkbox
     */
    public void clickCheckBox() {
      this.sessionCheckBox.click();
    }

    /**
     * Enter the username in the Username field, then proceed to 
     * click the next button to be presented with the password 
     * filed. Finally, enter the password in the password field 
     * and click next.
     * @return YahooHome The Page object for Yahoo Home page.
     */
    public YahooHome login(String username, String password) {
      // Enter username
      this.usernameField.sendKeys(username);
      this.nextButton.click();

      // Enter password
      this.passwordField.sendKeys(password);
      this.clickNextButton();

      return new YahooHome(this.driver);
    } 
  }
```

> Notice on the `login()` method how a new page object class is being returned as a successful login on the Yahoo login page presents you with an authenticated Yahoo Home page.
> 

## Example Test *with* POM pattern in use

``` java
  /**
   * Test Login Feature
   */
  public class Login {
    @Test
    public void testLogin() {
      YahooLogin yahooLogin = new YahooLogin(driver);
      YahooHome yahooHome = yahooLogin.login("Pravin", "password");
      assertThat(yahooHome.getUser(), is("Pravin"));
    }
  }
```

As can be seen there is significant code reduction, and if the web page were to experience any UI changes, all modifications will only need to be made within the page object class and not the test cases themselves.

## See it in action

> If you're interested in seeing more of this Design pattern in action within a real project, check out my [Automated portfolio tracking](https://github.com/modothprav/Automated-Portfolio-Tracking) project on GitHub. 
>
This Java Selenium project scrapes transaction data from Sharesies the New Zealand investing platform and enters that scraped data within a user's portfolio in Yahoo Finance. In this project, selenium is used more for automation purposes rather than its typical use in testing.