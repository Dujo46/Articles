# Army Height for Weight Explored
## Abstract
The Army’s Height/Weight standards are detailed in AR 600-9. For the purposes of this exploration, we will be creating a model that can be used in an application to calculate an individuals compliance with the prescribed standard.

The Army “weight for height screening table” is as follows:

> Minimum Weight Note: Male and female Soldiers who fall below the minimum weights shown in table B-1 [shown below] will be referred by the commander for immediate medical evaluation.  

> Height 80+ in. Note: Add 6 pounds per inch for males over 80 inches and 5 pounds per inch for females over 80 inches.  
### Weight for Height Data
```swift
[
    // Age Group 1: 17-20
    // Age Group 2: 21-27
    // Age Group 3: 28-39
    // Age Group 4: 40+

    // Height, Min Weight, Male Age Group 1-4 Max, Female Age Group 1-4 Max
    [58, 91, 0, 0, 0, 0, 119, 121, 122, 124],
    [59, 94, 0, 0, 0, 0, 124, 125, 126, 128],
    [60, 97, 132, 136, 139, 141, 128, 129, 131, 133],
    [61, 100, 136, 140, 144, 146, 132, 134, 135, 137],
    [62, 104, 141, 144, 148, 150, 136, 138, 140, 142],
    [63, 107, 145, 149, 153, 155, 141, 143, 144, 146],
    [64, 110, 150, 154, 158, 160, 145, 147, 149, 151],
    [65, 114, 155, 159, 163, 165, 150, 152, 154, 156],
    [66, 117, 160, 163, 168, 170, 155, 156, 158, 161],
    [67, 121, 165, 169, 174, 176, 159, 161, 163, 166],
    [68, 125, 170, 174, 179, 181, 164, 166, 168, 171],
    [69, 128, 175, 179, 184, 186, 169, 171, 173, 176],
    [70, 132, 180, 185, 189, 192, 174, 176, 178, 181],
    [71, 136, 185, 189, 194, 197, 179, 181, 183, 186],
    [72, 140, 190, 195, 200, 203, 184, 186, 188, 191],
    [73, 144, 195, 200, 205, 208, 189, 191, 194, 197],
    [74, 148, 201, 206, 211, 214, 194, 197, 199, 202],
    [75, 152, 206, 212, 217, 220, 200, 202, 204, 208],
    [76, 156, 212, 217, 223, 226, 205, 207, 210, 213],
    [77, 160, 218, 223, 229, 232, 210, 213, 215, 219],
    [78, 164, 223, 229, 235, 238, 216, 218, 221, 225],
    [79, 168, 229, 235, 241, 244, 221, 224, 227, 230],
    [80, 173, 234, 240, 247, 250, 227, 230, 233, 236]
]
```

## Preparation
There are 4 things we need to setup prior to creating our `HeightWeight` class:
1. Gender enum
2. Standard -> Minimum and maximum allowed
3. ComplianceResult -> Standard, actual state, and delta (or difference from standard)
4. Compliance -> Over the standard, under the standard, compliant with the standard, **or any other status that is not “compliant” (e.g. does not meet age requirement to check compliance, does not meet height requirement to check compliance, etc.)**

