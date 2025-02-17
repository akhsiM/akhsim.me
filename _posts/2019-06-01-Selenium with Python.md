---
title: Selenium with Python
layout: post-toc
tags: python programming
---

## Intro

I have been willing to learn about Selenium for a while and work just gave me a project to work on! So here I am reading thorugh the https://selenium-python.readthedocs.io/ and taking notes that I think are useful.

At some points in time I may have to come back to this and do an /actual/ tutorial. However this should do for now.

Selenium is good at mimicking human actions and do common actions. It works by using "locator".

### Installation

 Selenium API bindings provides a simple API to access functionalities of Selenium WebDriver by Python in an intuitive way.

 Selenium can be installed using `pip`.

#### Driver

You also need to install the relevant webdriver to interface with the chose driver.

Chrome: 	https://sites.google.com/a/chromium.org/chromedriver/downloads
Edge: 	https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/
Firefox: 	https://github.com/mozilla/geckodriver/releases
Safari: 	https://webkit.org/blog/6900/webdriver-support-in-safari-10/

#### Selenium server

This is only required if we want to use the *remote Webdriver*.
/([[https://selenium-python.readthedocs.io/getting-started.html#selenium-remote-webdriver][Using Selenium with remote WebDriver]])/

Seleium server is a Java program so we'd also need JRE of at least version 1.6.

## Getting started

### Simple Usage

If we have installed Selenium Python bindings, we can start using it like this:
```python
# python_org_search.py
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver = webdriver.Firefox()
driver.get("http://www.python.org")  ## go to python.org
assert "Python" in driver.title  ## confirm - "Python" is in the page title
elem = driver.find_element_by_name("q") ## find element name = "q" 
elem.clear()  ## clear
elem.send_keys("pycon")  ## enter pycon
elem.send_keys(Keys.RETURN)  ## hit ENTER
assert "No results found." not in driver.page_source  ## negative assert
driver.close()
```

*EXPLANATION*

The `selenium.webdriver` module provides all the WebDriver implementations. Currently the supported WebDrivers are Firefox (gecko), Chrome (Chromium), IE and remote.

The `Keys` class provide keys in the keyboard like `RETURN`, `F1`, `ALT`..etc..

The `driver.get("https://www.python.org/")` navigates to the page given by the URL. WebDriver will wait until the page has fully loaded before returning control to the sciprt.

This line is an assertion to confirm that title has "Python" word in it:
`assert "Python" in driver.title`

WebDriver offers a number of ways to find elements using one of the `find_element_by_*` methods.
```python
elem = driver.find_element_by_name("q") 
```

This is the element:
<img class="mx-auto w-1/2" src="{{site.baseurl}}/assets/img/orgNotesImages/memspace.png">


```
# all the find_element_by_* methods
'find_element', 'find_element_by_class_name', 'find_element_by_css_selector', 'find_element_by_id', 'find_element_by_link_text', 'find_element_by_name', 'find_element_by_partial_link_text', 'find_element_by_tag_name', 'find_element_by_xpath', 'find_elements', 'find_elements_by_class_name', 'find_elements_by_css_selector', 'find_elements_by_id', 'find_elements_by_link_text', 'find_elements_by_name', 'find_elements_by_partial_link_text', 'find_elements_by_tag_name', 'find_elements_by_xpath'
```

*(More about find_elements_* here: [[https://selenium-python.readthedocs.io/locating-elements.html#locating-elements][Locating Elements]].)*

Next, we can send keys which is similar to entering keys using the keyboard. *Special Keys* can be used using the `Keys` class imported via `from selenium.webdriver.common.keys import Keys`. It is good practice to first clear any pre-populated text in the input field.
```python
elem.clear()
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
```

After submission of the page, check if there is any result. For this we used negative assert - check that there is *no* "No results found." in page_source.

Finally the browser is closed. This can be done by either of the following:

```python :exports both :results output :session
driver.close()
driver.quit()
```

Note that the close() method closes one tab whereas the quit() method exit the entire browser.
## Navigating

### Common actions

   ```
get()
findElement()
click() 
sendKeys()
isDisplayed()
   ```

`isDisplayed()` in Selenium actually checks if the element is *actually* displayed to the human user.

### get()
This is the first thing we'd want to do and it's done by..
```python
driver.get("https://google.com/")
```

WebDriver will wait until the page hsa fully loaded before returning to the script.

### Locator strategies

- Class (good)
- CSS Selectors (for hard to reach)
- ID (good)
- Link Text (x)
- Partial Link Text (x)
- Tag Name (x)
- Xpath (for hard to reach)

Good locators are:
- Unique
- Descriptive
- Unlikely to change
 
#### Finding Quality Locators

- Using *Inspect the page*. 
- Verify our selection

>> *Easy tip:* We can verify our selection by using FirePath or FireFinder which gives better feedback than keep re-testing.

## Assertion

Assertion is used for verification.

Example:
```java
assertTrue("success message not present", 
	driver.findElement(By.cssSelector(".flash.success")).isDisplayed());
```

*Tip*: When doing assertion, trying doing a few `Fail` tests.

## Exception Handling

*NoSuchElementException* is the most common of exceptions. This include the two `NoSuchElement` and `StaleElementReferenceError`.

We can catch the Exception and return `False`.

## Page Objects

We can create a page object that represent the application under test  and run our test scripts against the page object. Therefore if say the application is updated we dont need to update every single test scripts.

## Making tests resilient

### Waiting

There are three ways.

Do not use `Thread.sleep()`: When we do this we actually force our script to always wait. There is no dynamicity.

There are:
- Implicit wait: Wait for x amount of time on default. It doesn't all conditions.
- _Explicit wait_: This is the one that we should use.

Note: Never mix Explicit Waits with Implicit Waits. This is because it will cause failure in our script because there different implementation of Implicit Waits in different drivers and Seleniums?

#### Explicit Waits

- Specify an amount of time and an action, i.e do an action and wrap it within an Explicit Wait.
>> Selenium will keep trying an action until success, if not throws a TimeoutError.

#### Browser Timing Considerations

The easiest driver for Selenium is *Firefox*. 

A script that is working on Firefox could not work on a different browser i.e IE or Chrome. It could be that a browser is slower. For example IE is slower than Firefox so the timing would need to be tweaked.






## Perform actions using Javascript in Python Selenium Webdriver

It may so happen that when working on a page, Selenium WebDriver cannot perform an action on a particular web element. Since WebDriver simulates end-user interaction, it is natural that it refuses to click on an element that is not visisble to end-user. Sometimes it also happens on elements that are visible on the page but may have been stylesheet'd to be "hidden". There can be several other similar reasons or scenarios.

In these cases, we can rely on JavaScript to click or perform actions on that web element.

Python Selenium WebDriver provides a built-in method:
```python
driver.execute_script("some javascript code here")
```

There are two ways we can execute JavaSCript within the browser:

### Method 1: Execute JS at Document Root Level

In this method, we capture the element that we want to work with using JS-provided methods then declare some actions on it and execute this JS using WebDriver

```
element = "document.getElementsByName('username')[0].click();"
driver.execute_script(element)
```

Step 1. We are inspecting and fetching the element by a property name (`getElementByName`). You can do the same for 'id' and 'class' properties.

Step 2. Declare and perform a click action on an element using JS.

Step 3. Call `execute_script()` method and pass the JS string that we created.

Notice the `[0]` bit above. JS functions `getElementby<X>` returns all matching elements as an array. In our case we need to act on the first matching element that can be accessed via `index[0]`.  If we know the index of the element we want to operate, we can directly use the index i.e `index[n]`.

If we are using `getElementbyID` we don't need to use and indexing because the ID should be unique.

While executing, WebDriver will inject the JS statement into the browser and the script will perform the job.

### Method 2: Executing Javascript at Element level

In this case we capture the element that we want to work with using WebDriver, then declare some actions on it using JavaScript and execute this JS using WebDriver by passing the web element as an argument to the JS.

```python
userName = driver.find_element_by_xpath("//button[@name='username']")
driver.execute_script("arguments[0].click();", userName)
```

Step 1. Inspect and capture the element using WebDriver method `find_element_by_xpath`.

Step 2. Declare and perform the click action using JS `arguments[0].click()`

Step 3. Call `execute_script()` method with the JS statement that we created as a string value and the web element captured as the second argument.


We can also have more than one JS actions in our statements. These actions need to be separated by `;`.

```python
userName = driver.find_element_by_xpath("//button[@name='username']")
password = driver.find_element_by_xpath("//button[@name='password']")

driver.execute_script("arguments[0].click();arguments[1].click();", userName, password)
```

### Returning value

We can also use JS executor to fetch values from web elements:
```python
print driver.execute_script('return document.getElementByID("fsr").innerText')
```
