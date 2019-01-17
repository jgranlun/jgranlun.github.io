# Ramblings about Software Test Automation

Various findings about the fascinating world of software test automation.


### Robot Framework & Selenium
*18.01.2018*

I must admit that I am not that familiar (yet) with Selenium and HTML based UI testing in general. 

Here are some of the findings which I had whilst trying to automate the front end part of one of our products with Robot Framework and Selenium.

I was using the following versions of Robot Libraries:
- robotframework-selenium2library==1.8.0
- robotframework-seleniumlibrary==3.3.0
- selenium==3.141.0

Webdriver and browser versions:
- geckodriver 0.23.
- Mozilla Firefox 61.0.1

#### How to configure the used Webdriver options

There seemed to be contradicting instructions on how one can configure options to the used webdriver, and the instructions differed depending on what webdriver was used. 

Also it seemed that no matter which instructions I followed, and whatever parameters I updated, there was no difference in the webdriver's behaviour. So clearly it seemed that I was configuring the parameters incorrectly and the webdriver was ignoring my custom parameter values.

*Maybe if I look the webdriver's debug logs, I can see what I'm doing wrong?* 

Yeah - good idea unfortunately, geckodriver webdriver is not very chatty about it's inner workings by default.

*But to enable the debug logs of the webdriver, you have to set the correct options to the webdriver. Wasn't that the problem  I was trying to solve with goddamn the debug logs in the first place?!.*

##### Firefox/Geckodriver: enable Geckodriver trace logs via Robot Framework and Selenium:

This part was really frustrating. Below were some of the things that confused me:

>So, If I want to enable the webdriver debug logs do I configure the parameter to the webdriver's 'capabilities' or 'desired_capabilities' parameters?  

>Should I use the 'moz:firefoxProfile' to set the debug logs on or should I set the 'LoggingPreferences' object which has settings for 'driver', 'server' and 'browser' each specifically? 

>Should I try to call the Selenium library's functions to set these options, or should I try to pass the parameters directly to the Webdriver via SeleniumLibrary KWs? 

>Can I use the 'Open Browser's 'firefox_capabilites' parameter to enable the debug logs or should I use a custom Firefox profile to do this? 

>To use a custom Firefox profile, one must give the path the directory where the profile is located, as a parameter to the 'Open Browser' KW. How should I name the firefox profile file if manually generate it, so that the webdriver correctly finds it?

Probably there are multiple ways to enable the *(bloody)* debug logs, but what worked for me was to set the 'moz:firefoxOptions' parameter when the Webdriver was created.

```
${log_levels}=    Create Dictionary    level    trace
${log_settings}=    Create Dictionary    log    ${log_levels}

${ff_capabilities}=    Create Dictionary    marionette    ${True}    acceptInsecureCerts    ${True}    browserName    firefox     moz:firefoxOptions    ${log_settings}

Create Webdriver     Firefox     desired_capabilities=${ff_capabilities}
```
Trace level logging should then appear in the 'geckodriver.log' file, which by default is written to the same directory from where Robot Framework is run. So dont go digging into journalctl or any other file located in /var/log/.

Possible log levels are listed [here](https://firefox-source-docs.mozilla.org/testing/geckodriver/geckodriver/TraceLogs.html).

Also note that 'marionette=True' seems to be a mandatory parameter with Geckodriver. If you do not have it set you will see the following error:
```
WebDriverException: Message: Can't load the profile. Possible firefox version mismatch. You must use GeckoDriver instead for Firefox 48+. Profile Dir: /tmp/tmp1snH2n If you specified a log_file in the FirefoxBinary constructor, check it for details.
```

##### Firefox/Geckodriver: Disable the checking of element visibility

If you try to use the SeleniumLibrary's 'Choose File' keyword to upload a file, you may get the following error:
```
ElementNotInteractableException: Message: Element <input ... type="file"> is not reachable by keyboard
```
This issue is explained in detail [here](https://github.com/mozilla/geckodriver/issues/1173). Note that it is Geckodriver specific issue, chromedriver works since it is not (alledgedly) following specifications.

So as a workaround you can either switch to using chromedriver, or configure the 'moz:webdriverClick' parameter to be 'False' with Geckodriver. 

To do the latter you can use the same code as above,  but this time include also the 'moz:webdriverClick' parameter:
```
${ff_capabilities}=    Create Dictionary    moz:webdriverClick    ${False}    marionette    ${True}    acceptInsecureCerts    ${True}    browserName    firefox     moz:firefoxOptions    ${log_settings}        
```

#### Robot Framework, Selenium and AngularJS pages 

ExtendedSelenium2Library is one of the available Robot Framework libraries that can be used to test web pages implemented with AngularJS.

Whilst using the latest version (0.9.2) of [ExtendedSelenium2Library](https://pypi.org/project/robotframework-extendedselenium2library/) and chromedriver I was getting the following error when trying to input text with 'Input Text' KW:

```
JavascriptException: Message: javascript error: $ is not defined
JavaScript stack:
ReferenceError: $ is not defined
    at eval (eval at executeAsyncScript (:457:5), <anonymous>:3:366)
    at lt.c.notifyWhenNoOutstandingRequests (https://10.39.96.73/vendor.bundle.js?2e4fd552993185fc6b8d:22:21307)
    at eval (eval at executeAsyncScript (:457:5), <anonymous>:3:323)
    at eval (eval at executeAsyncScript (:457:5), <anonymous>:3:439)
    at executeAsyncScript (<anonymous>:457:26)
    at apply.ELEMENT (<anonymous>:473:29)
    at callFunction (<anonymous>:361:33)
    at <anonymous>:371:23
    at <anonymous>:372:3
  (Session info: chrome=71.0.3578.98)
  (Driver info: chromedriver=2.45.615279 (12b89733300bd268cff3b78fc76cb8f3a7cc44e5),platform=Linux 4.15.0-29-generic x86_64)
```
The error is a bit odd, since for some elements 'Input Text' works fine, but for some elements this error appears. :confused:

Anyways the error seems to be coming from the used webdriver (in this case chromedriver), so one alternative is to switch to using geckodriver (Firefox) which does not crash in these text inputs. 

Another workaround is to downgrade to 'ExtendedSelenium2Library==0.4.13' version, which does not have this issue. Note though that version 0.4.13 is quite old and some Selenium2Library KWs are missing from that version. For example 'Scroll Element Into View' KW is not found in that version.