### Aside: Make an enum identifiable automatically
Enumerations in swift are not inherently Identifiable. Making an enum conform to the `AutoIdentifiable` protocol example as seen below makes it automatically identifiable based on it’s case names.
```swift
protocol AutoIdentifiable: Identifiable {}
extension AutoIdentifiable {
    public var id: String { String(describing: self) }
}
```
## Gender enum
Here, we make the enumeration `CaseIterable` (which would be used in an instance where the enum has many cases that you want to iterate through in an ordered manner.).  In the event that your enum has many cases and you want to use SwiftUI’s `ForEach`  with automatic ID conformance, you will need to make your object  `Hashable`. See [Apple Hashable Documentation](https://developer.apple.com/documentation/swift/hashable) and [Apple ForEach Documentation](https://developer.apple.com/documentation/swiftui/foreach/) for more information.
```swift
enum Gender: CaseIterable, Hashable, AutoIdentifiable {
    case male, female
}
```

## Standard enum
```swift
struct Standard<T: Numeric> {
    let min, max: T
}
```
## Compliance Result enum
For more information on Swift’s Numeric type, see [Apple’s Documentation](https://developer.apple.com/documentation/swift/numeric) - “The Numeric protocol provides a suitable basis for arithmetic on scalar values, such as integers and floating-point numbers. ” As a basic concept, it can allow for integers and floats.

As seen above, the `Standard` has a minimum and maximum value, which are also generic to Numeric.

When creating this struct, ask the questions:
1. What is the standard?
2. What is the actual datum?
3. What is the delta (or difference)?
```swift
struct ComplianceResult<T: Numeric> {
    let standard: Standard<T>
    let actual: T
    let delta: T
}
```

## Compliance enum

Including a computed property named `description` allows for output to the console that explains in plain words what the object is.

```swift
enum Compliance<T: Numeric> {
    case under(ComplianceResult<T>)
    case over(ComplianceResult<T>)
    case compliant(ComplianceResult<T>)

    case doesNotMeetMinimumAge
    case heightNotWithinBounds
    case noStandardAvailable
    
// Used for console output
    var description: String {
        switch self {
            // Description of result for those who are under the standard
        case .under(let result): return "Under by \(result.delta). Actual: \(result.actual).  Min: \(result.standard.min) - Max: \(result.standard.max)."
            // Description of result for those who are over the standard
        case .over(let result): return "Over by \(result.delta). Actual: \(result.actual).  Min: \(result.standard.min) - Max: \(result.standard.max)."
            // Description of result for those who are compliant with the standard
        case .compliant(let result): return "Compliant, actual: \(result.actual).  Min: \(result.standard.min) - Max: \(result.standard.max)."
            
            // Description for those who do not meet the minimum age requirement
            // as laid out in the regulation
        case .doesNotMeetMinimumAge: return "Does not meet minimum age requirement."
            // Description for those who do not have easiliy identified standrads
        case .noStandardAvailable: return "No standard data available."
            // Example, those who are 80+ in. in height
        case .heightNotWithinBounds: return "Height must be between 58in. and 80in."
        }
    }
}
```

# Calculating Compliance
```swift
static func calculate(gender: Gender, age: Int, height: Int, weight: Int) -> Compliance<Int> {
    // Height within screening table bounds
    guard (58...80).contains(height) else { return .heightNotWithinBounds }
        
    // Meets minimum age
    guard age > 16 else { return .doesNotMeetMinimumAge }
    
    // Get the row for the specific height being evaluated
    let data = HeightWeight.HeightForWeightScreeningTable
        .first(where: { $0.first == height }) ?? []
        .compactMap { $0 }
    
    // Get max index for specified gender    
    var maxIndex: Int {
        switch age {
        case 17...20: return gender == .male ? 2 : 6
        case 21...27: return gender == .male ? 3 : 7
        case 28...39: return gender == .male ? 4 : 8
        case 40...Int.max: return gender == .male ? 5 : 9
        default: return 0
        }
    }
    
    // Standard
    let min = data[1] // always the column at index 1
    let max = data[maxIndex] // based on gender and ageGroup
    let standard = Standard(min: min, max: max)
    
    // Check if exception to regulation policy (too high/low)
    // Personnel who are out of the height bounds will fall
    // into this category
    guard min != 0 && max != 0 else { return .noStandardAvailable }
    
    // Over or under screening table weight
    guard (min...max).contains(weight) else {
        return weight < min
        ? .under(ComplianceResult(standard: standard, actual: weight, delta: min-weight))
        : .over(ComplianceResult(standard: standard, actual: weight, delta: weight-max))
    }
    
    // Compliant, within screening table weight
    return .compliant(ComplianceResult(standard: standard, actual: weight, delta: .zero))
}
```

# Testing the class
```swift
let gender = Gender.female
let age = 30
let height = 60

// Check compliance with minimum and maximum
var exampleWeights = [96, 97, 131, 132]

exampleWeights.forEach { exampleWeight in
    let result = HeightWeight.calculate(gender: gender,
                                        age: age,
                                        height: height,
                                        weight: exampleWeight) // test
    
    print(“Using a weight of \(exampleWeight)”)
    print(“\(result.description)\n”)
}

//Using a weight of 96
//Under by 1. Actual: 96.  Min: 97 - Max: 131.

//Using a weight of 97
//Compliant, actual: 97.  Min: 97 - Max: 131.

//Using a weight of 131
//Compliant, actual: 131.  Min: 97 - Max: 131.

//Using a weight of 132
//Over by 1. Actual: 132.  Min: 97 - Max: 131.
```
