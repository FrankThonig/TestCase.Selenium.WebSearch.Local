# Browser Test Automation with Selenium
## Test Case: Web Search

##### Table of Contents
- [Testing Paradigms / Test Setup Specifications](#testing-paradigms--test-setup-specifications)
- [System / Software Environment](#system--software-environment)
  - [Third Party Software](#third-party-software)
  - [Test Data Creation / Data Source](#test-data-creation--data-source)
  - [Execution / Deployment](#execution--deployment)
  - [General Notes](#general-notes)
- [User Guide](#user-guide)
  - [Test Case](#test-case)
  - [Implementation](#implementation)
  - [Execution / Deployment](#execution--deployment-1)

## Testing Paradigms / Test Setup Specifications

- automated testing
- data-driven testing
- parallel test execution
- cross browser / multi browser testing
- local test execution, i.e. no distributed test execution

## System / Software Environment

### Third Party Software

#### Programming Language

- Java

#### Computing Platform

- JDK 1.8.0_121 (32 bit, 64 bit)

#### Browser Automation Framework

- Selenium 3.4.0

#### Testing Framework

- TestNG 6.11

#### Browsers

- Firefox 45, 48, 52 (32 bit)
- Chrome 57 (32 bit, 64 bit)
- Internet Explorer 11 (32 bit, 64 bit)

#### WebDrivers

- Geckodriver 0.15.0 (32 bit)
- Chromedriver 2.28
- IEDriverServer 3.3.0.0 (32 bit)

#### Build / Deployment Tools

- Eclipse Neon
- Maven 3.5.0
- Jenkins 2.57

#### Operating Systems (OS)

- Windows 10 (64 bit)
- Windows 7 (32 bit)

### Test Data Creation / Data Source

- TestNG DataProvider

### Execution / Deployment

- Java Application
- TestNG Suite
- Maven build
- Jenkins:
  - Freestyle project
  - Pipeline

### General Notes

#### Bit Architecture

Under certain circumstances 32 bit application software can be run on 64 bit OS. However, 64 bit application software can usually not be run on 32 bit OS. For that reason this project prefers the 32 bit versions of application software over the 64 bit versions - for both the 32 bit and the 64 bit OS project setup. The aim is to only need to store one single bit architecture specific version of each application software in the file system and to avoid needing to implement bit architecture related distinctions in any of the test project parts. This again keeps maintenance efforts lower and facilitates system portability between 32 bit and 64 bit OS.

#### Third Party Software Files

The third party software files are not part of this project's file repository. However, the project's setup contains notations of paths and file names of that software to be used for the project setup. With the help of these notations, after obtaining the files, a project user can place them in the respective directories.

#### File Directory Structure

The file directories are set up in a way to facilitate the use of Maven.

Within the "resources" directory there are two subdirectories:

- "int"
- "ext"

"int" stands for "internal". This directory contains all the files whose content is customized by the project user and which are therefore related to the internal setup of the project.

"ext" stands for "external". This directory contains all the third party application software files.

#### Utility Files

This projects java implementations contain a number of files serving as utilities for common tasks that are not explicitly related to the specific test scope. Therefore, in this documentation, these files are only described more in detail if relevant for the understanding of the test topic.

## User Guide

### Test Case

The project comprehends one single test case.

#### Test Sequence

1. start browser
2. navigate to search web site
3. enter search term
4. execute search process
5. retrieve search results
6. count occurences of search term in search results
7. execute test assert

#### Test Assert Textual Phrasing

Variables:

- `X`: expected result

Assert:

_Each of the first `X` result titles of the result list contains the search term._

The concrete test assert implementation is given in a different segment of this documentation.

### Implementation

The project contains one single test class `CountSearchResultsTestCase` which again contains one single test method `testSearchForSearchTermAndCountOccurencesInResults`.

#### Test Data

#### *Common Test Data*

`CountSearchResultsTestCase.xml` is a TestNG XML file and contains test data that is common for all test runs:

- URL of search web site
- expected result `X` for assert

```
<parameter
	name="urlOfSearchWebsite"
	value="https://www.google.com" />
	
<parameter
	name="amountOfResultsToBeChecked"
	value="3" />
```

The common test data provided by the `CountSearchResultsTestCase.xml` TestNG XML file is handed over to the  `beforeMethod` in the `CountSearchResultsTestCase` class, i.e. the handover takes place before executing the test method itself.

```
@BeforeMethod
@Parameters({
	"urlOfSearchWebsite",
	"amountOfResultsToBeChecked"})
public void beforeMethod(
	String urlOfSearchWebsite,
	int amountOfResultsToBeChecked)
```

#### *Individual Test Data*

The `TestData` class is a Test NG `DataProvider` and supplies individual test data for each test run:

- name of browser to be used
- search term

```
@DataProvider(
	name=
		"testData",
	[...])
public static Object[][] createTestData()
```

```
_Data=
	new Object[8][2];

_Data[0][0]=
		chrome;

_Data[0][1]=
		monday;

_Data[1][0]=
		chrome;

_Data[1][1]=
		tuesday;

_Data[2][0]=
		ie;

_Data[2][1]=
		monday;
```

The individual test data provided by the `TestData` class is handed over to the test method `testSearchForSearchTermAndCountOccurencesInResults` in the `CountSearchResultsTestCase` class, i.e. the handover takes place when executing the test method itself - as opposed to the handover of the common test data.

```
@Test(
	dataProvider=
		"testData",
	dataProviderClass=
		TestData.class)
public void testSearchForSearchTermAndCountOccurencesInResults(
	String browserName,
	String searchTerm)
```

#### *Test Data Supply for Data Driven Testing*

This project setup uses the TestNG `DataProvider` for handling the individual, non common test data. It is also possible to use the TestNG XML file for this task. `DataProvider` was chosen here to facilitate maintainability.

#### *Test Data Handling for Parallel Test Execution*

Executing the test method in parallel test runs for each set data package from the `DataProvider` is achieved by setting the `DataProvider` to parallel mode in the `TestData` class.

```
@DataProvider(
	[...],
	parallel=
		true)
public static Object[][] createTestData()
```

In the `CountSearchResultsTestCase.xml` TestNG XML no setup of any parallel execution mode for the test is made.

In the TestNG XML the size of the pool of threads available for the `DataProvider` is set to the value calculated from the structure of the test data specified in the `TestData` class.

```
_Data=
	new Object[8][2];
```

```
<suite
	[...]
	data-provider-thread-count="8"
	[...]>
```

#### Test class `CountSearchResultsTestCase`

Since the test method will be run in parallel by different threads, the class variables need to be thread safe. For that purpose `ThreadLocal` is used.

```
ThreadLocal<Integer> _AmountOfResultsToBeChecked
```

`ThreadLocal` variables need a certain handling, including initialization via constructor `new ThreadLocal()` as well as asigning and reading the values via `set` and `get` methods.

The `beforeMethod` takes care of instantiating a `WebDriverManagerFactory` which will later be used to retrieve a `WebDriver` instance. The method also handles the constant test data, as explained above.

The test method `testSearchForSearchTermAndCountOccurencesInResults` manages the realization of the test sequence.

First, it takes care of starting the browser. For this purpose, an instance of a `WebDriver` implementation is retrieved.

```
_WebDriverManager.set(
	_WebDriverManagerFactory.get().createWebDriverManager(
		browserName));

if(_WebDriverManager.get()!=null)
{
	
	webDriver=
		_WebDriverManager.get().createWebDriver();
	
}
```

Then the browser is called to navigate to the search web site.

```
WebDriverNavigate.navigateToURL(
	webDriver,
	_URLOfSearchWebsite.get());
```

The search term gets entered, the search button pressed and the result list is retrieved.

```
resultsList=
	new SearchPage(
		webDriver).executeSearch(
			searchTerm).getResultsList();
```

In the end the assertion is called. A specific error message is set for when an assertion error gets thrown.

```
Assert.assertTrue(
	isSearchTermContainedInResultsListWithExpectedAmountOfOccurences(
		searchTerm,
		resultsList),
	"Search term not contained in the first " +
	_AmountOfResultsToBeChecked.get() +
	" search results.");
```

The auxiliary methods `isSearchTermContainedInResultsListWithExpectedAmountOfOccurences` and `isSearchTermContainedInCurrentResult` determine how many of the first `X` result titles of the result list contain the search term. This value gets used by the assert.

The `afterMethod` takes care of closing the respective browser instance after each individual test run.

#### Browser Executable and `WebDriver` Handling Overview

Browser Executable and `WebDriver` handling in this project happens in a three level system.

1. `WebDriverManagerFactory`
2. `WebDriverManager`
3. browser specific `[...]WebDriverManager`

The `WebDriverManagerFactory` uses the browser name parameter to instantiate the appropriate `WebDriverManager`.

`WebDriverManager` is an abstract class implementing common behaviour of all browser specific `WebDriver` implementations.

The individual `[...]WebDriverManager` classes contain browser specific information and system setups for instantiating the respective `WebDriver` implementation.

#### *Browser Executable and `WebDriver` Handling Overview for Firefox*

Each Firefox version used in this project comes with its own `FirefoxV[...]WebDriverManager` class holding the version specific values. Instantiation of the `WebDriver` for all versions is handled in `FirefoxWebDriverManager`.

#### *Browser Executable and `WebDriver` Handling Overview for Chrome*

This project uses one single Chrome version. Instantiation of the `WebDriver` is handled in `ChromeWebDriverManager`.

#### *Browser Executable and `WebDriver` Handling Overview for Internet Explorer*

This project uses one single Internet Explorer version. Instantiation of the `WebDriver` is handled in `IEWebDriverManager`.

#### Browser Executable Handling Details

#### *Browser Executable Handling Details for Firefox*

Firefox browser files need to be placed in the respective directories specified in the `FirefoxV[...]WebDriverManager` and in the `FirefoxWebDriverManager` classes. The directories and the browser release versions can be chosen by the tester.

##### *Firefox version 47 and below*

The directory to the Firefox browser binary is getting set via a system property. The Constructor of the `FirefoxDriver` does not need any arguments. 

```
System.setProperty(
	"webdriver.firefox.bin",
	createPathToFirefoxBinary(
		versionXX));

_WebDriver=
		new FirefoxDriver();
```

##### *Firefox version 48 and above*

For the purpose of demonstrating the options available, this test project contains *two* different ways of initializing the `FirefoxDriver` for Firefox version 48 and above. When running the projects source code, only *one* of the two options needs to be executed.

*Option 1: `DesiredCapabilities`*

The directory to the Firefox browser binary is getting set via `DesiredCapabilities`. The Constructor of the `FirefoxDriver` takes the `DesiredCapabilities` as an argument. 

```
_DesiredCapabilities.setCapability(
	"firefox_binary",
	createPathToFirefoxBinary(
		versionXX));
		
_WebDriver=
	new FirefoxDriver(
		_DesiredCapabilities);
```

*Option 2: `FirefoxOptions`*

The directory to the Firefox browser binary is getting set via `FirefoxOptions`. The Constructor of the `FirefoxDriver` takes the `FirefoxOptions` as an argument.

```
_FirefoxOptions.setBinary(
	createPathToFirefoxBinary(
		versionXX));
		
_WebDriver=
	new FirefoxDriver(
		_FirefoxOptions);
```

#### *Browser Executable Handling Details for Chrome*

This project uses one single Chrome release version but both the 32 bit and 64 bit architecture version of it.

Chrome browser files need to be placed in the respective directories specified in the `ChromeWebDriverManager` class. The directories and the browser release versions can be chosen by the tester.

Depending on the OS bit architecture, the corresponding Chrome bit architecture version needs to be used.

The directory to the Chrome browser binary is getting set via `ChromeOptions`. The Constructor of the `ChromeDriver` takes the `ChromeOptions` as an argument.

```
if(OperatingSystem.is64bitOS())
{
	
	_ChromeOptions.setBinary(
		new File(
			createPathToChromeBinary(
				"64")));
	
}
else
{
	
	_ChromeOptions.setBinary(
		new File(
			createPathToChromeBinary(
				"32")));
	
}

_WebDriver=
	new ChromeDriver(
		_ChromeOptions);
```

#### *Browser Executable Handling Details for Internet Explorer*

This project uses one single Internet Explorer release version. The version to be used is the one installed on the Windows OS on which this project runs. It is not possible - with reasonable effort - to choose a different than the default Internet Explorer installation.

Internet Explorer browser files must not / do not need to be moved from their default directories in the Windows OS. The directory and the browser release version cannot be chosen by the tester.

#### `WebDriver` handling details

The path to the respective `WebDriver` executables is being set in the system properties.

#### *`WebDriver` handling details for Firefox*

Geckodriver / Marionette driver is not needed for Firefox version 47 and below, but it is needed for version 48 and above. With the version of Selenium used in this project it is mandatory to set the path to the Geckodriver executable - no matter if it will later be needed or not by the concrete Firefox version used in a specific test run.

In this test project, even with the 64 bit OS setup, the 32 bit version of the Geckodriver executable is used because it has run more stable than the 64 bit version.

```
System.setProperty(
	"webdriver.gecko.driver",
	PathRetrieve.retrieveAbsolutePathStringFromRelativePathString(
		this,
		"/ext/bin/selenium/driver/firefox/geckodriver-v0.15.0-win32.exe"));
```

##### *Firefox version 47 and below*

Geckodriver / Marionette driver is activated in Selenium by default. It needs to be explicitly deactivated for Firefox version 47 and below.

```
System.setProperty(
	"webdriver.firefox.marionette",
	"false");
```

Deactivation of the Geckodriver is done by setting the corresponding system property to `false`. However, the driver needs to be activated for Firefox version 48 and above to be executed, so for them, the same system property needs to be set `true`.
Running the tests in parallel for all Firefox versions would result in the need for the Geckodriver system property to be set at the same time to `false` (for Firefox version 47 and below) and to `true` (for Firefox version 48 and above). Since the two values `false` and `true` are mutually exclusive, it is not possible to run tests using Firefox version 47 and below in parallel to tests using Firefox version 48 and above. Only non-parallel, i.e. sequential, test execution could have tests for all the Firefox versions mentioned above within one test setup.

##### *Firefox version 48 and above*

Since Geckodriver / Marionette driver is activated in Selenium by default, there is no need for any specific setup for Firefox version 48 and above regarding this topic.

In this setup, it is not possible to run tests using Firefox version 47 and below in parallel to tests using Firefox version 48 and above. A detailed explanation is given in a different segment of this documentation.

#### *`WebDriver` handling details for Chrome*

For Windows OS there is only one Chromedriver bit architecture version available of this Chromedriver release version. It works for both the 32 bit and the 64 bit Chrome executable.

```
System.setProperty(
	"webdriver.chrome.driver",
	PathRetrieve.retrieveAbsolutePathStringFromRelativePathString(
		this,
		"/ext/bin/selenium/driver/chrome/chromedriver.exe"));
```

#### *`WebDriver` handling details for Internet Explorer*

There are 32 bit and 64 bit IEDriverServer version available. On 64 bit Windows OS the 64 bit IEDriverServer version has shown to execute browser actions more slowly compared to the 32 bit IEDriverServer version. For this reason the 32 bit IEDriverServer version is used for both the 32 bit and the 64 bit Windows OS.

```
System.setProperty(
	"webdriver.ie.driver",
	PathRetrieve.retrieveAbsolutePathStringFromRelativePathString(
		this,
		"/ext/bin/selenium/driver/ie/32 bit/IEDriverServer.exe"));
```

#### Browser Automation

Following are the details of the browser automation parts of this project's specific test case described in a different segment of this documentation.

#### *General Notes*

Browser actions are executed as "safe" actions, i.e. before interacting with a certain web element, the algorithm waits for that element to be ready for interaction.

#### *Page Objects*

The test case covers two web sites:

- search web site
- results web site

In the source code, each of these web sites is represented by its own `[...]Page` class. For implementation the page object pattern was applied.

##### *`PageObject`* class

`PageObject` is an abstract class implementing common behaviour of all `[...]Page` classes.

##### *`SearchPage`* class

The `SearchPage` class represents the search web site. Following are the web elements relevant to the test case:

- search text field
- search button

```
@FindBy(
	name=
		"q")
private WebElement _SearchTextField;

@FindBy(
	id=
		"_fZl")
private WebElement _SearchButton;
```

##### *`ResultsPage`* class

The `ResultsPage` class represents the results web site. Following are the web elements relevant to the test case:

- results list

However, of the result list only the result titles in particulare are relevant to the test case. For that reason, a sublist of the results list is regarded:

- result titles list

```
@FindBy(
	id=
		"resultStats")
private WebElement _ResultsList;

@FindBy(
	xpath=
		".//*[@id='rso']//*[@class='rc']//h3/a")
private List<WebElement> _ResultTitleWebElementsList;
```

#### *Test Sequence Browser Automation*

The test sequence is executed by the `testSearchForSearchTermAndCountOccurencesInResults` method of the `CountSearchResultsTestCase` class. Following are the test sequence parts concerning browser automation.

```
webDriver=
	_WebDriverManager.get().createWebDriver();

[...]

WebDriverNavigate.navigateToURL(
	webDriver,
	_URLOfSearchWebsite.get());

resultsList=
	new SearchPage(
		webDriver).executeSearch(
			searchTerm).getResultsList();
```

##### *Start Browser*

The browser is getting started by instantiating the concrete `WebDriver` implementiation in the respective `[...]WebDriverManager` class.

```
webDriver=
	_WebDriverManager.get().createWebDriver();
```

The implementation for instantiating the `WebDriver` is shown in a different segment of this documentation.

##### *Navigate to Search Web Site*

```
WebDriverNavigate.navigateToURL(
	webDriver,
	_URLOfSearchWebsite.get());
```

The navigation is implemented in the `WebDriverNavigate` class.

```
public static void navigateToURL(
	WebDriver webDriver,
	String url)
{
	
	if(webDriver!=null)
	{
		
		webDriver.get(
			url);
		
	}
	
}
```

##### *Enter Search Term & Execute Search Process*

```
new SearchPage(
	webDriver).executeSearch(
		searchTerm)
```

First, the search field is cleared from any previous entries. Then the search term is entered. Finally the search button is pressed. The sequence of these steps is defined in the `SearchPage` class.

```
public ResultsPage executeSearch(
	String searchTerm)
{
	
	return clearSearchTextFieldAndEnterSearchTerm(
		searchTerm).clickSearchButton();
	
}
```

The concrete actions are implemented in the `WebElementActions` class.

```
static void clear(
	WebElement webElement)
{
	
	if(webElement!=null)
	{
		
		webElement.clear();
		
	}
	
}

static void sendKeys(
	WebElement webElement,
	String keysToSend)
{
	
	if(webElement!=null)
	{
		
		webElement.sendKeys(
				keysToSend);
		
	}
	
}

static void click(
	WebElement webElement)
{
	
	if(webElement!=null)
	{
		
		webElement.click();
		
	}
	
}
```

##### *Retrieve Search Results*

```
getResultsList();
```

After the search button has been pressed the results page containing the results list is shown in the browser. The `ResultsPage` class retrieves this list and extracts the result titles as text elements.

```
for(WebElement currentResultTitleWebElement : _ResultTitleWebElementsList)
{
	
	_ResultTitleTextsList.add(
		WebElementActions.getText(
			currentResultTitleWebElement));
	
}
```

The concrete action is implemented in the `WebElementActions` class.

```
public static String getText(
	WebElement webElement)
{

	String text=
		null;
	
	if(webElement!=null)
	{
		
		text=
			webElement.getText();
		
	}
	
	return text;
	
}
```

#### Test Assert Implementation

The test assert is executed by the `testSearchForSearchTermAndCountOccurencesInResults` method of the `CountSearchResultsTestCase` class.

The test assert textual phrasing is given in a different segment of this documentation.

The assert states that each of the first `X` result titles of the result list contains the search term, with `X` being the expected result.

The value of `X` is given to the test by the TestNG XML and stored as `_AmountOfResultsToBeChecked`.

The actual amount of first result list titles containing the search term needs to be retrieved for the assert. This is done by the auxiliary method `isSearchTermContainedInResultsListWithExpectedAmountOfOccurences`.

```
Assert.assertTrue(
	isSearchTermContainedInResultsListWithExpectedAmountOfOccurences(
		searchTerm,
		resultsList),
	"Search term not contained in the first " +
	_AmountOfResultsToBeChecked.get() +
	" search results.");
```

### Execution / Deployment

#### Java Application

For convenience the project contains the runnable java class `Launcher`. Its purpose is to initiate test execution directly from a java application.

Test execution is prepared by instantiating a `TestNG` object and setting the `CountSearchResultsTestCase.xml` TestNG XML as the reference file for the test. Test execution is then started by calling the `TestNG` `run` method.

```
TestNG testNG=
	new TestNG();

List<String> suiteList=
	Lists.newArrayList();

suiteList.add(
	PathRetrieve.retrieveAbsolutePathStringFromRelativePathString(
		this,
		"/int/testng/xml/CountSearchResultsTestCase.xml"));

testNG.setTestSuites(
	suiteList);

testNG.run();
```

Additionally user information about the application status is displayed.

```
InfoAlert infoAlert=
	new InfoAlert();

infoAlert.createAlert(
	"Executing Test[...]");

[...]

infoAlert.closeAlert();

infoAlert.createAlert(
	"Showing Test Results[...]");

[...]

infoAlert.closeAlert();
```

After test execution the test results are being opened in a browser window.

```
BrowserShowFile.showFileInBrowser(
	".\\test-output\\emailable-report.html");
```

#### TestNG Suite

`CountSearchResultsTestCase.xml` is the TestNG XML file which can be run directly.

#### Maven build

The projects directory structure is suited to be used with a Maven build. `pom.xml` is the respective Maven XML file. The Maven build can be used to compile the source, copy the resources and run the test by using the Maven goal `test`.

#### Jenkins

This project contains a setup for using continuous integration in Jenkins. The project specific integration process comprehends:

- Git clone / pull of the project repository
- Maven build with goal `test`
- filing test results

Jenkins needs to be configured to set up the respective software for this process.
Applying the `test` goal to the maven build includes:

- compiling of the java source files
- running the tests

Therefore, a JDK and TestNG need to be set up, too. The complete setup consists of:

- Git setup
- JDK setup
- TestNG setup
- Maven setup
- Test Result setup

The setup itself comprehends two parts:

- installation
- configuration

It is not necessary for all setups to include both the installation and the configuration.

##### Installations

##### *Git Installation*

The Git installation is given the name of `gitInstallation` and the directory of a local Git executable is specified.

![](/img/Jenkins_Config_Git.JPG)

##### *JDK Installation*

The JDK installation is given the name of `jdkInstallation` and the directory of a local JDK executable is specified.

![](/img/Jenkins_Config_JDK.JPG)

##### *Maven Installation*

The Maven installation is given the name of `mavenInstallation` and the directory of a local Maven executable is specified.

![](/img/Jenkins_Config_Maven.JPG)

##### Configurations

The configurations depend on the type of Jenkins integration used. There are *two* options given in this project:

- Freestyle project
- Pipeline

When applying the continuous integration, only *one* of them needs to be used.

##### *Freestyle project*

*Git configuration*

The project user needs to specify his Git repository URL and the credentials. The credentials can be managed via Jenkins.

![](/img/Freestyle_Config_Git.JPG)

*Maven configuration*

To configure the Maven build a build step needs to be added.
The project user needs to specify the name of the Maven Version. Here, the name of the Maven Installation `mavenInstallation` is to be used.
For the Goals the configuration gets set to `test`. Additionally, the behaviour in case of failing tests can be specified.
Finally, the POM is set to the respective project directory.

![](/img/Freestyle_Config_Maven.JPG)

*Test Result configuration*

To configure the handling of test results, a post-build action needs bo be added.
The project user needs to specify his TestNG XML report pattern to match this project's directory structure.

![](/img/Freestyle_Config_TestNG.JPG)

##### *Pipeline*

Configuration of a Pipeline is handled by the `seleniumWebSearchLocalPipeline_Jenkinsfile` Jenkinsfile.

*Git configuration*

The project user needs to specify his Git repository URL and the credentials ID. The credentials can be managed via Jenkins.

```
git([
	url: 'YOUR_GIT_REPO_URL',
	credentialsId: 'YOUR_JENKINS_CREDENTIALS_ID'])
```

*Maven configuration*

The project user needs to specify the name of the JDK and Maven versions. Here, the name of the JDK Installation `jdkInstallation` and the name of the Maven Installation `mavenInstallation` are to be used.
For the Goals the configuration gets set to `test`. Additionally, the behaviour in case of failing tests can be specified.
The POM file is taken from the directory set in the `bat` command. This directory needs to match the project's diretory structure.

```
String javaHome = tool("jdkInstallation");		
String mvnHome = tool("mavenInstallation") + "\\bin";

[...]

withEnv([
	"PATH+WHATEVER=${javaHome}",
	"PATH+WHATEVER2=${mvnHome}",
	"JAVA_HOME=${javaHome}"])
	{
	
		bat "mvn -Dmaven.test.failure.ignore=true test"
	
	}
```

*Test Result configuration*

The project user needs to specify the report filename pattern to match this project's directory structure.

```
step([
	$class: 'hudson.plugins.testng.Publisher',
	reportFilenamePattern: '**/target/surefire-reports/testng-results.xml'])
```
