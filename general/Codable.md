# Codable

Snippet from documentation

```
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    var vantagePoints: [Coordinate]
    
    enum CodingKeys: String, CodingKey {
        case name = "title"
        case foundingYear = "founding_date"
        
        case location
        case vantagePoints
    }
}
```

Source: https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types

## Dates
If you want to decode strings into Date objects automatically, you will ned to set it up in your JSONDecoder. If its simple decoding, try:
```
decoder.dateDecodingStrategy = .iso8601
```

If not, read, understand, and leave me a note lol

Source: https://useyourloaf.com/blog/swift-codable-with-custom-dates/

## Manual Decoding
If you need to do some transformation during the decoding process, it is a good chance you will need to manually override the init for the model.

It should look something like this:
```
struct MovieModel: Decodable {

    let title: String
    let overview: String?
    let popularity: Double
    let releaseDate: Date?


    enum CodingKeys: String, CodingKey {
        case title
        case overview
        case popularity
        case releaseDate = "release_date"
    }


    // Had to make decoding manual because Date needed to be taken from a string to the correct date.
    // smh.
    init(from decoder: Decoder) throws {

        let container = try decoder.container(keyedBy: CodingKeys.self)

        title = try container.decode(String.self, forKey: .title)
        overview = try container.decode(String.self, forKey: .overview)
        popularity = try container.decode(Double.self, forKey: .popularity)

        let releaseDateString = try container.decodeIfPresent(String.self, forKey: .releaseDate)
        let formatter = DateFormatter.yyyyMMdd

        guard let releaseDateString = releaseDateString,
            let date = formatter.date(from: releaseDateString) else {
            throw DecodingError.dataCorruptedError(forKey: .releaseDate, in: container, debugDescription: "Date string does not match expected")
        }
        
        releaseDate = date
    }
}

```

* Be warned: If it can't find one of your properties that you believe must be in the payload, it will throw a decoding error. A good way to prevent this is by calling `container.decodeIfPresent(_: forKey:)` on your optional values.
