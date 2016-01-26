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

Important: As with most performance tooling the monitoring of the system will have varying effects
on the system. Therefore, when interpreting the data from the rules it is suggested that you keep
as many system variables constant as possible. It wouldn't make sense for you to compare data from
different devices or major Android versions, for instance. It would make sense for you to collect
performance data, make a change, then determine if that change improved overall system performance
prior to committing the change to source control.

Quick summary of the Logging Rules:
    EnableTestTracingRule calls the Trace#beginSection and Trace#endSection methods before and
        after each test. This records the start and end of the tests in Systrace output to give you
        more context when debugging a performance trace.
    LogGraphicsStatsRule collects a graphics subsystem dumpsys output after each test. On Android
        Marshmallow+ this includes a "Janky" percentage summary which can be monitored in a
        post processing script to programmatically detect jank above acceptable thresholds.
    LogLogcatRule resets the Logcat buffer on the device prior to running a test and then extracts
        the buffer after a test. This is useful when researching a regression failure or when
        wanting to see Logcat output for a specific test as it ran on a device.
    LogNetStatsRule dumps the current and historical network information using dumpsys. This can be
        useful context when debugging networking related issues. For instance, if a file download
        test failed you might double check to see if the network became unavailable during a test
        run before wasting time digging into the issue.
    LogBatteryInformationRule resets the battery stats before a test and collects the battery
        information after a test. This should be used for large or long running test suites since
        battery stats are affected by various things that could be happening on the system.
    LogDeviceGetPropInfoRule collects the results of running `getprops` on the Android device. These
        properties contain Android and device configuration information. If you have tests that
        pass on some devices but not others this file might be helpful in identifying the issue.
        For instance, if there was a bug in your code that only presented in devices using the Art
        runtime you might be able to uncover that with this output and post-test analysis tools
        since this file specifies whether Art or Dalvik is in use (among many other things).
    Coming soon: An atrace logger which collects raw system atraces which can be converted to
        systrace html files to perform performance analysis on tests in a test suite.

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
{% highlight java %}
TBD
{% endhighlight %}
