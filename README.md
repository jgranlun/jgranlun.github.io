# Robot Framework & Selenium

There are a bunch of confusing instructions are floating around the use of Selenium and Robot Framework. 

For example the way how used options are configured to underlying webdriver was a very confusing. Also I could not find any examples online how to do it with Robot Framework.

So I listed here some examples which may be helpful to other people.

Info is valid at least for the following SW releases:
- robotframework-selenium2library==1.8.0
- robotframework-seleniumlibrary==3.3.0
- selenium==3.141.0
- geckodriver 0.23.
- Mozilla Firefox 61.0.1

## Firefox & Geckodriver 
### Enable Firefox/Geckodriver trace logs via Robot Framework and Selenium:
```
${log_levels}=    Create Dictionary    level    trace
${log_settings}=    Create Dictionary    log    ${log_levels}

${ff_capabilities}=    Create Dictionary    marionette    ${True}    acceptInsecureCerts    ${True}    browserName    firefox     moz:firefoxOptions    ${log_settings}

Create Webdriver     Firefox     desired_capabilities=${ff_capabilities}
```
Trace level logging should then appear in the 'geckodriver.log' file, which by default is written to the same directory from where Robot Framework is run.

Possible log levels are listed [here](https://firefox-source-docs.mozilla.org/testing/geckodriver/geckodriver/TraceLogs.html).

Also note that 'marionette=True' seems to be a mandatory parameter with Geckodriver. If you do not have it set you will see the following error:
```
WebDriverException: Message: Can't load the profile. Possible firefox version mismatch. You must use GeckoDriver instead for Firefox 48+. Profile Dir: /tmp/tmp1snH2n If you specified a log_file in the FirefoxBinary constructor, check it for details.
```

### To disable the checking of element visibility with Firefox / Geckodriver

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

## Chrome 

### Testing AngularJS pages with Chrome and with ExtendedSelenium2Library

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
The error is a bit odd, since for some elements 'Input Text' works fine, but for some elements this error appears.

Anyways the error seems to be coming from the used webdriver (in this case chromedriver), so one alternative is to switch to using geckodriver (Firefox) which does not crash in these text inputs. 

Another workaround is to downgrade to 'ExtendedSelenium2Library==0.4.13' version, which does not have this issue. Note though that version 0.4.13 is quite old and some Selenium2Library KWs are missing from that version. For example 'Scroll Element Into View' KW is not found in that version.

