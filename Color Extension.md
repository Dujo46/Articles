**Purpose**: Easily instantiate a SwiftUI Color using RGB values.

```swift
// MARK: - Easy init
extension Color {
    init(_ red: Double, _ green: Double, _ blue: Double) {
        self.init(red: red / 255,
                  green: green / 255,
                  blue: blue / 255, opacity: 1.0)
    }
}
```

# Usage
```swift
let colorName = Color(245, 198, 77)
```
