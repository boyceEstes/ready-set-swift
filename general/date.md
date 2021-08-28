# Date

* [Foundations](#foundations)
* [Get current date and time](#get-current-date-and-time)
* [Creating another date and time](#creating-another-date-and-time)
	* [Seconds since reference date](#seconds-since-reference-date)
	* [Using specific date components](#using-specific-date-components)
	* [DateFormatter](#dateformatter)
* [Displaying date and time](#displaying-date-and-time)
	* [Calendar](#calendar)
		* [Current vs AutoupdatingCurrent](#current-vs-autoupdatingcurrent)
	* [DateFormatter](#dateformatter)
* [ISO8601DateFormatter](#iso8601dateformatter)
* [Locale](#locale)
* [Date encoding](#date-encoding)
* [Date decoding](#date-decoding)

## Foundations
Foundational information that is necessary to really understand everything:
* Coordinated Universal Time (UTC)
* UTC is a *time standard* that is used across the world to keep time synchronized. It will often be referenced at the end of a timestamp `+0000` means that you are in the GMT timezone.
* Greenwich Mean Time (GMT) - This is a timezone. It is the same time that is used in the UTC standard. But it is technically different as this is a timezone instead of a standard unit.
* Eastern Standard Time (EST) - This is a timezone (The best one 😁). 
* Eastern Daylight Time (EDT) - This is a timezone (The other best one). 
* For all date times, in various timezones, there is an area at the end that will indicate the number of hours that they are off of the UTC (universal time).

* iso8601 is a standard format for date times. if the time is already in utc time, it looks like this `2021-05-20T12:00:48.229Z`. It would also be acceptable to use `2021-05-20T12:00:48.229+0000` to represent the same thing in iso8601.
	* Never use the Z an *addition* to the timezone offset (`+0000`). Z in this context basically standards for "Zero offset"

* **NOTE**: For some reason Swift handles daylight savings time a little weird whenever you initialize a TimeZone like `TimeZone(abbreviation:)`  vs like `TimeZone(identifier:)`

```
let estAbbreviation = TimeZone(abbreviation: "EST")
let estIdentifier = TimeZone(identifier: "EST")

let dateTime2 = formatter.date(from: "1980-10-11 08:34")

print(dateTime2!.startOfDay(withTimeZone: estAbbreviation))
// Output: 1980-10-11 04:00:00 +0000
print(dateTime2!.startOfDay(withTimeZone: estIdentifier))
// Output: 1980-10-11 05:00:00 +0000
```
In summary, use the identifier for the stricter rules.

When a TimeZone is added on, you can think of it like `HH:mm`. So, for example, `-1200` would mean that the timezone is behind UTC by 12 hours and no minutes.

## Get current date and time
Get current time and date:
```
let currentDateTime = Date()
```

Date is a 64-bit floating point number measuring the number of seconds since the reference date of January 1, 2001 at 00:00:00 UTC. 

You can see this number by calling
```
secondsSinceReference = Date().timeIntervalSinceReferenceDate
// Outputs: 642696162.98563
```

## Creating another date and time

### Seconds since reference date
If you know the number of seconds before or after the reference date, you can use that.
```
let someNewDate = Date(timeIntervalSinceReferenceDate: -123456789.0)
// Outputs: 1997-02-02 02:26:51 +0000
```

### Using specific date components
A `DateComponent` refers to each component of a datetime object like day, hour, minute, second, etc., rather than just the number of relative seconds.

Whenever you specify the Date components, you will need to then specify the `Calendar` to create the actual date. The calendar gives us the proper information that we need to express date components in a given context. 

For example a Buddhist calendar is going provide a different Date given the same `DateComponents` than a Gregorian calendar would.

```
var dateComponents = DateComponents()
dateComponents.year = 1980
dateComponents.month = 7
dateComponents.day = 11
dateComponents.timeZone = TimeZone(abbreviation: "JST") // Japan Standard Time
dateComponents.hour = 8
dateComponents.minute = 34

// Create date from components
let userCalendar = Calendar.current
let someDateTime = userCalendar.date(from: dateComponents)

print(someDateTime!)
// Output: 1980-07-10 23:34:00 +0000 // Formatted the UTC DateComponents for JST
```
* **Note**: The date components are interpretted based on the Gregorian calendar in UTC timezone. 

* Timezones might not matter whenever you are using a different calendar to generate your Date object.
```
let buddhistCalendar = Calendar(identifier: .buddhist)
let someDateTime3 = buddhistCalendar.date(from: dateComponents(with: est)) 

buddhist version - est timezone 1437-07-11 13:30:02 +0000
buddhist version - pst timezone 1437-07-11 13:30:02 +0000
```

* **Note**: In the above example, `dateComponents(with:)` refers to a homemade function that will always return the time from the example before 1980-7-11 8:34 with the given timezone.

### DateFormatter
This is the *shortest* way, but maybe not necessarily the best.
* `date(from:)` returns an optional Date from some string that is given. The string provided in the argument *must* be in the same format as given in dateFormat. This can get you past the annoyance of dealing with individual DateComponents to specify your datetime.

Ex:
```
let formatter = DateFormatter()
formatter.dateFormat = "yyyy/MM/dd HH:mm"
let date1 = formatter.date(from: "1980-07-11 08:34")
print("Output: \(date1!)")
```
* **Important**: Timezones must be considered whenever you are using DateFormatter. By default, a DateFormatter will contain your `TimeZone.current` as its value for the `formatter.timeZone` property. This means that it will read in `1980-07-11 08:34` as UTC time, but convert it to ETC whenever you want to use it. This causes the below output from the above snippet.
```
Output: 1980-07-11 12:34:00 +0000 // Does not match inputted time.
```

To fix this: Make sure that you are inputted the UTC time to the dateformatter. And if you want the same time value no matter where the user is, make sure that you specify a timeZone: 
```
let formatter = DateFormatter()
formatter.dateFormat = "yyyy/MM/dd HH:mm"
formatter.timeZone = utc 								// previously defined TimeZone property
let date2 = formatter.date(from: "1980-07-11 08:34")
print("Output: \(date2!)")
// prints "Output: 1980-07-11 08:34:00 +0000"			// Yay it matches.
```
* **Note**: This also doesn't take into account calendars by default so be weary of anything non gregarian



## Displaying date and time

Sometimes we just wanna display the date/time in some format:

### Calendar

Use `CalendarUnit` to specify the components that you want to extract from the `Date` object.
```
// get the current date and time
let rightNow = Date()

// get the user's calendar
let userCalendar = Calendar.current

// choose which date and time components are needed
let requestedComponents: Set<Calendar.Component> = [
    .year,
    .month,
    .day
]

// Get the components you requested
let dateTimeComponents = userCalendar.dateComponents(requestedComponents, from: rightNow)
let arrangedComponents = "\(dateTimeComponents.day!) \(dateTimeComponents.month!), \(dateTimeComponents.year!)"

print("Output: \(arrangedComponents)")
// Output: 14 5, 2021
```
* There is not an "out of the box" method to get the month name from the calendar method. You must use DateFormatter to do that ("LLLL" is the format for month name)
* You can only use the date components that you requested. Anything else will return nil.

#### Current vs AutoupdatingCurrent
* `.current` will retrieve the current calendar for the user, but it does not track changes that the user makes to their settings.
* `.autoupdatingCurrent` tracks changes to user's preferred calendar.

So .autoupdatingCurrent calendar is following every update you do / system does to the date & time parameters.

This also applies if you change, for example, the Calendar from your device settings (Settings > General > Language & Region > Calendar).

Same pattern applies to other Foundation objects, such as Locale and TimeZone

If you just want to access user’s calendar and do not track changes that he does through settings later on, just use .current.

**Just to be clear** if you change the calendar type from Gregorian to Buddhist, `.current` will register it. It is other settings that it won't look for in time


### DateFormatter

This is where DateFormatter shines. It would be a pain to try to convert all of the data in `Date` objects to the way that they are expected in every style, language, and region. But luckily, DateFormatter does this for us.

There are helpful styles that can be set in the DateFormatter object that tell it how we would like dates to be interacted with. We could set it manually using:
```
let formatter = DateFormatter()
formatter.dateFormat = "yyyy/MM/dd HH:mm"
```
Or, the better approach would be to do it using the Apple shorthand styles so that it would change depending on the region. This makes things much less potentially confusing for the user (For example, Europe and America format dates differently: dd-MM-yyyy vs MM/dd/yyyy)

```
// initialize the date formatter and set the style
let newFormatter = DateFormatter()
newFormatter.timeStyle = .medium
newFormatter.dateStyle = .long
newFormatter.calendar = .current
newFormatter.timeZone = .current

// get the date time String from the date object
let prettyDateTime = newFormatter.string(from: Date())

print("Output: \(prettyDateTime)")
// Output: May 14, 2021 at 5:35:47 PM
```

### Common Date formats

| Character | Example | Description |
| --- | --- | --- |
| **Year** |
| yy | 08 | 2-digit year |
| yyyy | 2008 | 4-digit year |
| **Quarter** |
| Q | 4 | The quarter of the year. Use QQ if you want zero padding. |
| QQQ | Q4 | Quarter including "Q" |
| QQQQ | 4th quarter | Quarter spelled out |
| **Month** |
| M | 12 | The numeric month of the year. A single M will use '1' for January. |
| MM | 12 | The numeric month of the year. A double M will use '01' for January. |
| MMM | Dec | The shorthand name of the month |
| MMMM | December | Full name of the month |
| MMMMM | D | Narrow name of the month |
| **Day** |
| d | 14 | The day of the month. A single d will use 1 for January 1st. |
| dd | 14 | The day of the month. A double d will use 01 for January 1st. |
| F | 3rd Tuesday in December | The day of week in the month |
| E | Tues | The day of week in the month |
| EEEE | Tuesday | The full name of the day |
| EEEEE | T | The narrow day of week |
| **Hour** |
| h | 4 | The 12-hour hour. |
| hh | 04 | The 12-hour hour padding with a zero if there is only 1 digit |
| H | 16 | The 24-hour hour. |
| HH	16 | The 24-hour hour padding with a zero if there is only 1 digit. |
| a | PM | AM / PM for 12-hour time formats |
| **Minute** |
| m | 35 | The minute, with no padding for zeroes. |
| mm | 35 | The minute with zero padding. |
| **Second** |
| s | 8 | The seconds, with no padding for zeroes. |
| ss | 08 | The seconds with zero padding. |
| **Time Zone** |
| zzz |	CST |	The 3 letter name of the time zone. Falls back to GMT-08:00 (hour offset) if the name is not known. |
| zzzz | Central Standard Time | The expanded time zone name, falls back to GMT-08:00 (hour offset) if name is not known. |
| zzzz | CST-06:00 | Time zone with abbreviation and offset |
| Z | -0600 | RFC 822 GMT format. Can also match a literal Z for Zulu (UTC) time. |
| ZZZZZ | -06:00 | ISO 8601 time zone format |



## ISO8601DateFormatter

ISO 8601 is the defacto standard format of time for the Gregarian calendar. It utilizes military time (24 hour timekeeping) and an optional UTC offset. 

Always fixed at GMT time. don't bother trying to set it to another TimeZone like `formatter.timeZone = TimeZone(identifier: "EDT")`

ISO8601 formatted using DateFormatter in Swift: `yyyy-MM-dd'T'HH:mm:ssZ`
* To place the T character in the middle, you must use commas
* `Z` at the end will be converted to `+0000` (or whatever the timezone dictates)
* `// Output from given date formatter code: 2021-05-20T16:05:09+0000`

Using this in Swift can be done out of the box using an `ISO8601DateFormatter`.
```
// Must be in yyyy-MM-ddTHH:mm:ssZ format
let isoDateFormatter = ISO8601DateFormatter()
guard let date = isoDateFormatter.date(from: "2021-05-21T17:53:00 +0000") else { fatalError() }

print("Output \(date)")
// Output: 2021-05-21 17:53:00 +0000
```
If you do not place the date string in that exact format, it will return nil (in this case a fatalError - probably want to handle that more gracefully)


**Important**: What is the difference between using an ISO8601DateFormatter and using a DateFormatter with `yyyy-MM-dd'T'HH:mm:ssZ` as the format?

Answer! Not much. This will allow us to successfully get the correct Date object for any given timezone if we are decoding. However, if we ever want to get the string from this date it will be in this format: '2021-05-21T17:53:00+0000' instead of the standard '2021-05-21T17:53:00Z' format for nil timezones. As long as this is okay they are interchangable.


### FormatOptions

You can require less by using `formatOptions` property
```
let isoDateFormatter2 = ISO8601DateFormatter()
isoDateFormatter2.formatOptions = .withFullDate
guard let date = isoDateFormatter2.date(from: "2021-05-21") else { fatalError() }

print("Output: \(date)")
// Output: 2021-05-21 00:00:00 +0000
```

The above doesn't require as much information to create a Date object, since in this case it is assuming you don't need the time information.

You can specify many different format options.

| Format 					| Example 					  | Options 								  |
| ------ 					| ------- 					  | ------- 								  |
| Date with dash separators | `2016-06-13` 				  | `withFullDate`, `withDashSeparatorInDate` |
| RFC 3339 Date and Time    | `2016-06-13T16:00:00+00:00` | `withInternetDateTime` 					  |

### Convenience
You can make shortcuts to have a custom ISO8601DateFormatter:

```
public extension ISO8601DateFormatter {

    convenience init(_ formatOptions: Options) {
        self.init()
        self.formatOptions = formatOptions
    }
}
```

This will allow you to create whatever ISO8601DateFormatter you want by calling like this. For example, if you wanted a formatter that operated using fractional time (an additional .SSS at the end before the timezone offset), you could do this.
```
let formatter = ISO8601DateFormatter([.withInternetDateTime, .withFractionalSeconds])
```

An even easier way to get this would be to create this would be to create a static variable on its superclass, Formatter. 

```
extension Formatter {
	static let iso8601WithFractionalSeconds = ISO8601DateFormatter([.withInternetDateTime, .withFractionalSeconds])
}
```

This is nice and scalable if there are many different types of date formatter options that you want to make.


## Locale
Your locale is used to identify the format that dates should be in for the Apple syntactic sugar:
```
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
dateFormatter.timeStyle = .none
 
let date = Date(timeIntervalSinceReferenceDate: 118800)
 
// US English Locale (en_US)
dateFormatter.locale = Locale(identifier: "en_US")
print(dateFormatter.string(from: date)) // Jan 2, 2001
 
// French Locale (fr_FR)
dateFormatter.locale = Locale(identifier: "fr_FR")
print(dateFormatter.string(from: date)) // 2 janv. 2001
 
// Japanese Locale (ja_JP)
dateFormatter.locale = Locale(identifier: "ja_JP")
print(dateFormatter.string(from: date)) // 2001/01/02
```



## Date encoding
Whenever you are encoding date objects in Swift, you can choose how you want them to be encoded in your JSON data. 

By default (using `let encoder = JSONEncoder()`, they will be converted into seconds since the reference date (1 January, 2001 at midnight).

In order to change this default behavior, you must modify the DateEncodingStrategy property on the JSONEncoder (or whatever the equivalent is that you are using).

```
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
``` 

Now whenever you encode some date, instead of being transformed into `516081604` in your JSON payload, it would be `2017-05-10T04:00:04Z`


If you would like to create a custom date format for encoding Date objects, you can use `DateFormatter` like this:
```
let encoder = JSONEncoder()
let formatter = DateFormatter()
formatter.dateStyle = .full
formatter.timeStyle = .full
encoder.dateEncodingStrategy = .formatted(formatter)
```
That converts any Date properties to be the fullest possible string for your locale, e.g. "Monday, February 5, 2018 at 9:28:10 PM Greenwich Mean Time”.



## Date decoding
Whenever you are decoding date objects in Swift, you can choose how you want them to be decoded from your data. Dates will come in a String objects or some number of seconds.

By default, it will automatically convert some Int like `516081604` to a date object, using that number to mean number of seconds since the reference date.

You can change this date decoding strategy by modifying the JSONDecoder's DateDecodingStrategy.
```
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
```
This is useful if your JSON payload that you are converting into a Date object is delivered in a iso8601 string format ([More information](#iso8601dateformatter)).


### Errors
Whenever you are decoding dates using JSONDecoder, you will want to have specific errors thrown whenever you have a problem with your custom date decoding strategy:
```
guard let date = Formatter.iso8601WithFractionalSeconds.date(from: string) else {
     throw DecodingError.dataCorruptedError(in: container,
           debugDescription: "Invalid date: " + string)
}
```

This is a good practice. Something to look out for when you are unit testing is that if you have a COMPLETELY different error, we won't even get to this point. For example, if you are unit testing that you get an error whenever you are decoding a date that is in the form of `secondsSinceReference` , if your decoding is expecting a String, it will error before ever getting to your custom error.

```
typeMismatch(Swift.String, Swift.DecodingError.Context(codingPath: [CodingKeys(stringValue: "date", intValue: nil)], debugDescription: "Expected to decode String but found a number instead.", underlyingError: nil))
```

You will need have the correct type (String in this case) before you would get the expected error.


## Sources
* [StackOverflow - how to get the current time as datetime](https://stackoverflow.com/questions/24070450/how-to-get-the-current-time-as-datetime)
	* [Dates, Calendars, DateComponents in Swift 3](http://www.globalnerdy.com/2016/08/18/how-to-work-with-dates-and-times-in-swift-3-part-1-dates-calendars-and-datecomponents/)

* [Datetime display codes](https://www.zerotoappstore.com/get-year-month-day-from-date-swift.html)
* [Formatting dates](https://schiavo.me/2019/formatting-dates/)
