# Measuring performance

As with anything there is a process to improving performance of the app: **Measure** -> **Change** -> **Verify** -> Repeat. 
	* Measure the original memory usage. 
	* Attempt to improve by changing the code
	* Run app and verify that the changes happened as expected.
	* Repeat until it is up to snuff.

## Memory Report
* Displays the amount of RAM being used by the application.
* If you use simulator the benchmarks will be using your computer's RAM instead of the same amount that your device has to offer.

To launch memory report: 
1. Click on the **Debug navigator** in the left navigator pane.
2. To get more information, expand the **running process** 
3. Click on the **Memory** row and and look at the top half of the memory gauge.

* The top half of the Memory Report:
	* **Memory Use** shows the amount and percentage of memory your app is using.
	* **Usage Comparison** depicts the memory your app is using as a fraction of the total available memory, as wel as the amount of available free RAM.
* The bottom half of the Memory Report:
	* Displays a chart showing RAM usage over time.


## Instruments Core Data
Open Core Data template in Instruments:
* Open Instruments
	* Product > Profile 
	* Shortcut to open: `Cmd+I`  
* Select `Core Data` from the menu of profiling templates.

Tools to monitor performance in Core Data profiling template:
* **Faults Instrument**: Captures information about fault events that result in cache misses. This can diagnose performance in low-memory situations.
* **Fetches Instrument**: Captures fetch count and duration of fetch operations. This will help you balance the number of fetch requests versus the size of each request.
* **Saves Instrument**: Captures information on managed object context save events. Writing data out to disk can be a performance and battery hit, so this can help you determine if you should batch things into one big save rather than many smaller ones.

* When you click on a tool, like Fetches, the details section at the bottom of Instruments will show more information regarding that tool.


## XCTest Framework
Although usually for Unit testing, the **XCTest** framework contains useful tools for testing performance.

Example test method to measure metrics:
```
func testTotalEmployeesPerDepartment() {

	measureMetrics([.wallClockTime], automaticallyStartMeasuring: false) {

		let departmentList = DepartmentListViewController()
		departmentList.coreDataStack = CoreDataStack(modelName: "EmployeeDirectory")

		startMeasuring()
		_ = departmentList.totalEmployeesPerDepartment()
		stopMeasuring()
	}
}
```
* The above code will indicate how long code takes to execute.
* Important to initialize the CoreDataStack each test case so that CoreData does not automatically cache the results and skew the subsequent runs to be faster than normal.
	* This can be done with the `setUp()` method.
* Cmd+U to test or press the Play button, or even hold the play button at the top and navigate to 'Run Tests'.

* Results might change based on your test device. What's important is that you use the same test device to measure before and after your change so results are reliable.
