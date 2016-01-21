---
layout: page
title: JUnit4 Rules with the ATSL
permalink: /docs/rules/index.html
site_nav_category: docs
is_site_nav_category2: true
site_nav_category_order: 120
---
{::options toc_levels="2"/}

* TOC
{:toc}

With the Android Testing Support Library we are providing a set of JUnit rules to be used with the `AndroidJUnitRunner`. JUnit rules provide more flexibility and reduce the boilerplate code required in tests.

`TestCase`s like `ActivityInstrumentationTestCase2` or `ServiceTestCase` are deprecated in favor of `ActivityTestRule` or `ServiceTestRule`.

## ActivityTestRule
This rule provides functional testing of a single activity. The activity under test will be launched before each test annotated with `@Test` and before any method annotated with `@Before`. It will be terminated after the test is completed and all methods annotated with `@After` are finished. The Activity under Test can be accessed during your test by calling `ActivityTestRule#getActivity()`.

{% highlight java %}
@RunWith(AndroidJUnit4.class)
@LargeTest
public class MyClassTest {

    @Rule
    public ActivityTestRule<MyClass> mActivityRule = new ActivityTestRule(MyClass.class);

    @Test
    public void myClassMethod_ReturnsTrue() { ... }
}
{% endhighlight %}

## ServiceTestRule

This rule provides a simplified mechanism to start and shutdown your service before and after the duration of your test. It also guarantees that the service is successfully connected when starting (or binding to) a service. The service can be started (or bound) using one of the helper methods. It will automatically be stopped (or unbound) after the test completes and any methods annotated with `@After` are finished.

Note: This rule doesn't support `IntentService` because it's automatically destroyed when `IntentService#onHandleIntent(android.content.Intent)` finishes all outstanding commands. So there is no guarantee to establish a successful connection in a timely manner.

{% highlight java %}
@RunWith(AndroidJUnit4.class)
@MediumTest
public class MyServiceTest {

    @Rule
    public final ServiceTestRule mServiceRule = new ServiceTestRule();

    @Test
    public void testWithStartedService() {
        mServiceRule.startService(
            new Intent(InstrumentationRegistry.getTargetContext(), MyService.class));
        // test code
    }

    @Test
    public void testWithBoundService() {
        IBinder binder = mServiceRule.bindService(
            new Intent(InstrumentationRegistry.getTargetContext(), MyService.class));
        MyService service = ((MyService.LocalBinder) binder).getService();
        assertTrue("True wasn't returned", service.doSomethingToReturnTrue());
    }
}
{% endhighlight %}
 
## Logging Rules - Beta

The `android.support.test.rule.logging` package contains Logging Rules that collect and
log certain pieces of system and app data to the Android external storage file system before,
during, and after test method execution. For example, the `LogLogcatRule` will clear the logcat
buffer before a test method is executed and then log the current state of the buffer after the test
method is executed.

The Logging Rules automate the exportation of the select pieces of data to allow
developers to have a better picture of what happened during a test in a robust test suite. The
extended information can be helpful if runnning tests on a large suite of devices, running tests on
remote devices, or automating extended validation for tests. One such extended validation might be
scanning all test that used the `LogGraphicsStatsRule` to ensure the jank percentage didn't go above
a certain threshold to programmatically detect jank issues.

When a test rule logs data to a file, by default, the file location uses a directory
structure composed of the test class package, test class name, test method name, and test iteration
number.

Usage: In order to use the Logging Rules add them to your test class as shown below then add the
Gradle snippet to your project's `build.gradle` file. The Gradle snippet will extract the log files
after a test suite is run.

Example test class using a Logging Rule.
{% highlight java %}
@RunWith(AndroidJUnit4.class)
@LargeTest
public class MyTest {

@Rule
public final LogLogcatRule mLogLogcatRule = new LogLogcatRule();

@Test
public void testWithLogcatRule() {
// test code
}
}
{% endhighlight %}

Example Gradle snippet that extracts the directory the Rules are logging to.
{% highlight gradle %}
TBD
{% endhighlight %}
