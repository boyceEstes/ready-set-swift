# Basics


## Code coverage
Whenever you are testing and checking code coverage you will see some stripes on the right of the code indicating "Partial code coverage". This happens often whenever a test has two paths that it could take and only takes one.

While this sounds like it makes sense it will hurt your code coverage numbers a little more than you might feel comfortable with for simple tests. For example:

```
func test_resource_exampleDateJSON_returnsData() {

    // when
    let resource = Resource(name: "ExampleDate", type: "json")
    let jsonData = resource.mockContentData

    // then
    XCTAssertNotNil(jsonData, "Data from \(resource.fullName()) does not exist")
    // Make sure this is returning Data
    XCTAssertTrue((jsonData as Any) is Data)
}
```

The `XCTAssertNotNil` will cause this candy-cane partial coverage indicator. This is because we are asserting that it `jsonData` is not nil and therefore we never go down the path of calling the error log "Data from \(resource.fullName()) does not exist".

This is a good lesson though. Some things you just need to take on the chin and keep going. The main thing to focus on while creating unit tests is that they are effective and easily-readible. Getting that code coverage percentage up is cool for the metrics but having 100% code coverage would probably mean that your tests are very specific and probably less helpful.